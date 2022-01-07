# How-To-Create-a-SSL-Certificate-for-Apache-in-Ubuntu-18.04
How To Create a SSL Certificate for Apache in Ubuntu 18.04

# Introduction

TLS, or transport layer security, and its predecessor SSL, which stands for secure sockets layer, are web protocols used to wrap normal traffic in a protected, encrypted wrapper.Using this technology, servers can send traffic safely between servers and clients without the possibility of messages being intercepted by outside parties. The certificate system also assists users in verifying the identity of the sites that they are connecting with.
In this guide, we will show you how to set up a self-signed SSL certificate for use with an Apache web server on Ubuntu 18.04.
Prerequisites

You will also need to have the Apache web server installed. If you would like to install an entire LAMP (Linux, Apache, MySQL, PHP) stack on your server, you can follow our guide on setting up LAMP on Ubuntu 18.04. If you just want the Apache web server, skip the steps pertaining to PHP and MySQL.

## Configuring Apache to Use SSL

We have created our key and certificate files under the /etc/ssl directory. Now we just need to modify our Apache configuration to take advantage of these.

We will create a configuration snippet to specify strong default SSL settings.We will modify the included SSL Apache Virtual Host file to point to our generated SSL certificates.(Recommended) We will modify the unencrypted Virtual Host file to automatically redirect requests to the encrypted Virtual Host.

When we are finished, we should have a secure SSL configuration.


Modifying the Default Apache SSL Virtual Host File
Next, let’s modify /etc/apache2/sites-available/default-ssl.conf, the default Apache SSL Virtual Host file. If you are using a different server block file, substitute its name in the commands below.
Before we go any further, let’s back up the original SSL Virtual Host file:

```
sudo cp /etc/apache2/sites-available/default-ssl.conf /etc/apache2/
```

sites-available/default-ssl.conf.bak”
Now, open the SSL Virtual Host file to make adjustments:

```
sudo nano /etc/apache2/sites-available/default-ssl.conf”
```
Inside, with most of the comments removed, the Virtual Host file should look something like this by default:
```
/etc/apache2/sites-available/default-ssl.conf
```
```
<IfModule mod_ssl.c>
<VirtualHost _default_:443>
ServerAdmin webmaster@localhost

DocumentRoot /var/www/html

ErrorLog ${APACHE_LOG_DIR}/error.log
CustomLog ${APACHE_LOG_DIR}/access.log combined

SSLEngine on

SSLCertificateFile      /etc/ssl/certs/ssl-cert-fullchain.pem
SSLCertificateKeyFile /etc/ssl/private/ssl-cert-private.key

<FilesMatch "\.(cgi|shtml|phtml|php)$">
SSLOptions +StdEnvVars
</FilesMatch>
<Directory /usr/lib/cgi-bin>
SSLOptions +StdEnvVars
</Directory>

</VirtualHost>
</IfModule>
```

We will be making some minor adjustments to the file. We will set the normal things we’d want to adjust in a Virtual Host file (ServerAdmin email address, ServerName, etc., and adjust the SSL directives to point to our certificate and key files.

After making these changes, your server block should look similar to this:
```
/etc/apache2/sites-available/default-ssl.conf
```

```
<IfModule mod_ssl.c>
        <VirtualHost _default_:443>
ServerAdmin your_email@example.com
ServerName dev.bitcot.com

DocumentRoot /var/www/html

ErrorLog ${APACHE_LOG_DIR}/error.log
CustomLog ${APACHE_LOG_DIR}/access.log combined

SSLEngine on

SSLCertificateFile      /etc/ssl/certs/apache-selfsigned.crt
SSLCertificateKeyFile /etc/ssl/private/apache-selfsigned.key
<
FilesMatch "\.(cgi|shtml|phtml|php)$">
SSLOptions +StdEnvVars
</FilesMatch>
<Directory /usr/lib/cgi-bin>
SSLOptions +StdEnvVars
</Directory>

</VirtualHost>
</IfModule>
```

Save and close the file when you are finished.


To create a symlink at current directory https.conf
```
	 ln -s /etc/apache2/sites-enable https.conf /etc/apache2/sites-enable/https.conf 
```

```
<IfModule mod_ssl.c>
<VirtualHost *:443>
ServerAdmin webmaster@localhost
ServerName  dev.bitcotapps.com
DocumentRoot /var/www/html
ErrorLog ${APACHE_LOG_DIR}/error.log
CustomLog ${APACHE_LOG_DIR}/access.log combined
<Directory /var/www/html>
Options -Indexes
AllowOverride All
</Directory>
<IfModule mod_dir.c>
DirectoryIndex index.php index.html index.cgi index.pl  index.xhtml index.html
</IfModule>
Include /etc/letsencrypt/options-ssl-apache.conf
SSLEngine on
SSLCertificateFile /home/bitcot/Downloads/fullchain.pem
SSLCertificateKeyFile /home/bitcot/Downloads/privkey.pem
</VirtualHost>
</IfModule>
```



now add the point of domain in local host
    			
```
sudo nano /etc/hosts
```



 Adjusting the Firewall

If you have the ufw firewall enabled, as recommended by the prerequisite guides, you might need to adjust the settings to allow for SSL traffic. Luckily, Apache registers a few profiles with ufw upon installation.

We can see the available profiles by typing:

                    
```                 		
sudo ufw app list”
```
You should see a list like this:   

```
Output
Available applications:
  Apache
  Apache Full
  Apache Secure
  OpenSSH
```

You can see the current setting by typing:

```
sudo ufw status
```

If you allowed only regular HTTP traffic earlier, your output might look like this:

Output
Status: active
```
To                         Action      From
--                         ------      ----
OpenSSH                    ALLOW       Anywhere
Apache                     ALLOW       Anywhere
OpenSSH (v6)               ALLOW       Anywhere (v6)
Apache (v6)                ALLOW       Anywhere (v6)
```
```
UFW status inactive
```

```
sudo ufw enable	
```
then 
 		                             
```
sudo ufw status”
```

```
Output
Status: active

To                         Action      From
--                         ------      ----
OpenSSH                    ALLOW       Anywhere
Apache                     ALLOW       Anywhere
OpenSSH (v6)               ALLOW       Anywhere (v6)
Apache (v6)                ALLOW       Anywhere (v6)
```



To additionally let in HTTPS traffic, we can allow the “Apache Full” profile and then delete the redundant “Apache” profile allowance:

```      
sudo ufw allow 'Apache Full'
```
```
sudo ufw delete allow 'Apache'
```
Your status should look like this now:

```
sudo ufw status
```
```
Output
Status: active

To                         Action      From
--                         ------      ----
OpenSSH                    ALLOW       Anywhere
Apache Full                ALLOW       Anywhere
OpenSSH (v6)               ALLOW       Anywhere (v6)
Apache Full (v6)           ALLOW       Anywhere (v6)
```


Enabling the Changes in Apache
We can enable mod_ssl, the Apache SSL module, and mod_headers, needed by some of the settings in our SSL snippet, with the a2enmod command:

```
sudo a2enmod ssl
```


Next, we can enable our SSL Virtual Host with the a2ensite command:

```
sudo a2ensite default-ssl
```

```
sudo apache2ctl configtest
```
If everything is successful, you will get a result that looks like this:

```
Output
AH00558: apache2: Could not reliably determine the server's fully qualified domain name, using 127.0.1.1. Set the 'ServerName' directive globally to suppress this message
Syntax OK
```

#enable mod_rewrite for Apache 2.2

```
sudo a2enmod rewrite
```


If your output has Syntax OK in it, your configuration file has no syntax errors. We can safely restart Apache to implement our changes:

```
sudo systemctl restart apache2
```

Testing Encryption

```
https://dev.bitcotapps.com
```

Conclusion
You have configured your Apache server to use strong encryption for client connections. This will allow you serve requests securely, and will prevent outside parties from reading your traffic.




