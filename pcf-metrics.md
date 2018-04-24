## PCF Metricsによるアプリケーションのモニタリング

PCFの独自機能として、[PCF Metrics](https://metrics.run.pivotal.io)というアプリケーションのメトリクスやログを蓄積して視覚化する仕組みが用意されています。PCFのライセンス内で無償で利用可能です。

PCF Metricsを利用することで直近1日間の

* コンテナのCPU、メモリ、ディスク使用率
* 単位秒あたりのリクエスト数、エラー数、リクエストのレイテンシー
* アプリケーションのイベント
* アプリケーションのログ

を確認することができます。

Apps Manager URL: https://apps.sys.pcflab.jp

管理コンソールから「View in PCF Metrics」リンクをクリックしてください。

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/215e8075-6064-47cf-c1f3-6792a73b5826.png)

**「PCF Metrics」へのリンク**

ダッシュボードへアクセスできます。ここではアプリケーションに発生したイベントと直近5分間のメトリクスを確認することができます。

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/06d03de0-a13c-a06a-1470-2c281f3aedbc.png)

**「PCF Metrics」のダッシュボード**

左側のメニューの「EXPLORE」をクリックしてください。ネットワークに関するメトリクスのグラフとアプリケーションログが表示されます。1分、1時間、1日という単位で状態を確認することができます。

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/1f46c343-2e1b-7577-0ff1-c12a19563417.png)

**ネットワークメトリクスとログ**

ログはキーワードで検索することもできます。

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/b9b6d9e2-3fce-48a8-d443-dea8a91b2fc3.png)

**ログの検索**

「Container Metrics」にチェックを入れることでコンテナに関するメトリクスのグラフも表示できます。

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/94dd63a3-2708-a84d-7ec2-05aab56a48d8.png)

**コンテナメトリクス**

「PCF Metrics」を利用することで直近でアプリケーションに何が起きたのか、どこでリクエストやCPU使用率のスパイクが発生したのかを把握することが可能になります。
アプリケーション開発者がこのような便利な機能をセットアップすることなく使えるようになるのはPaaSのメリットの一つです。

**ここまで完了したら進捗シートにチェックをしてください。**

## Metrics Forwarderを使ってアプリケーションメトリクスを転送

**アプリケーションの確認**
`hello-redis-<STUDENT>`のアプリケーションが起動しているか確認をしてください。
```console
$ cf start hello-redis-<STUDENT_ID>
$ cf app hello-redis-<STUDENT_ID>
$ cf set-env hello-redis-<STUDENT_ID> management.security.enabled false
```

**Metrics Forwarderインスタンスの作成**
```console
$ cf marketplace
Getting services from marketplace in org pcfdemoapp / space development as kab...
OK

service             plans                                    description
app-autoscaler      standard                                 Scales bound applications in response to load
metrics-forwarder   unlimited                                Custom metrics service
p-cloudcache        small, dev-plan                          Pivotal Cloud Cache offers the ability to deploy a GemFire cluster as a service in Pivotal Cloud Foundry.
p-redis             dedicated-vm, shared-vm                  Redis service to provide pre-provisioned instances configured as a datastore, running on a shared or dedicated VM.
p.redis             cache-small, cache-medium, cache-large   Redis service to provide on-demand dedicated instances configured as a cache.

TIP:  Use 'cf marketplace -s SERVICE' to view descriptions of individual plans of a given service.
```
Service MarketplaceにあるPCF Metrics Forwarderのインスタンスを払い出し、アプリケーションにバインドします。
これによりMetrics Forwarderからアプリケーションのメトリクスを外部に転送できます。
```console
$ cf create-service metrics-forwarder unlimited my-forwarder
$ cf bind-service my-forwarder hello-redis-<STUDENT_ID>
$ cf restaget hello-redis-<STUDENT_ID>
```

**PCF Metricsのダッシュボードの設定**
PCF Metrics Forwarderによる転送されるメトリクスをPCF MetricsのGUIから見てみます。
PCF Metrics: https://login.sys.pcflab.jp/login

[写真]

`ADD CHART`から自由にメトリクスを選択してみてください。

[写真]

**ここまで完了したら進捗シートにチェックをしてください。**


**PCF Apps ManagerによるSpring Bootアプリケーションのモニタリング**
PCF Apps Managerを利用するとSpring Boot Actuatorのエンドポイントを使ってSpring Bootアプリケーションの様々なモニタリングをGUI上で行うことができます。
PCF Apps Managerにログインをしてください。

Apps Manager URL: https://apps.sys.pcflab.jp

[写真]


**ここまで完了したら進捗シートにチェックをしてください。**
