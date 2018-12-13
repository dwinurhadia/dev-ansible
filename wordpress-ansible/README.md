# Demo #
Demo program dengan cara install wordpress yang akan dipasang di 3 buah server linux ubuntu 16.04

## Step 1 - Install ansible ##

    vagrant@ubuntuc:~/dev-ansible$ sudo apt-get install ansible -y

Kemudian cek versi ansible yang berjalan

    vagrant@ubuntuc:~/dev-ansible$ ansible --version

Maka akan tampil seperti berikut

    ansible 2.0.0.2
      config file = /etc/ansible/ansible.cfg
      configured module search path = Default w/o overrides
    vagrant@ubuntuc:~/dev-ansible$

## Step 2 - Setting struktur file dan folder ##

Buat folder dengan nama `wordpress-ansible` dan masuk ke `directory` tersebut

    vagrant@ubuntuc:~/dev-ansible$ mkdir wordpress-ansible && cd wordpress-ansible
    vagrant@ubuntuc:~/dev-ansible/wordpress-ansible$

Kemudian buatlah file dengan nama `playbook.yml` dan `hosts`

    vagrant@ubuntuc:~/dev-ansible/wordpress-ansible$ touch playbook.yml
    vagrant@ubuntuc:~/dev-ansible/wordpress-ansible$ touch hosts


Untuk mempermudah, bisa dibuat pemisah di playbook dengan beberapa roles

    - server
    - php
    - mysql
    - wordpress

Dari directory `dev-ansible/wordpress-ansible` buatlah directory dengan nama `roles`, kemudian masuk ke directory tersebut

	vagrant@ubuntuc:~/dev-ansible/wordpress-ansible$ mkdir roles && cd roles
	vagrant@ubuntuc:~/dev-ansible/wordpress-ansible/roles$

Kemudian jalankan `ansible-galaxy init` untuk generate roles tadi

	vagrant@ubuntuc:~/dev-ansible/wordpress-ansible/roles$ ansible-galaxy init server
	- server was created successfully
	vagrant@ubuntuc:~/dev-ansible/wordpress-ansible/roles$ ansible-galaxy init php
	- php was created successfully
	vagrant@ubuntuc:~/dev-ansible/wordpress-ansible/roles$ ansible-galaxy init mysql
	- mysql was created successfully
	vagrant@ubuntuc:~/dev-ansible/wordpress-ansible/roles$ ansible-galaxy init wordpress
	- wordpress was created successfully

## Step 3 - Tulis Playbooknya ##

Edit inventory ( daftarnya di file `hosts` )
Tambahkan pada file hosts dengan `[wordpress]` kemudian list domain/ipnya dibawahnya ( dalam hal ini saya menggunakan domain yang sudah di daftarkan di /etc/hosts )

    	[wordpress]
    	ubuntu1
    	ubuntu2
    	ubuntu3

Kemudian edit file `playbook.yml` seperti pada berikut

    - hosts: wordpress
      roles:
	    - server
	    - php
	    - mysql
	    - wordpress

Pindah ke directory `wordpress-ansible`. 

    cd ../wordpress-ansible

Kemudian coba jalankan playbook dengan perintah

	ansible-playbook playbook.yml -i hosts -u vagrant -K

Jika berhasil maka akan ada `state=ok`

	vagrant@ubuntuc:~/dev-ansible/wordpress-ansible$ ansible-playbook playbook.yml -i hosts -u vagrant -K
	SUDO password:
	
	PLAY ***************************************************************************
	
	TASK [setup] *******************************************************************
	ok: [ubuntu3]
	ok: [ubuntu1]
	ok: [ubuntu2]
	
	PLAY RECAP *********************************************************************
	ubuntu1                    : ok=1    changed=0    unreachable=0    failed=0
	ubuntu2                    : ok=1    changed=0    unreachable=0    failed=0
	ubuntu3                    : ok=1    changed=0    unreachable=0    failed=0
	
	vagrant@ubuntuc:~/dev-ansible/wordpress-ansible$

Menandakan bahwa ansible server sudah bisa terhubung ke client client yand dideklarasikan

## Step 4 - Membuat Roles ##

### Server

Edit file di `roles/server/task/main.yml`

	- name: Update apt cache
	  apt: update_cache=yes cache_valid_time=3600
	  sudo: yes
	
	- name: Install required software
	  apt: name={{ item }} state=present
	  sudo: yes
	  with_items:
	    - apache2
	    - mysql-server
	    - php-mysql
	    - php
	    - libapache2-mod-php
	    - php-mcrypt
	    - python-mysqldb

Maksudnya adalah untuk update package, install `apache, mysql, dan php` untuk ke-3 server `ubuntu1, ubuntu2,ubuntu3`

Kemudian jalankan kembali perintah

	vagrant@ubuntuc:~/dev-ansible/wordpress-ansible$ ansible-playbook playbook.                                                                                  yml -i hosts -u vagrant -K
	SUDO password:
	[DEPRECATION WARNING]: Instead of sudo/sudo_user, use become/become_user an                                                                                  d
	make sure become_method is 'sudo' (default). This feature will be removed i                                                                                  n a
	future release. Deprecation warnings can be disabled by setting
	deprecation_warnings=False in ansible.cfg.
	
	PLAY **********************************************************************                                                                                  *****
	
	TASK [setup] **************************************************************                                                                                  *****
	ok: [ubuntu1]
	ok: [ubuntu3]
	ok: [ubuntu2]
	
	TASK [server : Update apt cache] ******************************************                                                                                  *****
	ok: [ubuntu2]
	ok: [ubuntu1]
	ok: [ubuntu3]
	
	TASK [server : Install required software] *********************************                                                                                  *****
	changed: [ubuntu1] => (item=[u'apache2', u'mysql-server', u'php-mysql', u'php', u'libapache2-mod-php', u'php-mcrypt', u'python-mysqldb'])
	changed: [ubuntu2] => (item=[u'apache2', u'mysql-server', u'php-mysql', u'php', u'libapache2-mod-php', u'php-mcrypt', u'python-mysqldb'])
	changed: [ubuntu3] => (item=[u'apache2', u'mysql-server', u'php-mysql', u'php', u'libapache2-mod-php', u'php-mcrypt', u'python-mysqldb'])
	
	PLAY RECAP *********************************************************************
	ubuntu1                    : ok=3    changed=1    unreachable=0    failed=0
	ubuntu2                    : ok=3    changed=1    unreachable=0    failed=0
	ubuntu3                    : ok=3    changed=1    unreachable=0    failed=0
	
	vagrant@ubuntuc:~/dev-ansible/wordpress-ansible$

### PHP

Kemudian edit role untuk `php`

	---
	# tasks file for php
	- name: Install php extensions
	  apt: name={{ item }} state=present
	  sudo: yes
	  with_items:
	    - php5-gd
	    - libssh2-php

jalankan script ansible playbook lagi

	vagrant@ubuntuc:~/dev-ansible/wordpress-ansible$ ansible-playbook playbook.yml -                                                                             i hosts -u vagrant -K
	SUDO password:
	[DEPRECATION WARNING]: Instead of sudo/sudo_user, use become/become_user and
	make sure become_method is 'sudo' (default). This feature will be removed in a
	future release. Deprecation warnings can be disabled by setting
	deprecation_warnings=False in ansible.cfg.
	
	PLAY ***************************************************************************
	
	TASK [setup] *******************************************************************
	ok: [ubuntu2]
	ok: [ubuntu1]
	ok: [ubuntu3]
	
	TASK [server : Update apt cache] ***********************************************
	ok: [ubuntu1]
	ok: [ubuntu2]
	ok: [ubuntu3]
	
	TASK [server : Install required software] **************************************
	ok: [ubuntu1] => (item=[u'apache2', u'mysql-server', u'php-mysql', u'php', u'lib                                                                             apache2-mod-php', u'php-mcrypt', u'python-mysqldb'])
	ok: [ubuntu2] => (item=[u'apache2', u'mysql-server', u'php-mysql', u'php', u'lib                                                                             apache2-mod-php', u'php-mcrypt', u'python-mysqldb'])
	ok: [ubuntu3] => (item=[u'apache2', u'mysql-server', u'php-mysql', u'php', u'lib                                                                             apache2-mod-php', u'php-mcrypt', u'python-mysqldb'])
	
	TASK [php : Install php extensions] ********************************************
	changed: [ubuntu1] => (item=[u'php-gd', u'php-ssh2'])
	changed: [ubuntu2] => (item=[u'php-gd', u'php-ssh2'])
	changed: [ubuntu3] => (item=[u'php-gd', u'php-ssh2'])
	
	PLAY RECAP *********************************************************************
	ubuntu1                    : ok=4    changed=1    unreachable=0    failed=0
	ubuntu2                    : ok=4    changed=1    unreachable=0    failed=0
	ubuntu3                    : ok=4    changed=1    unreachable=0    failed=0
	
	vagrant@ubuntuc:~/dev-ansible/wordpress-ansible$

Maka akan muncul status berhasil install php-ekstensi

### MySQL

Kemudian deklarasikan variable database `mysql` di role `mysql`

Edit file `role/mysql/default/main.yml`

	wp_mysql_db: wordpress
	wp_mysql_user: wordpress
	wp_mysql_password: Hello-123 <- isikan sesuai keinginan

Kemudian buat task mysql di `roles/mysql/tasks/main.yml`

	- name: Create mysql database
	  sudo : yes
      mysql_db: name={{ wp_mysql_db }} state=present
	
	- name: Create mysql user
	  sudo : yes
	  mysql_user: 
	    name={{ wp_mysql_user }} 
	    password={{ wp_mysql_password }} 
	    priv=*.*:ALL

Kemudian jalankan kembali ansible-playbooknya

	vagrant@ubuntuc:~/dev-ansible/wordpress-ansible$ ansible-playbook playbook.yml -i hosts -u vagrant -K
	SUDO password:
	[DEPRECATION WARNING]: Instead of sudo/sudo_user, use become/become_user and make sure become_method is 'sudo' (default). This feature will be removed in a
	future release. Deprecation warnings can be disabled by setting deprecation_warnings=False in ansible.cfg.
	
	PLAY ***************************************************************************
	
	TASK [setup] *******************************************************************
	ok: [ubuntu1]
	ok: [ubuntu2]
	ok: [ubuntu3]
	
	TASK [server : Update apt cache] ***********************************************
	ok: [ubuntu1]
	ok: [ubuntu2]
	ok: [ubuntu3]
	
	TASK [server : Install required software] **************************************
	ok: [ubuntu1] => (item=[u'apache2', u'mysql-server', u'php-mysql', u'php', u'libapache2-mod-php', u'php-mcrypt', u'python-mysqldb'])
	ok: [ubuntu2] => (item=[u'apache2', u'mysql-server', u'php-mysql', u'php', u'libapache2-mod-php', u'php-mcrypt', u'python-mysqldb'])
	ok: [ubuntu3] => (item=[u'apache2', u'mysql-server', u'php-mysql', u'php', u'libapache2-mod-php', u'php-mcrypt', u'python-mysqldb'])
	
	TASK [php : Install php extensions] ********************************************
	ok: [ubuntu1] => (item=[u'php-gd', u'php-ssh2'])
	ok: [ubuntu2] => (item=[u'php-gd', u'php-ssh2'])
	ok: [ubuntu3] => (item=[u'php-gd', u'php-ssh2'])
	
	TASK [mysql : Create mysql database] *******************************************
	changed: [ubuntu3]
	changed: [ubuntu1]
	changed: [ubuntu2]
	
	TASK [mysql : Create mysql user] ***********************************************
	changed: [ubuntu2]
	changed: [ubuntu1]
	changed: [ubuntu3]
	
	PLAY RECAP *********************************************************************
	ubuntu1                    : ok=6    changed=2    unreachable=0    failed=0
	ubuntu2                    : ok=6    changed=2    unreachable=0    failed=0
	ubuntu3                    : ok=6    changed=2    unreachable=0    failed=0
	
	vagrant@ubuntuc:~/dev-ansible/wordpress-ansible$

### Wordpress

Kemudian edit task `wordpress`

	- name: Download WordPress  get_url: 
	    url=https://wordpress.org/latest.tar.gz 
	    dest=/tmp/wordpress.tar.gz
	    validate_certs=no
	
	- name: Extract WordPress  unarchive: src=/tmp/wordpress.tar.gz dest=/var/www/ copy=no
	  sudo: yes
	
	- name: Update default Apache site
	  sudo: yes
	  lineinfile: 
	    dest=/etc/apache2/sites-enabled/000-default.conf 
	    regexp="(.)+DocumentRoot /var/www/html"
	    line="DocumentRoot /var/www/wordpress"
	  notify:
	    - restart apache
	
	- name: Copy sample config file
	  command: mv /var/www/wordpress/wp-config-sample.php /var/www/wordpress/wp-config.php creates=/var/www/wordpress/wp-config.php
	  sudo: yes
	
	- name: Update WordPress config file
	  lineinfile:
	    dest=/var/www/wordpress/wp-config.php
	    regexp="{{ item.regexp }}"
	    line="{{ item.line }}"
	  with_items:
	    - {'regexp': "define\\('DB_NAME', '(.)+'\\);", 'line': "define('DB_NAME', '{{wp_mysql_db}}');"}        
	    - {'regexp': "define\\('DB_USER', '(.)+'\\);", 'line': "define('DB_USER', '{{wp_mysql_user}}');"}        
	    - {'regexp': "define\\('DB_PASSWORD', '(.)+'\\);", 'line': "define('DB_PASSWORD', '{{wp_mysql_password}}');"}
	  sudo: yes

Kemudian tambahkan `handler` wordpress

	---
	# handlers file for wordpress
	- name: restart apache
	  service: name=apache2 state=restarted
	  sudo: yes

Jalankan semua asible-playbooknya

	vagrant@ubuntuc:~/dev-ansible/wordpress-ansible$ ansible-playbook playbook.yml -i hosts -u vagrant -K
	SUDO password:
	[DEPRECATION WARNING]: Instead of sudo/sudo_user, use become/become_user and make sure become_method is 'sudo' (default). This feature will be removed in a
	future release. Deprecation warnings can be disabled by setting deprecation_warnings=False in ansible.cfg.
	
	PLAY ***************************************************************************
	
	TASK [setup] *******************************************************************
	ok: [ubuntu1]
	ok: [ubuntu2]
	ok: [ubuntu3]
	
	TASK [server : Update apt cache] ***********************************************
	ok: [ubuntu1]
	ok: [ubuntu3]
	ok: [ubuntu2]
	
	TASK [server : Install required software] **************************************
	ok: [ubuntu1] => (item=[u'apache2', u'mysql-server', u'php-mysql', u'php', u'libapache2-mod-php', u'php-mcrypt', u'python-mysqldb'])
	ok: [ubuntu2] => (item=[u'apache2', u'mysql-server', u'php-mysql', u'php', u'libapache2-mod-php', u'php-mcrypt', u'python-mysqldb'])
	ok: [ubuntu3] => (item=[u'apache2', u'mysql-server', u'php-mysql', u'php', u'libapache2-mod-php', u'php-mcrypt', u'python-mysqldb'])
	
	TASK [php : Install php extensions] ********************************************
	ok: [ubuntu1] => (item=[u'php-gd', u'php-ssh2'])
	ok: [ubuntu3] => (item=[u'php-gd', u'php-ssh2'])
	ok: [ubuntu2] => (item=[u'php-gd', u'php-ssh2'])
	
	TASK [mysql : Create mysql database] *******************************************
	ok: [ubuntu1]
	ok: [ubuntu3]
	ok: [ubuntu2]
	
	TASK [mysql : Create mysql user] ***********************************************
	ok: [ubuntu1]
	ok: [ubuntu2]
	ok: [ubuntu3]
	
	TASK [wordpress : Download WordPress] ******************************************
	changed: [ubuntu3]
	changed: [ubuntu1]
	changed: [ubuntu2]
	
	TASK [wordpress : Extract WordPress] *******************************************
	changed: [ubuntu2]
	changed: [ubuntu3]
	changed: [ubuntu1]
	
	TASK [wordpress : Update default Apache site] **********************************
	changed: [ubuntu2]
	changed: [ubuntu1]
	changed: [ubuntu3]
	
	TASK [wordpress : Copy sample config file] *************************************
	changed: [ubuntu1]
	changed: [ubuntu3]
	changed: [ubuntu2]
	
	TASK [wordpress : Update WordPress config file] ********************************
	changed: [ubuntu1] => (item={u'regexp': u"define\\('DB_NAME', '(.)+'\\);", u'line': u"define('DB_NAME', 'wordpress');"})
	changed: [ubuntu1] => (item={u'regexp': u"define\\('DB_USER', '(.)+'\\);", u'line': u"define('DB_USER', 'wordpress');"})
	changed: [ubuntu1] => (item={u'regexp': u"define\\('DB_PASSWORD', '(.)+'\\);", u'line': u"define('DB_PASSWORD', 'Hello-123');"})
	changed: [ubuntu2] => (item={u'regexp': u"define\\('DB_NAME', '(.)+'\\);", u'line': u"define('DB_NAME', 'wordpress');"})
	changed: [ubuntu2] => (item={u'regexp': u"define\\('DB_USER', '(.)+'\\);", u'line': u"define('DB_USER', 'wordpress');"})
	changed: [ubuntu2] => (item={u'regexp': u"define\\('DB_PASSWORD', '(.)+'\\);", u'line': u"define('DB_PASSWORD', 'Hello-123');"})
	changed: [ubuntu3] => (item={u'regexp': u"define\\('DB_NAME', '(.)+'\\);", u'line': u"define('DB_NAME', 'wordpress');"})
	changed: [ubuntu3] => (item={u'regexp': u"define\\('DB_USER', '(.)+'\\);", u'line': u"define('DB_USER', 'wordpress');"})
	changed: [ubuntu3] => (item={u'regexp': u"define\\('DB_PASSWORD', '(.)+'\\);", u'line': u"define('DB_PASSWORD', 'Hello-123');"})
	
	RUNNING HANDLER [wordpress : restart apache] ***********************************
	changed: [ubuntu1]
	changed: [ubuntu3]
	changed: [ubuntu2]
	
	PLAY RECAP *********************************************************************
	ubuntu1                    : ok=12   changed=6    unreachable=0    failed=0
	ubuntu2                    : ok=12   changed=6    unreachable=0    failed=0
	ubuntu3                    : ok=12   changed=6    unreachable=0    failed=0
	
	vagrant@ubuntuc:~/dev-ansible/wordpress-ansible$
	
Wordpress telah berhasil terpasang pada ke-3 server tersebut
