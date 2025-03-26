# Install Lib to use
```
apt install libtool-bin build-essential libfuse-dev libcurl4-openssl-dev libxml2-dev automake libtool pkg-config autoconf
apt-get install autoconf automake libtool cmake mime-support unzip
apt install zlib1g zlib1g-dev
apt install libjwt-dev
```

# install jansson https://github.com/benmcollins/libjwt/tree/v1.12.0
```
unzip jansson-2.12.zip
```

# goto jansson-2.12.zip
```
cmake .
make install
```

# install libjwt https://github.com/benmcollins/libjwt/tree/v1.17.2
```
unzip libjwt-1.17.2.zip
apt install libssl-dev
cmake .
make install
```

# load ngx module https://github.com/TeslaGov/ngx-http-auth-jwt-module/tree/2.2.0
```
unzip ngx-http-auth-jwt-module-2.2.0.zip
```

# edit file ngx_http_auth_jwt_module.c
```
# Remove
  #ifndef NGX_LINKED_LIST_COOKIES
  if (ngx_http_parse_multi_header_lines(&r->headers_in.cookies, &jwt_location, &jwtCookieVal) != NGX_DECLINED)
  {
    has_cookie = true;
  }
  #else
# save file
```
# get ngix wget http://nginx.org/download/nginx-1.27.2.tar.gz
```
wget http://nginx.org/download/nginx-1.27.2.tar.gz
tar -xvzf nginx-1.27.2.tar.gz
```

# run configure
# path of --add-module=/home/adm_uatapigw/ngx-http-auth-jwt-module-2.2.0 \
```
./configure \
    --add-module=/home/adm_uatapigw/ngx-http-auth-jwt-module-2.2.0 \
    --prefix=/usr/share/nginx \
    --sbin-path=/usr/sbin/nginx \
    --modules-path=/usr/lib64/nginx/modules \
    --conf-path=/etc/nginx/nginx.conf \
    --error-log-path=/var/log/nginx/error.log \
    --http-log-path=/var/log/nginx/access.log \
    --http-client-body-temp-path=/var/lib/nginx/tmp/client_body \
    --http-proxy-temp-path=/var/lib/nginx/tmp/proxy \
    --http-fastcgi-temp-path=/var/lib/nginx/tmp/fastcgi \
    --http-uwsgi-temp-path=/var/lib/nginx/tmp/uwsgi \
    --http-scgi-temp-path=/var/lib/nginx/tmp/scgi \
    --pid-path=/run/nginx.pid \
    --lock-path=/run/lock/subsys/nginx \
    --user=nginx \
    --group=nginx \
    --with-file-aio \
    --with-ipv6 \
    --with-http_ssl_module \
    --with-http_v2_module \
    --with-http_realip_module \
    --with-http_addition_module
```
#Build nginx
```
make install .
```
#Install Nginx
```
apt install nginx
systemctl status nginx
```
#Config Nginx
## edit nginx.conf
```
user  nginx;
worker_processes  auto;

error_log  /var/log/nginx/error.log notice;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    include /etc/nginx/conf.d/*.conf;
    include /etc/nginx/sites-enabled/*.conf;
}
```


```
server { 
	listen 443 ssl default_server;
	#server_name uat-apigateway.kingpower.com;
	ssl_certificate /opt/certs/certificate.crt;
	ssl_certificate_key /opt/certs/private.key;
	location /admin-api/ {
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_set_header X-Forwarded-Port 443;

            proxy_pass http://localhost:8001;
            rewrite ^/admin-api(/.*)? $1 break;
        }
	root /var/www/protected;
        index index.html;
        location /api {
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_set_header X-Forwarded-Port 443;

            proxy_pass http://localhost:8000;
            #rewrite ^/api(/.*)? $1 break;
        }

        location /manager/ {
            #auth_jwt_key "5cbd0fc3b24911163b2c076f11c71711ae3f71e5e83a761eab9e9101d900ac62";
            auth_jwt_enabled on;
            auth_jwt_algorithm RS256; # or RS256
            auth_jwt_location COOKIE=auth-token;
            auth_jwt_use_keyfile on;
            auth_jwt_keyfile_path "/etc/nginx/keys/public.pem";
            #auth_jwt_validate_email off;  # or off
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            #proxy_set_header X-Forwarded-Port 443;

            proxy_pass http://localhost:8002;
            # Your protected content
            #root /var/www/protected;
            #index index.html;
        }
	error_page 401 /unauthorized.html;
}
```
