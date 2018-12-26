# 			**Samba and LDAP**



 This section covers the integration of Samba with LDAP. The Samba  server's role will be that of a "standalone" server and the LDAP 	directory will provide the authentication layer in addition to  containing the user, group, and machine account information that Samba 	requires in order to function (in any of its 3 possible roles). The  pre-requisite is an OpenLDAP server configured with a directory 	that can accept authentication requests. See [OpenLDAP Server](https://help.ubuntu.com/lts/serverguide/openldap-server.html.en) for details on fulfilling this requirement. Once this 	section is completed, you will need to decide what specifically you want Samba to do for you and then configure it accordingly. 	

 	This guide will assume that the LDAP and Samba services are running on  the same server and therefore use SASL EXTERNAL authentication 	whenever changing something under cn=config. If that is not your scenario, you will have to run those ldap 	commands on the LDAP server.

- [Software Installation](https://help.ubuntu.com/lts/serverguide/samba-ldap.html.en#samba-ldap-installation)
- [LDAP Configuration](https://help.ubuntu.com/lts/serverguide/samba-ldap.html.en#samba-ldap-openldap-configuration)
- [Samba Configuration](https://help.ubuntu.com/lts/serverguide/samba-ldap.html.en#samba-ldap-samba-configuration)
- [Resources](https://help.ubuntu.com/lts/serverguide/samba-ldap.html.en#samba-ldap-resources)

## Software Installation

 	There are two packages needed when integrating Samba with LDAP: samba 	and smbldap-tools, install them on samba server.

 	Strictly speaking, the smbldap-tools package isn't needed, but unless you have some other way to manage the various 	Samba entities (users, groups, computers) in an LDAP context then you should install it.   	

 	Install these packages now: 	

```
sudo apt install samba smbldap-tools
```

## LDAP Configuration

 	We will now configure the LDAP server so that it can accomodate Samba data. We will perform three tasks in this section: 	

1. Import a schema(on LDAP server)
2. Index some entries(on LDAP server)
3. Add objects (on samba server)

### Samba schema

​       	In order for OpenLDAP to be used as a backend for Samba,  logically, the DIT will need to use attributes that can properly  describe 	Samba data. Such attributes can be obtained by introducing a Samba LDAP  schema. Let's do this now, there are 2 methods:

**Method 1**:

 		For more information on schemas and their installation see [Modifying the slapd Configuration Database](https://help.ubuntu.com/lts/serverguide/openldap-server.html.en#openldap-configuration). 		

1.  The schema(samba.ldif.gz) is found in the now-installed samba package and is already in the ldif format. 	We can transfer this file to LDAP server.

   ```
   sudo scp /usr/share/doc/samba/examples/LDAP/samba.ldif.gz user@LDAPip:/etc/ldap/
   ```

2. Import samba.ldif.gz with one simple command: 		

   ```
   zcat /etc/ldap/samba.ldif.gz | sudo ldapadd -Q -Y EXTERNAL -H ldapi:///
   ```

3.  To query and view this new schema: 		

   ```
   sudo ldapsearch -Q -LLL -Y EXTERNAL -H ldapi:/// -b cn=schema,cn=config 'cn=*samba*'
   ```

**Method 2:**

1.   The schema(samba.schema) is found in the now-installed samba package, transfer this file to LDAP server's directory /etc/ldap/schama/。

2. Import this file in slapd.conf:

   ```
   vim /usr/share/slapd/slapd.conf, add the following file to slapd.conf:
   include         /etc/ldap/schema/samba.schema
   ```

### Samba indices

 	Now that slapd knows about the Samba attributes, we can set up some indices based on them. Indexing entries is a way to improve 	performance when a client performs a filtered search on the DIT. 	

 	Create the file samba_indices.ldif with the following contents: 	

```
dn: olcDatabase={1}mdb,cn=config
changetype: modify
replace: olcDbIndex
olcDbIndex: objectClass eq
olcDbIndex: uidNumber,gidNumber eq
olcDbIndex: loginShell eq
olcDbIndex: uid,cn eq,sub
olcDbIndex: memberUid eq,sub
olcDbIndex: member,uniqueMember eq
olcDbIndex: sambaSID eq
olcDbIndex: sambaPrimaryGroupSID eq
olcDbIndex: sambaGroupType eq
olcDbIndex: sambaSIDList eq
olcDbIndex: sambaDomainName eq
olcDbIndex: default sub,eq
```

 	Using the ldapmodify utility load the new indices: 	

```
sudo ldapmodify -Q -Y EXTERNAL -H ldapi:/// -f samba_indices.ldif
```

 	If all went well you should see the new indices using ldapsearch: 	

```
sudo ldapsearch -Q -LLL -Y EXTERNAL -H \
ldapi:/// -b cn=config olcDatabase={1}mdb olcDbIndex
```

### Adding Samba LDAP objects

 	Next, configure the smbldap-tools package to match your environment.  The package  	comes with a configuration helper script called smbldap-config. Before running it, 	though, you should decide on two important configuration settings in /etc/samba/smb.conf: 	

- netbios name: how this server will be known. The default value is derived 		from the server's hostname, but truncated at 15 characters.
- workgroup: the workgroup name for this server, or, if you later decide to make it 		a domain controller, this will be the domain.

It's important to make these choices now because smbldap-config will use them 	to generate the config that will be later stored in the LDAP directory. If you run 	smbldap-config now and later change these values in /etc/samba/smb.conf 	there will be an inconsistency. 

Notice: *smbldap-config always use the configuration in smb.conf, so we can config smb.conf first, and then run smbldap-config.*

Once you are happy with netbios name and workgroup, proceed to generat 	the smbldap-tools configuration by running the configuration script which will ask you 	some questions: 	

```
sudo smbldap-config
```

Some of the more important ones:

- workgroup name: has to match what you will configure in 			/etc/samba/smb.conf later on.
- ldap suffix: has to match the ldap suffix you chose when you configured the LDAP server.
- other ldap suffixes: they are all relative to ldap suffix above. For example, for 			ldap user suffix you should use ou=People.
- ldap master bind dn and bind password: use the rootDN credentials.

 	The smbldap-populate script will then add the LDAP objects required for Samba. It is a good idea to first 	make a backup of your DIT using slapcat: 	

```
sudo slapcat -l backup.ldif
```

 	Once you have a backup proceed to populate your directory. It will ask you for a password for the "domain root" 	user, which is also the "root" user stored in LDAP: 	

```
sudo smbldap-populate -g 10000 -u 10000 -r 10000
```

The -g, -u and -r parameters tell 	smbldap-tools where to start the numeric uid and gid allocation for the LDAP 	users. You should pick a range start that does not overlap with your local 	/etc/passwd users.

 	You can create a LDIF file containing the new Samba objects by executing sudo smbldap-populate -e samba.ldif. 	This allows you to look over the changes making sure everything is correct. If it is, rerun the script without the '-e' 	switch. Alternatively, you can take the LDIF file and import its data per usual. 	

 	Your LDAP directory now has the necessary information to authenticate Samba users. 	

## Samba Configuration

 	There are multiple ways to configure Samba. For details on some common configurations see [Samba](https://help.ubuntu.com/lts/serverguide/samba.html.en).       	To configure Samba to use LDAP, edit its configuration file /etc/samba/smb.conf commenting out 	the default passdb backend parameter and adding some ldap-related ones. Make sure to use the 	same values you used when running smbldap-populate: 	

```
   workgroup = EXAMPLE
   
   #how this server will be known. The default value is derived from the server's hostname
   netbios name = samba

# LDAP Settings
   #  passdb backend = tdbsam
   passdb backend = ldapsam:ldap://hostname
   ldap suffix = dc=auto-link,dc=com,dc=cn
   ldap user suffix = ou=Devel,dc=auto-link,dc=com,dc=cn
   ldap group suffix = ou=Groups,dc=auto-link,dc=com,dc=cn
   #ldap machine, idmap suffix can not be configured
   ldap machine suffix = ou=Computers
   ldap idmap suffix = ou=Idmap
   ldap admin dn = cn=admin,dc=auto-link,dc=com,dc=cn
   # no if TLS/SSL is not configured, of start tls if TLS/SSL is configured
   ldap ssl = no
   ldap passwd sync = yes
```

 	Change the values to match your environment. 	

The smb.conf as shipped by the package is quite long and has many configuration 		examples. An easy way to visualize it without any comments is to run testparm -s.

Notice: 

1. netbios name can used to bind your samba SID，ervery samba server has one SID, different samba server can has the same samba SID when configure the same netbios name.
2.  The sambaSID attribute of every samba account on LDAP is made up of sambaSID and a unique numeric suffix, and this note that the account can only be known with the samba server who has the same samba SID. If many samba server want to use the same samba account in the same LDAP, Having the same netbios name on different samba server can help.

Use net to get samba SID on samba server:

```
net getlocalsid
```

Now inform Samba about the rootDN user's password (the one set during the installation of the slapd package): 	

```
sudo smbpasswd -W
```

 	As a final step to have your LDAP users be able to connect to samba and authenticate, we need these users to also show up 	in the system as "unix" users. One way to do this is to use libnss-ldap. Detailed instructions 	can be found in the [LDAP Authentication](https://help.ubuntu.com/lts/serverguide/openldap-server.html.en#openldap-auth-config) section, but we only need the NSS part. 	

1. Install libnss-ldap

   ```
   sudo apt install libnss-ldap
   ```

   There is no need to use the LDAP rootDN login credentials, so you can skip that step.

2. Configure the LDAP profile for NSS:

   ```
   sudo auth-client-config -t nss -p lac_ldap
   ```

3. Restart the Samba services:

   ```
   sudo systemctl restart smbd.service nmbd.service
   ```

   Notice: after restart samba and LDAP, We can see sambaDomain object named "SAMBA" on LDAP Web page, the name is the netbios name configure on samba server.

4. To quickly test the setup, see if getent can list the Samba groups:

   ```
   getent group
   
   ...
   Account Operators:*:548:
   Print Operators:*:550:
   Backup Operators:*:551:
   Replicators:*:552:
   ```

 	If you have existing LDAP users that you want to include in your new  LDAP-backed Samba they will, of course, also need to be given 	some of the extra Samba specific attributes. The smbpasswd utility can do this for you: 	

```
sudo smbpasswd -a username
```

 	You will prompted to enter a password. It will be considered as the new  password for that user. Making it the same as before is reasonable. 	Note that this command cannot be used to create a new user from scratch  in LDAP (unless you are using ldapsam:trusted 	and ldapsam:editposix, not covered in this guide). 	

 	To manage user, group, and machine accounts use the utilities provided by the smbldap-tools package. 	Here are some examples: 	

-  		To add a new user with a home directory: 		

  ```
  sudo smbldap-useradd -a -P -m username
  ```

   		The -a option adds the Samba attributes, and the -P option calls the  		smbldap-passwd utility after the user is created allowing you to enter a password for the user. 		Finally, -m creates a local home directory. 		Test with the getent command: 		

  ```
  getent passwd username
  ```

  Notice: If you don't get a response, then your libnss-ldap configuration is incorrect.

-  		To remove a user: 		

  ```
  sudo smbldap-userdel username
  ```

   		In the above command, use the -r option to remove the user's home directory. 		

-  		To add a group: 		

  ```
  sudo smbldap-groupadd -a groupname
  ```

   		As for smbldap-useradd, the -a adds the Samba attributes. 		

-  		To make an existing user a member of a group: 		

  ```
  sudo smbldap-groupmod -m username groupname
  ```

   		The -m option can add more than one user at a time by listing them in comma-separated format. 		

-  		To remove a user from a group: 		

  ```
  sudo smbldap-groupmod -x username groupname
  ```

-  		To add a Samba machine account: 		

  ```
  sudo smbldap-useradd -t 0 -w username
  ```

   		Replace username with the name of the workstation.  The -t 0 option creates the machine account 		without a delay, while the -w option specifies the user as a machine account.  Also, note the  		add machine script parameter in /etc/samba/smb.conf was changed to use smbldap-useradd. 		

 	There are utilities in the smbldap-tools package that were not covered here. Here is a complete list: 	

```
smbldap-groupadd
smbldap-groupdel
smbldap-groupmod
smbldap-groupshow
smbldap-passwd
smbldap-populate
smbldap-useradd
smbldap-userdel
smbldap-userinfo
smbldap-userlist
smbldap-usermod
smbldap-usershow
```

Resources

-  		For more information on installing and configuring Samba see [Samba](https://help.ubuntu.com/lts/serverguide/samba.html.en) of this Ubuntu Server Guide. 		
-  		There are multiple places where LDAP and Samba is documented in the upstream 		[Samba HOWTO Collection](http://samba.org/samba/docs/man/Samba-HOWTO-Collection/). 		
-  		Regarding the above, see specifically the [passdb section](http://samba.org/samba/docs/man/Samba-HOWTO-Collection/passdb.html). 		
-  		Although dated (2007), the [Linux Samba-OpenLDAP HOWTO](http://download.gna.org/smbldap-tools/docs/samba-ldap-howto/) contains valuable notes. 		
-  		The main page of the [Samba Ubuntu community documentation](https://help.ubuntu.com/community/Samba#samba-ldap) has a plethora of links to 		articles that may prove useful. 		