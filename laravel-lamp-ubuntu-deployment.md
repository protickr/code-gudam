# How to deploy a laravel application on digitalocean droplet (Ubuntu LAMP stack).  
  
### Including  
	* SSH   
	* GIT   
	* Composer  
	* MySQl   
	* phpMyAdmin   
	* htaccess  
	* Laravel deployment and permission  
	* Domain and sub-domain configuration  
	* Free SSL using "certbot" and traffic encryption (HTTPS) using Lets Encrypt  

> This is a deployment note for my future self, sharing with you hoping it might help.  

0. Log into you DigitalOcean dashbaord choose option and create a virtual machine that runs Ubuntu.  

1. 	login to your droplet (ubuntu VM) by,  
	> ssh root@IP_Address  
	enter password  
  
	if SSH port is different than the default port 22 you can specify port by,  
	> ssh user_name@ip_address -p <port_number>  

2. 	> sudo apt-get update  
	> sudo apt-get upgrade  
	> sudo apt-get dist-upgrade  

3.	lets install all required PHP modules in our LAMP VM,  
	> sudo apt-get install apache2 apache2-utils curl mysql-server mysql-client php libapache2-mod-php php-mysql php-common php-mbstring php-xmlrpc php-soap php-gd php-xml php-mysql php-cli php-zip  

4.	run `sudo reboot now` to reboot our vm, you will be disconnected as it reboots  

5.	now lets generate a pair of ssh keys and use that to log in to our server instead of a password.  
	we will be using our private key to make a request to the ssh server and that key-pair's public counterpart need to be stored in the target server.  

6. 	open powershell/terminal  
	> cd ~/.ssh  
  
7.	using openssh,  
	> ssh-keygen -t rsa -b 4096 -C "your comment / name signature" -f "output_file_name"  
	> ssh-keygen -t rsa -b 4096 -C "protickrr@gmail.com" -f "posx"  
  
	you will be prompted to enter a passphrase, you can enter and then confirm it or ignore it by pressing enter twice.  
	if you enter a passphrase while generating the key-pair then you will be asked for the passphrase in the time of logging in while using that key  
  
8.	Now lets add our newly generated ssh key-pair's public key to our droplet's ssh server,  

	UNIX: > cat ~/.ssh/output_file_name.pub | ssh root@<server_ip> 'cat - >> ~/.ssh/authorized_keys'  
	Windows: > get-content ~/.ssh/posx.pub | ssh root@<server_ip> $ 'cat - >> ~/.ssh/authorized_keys'  

9. 	Now lets login to our server vm using ssh and public key  
	> ssh -i <path/to/your/private/key> root@server_ip  
	> ssh -i ~/.ssh/posx root@server_ip  

10. You will be asked for a passphrase or not, depending on your previous steps.  

11. If our login attempt was successful using the private key we generated earlier, that means now we can login to our server using ssh key so lets turn off  	Password Authentication  
	> sudo nano /etc/ssh/sshd_config  
	
	change,  
	`# PasswordAuthentication yes`  
	to  
	PasswordAuthentication no  
	  
	> sudo service sshd restart  
  
12.	Lets add a general system-user,  
	> adduser dev  

	will ask for password and confirm password, enter password and remember  
	will ask for name, email address etc.  
	press enter to skip.  
  
13.	Add new user to sudo group.  
	> usermod -aG sudo dev  
	> sudo su dev  
	> exit  
  
14.	now lets switch back to our new user,  
	> sudo su dev  
	> cd ~  
	> ll  
	  
	if no .ssh directory is listed there we have to create one by the following,  
	> mkdir .ssh  
	> chmod 700 ~/.ssh  
	
	create a file called authorized_keys in the .ssh directory  
	> touch authorized_keys  
	 
	paste the public key contents of the generated key pair from step 8 in the "authorized_keys" file and save.	 
	then  
	> sudo service sshd restart  

15.	Now we can connect to our server as "dev" user with our private ssh key  
	> ssh -i ~/.ssh/posx dev@server_ip  

16. lets tidy up the login process,  
  
	create a file named "config" in .ssh directory of your local computer,  
	then in the config file insert the following, save and exit.  
	  
	`# POSX@DEV
	Host posx-dev
    	  HostName <server_ip or server_address>
    	  User <user_name>
    	  Port <port>
    	  PreferredAuthentications publickey
    	  IdentityFile <path/to/private/key>
	`
	Now to login into our server,  
	> ssh posx-dev  

17. As we have other means to access the server as a non root user so, we should revoke remote login permission of root user.  
	
	Login to server as non-root user. e.g., dev  
	> sudo nano /etc/ssh/sshd_config  
	change  
	"PermitRootLogin yes" to "PermitRootLogin no" and  
	"PasswordAuthentication yes" to "PasswordAuthentication no"  
	  
	restart the ssh server  
	> sudo service ssh restart  
	
	exit out and test root login, it should be denied by server.  
  
18.	Login as non-root "dev" user,  
	> cd ~/.ssh  
	> ssh-keygen -t rsa -b 4096 -C "posx-lamp server" -f "posx-lamp"  
	> cat posx-lamp.pub  
  
	copy contents of posx-lamp.pub  

19. go to github / bitbucket / gitlab > settings > deploy keys > add key >  
	enter title and paste key content.  
	confirm and exit  
  
20.	on your server terminal,  
	> cd /var/www/  
	> sudo rm -rf html/  
	> sudo git clone <your_repo_url>  
  
21.	Install composer  
	> sudo apt-get install composer  
  
22. Install composer packages,  
	> cd /var/www/laravel-pos  
	> composer install  

23. Change permission so apache-user (www-data) can read write and execute.  
	and "dev" user can read write and execute as well  
	  
	> sudo chown -R dev:www-data /var/www/laravel-pos  
	> sudo find /var/www/laravel-pos -type f -exec chmod 664 {} \;  
	> sudo find /var/www/laravel-pos -type d -exec chmod 775 {} \;  
	  
	before running the following commands run,  
	> git config --global core.filemode false  
  
	> sudo chgrp -R www-data storage bootstrap/cache  
	> sudo chmod -R ug+rwx storage bootstrap/cache  
  
24.	Install and configure mysql server,  
	should be installed already as per step 3.  
	  
	to configure mysql,  
	we need to create a databse and a non root user,  
	> sudo mysql_secure_installation  
	  
	"Would you like to setup VALIDATE PASSWORD component?"  
	enter y  
  
	enter password and confirm  
	answer y(yes) to all four questions.  
	  
	From this step you will get,  
	MySql  
	user: root  
	pass: posX1@Dev  
  
25.	Now we will create a new database for our application.  
	login as root by,  
  
	> sudo mysql -u root -p  
	enter password for mysql user root.  
  
	> create databse databse_name;  
	  
26.	Lets create a mysql user,  
	> create user 'username'@'localhost' identified by 'password';  
	  
	grant permission on our created database,  
	> grant all on databse_name.* to 'username'@'localhsot';  
	  
	> flush privileges;  
	> exit;  
  

27. Configure our laravel application using .env,  
	first copy .env.example to .env  
	> cp .env.example .env  
	edit .env and set database username password, other API credentials etc.  
  
	Generate application key,  
	> php artisan key:generate  

28.	Now run migration,  
	> php artisan migrate:fresh --seed  
	if your mysql and application configuration were right then migration will run successfully.  
  
29. Our application is ready. Now we have to configure apache2  
	we will be creating individual config file for each virtual host.  
  
	> sudo cp 000-default.conf laravelpos.conf  
	> sudo nano laravelpos.conf  
	  
	edit the content and point DocumnetRoot, Directory to your project's public directory.  
	`
	<VirtualHost *:80>
		ServerAdmin webmaster@localhost
		DocumentRoot /var/www/laravel-pos/public
		
		<Directory /var/www/laravel-pos/public>
			Options Indexes FollowSymLinks
			AllowOverride All
			Require all granted
		</Directory>
		
		ErrorLog ${APACHE_LOG_DIR}/error.log
		CustomLog ${APACHE_LOG_DIR}/access.log combined

		<IfModule mod_dir.c>                                                                   
			DirectoryIndex index.php index.pl index.cgi index.html index.xhtml index.htm
		</IfModule>  
	</VirtualHost>
    `  
	In case your are using a domain,  
	> sudo nano /etc/apache2/sites-available/your_domain.conf  
	Paste in the following configuration block, which is similar to the default, but updated for our new directory and domain name:
`
	<VirtualHost *:80>
		ServerAdmin webmaster@localhost
		ServerName your_domain
		ServerAlias www.your_domain
		DocumentRoot /var/www/project/public
		  
		<Directory /var/www/project/public>
			AllowOverride All
			Require all granted
		</Directory>

		ErrorLog ${APACHE_LOG_DIR}/error.log
		CustomLog ${APACHE_LOG_DIR}/access.log combined
	</VirtualHost>
`
30. Enable configuration file and disable deafault config file,  
	> sudo a2ensite laravelpos.conf  
	> sudo a2dissite 000-default.conf  
	> sudo systemctl restart apache2  
	> sudo apache2ctl configtest  
  
#### Now lets add domain to our server.

1.	Click on 3 dots icon besides droplet entry in droplet listing.  
	click on add a domain  
	enter domain and click add domain,  
	in the next page add records such as,  
	www.  
	pma.  
	  
	then keep note of the digital ocean name server addresses  
	go to your domain registerer's dashbaord and add digitalocean's name server addresses under "custom dns"  
	click on the small tick mark to submit custom nameserver entries;  
	  
	it may take upto 48 hours for changes to take effect.  
	NB: as we have not enabled any traffic encryption we wont be able to access our site via https:// but,  
	site will be accessible via,  
	http://domain_name or http://www.domain_name (if you have added "www." dns record that point to your server)  
  
#### Now lets install phpMyAdmin for easier database access, later we will configure it on a sub-domain  
  
1.	> sudo mysql -u root -p  
	> mysql > UNINSTALL COMPONENT "file://component_validate_password";  
	> mysql > exit;  
  
2.	we need to install few packages along with phpMyAdmin  
	> sudo apt install phpmyadmin php-mbstring php-zip php-gd php-json php-curl  
  
	For the server selection, choose apache2  
	Warning: When the prompt appears, “apache2” is highlighted, but not selected.  
	If you do not hit SPACE to select Apache,  
	the installer will not move the necessary files during installation. Hit SPACE, TAB, and then ENTER to select Apache.  
  
	Select Yes when asked whether to use dbconfig-common to set up the database  
	You will then be asked to choose and confirm a MySQL application password for phpMyAdmin  
	enter and confirm password for mysql user "phpmyadmin".  
  
	> sudo phpenmod mbstring  
	> sudo systemctl restart apache2  
	> sudo mysql -u root -p  
	> mysql > INSTALL COMPONENT "file://component_validate_password";  
	> exit  
	  
	phpmyadmin should be available at, your_domain/phpmyadmin  
	now, you can login and operate on database(s) by providing your msyql user credentials in phpmyadmin.  
  
3. Securing phpmyadmin installation,  
	Implement .htaccess gateway,  
	> sudo nano /etc/apache2/conf-available/phpmyadmin.conf  
	Add an  
		AllowOverride All  
	directive within the <Directory /usr/share/phpmyadmin> section of the configuration file, like this:  
	  
	`
	<Directory /usr/share/phpmyadmin>  
    	Options SymLinksIfOwnerMatch
    	DirectoryIndex index.php
    	AllowOverride All
    	. . .
	`  
	> press ctrl + o and enter to save and ctrl + x to exit  
  
	> sudo systemctl restart apache2  
  
	create a .htaccess file within the phpmyadmin directory  
	> sudo nano /usr/share/phpmyadmin/.htaccess  
		paste in the following,  
	  
	`
	AuthType Basic  
	AuthName "Restricted Files"  
	AuthUserFile /etc/phpmyadmin/.htpasswd  
	Require valid-user  
	`
	> sudo htpasswd -c /etc/phpmyadmin/.htpasswd user_name  
	enter pasword and confirm password,  
	> sudo systemctl restart apache2  
  
	lets make it available under a subdomain rather than plain diretory url,  
  
	first we will disable the sub-directory url /phpmyadmin by,  
	> sudo a2disconf phpmyadmin.conf  
	> sudo systemctl restart apache2  

	go to digital ocean domain control panel  
	add a dns record e.g., dbrowser.mapsit.link  
  
	now go to,  
	> cd /etc/apache2/sites-available/  
	> sudo cp ../conf-available/phpmyadmin.conf dbrowser.mapsit.link.conf  
  
	> sudo nano dbrowser.mapsit.link.conf  
  
	append the following at the start,  
	  
	`
	<VirtualHost *:80>  
		ServerName dbrowser.mapsit.link
		DocumentRoot /usr/share/phpmyadmin

		ErrorLog ${APACHE_LOG_DIR}/dbrowser.error.log
		CustomLog ${APACHE_LOG_DIR}/dbrowser.access.log combined
    `
	and add  
		</VirtualHost>  
	at the very last of the file,  
  
	also please note that you need to replace "dbrowser" with your chosen subdomain name and  
	name your phpmyadmin configuration file according to your records e.g., "subdomain.example.com.conf"  
  
	and comment out this line,  
		Alias /phpmyadmin /usr/share/phpmyadmin  
	to,  
		`#Alias /phpmyadmin /usr/share/phpmyadmin`  

	Finally save, exit and enable phpmyadmin site configuration, and restart apache  
  
	> sudo a2ensite dbrowser.mapsit.link.conf  
	> sudo systemctl restart apache2  

	[In case of apache start-up failure run,  
	> sudo apache2ctl configtest  
	and fix reported errors accordingly.]  
  
	Now, your site should be available at,  
	http://www.mapsit.link  
	and your phpmyadmin at,  
	http://dbrowser.mapsit.link  

#### Now lets secure Apache with "Let's Encrypt "

1. First we need install some packages,  
    > sudo apt install certbot python3-certbot-apache  
  
2. Let in HTTPS traffic, allow the “Apache Full” profile and delete the redundant “Apache” profile:  
	> sudo ufw allow 'Apache Full'  
	> sudo ufw delete allow 'Apache'  
  
3. Certbot provides a variety of ways to obtain SSL certificates through plugins.  
    The Apache plugin will take care of reconfiguring Apache and reloading the configuration whenever necessary.  
	To use this plugin, type the following:  
    > sudo certbot --apache  

	a. provide a valid email address when prompted.  
	  
	b. To Agreee to their terms and service, press A and enter.  

	c. To share your email address with Electronic Frontier Foundation press Y or N if you dont then press enter.  
  
	d. Select the appropriate numbers separated by commas and/or spaces, or leave input  
		blank to select all options shown (Enter 'c' to cancel).  
	
	e. Choose 2 to enable the redirection from http::// to https://,  
	    or 1 if you want to keep both HTTP and HTTPS as separate methods of accessing your website.  

	f. To check the status of this service and make sure it’s active and running, you can use:  
	    > sudo systemctl status certbot.timer  

	g. To test the renewal process, you can do a dry run with certbot:  
		sudo certbot renew --dry-run  

	The certbot package we installed takes care of renewals by including a renew script to /etc/cron.d,  
	which is managed by a systemctl service called certbot.timer.  
	This script runs twice a day and will automatically renew any certificate that’s within thirty days of expiration.  
  
	Restart your server,  
	https://domain_name  
	https://www.domain_name  
	https://sub-domain.domain_name  

	all should be available online.  


Additional troubleshooting tips, on error,  
"Failed to clear cache. Make sure you have the appropriate permissions"  
	If the data directory doesn't exist under (storage/framework/cache/data), then you will have this error.  
	This data directory doesn't exist by default on a fresh/new installation.  
	Creating the data directory manually at (storage/framework/cache) should fix this issue.  
  
"laravel.log could not be opened"  
	sudo chown -R $USER:www-data storage  
	sudo chown -R $USER:www-data bootstrap/cache  
	then to set directory permission try this:  
  
	chmod -R 775 storage  
	chmod -R 775 bootstrap/cache  
  
“Could not reliably determine the server's fully qualified domain name”  
	This is just a friendly warning and not really a problem (as in that something does not work).  
	If you go to:  
	/etc/apache2/apache2.conf  
	and insert:  
  
	ServerName 127.0.0.1  
	and then restart apache by typing into the terminal:  
  
	sudo systemctl reload apache2  
	the notice will disappear.  
  
-------------------------------------------------------------------------------------------------------------------------
Tutorials that we followed,  
References:  
	1.   https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys-2
	2.   https://kaloraat.com/articles/how-to-deploy-laravel-to-digital-ocean
	3.   https://devdojo.com/bobbyiliev/laravel-app-on-digital-ocean-ubuntu-1904-droplet-step-by-step-guide#create-a-droplet
	4.   https://devdojo.com/bobbyiliev/laravel-app-on-digital-ocean-ubuntu-1904-droplet-step-by-step-guide#create-a-droplet
	5.   https://gist.github.com/harryfinn/829bba2a9ad8c8a471b07016165ef82d
	6.   https://www.digitalocean.com/community/tutorials/how-to-install-and-secure-phpmyadmin-on-ubuntu-20-04
	7.   https://www.linuxbabe.com/ubuntu/install-phpmyadmin-apache-lamp-ubuntu-18-04
	8.   https://www.digitalocean.com/community/tutorials/how-to-secure-apache-with-let-s-encrypt-on-ubuntu-20-04
	9.   https://stackoverflow.com/questions/52231248/laravel-showing-failed-to-clear-cache-make-sure-you-have-the-appropriate-permi
	10.  https://stackoverflow.com/questions/23411520/how-to-fix-error-laravel-log-could-not-be-opened
	11.  https://stackoverflow.com/questions/1580596/how-do-i-make-git-ignore-file-mode-chmod-changes
	12.  https://askubuntu.com/questions/256013/apache-error-could-not-reliably-determine-the-servers-fully-qualified-domain-n