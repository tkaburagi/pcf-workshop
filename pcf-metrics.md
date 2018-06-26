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
PCF Metrics: https://metrics.sys.pcflab.jp

![image](https://storage.googleapis.com/pcf-workshop/metrics1.png)

`ADD CHART`から自由にメトリクスを選択してみてください。

![image](https://storage.googleapis.com/pcf-workshop/metrics2.png)

メトリクス一覧にSpring Actuatorのメトリクスが可視化されます。
![image](https://storage.googleapis.com/pcf-workshop/metrics3.png)

**ここまで完了したら進捗シートにチェックをしてください。**


**PCF Apps ManagerによるSpring Bootアプリケーションのモニタリング**
PCF Apps Managerを利用するとSpring Boot Actuatorのエンドポイントを使ってSpring Bootアプリケーションの様々なモニタリングをGUI上で行うことができます。
PCF Apps Managerにログインをしてください。

Apps Manager URL: https://apps.sys.pcflab.jp

**ログレベルの変更**
Spring BootアプリケーションのログレベルをApps Managerから動的に変更可能です。
Apps Managerから`hello-redis-<STUDENT_ID>`のアプリケーションを選択し、`Logs`メニューから`CONFIGURE LOG LEVEL`を選択してください。
ROOTのログレベルを`DEBBUG`に変更してみます。
![image](https://storage.googleapis.com/pcf-workshop/loglevel.png)

```console
$ cf logs hello-redis-<STUDENT_ID>
Retrieving logs for app pcfdemoapp-dev in org pcfdemoapp / space development as kab...

   2018-04-26T15:21:35.46+0900 [APP/PROC/WEB/0] OUT 2018-04-26 06:21:35.463 DEBUG 15 --- [pool-1-thread-1] s.n.www.protocol.http.HttpURLConnection  : sun.net.www.MessageHeader@3e0111c08 pairs: {POST /v1/metrics HTTP/1.1: null}{Accept: application/json, application/*+json}{Authorization: 4fe4231e-f775-4e2f-4ccb-3117c47b70a5}{Content-Type: application/json;charset=UTF-8}{User-Agent: Java/1.8.0_162}{Host: metrics-forwarder.sys.pcflab.jp}{Connection: keep-alive}{Content-Length: 5593}
   2018-04-26T15:21:35.47+0900 [APP/PROC/WEB/0] OUT 2018-04-26 06:21:35.469 DEBUG 15 --- [pool-1-thread-1] s.n.www.protocol.http.HttpURLConnection  : sun.net.www.MessageHeader@17eb829113 pairs: {null: HTTP/1.1 202 Accepted}{Content-Length: 0}{Content-Type: text/plain; charset=utf-8}{Date: Thu, 26 Apr 2018 06:21:35 GMT}{X-Ratelimit-Limit: unlimited}{X-Ratelimit-Metrics-Limit: unlimited}{X-Ratelimit-Metrics-Remaining: unlimited}{X-Ratelimit-Period: 60}{X-Ratelimit-Remaining: unlimited}{X-Ratelimit-Retry-After: 0}{X-Vcap-Request-Id: 9290632f-298d-438b-61b0-e89cbf4d416e}{Via: 1.1 google}{Alt-Svc: clear}
   2018-04-26T15:21:35.47+0900 [APP/PROC/WEB/0] OUT 2018-04-26 06:21:35.470 DEBUG 15 --- [pool-1-thread-1] o.s.web.client.RestTemplate              : POST request for "https://metrics-forwarder.sys.pcflab.jp/v1/metrics" resulted in 202 (Accepted)
   2018-04-26T15:21:35.47+0900 [APP/PROC/WEB/0] OUT 2018-04-26 06:21:35.471 DEBUG 15 --- [pool-1-thread-1] o.c.m.RestOperationsMetricPublisher      : Sent Spring Boot metrics to Metrics Forwarder service
   2018-04-26T15:21:47.27+0900 [APP/PROC/WEB/0] OUT 2018-04-26 06:21:47.260 DEBUG 15 --- [8080-Acceptor-0] o.apache.tomcat.util.threads.LimitLatch  : Counting up[http-nio-8080-Acceptor-0] latch=1
   2018-04-26T15:21:47.27+0900 [APP/PROC/WEB/0] OUT 2018-04-26 06:21:47.261 DEBUG 15 --- [nio-8080-exec-8] o.a.tomcat.util.net.SocketWrapperBase    : Socket: [org.apache.tomcat.util.net.NioEndpoint$NioSocketWrapper@66cadff2:org.apache.tomcat.util.net.NioChannel@4dc9bea8:java.nio.channels.SocketChannel[connected local=/10.255.107.49:8080 remote=/10.255.107.49:40998]], Read from buffer: [0]
   2018-04-26T15:21:47.27+0900 [APP/PROC/WEB/0] OUT 2018-04-26 06:21:47.275 DEBUG 15 --- [nio-8080-exec-8] o.apache.coyote.http11.Http11Processor   : Error parsing HTTP request header
```
上記のように再起動等することなくデバッグログを取得できます。

**スレッドダンプの取得**
次にApps Managerの`Threads`メニューからスレッドダンプを確認します。
![image](https://storage.googleapis.com/pcf-workshop/thread.png)

`DOWNLOAD`を押すとスナップショットをローカルに保存できます。

**ここまで完了したら進捗シートにチェックをしてください。**
