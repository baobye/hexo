---
title: jenkins
date: 2018-11-19 20:25:24
tags:
---


# 1、tomcat配置 
   
* web.xml 修改
``` bash
  <role rolename="manager"/>　
  <role rolename="manager-gui"/>　
  <role rolename="admin"/>　
  <role rolename="admin-gui"/>　
  <role rolename="manager-script"/>
  <user username="xxx" password="xxx" roles="admin-gui,admin,manager-gui,manager,manager-script"/>

```

* server.xml 修改
<!-- more -->

``` bash

```

# 2、Jenkins 配置  高级  使用自定义的工作空间

``` bash
    项目名 deploy_xxx_web
    丢弃旧的构建

    git 地址
    clean
    install
    -Dmaven.test.skip=true

    decision_api/prediction_web/target/prediction_web.war
	
	decision_api/prediction_web/pom.xml
	
    http://172.24.8.125:8018

```


3、前端部署

``` bash
export LD_LIBRARY_PATH=/usr/local/bin/
export PATH=/opt/node/bin
export PATH=$PATH:/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
cd /opt/jenkins/decision_webapp
yarn config set registry https://registry.npm.taobao.org
yarn install
yarn build:dev
scp -r dist/ 172.24.2.194:/opt/www/webapp/tmp/v2_tmp/
echo 'Copy to dist server end...'
ssh 172.24.2.194 'sh /opt/www/webapp/sh/deploy_v2.sh'

```
* sh脚本

``` bash
#!/bin/bash

cd /opt/www/

mv ./webapp/dist_index/v2_index/index.html ./assets/v2
tar -czPf /opt/www/webapp/bak/v2_bak/webapp_v2.$(date '+%Y%m%d%H%M%S' ).bak.tar.gz  ./assets/v2
rm -rf ./assets/v2/*
cd /opt/www/webapp/tmp/v2_tmp
mv dist/index.html  ../../dist_index/v2_index
mv dist/* ../../../assets/v2
rm -rf /opt/www/webapp/tmp/v2_tmp/dist
echo 'Deploy finished.'
```
