# Redmine_on_GCP
Readmine (http://www.redmine.org/) Installation on Google Cloud Platform (GCP)

## Installation
* Step 1 
Install and Configure LAMP

* Step 2 
Install Ruby and Rails
```
[[備註] 安裝 RVM and Ruby@Ubuntu 13.04 x64](http://www.kenming.idv.tw/note_install_rvm-and-ruby_at-ubuntu-13-04-x64/)
```

And then,
```
$ gem install rails -v 5.2
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



## Environment Configuration
o Ubuntu 16.04.5 LTS (GNU/Linux 4.15.0-1026-gcp x86_64)。
o Apache 2.4。
o MySQL 5。
o Ruby 2.4 (via RVM Installation)。
o Rails 5.2 (via GEM Installation)。
o Redmine 4.0.0 (2018-12-09)。

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


![alt tag](https://www.luisblasco.com/wp-content/uploads/2018/12/redmine-40-version.jpg)
* [Redmine.JP Blog 分類 "4.0" に含まれるページ](http://blog.redmine.jp/tags/4.0/)