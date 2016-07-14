# 基于docker,nginx,let's encrypt 的https服务器配置指南

https服务器，可以采用基于docker容器的方式布署，使用nginx web server 做的反向代理，后端的业务系统都通过nginx来对外访问。


https证书使用[let's encrypt](https://letsencrypt.org/)的免费证书方案。
服务端使用两个开源镜像，需要提前docker pull 好到电脑上，另外，需要安装[docker-compose](http://www.docker.com/products/docker-compose)命令。
- [docker-nginx-letsencrypt](https://github.com/bringnow/docker-nginx-letsencrypt) :配置好letsencrypt https的nginx 镜像。
- [docker-letsencrypt-manager](https://github.com/bringnow/docker-letsencrypt-manager) :letsencrypt 证书管理的镜像，可以实现证书的申请，撤销，创建定时任务来更新证书。
  可以按[docker-nginx-letsencrypt](https://github.com/bringnow/docker-nginx-letsencrypt)和[docker-letsencrypt-manager](https://github.com/bringnow/docker-letsencrypt-manager) 中的说明来一步步操作即可。

## 需要注意的问题：
- docker-nginx-letsencrypt 在第一次启动时，会生成DH Parameter,以提高https的安全系数。这是一个很漫长的过程，可以提前生成这个文件：`openssl dhparam -out /etc/nginx/dhparam/dhparam.pem 4096`
- `docker-letsencrypt-manager`在申请证书时，需要nginx需要启动起来，但这个docker-nginx-letsencrypt中配置的证书还没有生成，这时直接启动的话，会报错，这时可以先用一个自定义的nginx.conf,这个nginx.conf只配置http,不配置https，等证书申请成功后，再还原原回来，只用于申请证书最小的配置：


    events {
    	worker_connections  1024;
    }
    http {
        server {
            listen 80;
       
            charset utf-8;
       
            location /.well-known/acme-challenge {
                alias /var/acme-webroot/.well-known/acme-challenge;
                location ~ /.well-known/acme-challenge/(.*) {
                    add_header Content-Type application/jose+json;
                }
            }
        }
    }
- docker-compose 后台运行：`docker-compose up -d`
- let's encrypt 的证书有效期为90天，在失效之前，需要向letsencrypt续期。docker-letsencrypt-manager已提供了相应的定时任务，执行下对应的命令即可。

## 相关资料

- [let'ss encrypt中docker使用](https://letsencrypt.readthedocs.io/en/latest/using.html#running-with-docker)
- [在Docker容器环境中用Let's Encrypt部署HTTPS](http://www.ituring.com.cn/article/217692)


