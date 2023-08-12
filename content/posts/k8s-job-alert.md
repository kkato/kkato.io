---
title: "KubernetesのCronJobが失敗したときにアラートをあげる"
date: 2023-08-08T19:48:39+09:00
draft: false
tags: ["kubernetes", "alertmanger"]
---

CronJobは、指定したcronフォーマットに基づいて定期的にJobを実行するという機能です。
主にバックアップの取得やメール送信など、定期的な処理を実行する際に便利です。

一方で、CronJobが失敗した際にどのように検知すべきかについては、あまり情報がありませんでした。
なので、今回はそちらについて考えてみたいと思います。

## 案
CronJobが失敗した際にどのように検知すべきか、個人的に思いついた案としては以下になります。
1. 特定の文字列を含むログを検知する
2. Jobによって起動されたPodのメトリクスを活用する
3. kube-state-metricsを活用する

### 案1. 特定の文字列を含むログを検知する
Jobによって起動されたPodが標準出力にログを吐くようであれば、そのログをもとにアラートを発火するのが簡単だと思います。
しかし、標準出力にログを吐かない場合や、失敗時のログが把握できない場合には難しいかもしれません。

### 案2. Jobによって起動されたPodのメトリクスを活用する
Jobによって起動されたPodから取得したメトリクスに基づきアラートを発火するのも可能だと思います。
しかし、Jobによって起動されたPodは処理終了後に削除されるので、Prometheusがメトリクスを取得できないことがあるかもしれません。
Pushgatewayなどを使うことで回避できそうですが、設定がやや複雑になりそうです。また、Podにexporterが備わっておらずメトリクスが取得できない場合もあるかもしれません。

### 案3. kube-state-metricsを活用する（採用）
kube-state-metricsとはKubernetesクラスタ内のリソースの状態をメトリクスとして提供するというものです。
kube-state-metricsではCronJobやJobの状態をメトリクスとして取得することができます。
調べてみた際※には、こちらの案を採用している記事が多かったので、こちらで検討を進めてみたいと思います。

※調べている際に見つけた記事
- https://medium.com/@tristan_96324/prometheus-k8s-cronjob-alerts-94bee7b90511
- https://www.giffgaff.io/tech/monitoring-kubernetes-jobs


## kube-state-metricsのメトリクス(CronJob, Job, Pod)
CronJobはcronフォーマットで指定された時刻にJobを生成し、そのJobがPodを生成するという3層の親子構造になっています。
また、CronJobとJobの関係は1対多で、JobとPodの関係は1対多になります。

CronJobとJobの関係はわかりやすいと思いますが、JobとPodの関係は少しわかりにくい部分があるので説明します。
Jobは複数のPodを生成します。というのも、Jobが完了するのは`completions`個のPodが正常終了した場合、もしくは`backoffLimit`個のPodが異常終了した場合に限ります。
- completions
    - completions個のPodが成功すると、Jobが完了したとみなされる
    - デフォルトは1
- backoffLimit
    - 失敗したと判断するまでの再試行回数で、この回数を超えるとJobは失敗とみなされる
    - デフォルトは6

そして、アラートを上げたいのはJobが失敗した時だと思います。言い換えると、`backofLimit`個のPodが異常終了したときにアラートを上げたいということになると思います。
各Podを監視し、何個のPodが異常終了したのか監視することも可能だと思いますが、ここではJobのステータスを監視するのが適切かと思います。

### Jobに関するメトリクス

| メトリクス | 対応する項目 | 説明|
|---|---|---|
| kube_job_status_succeeded | .status.succeeded | Jobの管理下で正常終了したPodの数 |
| kube_job_status_failed | .status.failed | Jobの管理下で異常終了したPodの数 |
| kube_job_complete | .status.conditions.type | Jobが成功したかどうか |
| kube_job_failed | .status.conditions.type | Jobが失敗したかどうか |

Jobに関するメトリクスをピックアップしてみました。
この中でも特に`kube_job_failed`を使ってアラートを上げるのが良さそうです。
```
kube_job_failed > 0
```

## 課題
しかし、失敗したJobは削除されずに残り続けるので、アラートが発火し続けてしまいます。
なので、Jobの`ttlSecondsAfterFinished`を設定しましょう。
- ttlSecondsAfterFinished
    - 指定秒後にJobを削除できる


CronJobの`failedJobsHistoryLimit`を設定するという方法も思いつきましたが、0にしてしまうとそもそもアラートが上がりません。
- failedJobsHistoryLimit
    - 失敗したJobを指定個数分残しておける
    - デフォルトは1
