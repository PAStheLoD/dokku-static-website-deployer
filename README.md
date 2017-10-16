# dokku-static-website-deployer

Build modern fancy SPA web3.0 apps with a Dockerfile ( https://hub.docker.com/r/pasthelod/nodejs/ ), and then get the resulting /dist from the container, and host it via the nginx docker already uses.

Recommended nginx.conf.sigil:

```
{{ range $port_map := .PROXY_PORT_MAP | split " " }}
    {{ $port_map_list := $port_map | split ":" }}
    {{ $scheme := index $port_map_list 0 }}
    {{ $listen_port := index $port_map_list 1 }}
    {{ $upstream_port := index $port_map_list 2 }}


    {{ if eq $scheme "http" }}

        server {
            listen      [::]:{{ $listen_port }};
            listen      {{ $listen_port }};
            {{ if $.NOSSL_SERVER_NAME }}
                server_name {{ $.NOSSL_SERVER_NAME }};
            {{ end }}
            access_log  /var/log/nginx/{{ $.APP }}-access.log;
            error_log   /var/log/nginx/{{ $.APP }}-error.log;
            {{ if (and (eq $listen_port "80") ($.SSL_INUSE)) }}
                return 301 https://$host:{{ $.NGINX_SSL_PORT }}$request_uri;
            {{ else }}
                location    / {
                    gzip on;
                    gzip_min_length  1100;
                    gzip_buffers  4 32k;
                    gzip_types    text/css text/javascript text/xml text/plain text/x-component application/javascript application/x-javascript application/json application/xml  application/rss+xml font/truetype application/x-font-ttf font/opentype application/vnd.ms-fontobject image/svg+xml;
                    gzip_vary on;
                    gzip_comp_level  6;

                    proxy_pass  http://{{ $.APP }}-{{ $upstream_port }};
                    proxy_http_version 1.1;
                    proxy_set_header Upgrade $http_upgrade;
                    proxy_set_header Connection "upgrade";
                    proxy_set_header Host $http_host;
                    proxy_set_header X-Forwarded-Proto $scheme;
                    proxy_set_header X-Forwarded-For $remote_addr;
                    proxy_set_header X-Forwarded-Port $server_port;
                    proxy_set_header X-Request-Start $msec;
                }
                include {{ $.DOKKU_ROOT }}/{{ $.APP }}/nginx.conf.d/*.conf;

                error_page 400 401 402 403 405 406 407 408 409 410 411 412 413 414 415 416 417 418 420 422 423 424 426 428 429 431 444 449 450 451 /400-error.html;
                location /400-error.html {
                  root {{ $.DOKKU_LIB_ROOT }}/data/nginx-vhosts/dokku-errors;
                  internal;
                }

                error_page 404 /404-error.html;
                location /404-error.html {
                  root {{ $.DOKKU_LIB_ROOT }}/data/nginx-vhosts/dokku-errors;
                  internal;
                }

                error_page 500 501 502 503 504 505 506 507 508 509 510 511 /500-error.html;
                location /500-error.html {
                  root {{ $.DOKKU_LIB_ROOT }}/data/nginx-vhosts/dokku-errors;
                  internal;
                }
            {{ end }}
        }

    {{ else if eq $scheme "https" }}

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
            server_name {{ .NOSSL_SERVER_NAME }}  ;

            access_log  /var/log/nginx/{{ .APP }}-access.log;
            error_log   /var/log/nginx/{{ .APP }}-error.log;

            ssl_certificate     {{ .APP_SSL_PATH }}/server.crt;
            ssl_certificate_key {{ .APP_SSL_PATH }}/server.key;
            ssl_protocols       TLSv1 TLSv1.1 TLSv1.2;

            #  plenuum plenuum

            gzip on;
            gzip_min_length  1100;
            gzip_buffers  4 32k;
            gzip_types    text/css text/javascript text/xml text/plain text/x-component application/javascript application/x-javascript application/json application/xml application/rss+xml font/truetype application/x-font-ttf font/opentype application/vnd.ms-fontobject image/svg+xml;
            gzip_vary on;
            gzip_comp_level  6;

            location / {
                 root {{ .DOKKU_ROOT }}/{{ .APP }}/static-website-data/;
            }
            include {{ .DOKKU_ROOT }}/{{ .APP }}/nginx.conf.d/*.conf;
        }

    {{ end }}

{{ range $upstream_port := $.PROXY_UPSTREAM_PORTS | split " " }}
    upstream {{ $.APP }}-{{ $upstream_port }} {
        {{ range $listeners := $.DOKKU_APP_LISTENERS | split " " }}
            {{ $listener_list := $listeners | split ":" }}
            {{ $listener_ip := index $listener_list 0 }}
            {{ $listener_port := index $listener_list 1 }}
              server {{ $listener_ip }}:{{ $listener_port }};
        {{ end }}
    }
{{ end }}
```

Recommended to skip deploy:

```
dokku config:set <app> DOKKU_SKIP_DEPLOY=true
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
