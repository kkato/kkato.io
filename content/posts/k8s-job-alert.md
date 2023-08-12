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

##  kube-state-metricsを活用する
kube-state-metricsとはKubernetesクラスタ内のリソースの状態をメトリクスとして提供するというものです。
kube-state-metricsではCronJobやJobの状態をメトリクスとして取得することができます。
調べてみた際※には、この方式採用している記事が多かったので、こちらで検討を進めてみたいと思います。

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

そして、アラートを上げたいのはJobが失敗した時だと思います。言い換えると、`backoffLimit`個のPodが異常終了したときにアラートを上げたいということになると思います。
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

## アラートが発火し続けてしまう問題
失敗したJobは削除されずに残り続けるので、アラートが発火し続けてしまいます。
なので、Jobの`ttlSecondsAfterFinished`を設定し、数分後にJobが削除されるようにします。
- ttlSecondsAfterFinished
    - 指定秒後にJobを削除できる


CronJobの`failedJobsHistoryLimit`を設定するという方法も思いつきましたが、0にしてしまうとそもそもアラートが上がりません。
- failedJobsHistoryLimit
    - 失敗したJobを指定個数分残しておける
    - デフォルトは1
