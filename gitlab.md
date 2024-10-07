# Настройка своего Gitlab

gitlab_url: gitlab.mycloud.ru
registry_url: registry.gitlab.mycloud.ru

### Базовое поднятие Gitlab

1. apt-get update && apt-get upgrade
2. apt-get install ca-certificates curl openssh-server
3. mkdir tmp
4. cd /tmp
5. curl -LO https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.deb.sh
6. bash /tmp/script.deb.sh
7. apt-get install gitlab-ce
8. gitlab-ctl reconfigure
9. iptables -A INPUT -p tcp --dport 80 -j ACCEPT
10. external_url ‘http://gitlab.mycloud.ru’
11. gitlab-ctl reconfigure


### Настройка Gitlab Container Registry
1. nano /etc/gitlab/gitlab.rb
2. nginx['custom_nginx_config'] = "include /etc/nginx/conf.d/*.conf;"
3. mkdir -p /etc/nginx/conf.d &&  mkdir /var/log/nginx && mkdir -p /usr/share/nginx/html
4. gitlab-ctl reconfigure
5. certbot certonly -a webroot -w /usr/share/nginx/html -d registry.gitlab.mycloud.ru
6. nano /etc/nginx/conf.d/nginx.conf
server {
    listen       443 ssl;
    server_name  registry.gitlab.mycloud.ru;

    ssl_certificate /etc/letsencrypt/live/registry.gitlab.mycloud.ru/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/registry.gitlab.mycloud.ru/privkey.pem;

    ssl_protocols TLSv1.1 TLSv1.2;
    ssl_ciphers 'EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH';
    ssl_prefer_server_ciphers on;
    ssl_session_cache shared:SSL:10m;

    root /usr/share/nginx/html;

    location / {
        proxy_pass http://127.0.0.1:8090;
        proxy_read_timeout      300;
        proxy_connect_timeout   300;
        proxy_redirect          off;
        proxy_set_header        X-Forwarded-Proto https;
        proxy_set_header        Host              $http_host;
        proxy_set_header        X-Real-IP         $remote_addr;
        proxy_set_header        X-Forwarded-For   $proxy_add_x_forwarded_for;
        proxy_set_header        X-Forwarded-Ssl   on;
    }

    location ^~ /.well-known {
        default_type 'text/plain';
        allow all;
    }

    error_log   /var/log/nginx/rregistry_gitlab_mycloud_ru_error.log error;
    access_log  /var/log/nginx/registry_gitlab_mycloud_ru_access.log;
}
7.  nano /etc/gitlab/gitlab.rb
registry_external_url 'https://registry.gitlab.mycloud.ru'
gitlab_rails['registry_enabled'] = true
registry['enable'] = true
registry_nginx['enable'] = true
registry_nginx['proxy_set_headers'] = {
  "Host" => "$http_host",
  "X-Real-IP" => "$remote_addr",
  "X-Forwarded-For" => "$proxy_add_x_forwarded_for",
  "X-Forwarded-Proto" => "https",
  "X-Forwarded-Ssl" => "on"
}
registry_nginx['listen_port'] = 8090
registry_nginx['listen_https'] = false
8. gitlab-ctl reconfigure





