## Redmine_on_GCP
Installing Readmine (http://www.redmine.org/) that integrated with Git/GitHub  hosts on Google Cloud Platform (GCP)

## Installation
* Step 1 
Install and Configure LAMP

[Redmine 2.3.0 on Ubuntu 12.04](https://hirooka.pro/?p=1139)
[HowTo-安裝 Redmine 2.3.2＠EC2-Ubuntu 12.04](http://www.kenming.idv.tw/howto-install_redmine_232_at_ec2_ubuntu_1204/)

Setup privieges and owner of Apache directories
```
$ sudo usermod -a -G www-data user-account  # 讓網站管理者(login user)的帳號加入 www-data 群組
$ sudo chgrp www-data /var/www  # 改變 /var/www 所屬群組為 www-data
$ sudo chmod 775 /var/www
$ sudo chmod g+s /var/www  # 讓爾後在該目錄下所新增的檔案目錄群組擁有者為 www-data
```

* Step 2 
Install Ruby and Rails

[[備註] 安裝 RVM and Ruby@Ubuntu 13.04 x64](http://www.kenming.idv.tw/note_install_rvm-and-ruby_at-ubuntu-13-04-x64/)


And then,
```
$ gem install rails -v 5.2
```

* Step 3 
Check version of ruby, gem, and rails

```
$ ruby -v
ruby 2.4.5p335 (2018-10-18 revision 65137) [x86_64-linux]
```

```
$ gem -v
2.7.8
```

```
$ gem list --local
*** LOCAL GEMS ***

rails (5.2.0)
```

* Step 4 
DB Setup

```
$ mysql -u root -p

> CREATE DATABASE redmine CHARACTER SET utf8;
> CREATE USER 'redmine'@'localhost' IDENTIFIED BY 'my_password';
> GRANT ALL PRIVILEGES ON redmine.* TO 'redmine'@'localhost';
> flush privileges;
```

* Step 5 
Redmine Installation

Download and tar Readmine
```
 $ wget http://www.redmine.org/releases/redmine-4.0.0.tar.gz
 $ tar -zxvf redmine-4.0.0.tar.gz
 $ sudo mv redmine-4.0.0 /usr/local/
 $ sudo ln -s /usr/local/redmine-4.0.0 /usr/local/redmine
```

DB realted parameters configuration
``` 
 $ cp /usr/local/redmine/config/database.yml.example /usr/local/redmine/config/database.yml
 $ sudo nano /usr/local/redmine/config/database.yml

ex:
production:
  adapter: mysql2
  database: redmine
  host: localhost
  username: redmine
  password: "my_password"
  encoding: utf8
```

Redmine realted gem dependencies
```
$ gem install bundler
$ cd /usr/local/redmine
:/usr/local/redmine$ bundle install --without development test postgresql sqlite
```

DB generate session store secret, migrate and load default data
```
:/usr/local/redmine$ bundle exec rake generate_secret_token
:/usr/local/redmine$ RAILS_ENV=production bundle exec rake db:migrate
:/usr/local/redmine$ RAILS_ENV=production bundle exec rake redmine:load_default_data
```

Modify some directories privileges
(Apache 上の Passenger で Redmine を実行するための設定)
```
/usr/local/redmine$ sudo chown -R user-account:www-data files log tmp public/plugin_assets config.ru
/usr/local/redmine$ sudo chmod -R 755 files log tmp public/plugin_assets
```

Redmine Testing
```
/usr/local/redmine$ ruby script/rails server webrick -e production
ruby: No such file or directory -- script/rails (LoadError)

/usr/local/redmine$ ruby bin/rails server webrick -e production
```
![alt tag](https://i.imgur.com/1qVkvFh.jpg)


Apache Passenger Installation and Configuration
```
$ gem install passenger --no-rdoc --no-ri
$ passenger-install-apache2-module
```
```
執行 passenger-install-apache2-module 指令時，常會提示有缺失相關的 dependencies 套件，此時只要依照提示的訊息一一安裝即可。記得因為這些套件需要使用 root 權限才可安裝，所以安裝時須加上 sudo apt-get install xxx-package。
```

$ passenger-install-apache2-module sucessful msg:
![alt tag](https://i.imgur.com/W9kOOOZ.jpg)

$ cd /etc/apache2/mods-available/
$ sudo nano passenger.conf
```
<IfModule mod_passenger.c>
     PassengerRoot /home/XXXXX/.rvm/gems/ruby-2.4.5/gems/passenger-6.0.1
     PassengerDefaultRuby /home/XXXXX/.rvm/gems/ruby-2.4.5/wrappers/ruby
</IfModule>
```

sudo nano passenger.load
```
LoadModule passenger_module /home/XXXXX/.rvm/gems/ruby-2.4.5/gems/passenger-6.0.1/buildout/apache2/mod_passenger.so
```

Create symbolic link of above two files @/etc/apache2/mods-enabled
```
$ cd /etc/apache2/mods-enabled/
$ sudo ln -s /etc/apache2/mods-available/passenger.conf
$ sudo ln -s /etc/apache2/mods-available/passenger.load
```

* Step 6 
☆☆☆Virutal Host of Apache2 Configuration☆☆☆
```
$ cd /var/www/
$ rails new testrails
$ ln -s /var/www/testrails/public /var/www/iinfo
$ sudo nano /etc/apache2/sites-available/000-default.conf
```
![alt tag](https://i.imgur.com/BF0sPIv.jpg)

```
$ apache2ctl configtest
$ a2ensite 000-default.conf
$ sudo /etc/init.d/apache2 restart
```

Rails Testing
http://public-ip/iinfo
![alt tag](https://i.imgur.com/Vx0XDhX.jpg)

* Step 7
☆☆☆OpenSSL Configuration for Apache2☆☆☆

OpenSSL Installation
```
$ dpkg --get-selections | grep openssl
$ sudo a2enmod ssl
```
![alt tag](https://i.imgur.com/jSHw5sD.jpg)

SSL connection for Apache2
```
$ openssl genrsa -out myserver.key 2048
$ openssl req -new -key myserver.key -out myserver.csr
$ openssl x509 -req -days 365 -in myserver.csr -signkey myserver.key -out myserver.crt
```
![alt tag](https://i.imgur.com/mgbuHNM.jpg)

CA Installation
```
$ mkdir /var/www/ssl
$ cp myserver.crt /var/www/ssl
$ cp myserver.key /var/www/ssl
```

Modify Apache2 configuration file
```
$ sudo vi /etc/apache2/sites-available/000-default.conf
```
![alt tag](https://i.imgur.com/sJxH3k4.jpg)

Restart Apache2
```
$ a2ensite 000-default.conf
$ sudo /etc/init.d/apache2 restart
```
Open browser https://public-ip/iinfo
![alt tag](https://i.imgur.com/wU750gx.jpg)
![alt tag](https://i.imgur.com/H0SpF0p.jpg)

Modify Apache2 configuration file for Redmine
```
$ sudo vi /etc/apache2/sites-available/000-default.conf
```
![alt tag](https://i.imgur.com/zuUYfsX.jpg)

Restart Apache2
```
$ a2ensite 000-default.conf
$ sudo /etc/init.d/apache2 restart
```
Open browser https://public-ip/redmine
![alt tag](https://i.imgur.com/JpFs807.jpg)

## Troubleshooting
* RVM Installation issue
![alt tag](https://i.imgur.com/IqtsZpb.jpg)

Solution:
[GPG can't check signature](https://askubuntu.com/questions/56841/gpg-cant-check-signature)

Then,
```
$ source /home/amyfanpti/.rvm/scripts/rvm
```

* RVM is not a function, selecting rubies with 'rvm use ...' will not work.

Solution:
[RVM is not a function, selecting rubies with 'rvm use …' will not work](https://superuser.com/questions/695710/rvm-is-not-a-function-selecting-rubies-with-rvm-use-will-not-work)

Then,
```
export PATH="$PATH:$HOME/.rvm/bin" # Add RVM to PATH for scripting
[[ -s "$HOME/.rvm/scripts/rvm" ]] && source "$HOME/.rvm/scripts/rvm"
```

* bundle install --without development test postgresql sqlite ...error exit.

Solution:
```
$ sudo apt-get install libmysqlclient-dev imagemagick libmagickwand-dev
```

* Rails Testing Error msg
![alt tag](https://i.imgur.com/W77ph94.jpg)

Solution:
[Could not find a JavaScript runtime. See https://github.com/sstephenson/execjs for a list of available runtimes. (ExecJS::RuntimeUnavailable)](https://stackoverflow.com/questions/8059332/could-not-find-a-javascript-runtime-see-https-github-com-sstephenson-execjs-f)
```
sudo apt-get install nodejs
```

## Environment Configuration
```
o Ubuntu 16.04.5 LTS (GNU/Linux 4.15.0-1026-gcp x86_64)。
o Apache 2.4。
o MariaDB 10.0.36。
o Ruby 2.4 (via RVM Installation)。
o Rails 5.2 (via GEM Installation)。
o Redmine 4.0.0 (2018-12-09)。
```

## Reference 
![alt tag](https://upload.wikimedia.org/wikipedia/commons/thumb/3/3f/Redmine_logo.svg/2000px-Redmine_logo.svg.png)
* [Ubuntu安裝Redmine(1) --- 前置準備作業 ](http://white5168.blogspot.com/2015/11/ubunturedmine-1.html#.XDf7381-WUk)
```
安裝過程在網路上搜尋了很多網站內容，在這裡不免抱怨一下，關於網路上很多Redmine安裝教學文都是相互抄襲，完全不做驗證，在安裝過程中發生了很多問題卻也不見到有人額外說明，按照這樣安裝文章做的話，多半會卡死在重要的關鍵點上，如安裝路徑設定問題、Server啟動問題等，鑒於此筆者決定從頭一步步將所有元件安裝起來，完成最後的Redmine，中間過程雖然很耗時、很折騰人，但皇天不負苦心人讓筆者可以完成所有的安裝並做紀錄。 
```

* [Ubuntu安裝Redmine(2) --- Ruby + Rails安裝 ](http://white5168.blogspot.com/2015/11/ubunturedmine-2-ruby-rails.html#.XDf9Fc1-XV8)

* [Ubuntu安裝Redmine(3) --- Apache2 + MySQL + PHP安裝](http://white5168.blogspot.com/2015/11/ubuntu-ruby-rails-apache2-mysql-redmine.html#.XDf-Rc1-XV8)

* [Ubuntu安裝Redmine(4) --- Passenger安裝、Rails與Apache2整合佈署設定](http://white5168.blogspot.com/2015/11/ubuntu-ruby-rails-apache2-mysql-ssl_15.html#.XDf_Qs1-XV8)

* [Ubuntu安裝Redmine(5) --- OpenSSL啟用](http://white5168.blogspot.com/2015/11/ubuntu-ruby-rails-apache2-mysql-ssl.html#.XDf_uc1-XV8)

* [Ubuntu安裝Redmine(6) --- Redmine安裝](http://white5168.blogspot.com/2015/11/ubuntu-ruby-rails-apache2-mysql-ssl_18.html#.XDgAp81-XV8)

* [Ubuntu安裝Redmine(7) --- Plugin安裝](http://white5168.blogspot.com/2015/11/ubunturedmine7-plugin.html#.XDgBKs1-XV8)

* [Installing Redmine Ruby interpreter](http://www.redmine.org/projects/redmine/wiki/RedmineInstall)

![alt tag](https://www.luisblasco.com/wp-content/uploads/2017/07/Redmine-3-4.jpg)
* [Redmine.JP Blog 分類 "3.4" に含まれるページ](http://blog.redmine.jp/tags/3.4/)
* [HowTo-安裝 Redmine 2.3.2＠EC2-Ubuntu 12.04](http://www.kenming.idv.tw/howto-install_redmine_232_at_ec2_ubuntu_1204/)
* [HowTo-Redmine 整合 Git/GitHub](https://www.kenming.idv.tw/howto_redmine_integrate_git_and_github/)
* [Redmine 2.3.0 on Ubuntu 12.04](https://hirooka.pro/?p=1139)
* [[備註] 安裝 RVM and Ruby@Ubuntu 13.04 x64](http://www.kenming.idv.tw/note_install_rvm-and-ruby_at-ubuntu-13-04-x64/)
* [GPG can't check signature](https://askubuntu.com/questions/56841/gpg-cant-check-signature)
* [Clear cookies and site data in Firefox Clear cookies for a single website](https://support.mozilla.org/en-US/kb/clear-cookies-and-site-data-firefox?redirectlocale=en-US&redirectslug=delete-cookies-remove-info-websites-stored#w_clear-cookies-for-a-single-website)
* [Installing Redmine on CentOS 6.2 Wiht MySQL and Apache](http://mrtn.me/blog/2012/07/06/installing-redmine-on-centos-6-dot-2-wiht-mysql-and-apache/)
* [Install Redmine integrated with Apache HTTP Server CentOS 6.3 x86](https://blog.xuite.net/zack_pan/blog/65262495-Install+Redmine+integrated+with+Apache+HTTP+Server)
* [GIT over HTTP (GIT HTTP Transparent) CentOS 6.3 x86](https://blog.xuite.net/zack_pan/blog/65273998)

* [redmine crashed after installing MariaDB March 29, 2015](https://www.digitalocean.com/community/questions/redmine-crashed-after-installing-mariadb)

![alt tag](https://www.luisblasco.com/wp-content/uploads/2018/12/redmine-40-version.jpg)
* [Redmine.JP Blog 分類 "4.0" に含まれるページ](http://blog.redmine.jp/tags/4.0/)