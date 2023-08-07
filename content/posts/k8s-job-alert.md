---
title: "KubernetesのCronJobが失敗したときにアラートをあげる"
date: 2023-08-06T12:48:39+09:00
draft: true
tags: ["kubernetes", "alertmanger"]
---

KubernetesのCronJobが失敗したときにどうやってアラートをあげるべきか考えてみたいと思います。


## CronJobとは?
CronJobはcronフォーマットで指定した時間になるとJobを生成し、さらにそのJobがPodを生成して処理を行います。