---
title: "Alertmanagerのwebhookを確認する"
date: 2023-08-13T13:11:42+09:00
draft: false
tags: ["kubernetes", "prometheus"]
---

Kubernetes上でAlertmanagerがちゃんと通知できるか、どんな内容が通知されているのか確認してみようと思いました。
しかし、連携するためのSlackがなかったり、Emailを送信するにもメールサーバが必要だったりと、意外と気軽に試せないということがありました。

なので、今回はwebhookの機能を使って確認してみようと思います。

## webhookとは?
Alertmanagerのreceiverには以下が指定できます。
- Email
- [Opesgenie](https://www.atlassian.com/software/opsgenie)
- [PagerDuty](https://www.pagerduty.com/)
- [Pushover](https://pushover.net/)
- [Slack](https://slack.com/intl/ja-jp)
- [AWS SNS](https://aws.amazon.com/jp/sns/)
- [VictorOps](https://www.splunk.com/en_us/about-splunk/acquisitions/splunk-on-call.html)
- Webhook
- [Wechat](https://www.wechat.com/ja/)
- [Telegram](https://telegram.org/)
- [Webex](https://www.webex.com/ja/index.html)

Webhookとは特定のエンドポイントに対してHTTP POSTリクエストでアラートの情報を送信するというものです。
外部サービスではないので、利用者自身でエンドポイントを用意し、利用者自身で後続の処理を実装する必要があります。

例えば、以下のように設定します。
```
receivers:
- name: "nginx"
  webhook_configs:
  - url: 'http://nginx-svc.default:8080/'
```

## webhookの連携先としてnginxを使う
今回はwebhookの連携先としてnginxを使用します。

nginxを使って実現したいことは以下のとおりです。
- エンドポイントを用意する
- リクエスト内容を確認する

nginx.confの初期設定をベースにしていますが、そのままだとリクエスト内容を確認することができないので、設定を追加しました。
`log_format`で`$request_body`を指定し、`/`にアクセスした時に`$request_body`がログとして標準出力に出るように設定しています。

しかし、`$request_body`を有効化するには`proxy_pass`などの後続処理が必要になるみたいです。なので、`proxy_pass`で`/trash`というエンドポイントにリクエストを転送し、`/trash`で特に意味のない処理(1x1ピクセルのgifを返す)をしています。

>The variable’s value is made available in locations processed by the proxy_pass, fastcgi_pass, uwsgi_pass, and scgi_pass directives when the request body was read to a memory buffer.

http://nginx.org/en/docs/http/ngx_http_core_module.html#var_request_body

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

    # -------------------追加設定---------------------
    log_format  postdata escape=none $request_body;

    server {
        listen   8080;
        location / {
            access_log /dev/stdout postdata;
            proxy_pass http://127.0.0.1:8080/trash;
        }
        location /trash {
            access_log off;
            empty_gif;
            break;
        }
    }
    # ----------------------------------------------

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    include /etc/nginx/conf.d/*.conf;
}
```

上記の設定をConfigMapとして作成し、NginxのPodに対してConfigMapをマウントしてあげます。`nginx-svc`というServiceも作成します。
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-configmap
  labels:
    app: nginx
data:
  nginx.conf: |

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

        # -------------------追加設定---------------------
        log_format  postdata escape=none $request_body;

        server {
            listen   8080;
            location / {
                access_log /dev/stdout postdata;
                proxy_pass http://127.0.0.1:8080/trash;
            }
            location /trash {
                access_log off;
                empty_gif;
                break;
            }
        }
        # ----------------------------------------------
        
        sendfile        on;
        #tcp_nopush     on;

        keepalive_timeout  65;

        #gzip  on;

        include /etc/nginx/conf.d/*.conf;
    }
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
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
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
      volumes:
      - name: nginx-conf
        configMap:
          name: nginx-configmap
          items:
          - key: nginx.conf
            path: nginx.conf
```

先程例にあったように、`nginx-svc`をエンドポイントしてAlertmanagerのWebhookを設定します。
そうすると、以下のようにリクエストボディとして送られたアラートの情報を確認することができます。(本来は以下のようにフォーマットされないです。ブログ用にフォーマットしています。)
```sh
$ k logs nginx-6dcc55fd49-7kbpd 
...(略)...
{
   "receiver":"nginx",
   "status":"firing",
   "alerts":[
      {
         "status":"firing",
         "labels":{
            "alertname":"TargetDown",
            "job":"kube-proxy",
            "namespace":"kube-system",
            "prometheus":"monitoring/kube-prometheus-kube-prome-prometheus",
            "service":"kube-prometheus-kube-prome-kube-proxy",
            "severity":"warning"
         },
         "annotations":{
            "description":"100% of the kube-proxy/kube-prometheus-kube-prome-kube-proxy targets in kube-system namespace are down.",
            "runbook_url":"https://runbooks.prometheus-operator.dev/runbooks/general/targetdown",
            "summary":"One or more targets are unreachable."
         },
         "startsAt":"2023-08-13T03:48:56.603Z",
         "endsAt":"0001-01-01T00:00:00Z",
         "generatorURL":"http://kube-prometheus-kube-prome-prometheus.monitoring:9090/graph?g0.expr=100+%2A+%28count+by+%28job%2C+namespace%2C+service%29+%28up+%3D%3D+0%29+%2F+count+by+%28job%2C+namespace%2C+service%29+%28up%29%29+%3E+10\u0026g0.tab=1",
         "fingerprint":"65b77eed2fe8b9c7"
      }
   ],
   "groupLabels":{
      "namespace":"kube-system"
   },
   "commonLabels":{
      "alertname":"TargetDown",
      "job":"kube-proxy",
      "namespace":"kube-system",
      "prometheus":"monitoring/kube-prometheus-kube-prome-prometheus",
      "service":"kube-prometheus-kube-prome-kube-proxy",
      "severity":"warning"
   },
   "commonAnnotations":{
      "description":"100% of the kube-proxy/kube-prometheus-kube-prome-kube-proxy targets in kube-system namespace are down.",
      "runbook_url":"https://runbooks.prometheus-operator.dev/runbooks/general/targetdown",
      "summary":"One or more targets are unreachable."
   },
   "externalURL":"http://kube-prometheus-kube-prome-alertmanager.monitoring:9093",
   "version":"4",
   "groupKey":"{}:{namespace=\"kube-system\"}",
   "truncatedAlerts":0
}
```