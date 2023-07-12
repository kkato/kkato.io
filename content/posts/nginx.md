---
title: "Nginx"
date: 2023-07-12T13:11:42+09:00
draft: true
---

# nginx

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-configmap
  labels:
    app: nginx
data:
  nginx.conf: |2

    user  nginx;
    worker_processes  1;
    error_log  /var/log/nginx/error.log warn;
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
        log_format  postdata escape=none '{"time": "$time_iso8601",'
                                         '"request_body": "$request_body" }';
        access_log  /var/log/nginx/access.log  main;

        server {
            listen   8080;
            location / {
                root /usr/share/nginx/html;
            }
            location /stub_status {
                stub_status;
            }
            location /json {
                root /tmp;
                index json;
                access_log /dev/stdout postdata;
                proxy_pass http://127.0.0.1:8080/post_gif;
            }
            location /post_gif {
                 access_log off;
                 empty_gif;
                 break;
            }
            error_page  405 =200 $uri;
        }
        sendfile        on;
        keepalive_timeout  65;
        include /etc/nginx/conf.d/*.conf;
    }
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-jsonpage-configmap
  labels:
    app: nginx
data:
  json: |
    Nginx_Page
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-svc
  labels:
    app: nginx
spec:
  selector:
    app: nginx
  ports:
  - name: nginx-http
    protocol: TCP
    port: 8080
    targetPort: 8080
  - name: nginx-exporter
    protocol: TCP
    port: 9113
    targetPort: 9113
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deploy
  labels:
    app: nginx
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 1
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 8080
        volumeMounts:
        - name: nginx-conf
          mountPath: /etc/nginx/nginx.conf
          subPath: nginx.conf
        - name: nginx-jsonpage-configmap
          mountPath: /tmp
      - name: nginx-exporter
        image: nginx/nginx-prometheus-exporter:0.11.0
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 9113
        args:
          - -nginx.scrape-uri=http://localhost:8080/stub_status
      nodeSelector:
        node-role.kubernetes.io/infra: ""
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 1
            podAffinityTerm:
              labelSelector:
                matchExpressions:
                - key: app
                  operator: In
                  values:
                  - nginx
              topologyKey: "kubernetes.io/hostname"
      volumes:
      - name: nginx-conf
        configMap:
          name: nginx-configmap
          items:
          - key: nginx.conf
            path: nginx.conf
      - name: nginx-jsonpage-configmap
        configMap:
          name: nginx-jsonpage-configmap
          items:
          - key: json
            path: json
```
