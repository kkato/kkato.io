---
title: "KubernetesのCronJobが失敗したときにアラートをあげる"
date: 2023-08-08T19:48:39+09:00
draft: false
---

CronJobは、指定したcronフォーマットに基づいて定期的にJobを実行するという機能です。
主にバックアップの取得やメール送信など、定期的な処理を実行する際に便利です。

一方で、CronJobが失敗した際にどのように検知すべきかについては、あまり情報がありませんでした。
なので、今回はそちらについて考えてみたいと思います。

##  kube-state-metricsを活用する
kube-state-metricsとはKubernetesクラスタ内のリソースの状態をメトリクスとして提供してくれるというものです。
kube-state-metricsではCronJobやJobの状態をメトリクスとして取得することができます。
この方式採用している記事が多かったので、こちらで検討を進めてみたいと思います。

※記事
- https://medium.com/@tristan_96324/prometheus-k8s-cronjob-alerts-94bee7b90511
- https://www.giffgaff.io/tech/monitoring-kubernetes-jobs


## どのメトリクスを使い監視するか
CronJobはcronフォーマットで指定された時刻にJobを生成し、そのJobがPodを生成するという3層の親子構造になっています。
また、CronJobとJobの関係は1対多で、JobとPodの関係は1対多になります。

Jobが完了するのは`completions`個のPodが正常終了した場合、もしくは`backoffLimit`個のPodが異常終了した場合に限ります。
- completions
    - completions個のPodが成功すると、Jobが完了したとみなされる
    - デフォルトは1
- backoffLimit
    - 失敗したと判断するまでの再試行回数で、この回数を超えるとJobは失敗とみなされる
    - デフォルトは6

そして、アラートを上げたいのはJobが失敗した時です。言い換えると、`backoffLimit`個のPodが異常終了したときにアラートを上げるということになります。
何個のPodが異常終了したのか監視することも可能だと思いますが、ここではJobのステータスを監視するのが適切かと思います。

| メトリクス | 対応する項目 | 説明|
|---|---|---|
| kube_job_status_succeeded | .status.succeeded | Jobの管理下で正常終了したPodの数 |
| kube_job_status_failed | .status.failed | Jobの管理下で異常終了したPodの数 |
| kube_job_complete | .status.conditions.type | Jobが成功したかどうか |
| kube_job_failed | .status.conditions.type | Jobが失敗したかどうか |

参考: https://github.com/kubernetes/kube-state-metrics/blob/main/docs/job-metrics.md

Jobに関するメトリクスは複数ありますが、この中でも特に`kube_job_failed`を使ってアラートを上げるのが良さそうです。
以下の設定だと、Jobが失敗したときにアラートが上がります。
```
kube_job_failed > 0
```

## アラートが発火し続けてしまう問題
失敗したJobは削除されずに残り続けるので、アラートが発火し続けてしまいます。
なので、Jobの`ttlSecondsAfterFinished`を設定し、数分後にJobが削除されるようにします。
- ttlSecondsAfterFinished
    - 指定秒後にJobを削除できる


CronJobの`failedJobsHistoryLimit`を設定するという方法も思いつきましたが、0にしてしまうとそもそもアラートが上がらないと思うので、こちらは1のままにしておきました。
- failedJobsHistoryLimit
    - 失敗したJobを指定個数分残しておける
    - デフォルトは1

## まとめ
今回はkube-state-metricsを活用して、CronJobが失敗した時のアラートを設定しました。他にもっといい方法をご存知の方は教えていただけるとありがたいです。
