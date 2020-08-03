# nginx部署及配置
nginx我们采用的是docker部署，可以参考github项目[nginx-template](https://github.com/flashtd1/nginx-template)

Dockerfile中定义了容器构建中需要做的事情，主要是把同目录的nginx.conf复制到容器中

在jenkins的构建中，我们将编写nginx.conf的步骤放到了参数中，您可以在jenkins的构建参数中将真实的nginx.conf内容填写在WriteFile参数,以下为内容参考

**注意**
nginx配置中只编写服务类的和ecs本机安装的其他服务的配置，部署在oss上的静态网站不需要在这里配置，可以直接在oss中绑定域名

```bash
cat>nginx.conf<<EOF
worker_processes  1;

events {
    worker_connections  1024;
}

http {
    include mime.types;
    default_type    application/octet-stream;

    client_max_body_size 30M;

    sendfile    on;
    keepalive_timeout   65;

    gzip    on;

    server {
        listen 80;
        # 这里是您使用的域名
        server_name api.yourdomain.com;
        
        location / {
            proxy_set_header X-Real-IP \$remote_addr;
            proxy_set_header X-Forwarded-For \$proxy_add_x_forwarded_for;
            proxy_set_header Host \$http_host;
            proxy_set_header X-NginX-Proxy true;
            # 下面写内网地址和对应构建任务的out_ip的端口号，ip写您的ecs的内网地址可以避免频繁去修改安全组
            proxy_pass http://120.0.0.1:8081;
            proxy_redirect off;
        }
    }

    server {
        listen 80;
        server_name build.yourdomain.com;
        
        location / {
            proxy_set_header X-Real-IP \$remote_addr;
            proxy_set_header X-Forwarded-For \$proxy_add_x_forwarded_for;
            proxy_set_header Host \$http_host;
            proxy_set_header X-NginX-Proxy true;
            # 下面写内网地址和对应构建任务的端口号，ip写您的ecs的内网地址可以避免频繁去修改安全组
            proxy_pass http://120.0.0.1:8080;
            proxy_redirect off;
        }
    }

    server {
        listen 80 default;
        location / {
            root html;
            index index.html index.htm;
        }
    }
}
EOF
```