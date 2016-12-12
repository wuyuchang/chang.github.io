---
title: Config web server with Nginx & PHP7 in Linux for beginner
---

We use **Debian** to finish the demo.
Both of below are basic command line, If you are not familiar with command line in Linux, you should learn it first.

## Update & Upgrade
``` bash
$ apt-get update
$ apt-get upgrade
```
You must curious what's difference between [update][apt-get] and [upgrade][apt-get].
> **update**
Used to re-synchronize the package index files from their sources. The indexes of available packages are fetched from the location(s) specified in /etc/apt/sources.list(5). An update should always be performed before an upgrade or dist-upgrade.
**upgrade**
Used to install the newest versions of all packages currently installed on the system from the sources enumerated in /etc/apt/sources.list(5). Packages currently installed with new versions available are retrieved and upgraded; under no circumstances are currently installed packages removed, nor are packages that are not already installed retrieved and installed. New versions of currently installed packages that cannot be upgraded without changing the install status of another package will be left at their current version. An update must be performed first so that apt-get knows that new versions of packages are available.

[apt-get]: https://linux.die.net/man/8/apt-get

## Htop, Vim & Screen
``` bash
$ apt-get install htop
$ apt-get install vim
$ apt-get install screen
```
These packages can improve your effective.
***htop:*** [htop][htop] command like [top][top] command, both of these are interactive process viewer for Unix systems. Its help you manage process and monitor resource easy. But [htop][htop] is more convenient than [top][top].
> - In 'htop' you can scroll the list vertically and horizontally to see all processes and complete command lines.
- In 'top' you are subject to a delay for each unassigned key you press (especially annoying when multi-key escape sequences are triggered by accident).
- 'htop' starts faster ('top' seems to collect data for a while before displaying anything).
- In 'htop' you don't need to type the process number to kill a process, in 'top' you do.
- In 'htop' you don't need to type the process number or the priority value to renice a process, in 'top' you do.
- 'htop' supports mouse operation, 'top' doesn't
- 'top' is older, hence, more used and tested.
From http://hisham.hm/htop/index.php?page=comparison

***vim:*** [vim][vim] is almost a proper superset of [vi][vi], everything that is in vi is available in vim.

***screen:***
> 1. use multiple shell window from a single SSH session
2. Keep a shell active even through network disruptions
3. Disconnect and re-connect to a shell sessions from multiple locations
4. Run a long running process without maintaining an active shell session

**Usage**
screen uses the command `ctrl-a` that's the control key and a lowercase "a" as a signal to send commands to screen instead of the shell.
*start*: `screen`
*creating*: `Ctrl-a c`
*switch*: `Ctrl-a n` for the next window or `Ctrl-a p` for the previous window.
*detach*: `Ctrl-a d`
*reattach*: `Ctrl-a r`
*logging*: `Ctrl-a h`
*lock*: `Ctrl-a x`
*stop*: `Ctrl-a k`

## Group & User
``` bash
$ groupadd
$ groupdel
$ useradd
$ passwd
$ usermod
$ userdel
```

### useradd
user add will auto create group that the name same as user
``` bash
$ useradd chang # craete user "chang"
$ useradd -d /data/projects chang # create user "chang" into /data/projects
$ useradd -u 999 -g 998 chang # create user with 999 for user id and 998 for group id
$ useradd -G admings,webadmin,developers chang #
$ useradd -M chang # add user "chang" without home directory
$ useradd -e 2017-01-01 chang # add user "chang" expire at 2017-01-02
$ chage -l chang # show/change user or password expire date
$ useradd -e 2017-01-01 -f 45 chang # password expire after 2017-01-01
$ useradd -c "Yuchang Wu" chang # add user with "Yuchang Wu" infomation
$ tail -1 /etc/passwd # print last 1 line of the file "/etc/passwd"
```

### the meaning of files that user and group saved
- /etc/passwd – User account information.
- /etc/shadow – Secure account information.
- /etc/group – Group account information.
- /etc/gshadow – Secure group account information.
- /etc/login.defs – Shadow password suite configuration..

/etc/passwd format:
username:password:uid:gid:userInfo:homeDirectory:shell

/etc/group format:
groupname:password:gidd:groupMembers

### usermod
for more detail, to below to see
```bash
$ usermod --help
```
``` bash
$ usermod -G www chang # add www group to chang, Be careful, it will remove all existing groups that user belongs, so always add the '-a' with '-G' to append new groups
$ usermod -G www -a chang
```

### chown
change file owner
``` bash
$ chown -hR www:www /home/www # chang the owner of /homw/www to www user and www group
```

## Nginx Install
As you know, you can install an application with few different methods, below we introduce three common methods to install Nginx.
### Install from Debian repository
``` bash
$ apt-get install nginx
```
you can verify the installation through:
``` bash
$ nginx -v
```
and you can verify which module you are install through:
``` bash
$ nginx -V  # Capital
```
and you will get the default configuration like:
``` bash
nginx version: nginx/1.6.2
TLS SNI support enabled
configure arguments: --with-cc-opt='-g -O2 -fstack-protector-strong -Wformat -Werror=format-security -D_FORTIFY_SOURCE=2' --with-ld-opt=-Wl,-z,relro --prefix=/usr/share/nginx --conf-path=/etc/nginx/nginx.conf --http-log-path=/var/log/nginx/access.log --error-log-path=/var/log/nginx/error.log --lock-path=/var/lock/nginx.lock --pid-path=/run/nginx.pid --http-client-body-temp-path=/var/lib/nginx/body --http-fastcgi-temp-path=/var/lib/nginx/fastcgi --http-proxy-temp-path=/var/lib/nginx/proxy --http-scgi-temp-path=/var/lib/nginx/scgi --http-uwsgi-temp-path=/var/lib/nginx/uwsgi --with-debug --with-pcre-jit --with-ipv6 --with-http_ssl_module --with-http_stub_status_module --with-http_realip_module --with-http_auth_request_module --with-http_addition_module --with-http_dav_module --with-http_geoip_module --with-http_gzip_static_module --with-http_image_filter_module --with-http_spdy_module --with-http_sub_module --with-http_xslt_module --with-mail --with-mail_ssl_module --add-module=/build/nginx-AZ8ONw/nginx-1.6.2/debian/modules/nginx-auth-pam --add-module=/build/nginx-AZ8ONw/nginx-1.6.2/debian/modules/nginx-dav-ext-module --add-module=/build/nginx-AZ8ONw/nginx-1.6.2/debian/modules/nginx-echo --add-module=/build/nginx-AZ8ONw/nginx-1.6.2/debian/modules/nginx-upstream-fair --add-module=/build/nginx-AZ8ONw/nginx-1.6.2/debian/modules/ngx_http_substitutions_filter_module
```

> Note: this type can not get the latest version of the NGINX.

### Install from Nginx repository
1. Download the key used to sign NGINX packages and repository to the apt program keyring and add it:
``` bash
$ wget http://nginx.org/keys/nginx_signing.key
$ apt-key add nginx_signing.key
```
2. Add the "sources" from which NGINX Open Source packages can be obtained: open the */etc/apt/sources.list* file in any text editor, for example, vim:
``` bash
$ vim /etc/apt/sources.list
```
3. Append the lines to the file:
``` bash
deb http://nginx.org/packages/mainline/debian/ codename nginx
deb-src http://nginx.org/packages/mainline/debian/ codename nginx
```
 where:
 codename is a codename of a Debian release:
<table> <thead> <tr> <th>Version</th> <th>Codename</th> <th>Supported Platforms</th> </tr> </thead> <tbody> <tr> <td>7.x</td> <td>wheezy</td> <td>x86_64,i386</td> </tr> <tr> <td>8.x</td> <td>jessie</td> <td>x86_64,i386</td> </tr> </tbody> </table>
4. Install
``` bash
$ apt-get remove nginx-common
$ apt-get update
$ apt-get install nginx
```
5. list the default module that nginx installed
``` bash
$ nginx -V
```
 ``` bash
nginx version: nginx/1.11.6
built by gcc 4.9.2 (Debian 4.9.2-10)
built with OpenSSL 1.0.1t  3 May 2016
TLS SNI support enabled
configure arguments: --prefix=/etc/nginx --sbin-path=/usr/sbin/nginx --modules-path=/usr/lib/nginx/modules --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --pid-path=/var/run/nginx.pid --lock-path=/var/run/nginx.lock --http-client-body-temp-path=/var/cache/nginx/client_temp --http-proxy-temp-path=/var/cache/nginx/proxy_temp --http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp --http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp --http-scgi-temp-path=/var/cache/nginx/scgi_temp --user=nginx --group=nginx --with-compat --with-file-aio --with-threads --with-http_addition_module --with-http_auth_request_module --with-http_dav_module --with-http_flv_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_mp4_module --with-http_random_index_module --with-http_realip_module --with-http_secure_link_module --with-http_slice_module --with-http_ssl_module --with-http_stub_status_module --with-http_sub_module --with-http_v2_module --with-mail --with-mail_ssl_module --with-stream --with-stream_realip_module --with-stream_ssl_module --with-stream_ssl_preread_module --with-cc-opt='-g -O2 -fstack-protector-strong -Wformat -Werror=format-security -Wp,-D_FORTIFY_SOURCE=2' --with-ld-opt='-Wl,-z,relro -Wl,-z,now -Wl,--as-needed'

```
You can see the latest version of the NGINX installed to your system.
## Tip
``` bash
$ man order # to see detail of the order
$ apt-cache search package # search package
$ cat /etc/passwd # show all users
$ getent passwd # show all users
$ getent group # show all groups
$ getent group | grep username
$ id username # show username information
$ groups username # show which groups does username have
```

> Ref:
*screen* https://www.rackaid.com/blog/linux-screen-tutorial-and-how-to/#starting
*useradd* http://www.tecmint.com/add-users-in-linux/
*usermod* http://www.tecmint.com/usermod-command-examples/
*permission* http://www.tecmint.com/manage-users-and-groups-in-linux/
*nginx* https://www.nginx.com/resources/admin-guide/installing-nginx-open-source/

[htop]: https://hisham.hm/htop/index.php
[top]: https://linux.die.net/man/1/top
[vim]: http://www.vim.org/
[vi]: http://ex-vi.sourceforge.net/
