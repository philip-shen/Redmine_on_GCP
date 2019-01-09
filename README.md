# Redmine_on_GCP
Readmine Installation (http://www.redmine.org/) that integrated with Git/GitHub  hosts on Google Cloud Platform (GCP)

## Installation
* Step 1 
Install and Configure LAMP

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

create symbolic link of above two files @/etc/apache2/mods-enabled
```
$ cd /etc/apache2/mods-enabled/
$ sudo ln -s /etc/apache2/mods-available/passenger.conf
$ sudo ln -s /etc/apache2/mods-available/passenger.load
```


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
* [Installing Redmine Ruby interpreter](http://www.redmine.org/projects/redmine/wiki/RedmineInstall)

![alt tag](https://www.luisblasco.com/wp-content/uploads/2017/07/Redmine-3-4.jpg)
* [Redmine.JP Blog 分類 "3.4" に含まれるページ](http://blog.redmine.jp/tags/3.4/)
* [HowTo-安裝 Redmine 2.3.2＠EC2-Ubuntu 12.04](http://www.kenming.idv.tw/howto-install_redmine_232_at_ec2_ubuntu_1204/)
* [HowTo-Redmine 整合 Git/GitHub](https://www.kenming.idv.tw/howto_redmine_integrate_git_and_github/)
* [Redmine 2.3.0 on Ubuntu 12.04](https://hirooka.pro/?p=1139)
* [[備註] 安裝 RVM and Ruby@Ubuntu 13.04 x64](http://www.kenming.idv.tw/note_install_rvm-and-ruby_at-ubuntu-13-04-x64/)
* [GPG can't check signature](https://askubuntu.com/questions/56841/gpg-cant-check-signature)
* [Clear cookies and site data in Firefox Clear cookies for a single website](https://support.mozilla.org/en-US/kb/clear-cookies-and-site-data-firefox?redirectlocale=en-US&redirectslug=delete-cookies-remove-info-websites-stored#w_clear-cookies-for-a-single-website)

* [redmine crashed after installing MariaDB March 29, 2015](https://www.digitalocean.com/community/questions/redmine-crashed-after-installing-mariadb)

![alt tag](https://www.luisblasco.com/wp-content/uploads/2018/12/redmine-40-version.jpg)
* [Redmine.JP Blog 分類 "4.0" に含まれるページ](http://blog.redmine.jp/tags/4.0/)