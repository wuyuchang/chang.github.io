---
title: config ThinkPHP support memcache in php5
date: 2017-01-03 17:23:37
tags: [Linux, PHP, Memcache]
---

This guide is base on Ubuntu & PHP5, so you have to install PHP5 in Ubunut first, but debian is okay.

## Install Memcached
``` shell
$ sudo apt-get install memcached
$ service memcached start
```

then execute `ps aux | grep memcached` to check it is running.
``` shell
$ ps aux | grep memcached
```

## Install Memcache extension for PHP
Let's go to https://pecl.php.net/package-stats.php.
1. download memcache package from https://pecl.php.net/package-stats.php to your home directory.
2. extract it.
3. run phpize to gererate configure, make file.
4. configure
5. make
6. make install

``` shell
$ wget https://pecl.php.net/get/memcache-2.2.7.tgz
$ tar zxf memcache-2.2.7.tgz
$ phpize
$ ./configure
$ sudo make
$ sudo make install
```

## set php.ini
append extension=/path/to/memcache.so
`php.ini`
``` config
extension=/path/to/memcache.so
```

## config ThinkPHP
Go into Core/conf/conf.php

``` php
'DATA_CACHE_TYPE' => 'Memcache',
'MEMCACHE_HOST' => 'localhost',
'MEMCACHE_PORT' => '11211'
```

## restart php-fpm
``` shell
$ ps aux | grep memcached
$ kill pid
```
