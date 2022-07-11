---
title: KylinV10 桌面版SVN搭建
date: 2022-07-11 9:29:52
categories:
- 杂记
tags:
---

# KylinV10 桌面版SVN搭建

## 一、服务端配置

### 1.1 安装SVN

```shell
sudo apt-get install subversion
```

### 1.2 新建仓库文件夹

```shell
# 创建项目仓库文件夹
mkdir /data/svn/project

# 修改文件夹权限
chmod -R 777 /data/svn/project
```

### 1.3 创建版本库

```shell
svnadmin create /data/svn/project
```

完成后会在文件夹中生成一系列文件

```shell
greatwall@greatwall-GW-001M1A-FTF:/data/svn/project$ ls
conf  db  format  hooks  locks  README.txt
```

修改db权限

```shell
chmod -R 777 db
```

### 1.4 设置访问权限

```she
vim /data/svn/project/conf/svnserve.conf
```

做如下修改

```shell
greatwall@greatwall-GW-001M1A-FTF:/data/svn/project/conf$ cat svnserve.conf
### This file controls the configuration of the svnserve daemon, if you
### use it to allow access to this repository.  (If you only allow
### access through http: and/or file: URLs, then this file is
### irrelevant.)

### Visit http://subversion.apache.org/ for more information.

[general]
### The anon-access and auth-access options control access to the
### repository for unauthenticated (a.k.a. anonymous) users and
### authenticated users, respectively.
### Valid values are "write", "read", and "none".
### Setting the value to "none" prohibits both reading and writing;
### "read" allows read-only access, and "write" allows complete
### read/write access to the repository.
### The sample settings below are the defaults and specify that anonymous
### users have read-only access to the repository, while authenticated
### users have read and write access to the repository.
anon-access = none	# 非认证用户不让写
auth-access = write
### The password-db option controls the location of the password
### database file.  Unless you specify a path starting with a /,
### the file's location is relative to the directory containing
### this configuration file.
### If SASL is enabled (see below), this file will NOT be used.
### Uncomment the line below to use the default password file.
password-db = passwd
### The authz-db option controls the location of the authorization
### rules for path-based access control.  Unless you specify a path
### starting with a /, the file's location is relative to the
### directory containing this file.  The specified path may be a
### repository relative URL (^/) or an absolute file:// URL to a text
### file in a Subversion repository.  If you don't specify an authz-db,
### no path-based access control is done.
### Uncomment the line below to use the default authorization file.
authz-db = authz
### The groups-db option controls the location of the groups file.
### Unless you specify a path starting with a /, the file's location is
### relative to the directory containing this file.  The specified path
### may be a repository relative URL (^/) or an absolute file:// URL to a
### text file in a Subversion repository.
# groups-db = groups
### This option specifies the authentication realm of the repository.
### If two repositories have the same authentication realm, they should
### have the same password database, and vice versa.  The default realm
### is repository's uuid.
realm = My First Repository
### The force-username-case option causes svnserve to case-normalize
### usernames before comparing them against the authorization rules in the
### authz-db file configured above.  Valid values are "upper" (to upper-
### case the usernames), "lower" (to lowercase the usernames), and
### "none" (to compare usernames as-is without case conversion, which
### is the default behavior).
# force-username-case = none
### The hooks-env options specifies a path to the hook script environment
### configuration file. This option overrides the per-repository default
### and can be used to configure the hook script environment for multiple
### repositories in a single file, if an absolute path is specified.
### Unless you specify an absolute path, the file's location is relative
### to the directory containing this file.
# hooks-env = hooks-env

[sasl]
### This option specifies whether you want to use the Cyrus SASL
### library for authentication. Default is false.
### This section will be ignored if svnserve is not built with Cyrus
### SASL support; to check, run 'svnserve --version' and look for a line
### reading 'Cyrus SASL authentication is available.'
# use-sasl = true
### These options specify the desired strength of the security layer
### that you want SASL to provide. 0 means no encryption, 1 means
### integrity-checking only, values larger than 1 are correlated
### to the effective key length for encryption (e.g. 128 means 128-bit
### encryption). The values below are the defaults.
# min-encryption = 0
# max-encryption = 256
```

### 1.5 添加访问用户

修改passwd文件，添加访问用户

```shell
greatwall@greatwall-GW-001M1A-FTF:/data/svn/project/conf$ cat passwd
### This file is an example password file for svnserve.
### Its format is similar to that of svnserve.conf. As shown in the
### example below it contains one section labelled [users].
### The name and password for each user follow, one account per line.

[users]
# harry = harryssecret
# sally = sallyssecret
admin=mysecret
gs=mysecret
```

### 1.6 设置用户权限

修改authz文件，设置用户权限

```shell
greatwall@greatwall-GW-001M1A-FTF:/data/svn/project/conf$ cat authz
### This file is an example authorization file for svnserve.
### Its format is identical to that of mod_authz_svn authorization
### files.
### As shown below each section defines authorizations for the path and
### (optional) repository specified by the section name.
### The authorizations follow. An authorization line can refer to:
###  - a single user,
###  - a group of users defined in a special [groups] section,
###  - an alias defined in a special [aliases] section,
###  - all authenticated users, using the '$authenticated' token,
###  - only anonymous users, using the '$anonymous' token,
###  - anyone, using the '*' wildcard.
###
### A match can be inverted by prefixing the rule with '~'. Rules can
### grant read ('r') access, read-write ('rw') access, or no access
### ('').

[aliases]
# joe = /C=XZ/ST=Dessert/L=Snake City/O=Snake Oil, Ltd./OU=Research Institute/CN=Joe Average

[groups]
# harry_and_sally = harry,sally
# harry_sally_and_joe = harry,sally,&joe

# [/foo/bar]
# harry = rw
# &joe = r
# * =

# [repository:/baz/fuz]
# @harry_and_sally = rw
# * = r

team1=admin,gs	# 创建用户组

[/]
@team1 = rw # 用户组权限设置为读写
* = r
```

### 1.7 启动服务

```shell
svnserve -d -r /data/svn
```

查看服务是否在运行

```shell
ps aux | grep svnserve
```

干掉服务

```shell
killall svnserve
```

## 二、客户端连接

svn://10.47.76.90/project

