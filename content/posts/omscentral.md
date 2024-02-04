---
title: "ジョージア工科大学 OMSCSを振り返って"
date: 2023-04-10T18:05:06+09:00
draft: false
tags: ["omscs"]
---

2020年8月にジョージア工科大学 コンピューターサイエンス修士課程(通称OMSCS)に入学し、2022年12月に卒業しました。今回は入学から卒業までを振り返ってみようと思います。

## なぜOMSCSを始めたのか
実は途中でOMSAからOMSCSに編入していたりしますが、OMSCSを始めた主な目的は以下です。
- 専門的な知識を身につける

私は物理学の学士課程を卒業していますが、物理を学ぶと選択肢は広がるものの少し中途半端なんですよね。例えば、機械系に行きたいとすると機械工学の学生でいいじゃん、電子系に行きたいとすると電子工学の学生でいいじゃん、IT系に行きたいとすると情報系の学生でいいじゃん、となってしまいます。なので、物理学＋αでより専門的な知識を身につけて、今後のキャリアアップに役立てたいと考えました。

- 海外で就職する

実は私はカナダの大学で学士号を取得していますが、卒業当時の私は現地就職するだけの自信と実力がありませんでした。国際的に評価されている大学の修士号であれば、海外で就職するときもプラスに働くと思い、OMSCSを始めることにしました。

## 実際どうだったか

正直かなり大変でした。コロナで家にいる時間も多かったですが、平日の仕事終わりと週末は基本的に家でずっと勉強していました。遊びに誘われることもありましたが、毎回断っていたら誰からも誘われなくなりました笑

どれくらいの時間が必要かというと、前提知識があるかどうかにもよると思いますが、1コースにつき週10-20時間は必要だと思います。ちなみに私は基本的に1学期に2コース取っていたので、休む時間がほとんどありませんでした。

また、卒業には以下の条件があり、良い成績をとらないと卒業できないというプレッシャーがありました。
- 最初の12ヶ月でfundamental courseを最低2つ履修し、B以上の成績を収めなければならない
- 専門分野のコースは全てB以上の成績を収めなければならない
- 合計GPAが3.0/4.0以上でなければならない

## どんな授業を受けたのか

OMSCSではSpecialization[*1](#参考)といって専門分野を決める必要があるのですが、私はComputing Systemsにしました。その理由としては配属された部署がデータベースや分散システムなどのミドルウェアを扱うところだったから、また低レイヤーの技術に興味があったからです。OMSCSでは10コース履修する必要がありますが、そのうち5, 6コースを自分の専門分野から履修しなければなりません(いわゆる必修科目)。

| 学期        | 受講したコース | 難易度 | 作業量 |
|-------------|--------|--------|--------|
| Fall 2020   | CSE6040 Computing for Data Anlysis | 易しい  | 10時間/週
| Fall 2020   | ISYE6501 Introduction to Analytics Modeling | 普通  | 15時間/週 |
| Summer 2021 | CS6300 Software Development Process | 普通 | 10時間/週 |
| Fall 2021   | CS6250 Computer Networks | 普通 | 10時間/週 |
| Fall 2021   | CS7646 Machine Learning for Trading | 普通 | 15時間/週 |
| Spring 2022 | CS6035 Introduction to Information Security | 普通 | 15時間/週 |
| Spring 2022 | CS6200 Introduction to Operating Systems| 難しい | 20時間/週 |
| Summer 2022 | CS6262 Network Security | 普通 | 15時間/週 |
| Fall 2022   | CS6515 Introduction to Graduate Algorithms | 難しい | 15時間/週 |
| Fall 2022   | CS9903-O13 Quantum Computing | 易しい | 10時間/週 |

受講したコースについて少し紹介しようと思います。2023年現在、OMSCSでは61コース提供されており[*2](#参考)、定期的に追加・更新されます。また、OMSCSにはOMSCentral[*3](#参考)という非公式のレビューサイトがあるので、そちらも参考にしてみてください。

### CSE6040 Computing for Data Anlysis
OMSA限定のコースになります。データサイエンスで必要となるPythonとそのライブラリ(numpy, scipay, pandas)の使い方が学べるので、データサイエンス初心者にぴったりのコースです。講義も課題もしっかり整理されていて、とても分かりやすかったです。

Grade distribution:
- Notebooks(x15): 50%
- Midterm 1: 10%
- Midterm 2: 15%
- Final: 25%

### ISYE6501 Introduction to Analytics Modeling
こちらはOMSAのコースですが、OMSCSの生徒も受講できます。基礎的な統計モデル(Suppor Vector Machines、K-Nearest Neighbors、K-Means Clusteringなど)について学習します。課題のほとんどはRで、Rを使ったことがなかったので、結構時間がかかりました。採点は生徒同士で行うので、かなりばらつきがあります。課題と授業の内容と乖離があり、テストでは思うように点が取れず苦労しました。

Grade distribution:
- Homework(x15): 15%
- Course Project: 8%
- Midterm 1: 25%
- Midterm 2: 25%
- Final: 25%
- Syllabus Quiz: 2%

### CS6300 Software Development Process
こpのコースでは、ソフトウェア開発手法(ウォーターフォール、アジャイル、プロトタイプ、スパイラル)やテスト手法(ブラックボックス、ホワイトボックス)、バージョン管理、UMLなどソフトウェア工学全般について学びます。課題は主にJavaを使いますが、GitやJUnit、UMLを書く課題もあります。グループプロジェクトでは3, 4人のチームを組み、Androidアプリを作成しました。幸いチームメンバはとても協力的だったので、スムーズにプロジェクトを終えることができました。

Grade distribution:
- Assignments(x6): 44%
- Individual Project: 25%
- Group Project: 18%
- Collaboration (グループプロジェクトでチームメンバーから評価される): 10%
- Participation (Ed Discussionでの発言、Participation Quizなど): 3%

### CS6250 Computer Networks
このコースでは、TCPやUDPの基礎知識や、ルーティング・プロトコル(経路制御アルゴリズム、RIP、OSPF、BGP)、ルーターの仕組み、Software Defined Networkingなどについて学びます。課題はSpanning Tree Protocol、Distance Vector Routing、SDN Firewallなどを実際にPythonで実装します。授業のビデオは棒読みで分量も多く眠気を誘ってきますが、課題の手順はかなり明確に説明されているので良かったです。マスタリングTCP/IPを読んで補足的に勉強しました。

Grade Distribution:
- Assignments(x5): 66%
- Exam 1: 12%
- Exam 2: 12%
- Quizzes: 10%
- Extra credit: 3%

### CS7646 Machine Learning for Trading
このコースでは、線型回帰、Q-Learning、k-Nearest Neighbor、回帰木などの統計手法を、どのように株取引に応用できるか学習します。統計手法はもちろんのこと、テクニカル分析などで使われるSimple Moving Average、Bollinger Bands、Stochastic Indicatorなどについても学びます。課題では過去の株取引のデータを渡され、お題にあったモデルをPythonで実装し、その考察をレポートに書いて提出します。個人的には課題がとても楽しく、内容も難しすぎず易しすぎず、お気に入りのコースの1つです。

Grade Distribution:
- Projects(x8): 73%
- Exams(x2): 25%
- Surveys: 2%
- Extra credit: 2%

### CS6035 Introduction to Information Security
このコースでは、ソフトウェア、OS、データベースのセキュリティや暗号に関するアルゴリズム/プロトコル、リスクマネジメントについて学習します。授業で習うこととプロジェクトに必要な知識・能力に乖離があり、プロジェクトが80%と多くの比重を占めるので、プロジェクト優先で取り組むのが良さそうです。課題ではC、Python、Java、HTML/JavaScript/PHP、SQLなど様々な知識が必要になりますが、課題をこなしながらキャッチアップすることは十分可能です。課題ではスタックオーバーフローを実装したり、Cuckooというツールでマルウェア解析したり、ウェブサイトにXSSやSQLインジェクションを仕掛けたりと結構楽しい課題が多かったです。

Grade distribution:
- Projects(x4): 80%
- Quizzes(x4): 10%
- Exams(x2): 10%

### CS6200 Introduction to Operating Systems
このコースでは、プロセスやプロセスマネジメント、スレッド、MutexやSemaphoreなどを使った並行性、メモリ管理、仮想化などについて学習します。課題では、ファイルを転送するクライアント・サーバ、キャシュサーバー、プロキシサーバーの実装をCで行ったり、gRPCの実装をC++を行ったりと、C/C++の前提知識がないとかなり大変です。私はC/C++を触ったことがほとんどなかったので、とても苦労しました。(まさか誕生日に徹夜することになるとは...)しかし、CSにおいてとても重要な知識になるので、個人的にはOMSCSにおけるmust-takeだと思っています。

Grade distribution:
- Participation: 5%
- Projects(x3): 45%
- Midterm Exam: 25%
- Final Exam: 25% \
※Curveありです。85%以上でA、65%以上でBくらいだと思います。

### CS6262 Network Security
このコースでは、DDoSやWebのセッションマネジメント、マルウェア解析、ボットネット検出などについて学びます。CS6035とかなり似ており、こちらもプロジェクトが80%と多くの比重を占めます。課題では、VirtualBoxのディスクイメージが配られ、様々なツール(nmapコマンド、Metasploit、John the Ripper、cewl、Cuckoo、Wiresharkなど)を駆使してポートスキャンやパスワード解析、権限奪取、マルウェア解析、XSS、ボットネット検出などを行います。かなり実践的なコースとなっており、CS6035と同じく結構楽しく課題に取り組めました。私は前提知識が少ない分時間がかかりましたが、地道に取り組めば必ず解けると思います。

Grade distribution:
Quizzes: 10%
Projects(x5): 80%
Exam: 10%
Extra credit: 5%

### CS6515 Introduction to Graduate Algorithms
このコースでは、Dynamic Programming、Divide and Conquer、RSA、グラフ理論gra, Linear Programming, NP完全問題について学習します。このコースでは、Study Groupを作って勉強することが推奨されているので、私はタイムゾーンが近い韓国人とマレーシア人の方と一緒にわからないところを質問し合ったり、情報共有するようにしました。また、このコースでは回答時の「フォーマット」がすごく重視されるので、宿題でその「フォーマット」に慣れておくようにしました。過去の生徒がきれいにまとめてくれたノート(Joves Notes)があり、Joves Notesに答えが載っている問題は全て解くようにしました。

Grade distribution:
- Homework(x8): 12%.
- Polls: 4%
- Coding Projects(x3): 9%.
- Logistic quizzes: 3%.
- Exams(best 3 out of 4): 72% \
※Curveありです。85%以上でA、65%以上でBくらいだと思います。

### CS9903-O13 Quantum Computing
このコースでは、Single Qubit Gates、Multi-Qubit Gatesや線形代数、量子もつれ、基礎的なアルゴリズム(Deusch-Joza Algorithm, Bernstein-Vazirani Algorithm, Grover's Alogorithm, Simon's Algorithm, Shor's Algorithm)について学習します。課題ではQiskitを用いて、実際にいくつかのアルゴリズムを実装します。Qiskitのドキュメントを参考に課題を進め、分からないところはTAに質問するようにしました。量子コンピュータということでかなり難しいと予想していましたが、量子コンピュータに対する前提知識がなくても十分Aを取ることは可能だと思います。

Grade distribution:
- Reviews(x5): 20%
- Midterm: 20%
- Final: 20%
- Labs(x4): 40%

## まとめ
OMSCSを始めた理由や各授業の内容、その感想などをご紹介させていただきました。ほとんどCSを学んだことがない私にとってはかなり大変でしたが、CSを学ぶ良い機会となりました。とはいえすぐに実務に直結するような内容はあまりないので、後になってじわじわ役に立つ知識という印象です(役に立ってくれるといいな)。なので学部でCSを学んでいた方が取得するメリットはあまりないと思いますが、私のように今までちゃんとCSを学んだことがない方や改めてCSを学び直したい方にはおすすめかなと思います。

## 参考
*1 [Specializations](https://omscs.gatech.edu/program-info/specializations) \
*2 [Current Courses](https://omscs.gatech.edu/current-courses) \
*3 [OMSCentral](https://www.omscentral.com/)