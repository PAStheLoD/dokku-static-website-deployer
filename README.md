# dokku-static-website-deployer

Build modern fancy SPA web3.0 apps with a Dockerfile ( https://hub.docker.com/r/pasthelod/nodejs/ ), and then get the resulting /dist from the container, and host it via the nginx docker already uses.

Recommended nginx.conf.sigil:

```
server {
  listen      [::]:80;
  listen      80;
  server_name {{ .NOSSL_SERVER_NAME }};

  access_log  /var/log/nginx/{{ .APP }}-access.log;
  error_log   /var/log/nginx/{{ .APP }}-error.log;

  return 301 https://$http_host:443$request_uri;
  
}

server {
  listen      [::]:443 ssl spdy;
  listen      443 ssl spdy;
  server_name {{ .NOSSL_SERVER_NAME }};

  access_log  /var/log/nginx/{{ .APP }}-access.log;
  error_log   /var/log/nginx/{{ .APP }}-error.log;

  ssl_certificate     {{ .APP_SSL_PATH }}/server.crt;
  ssl_certificate_key {{ .APP_SSL_PATH }}/server.key;
  ssl_protocols       TLSv1 TLSv1.1 TLSv1.2;

  gzip on;
  gzip_min_length  1100;
  gzip_buffers  4 32k;
  gzip_types    text/css text/javascript text/xml text/plain text/x-component application/javascript application/x-javascript application/json application/xml  application/rss+xml font/truetype application/x-font-ttf font/opentype application/vnd.ms-fontobject image/svg+xml;
  gzip_vary on;
  gzip_comp_level  6;

  location    / {
	root {{ .DOKKU_ROOT }}/{{ .APP }}/static-website-data/;
  }
  include {{ .DOKKU_ROOT }}/{{ .APP }}/nginx.conf.d/*.conf;
}
```

## requirements

- dokku 0.4.0+
- docker 1.8.x

## installation

```shell
# on 0.4.x
dokku plugin:install https://github.com/PAStheLoD/dokku-static-website-deployer
```

## hooks

This plugin provides hooks:

* `post-build-buildpack`: copy files into app/static-website-data directory
* `post-build-dockerfile`: copy files into app/static-website-data directory
