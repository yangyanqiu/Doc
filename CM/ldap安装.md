## 						Install ldap in ubuntu

sudo apt-get update
sudo apt-get install slapd ldap-utils
sudo dpkg-reconfigure slapd
sudo apt-get install phpldapadmin

修改phpldapadmin的配置文件，vim /etc/phpldapadmin
Change the red value to the way you will be referencing your server, either through domain name or IP address.

    $servers->setValue(‘server’ , ‘host’, ‘domain_nam_or_IP_address‘;

For the next part, you will need to reflect the same value you gave when asked for the DNS domain name when we reconfigured “slapd”.

You will have to convert it into a format that LDAP understands by separating each domain component. Domain components are anything that is separated by a dot.

These components are then given as values to the “dc” attribute.

For instance, if your DNS domain name entry was “imaginary.lalala.com”, LDAP would need to see “dc=imaginary,dc=lalala,dc=com”. Edit the following entry to reflect the name you selected (ours is “test.com” as you recall):

    $servers->setValue(‘server’,’base’,array(‘dc=test,dc=com‘));

The next value to modify will use the same domain components that you just set up in the last entry. Add these after the “cn=admin” in the entry below:

    $servers->setValue(‘login’,’bind_id’,’cn=admin,dc=test,dc=com‘);

Search for the following section about the “hidetemplatewarning” attribute. We want to uncomment this line and set the value to “true” to avoid some annoying warnings that are unimportant.

    $config->custom->appearance[‘hide_template_warning’] = true;

Save and close the file.

配置ldap
sudo dpkg-reconfigure slapd

启动
#sudo service apache2 restart
sudo service slapd  restart

需要的端口：
389默认端口

# 636 SSL端口



添加在phpldapadmin中添加用户报错：
Error in phpldapadmin
Error trying to get a non-existant value (appearance,password_hash)
解决办法：
`/usr/share/phpldapadmin/lib/TemplateRender.php` on line 2469 to `password_hash_custom`.

导入已有的用户：
sudo ldapadd -H ldap://127.0.0.1 -x -D "cn=admin,dc=auto-link,dc=com,dc=cn" -f /root/t_back.ldif -w admin



