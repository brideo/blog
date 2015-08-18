---
layout: post
title: Installing Magento 2 with PuPHPet
categories: magento2
---

Head over to PuPHPet and setup a machine using the [config.yaml](https://github.com/brideo/magento2-yaml-file) file. To do this you can drag the file from your machine onto the PuPHPet homepage then click through the steps and download.

Once you have extracted the file PuPHPet has provided you with create a `magento2` directory, then provision Vagrant.

	cd ~/path/to/PuPHPet
	mkdir ./magento2
	vagrant up

Once that has finished running, it may take around 15/20 minutes, move into your `magento2` directory, remove the `html` folder and replace with the magento2 repository. Once Magento2 has downloaded lets install [composer](https://getcomposer.org/).

	cd magento2
	rm -rf html
	git clone git@github.com:magento/magento2.git html
	cd html
	composer install
	
Once that is all okay, you will need to add the following line into your hosts file:

	sudo vim /etc/hosts
	192.168.56.101 magento2.dev

Now you should be able to access your machine in the browser using the URL [magento2.dev](http://magento2.dev/). You can install Magento from here, however this didn't work for me. I had to SSH into my Vagrant Box and install Magento using their new tool.	

	cd ~/path/to/PuPHPet
	vagrant ssh
	cd /var/www/magento2/html
	bin/magento setup:install --db-host=localhost --db-name=magento --db-user=root --db-password=123 --base-url=http://magento2.dev/ --backend-frontname=admin --admin-firstname=Admin --admin-lastname=admin --admin-email=brideo@example.com --admin-user=admin --admin-password=password

It looks like Magento2 uses the [OyeJorge Less](https://github.com/oyejorge/less.php) library to compile it's css. If your CSS isn't loading you will need to compile that. I just bumped up my max execution time and loaded each file manually in the browser to be sure.

Done, Magento 2 should now be working!
