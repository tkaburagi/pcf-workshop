## スケールアウト/オートヒーリング (Java編)

本章では[バックエンドサービスRedisの利用](backend-service-redis_java.md)で作成したプロジェクトを利用します。

どちらのインスタンスにアクセスしているか分かるようにソースコードを修正しましょう。

``` java
    @GetMapping("/")
    String hello() {
        return greeter.hello() + " (" + System.getenv("CF_INSTANCE_INDEX") + ")" + " on " + System.getenv("CF_INSTANCE_IP"); // この行を変更
    }
```

インスタンス数1を指定して`cf push`します。

``` console
$ ./mvnw package -Dmaven.test.skip=true
$ cf push hello-redis-<STUDENT_ID> -i 1 -p target/hello-redis-0.0.1-SNAPSHOT.jar --no-start 
$ cf set-env hello-redis-<STUDENT_ID> JAVA_OPTS '-XX:ReservedCodeCacheSize=32M -XX:MaxDirectMemorySize=32M'
$ cf set-env hello-redis-<STUDENT_ID> JBP_CONFIG_OPEN_JDK_JRE '{ memory_calculator: { stack_threads: 30 } }'
$ cf scale -m 512m hello-redis-<STUDENT_ID>
```

別のターミナルを二つ立ち上げて次の2つのコマンドをそれぞれ実行しながら以後のステップ進むとわかりやすいです。

```
while true;do curl -s http://hello-redis-<STUDENT_ID>.apps.pcflab.jp/;echo;sleep 1;done
```

```
while true;do cf app hello-redis-<STUDENT_ID>;sleep 1;done
```

**ここまで完了したら進捗シートにチェックをしてください。**

Cloud Foundryではスケールアウトも簡単です。`cf scale -i <Instance Count> <App>`で指定したインスタンス数にスケールアウトできます。

``` console
$ cf scale -i 2 hello-redis-<STUDENT_ID>
```

ログを見て、2つ目のインスタンスであることを示す`[APP/1]`が出力されていることを確認してください。

``` console
$ cf logs hello-redis-<STUDENT_ID> --recent
...
2016-03-23T17:15:23.29+0900 [API/5]      OUT Updated app with guid 77e2f12a-8c1c-4efb-9258-0f8020b4ca27 ({"instances"=>2})
2016-03-23T17:15:23.54+0900 [CELL/1]     OUT Creating container
2016-03-23T17:15:31.15+0900 [APP/1]      OUT   .   ____          _            __ _ _
2016-03-23T17:15:31.15+0900 [APP/1]      OUT  /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
2016-03-23T17:15:31.15+0900 [APP/1]      OUT  \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
2016-03-23T17:15:31.15+0900 [APP/1]      OUT   '  |____| .__|_| |_|_| |_\__, | / / / /
2016-03-23T17:15:31.15+0900 [APP/1]      OUT  =========|_|==============|___/=/_/_/_/
2016-03-23T17:15:31.15+0900 [APP/1]      OUT  :: Spring Boot ::        (v1.3.3.RELEASE)
...
```

`cf apps`で対象のインスタンス数が`2/2`になっていることを確認できます。

``` console
$ cf apps
Getting apps in org tmaki / space development as ****@gmail.com...
OK

name                requested state   instances   memory   disk   urls   
hello-redis-tmaki   started           2/2         1G       1G     hello-redis-tmaki.cfapps.io
```

アプリケーションにアクセスをすると、異なるインスタンスにアクセスしていることがわかります。

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/6db798b2-9534-1d80-4a1b-d22ef7f02f58.png)

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/7a848882-f164-d0b0-6f0e-1f242b848499.png)

**ここまで完了したら進捗シートにチェックをしてください。**

3インスタンスにスケールアウトしましょう。

``` console
$ cf scale -i 3 hello-redis-<STUDENT_ID>
```

しばらくすると3番目のインスタンスにもアクセスできていることがわかります。

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/ba4cd5ca-157c-7feb-a58f-c0b1117eae85.png)

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/448d0cd6-bbf0-cef9-b52a-b4dc11b5c982.png)

### アプリケーションに障害を起こしてみます。

以下の環境変数を設定して、アプリケーションのシャットダウンを有効にしましょう。

``` console
$ cf set-env hello-redis-<STUDENT_ID> management.security.enabled false
$ cf set-env hello-redis-<STUDENT_ID> endpoints.shutdown.enabled true
$ cf restart hello-redis-<STUDENT_ID>
```

インスタンス0~2までアクセスできることを確認した後、
以下のコマンドを実行してください。

```console
$ curl -X POST https://hello-redis-<STUDENT_ID>.apps.pcflab.jp/shutdown --insecure
{"message":"Shutting down, bye..."}
```

3つのインスタンスのうちのインスタンスがダウンしたため、アプリケーションにアクセスすると残りのインスタンスからのみレスポンスがあります。

Cloud Foundryは指定したインスタンス数を保つために自動でインスタンスを再起動させます。しばらくすると再び3つのインスタンスからレスポンスがあることを確認できるでしょう。

**ここまで完了したら進捗シートにチェックをしてください。**

次に、VMを一つ削除しVMとアプリケーションコンテナの自動復旧の様子を見てみます。
現在3インスタンスあるそれぞれのインデックス番号のコンテナがホストされているVMを各自ホワイトボードに記述してください。記述後講師がVMを一つ削除します。アプリケーションのレスポンスとアプリケーションインスタンスの様子をモニタリングするため下記の二つのコンソールが立ち上がっていることを確認してください。
```
while true;do curl -s https://hello-redis-<STUDENT_ID>.apps.pcflab.jp/;echo;sleep 1;done
```

```
while true;do cf app hello-redis-<STUDENT_ID>;sleep 1;done
```

**オプション: PCF Auto-scalerの利用**
PCFのライセンスに付随するApp Autoscalerを利用するとアプリケーションのパフォーマンスやカレンダーに基づいてアプリケーションの自動スケールアウト、スケールインを実現できます。

Apps Manager: https://apps.sys.pcflab.jp
Apps Managerにログインし、左カラムの`Marketplace`を選択し、`App Autoscaler`を選択してください。

次に`Standard Plan`を選択し、下記を入力します。
* Instance Name: my-autoscaler-<STUDENT_ID>
* Add to Space: development
* Bind to App: hello-redis-<STUDENT_ID>

インスタンスが出来たら作成したら`hello-redis-<STUDENT_ID>`の画面に移動し、Autoscalingをオンにします。
![image](https://storage.googleapis.com/pcf-workshop/autoscale.png)

`Manage Autoscaling`を押し、Autoscalerの設定を実施します。

1. 最大と最小のインスタンス数

![image](https://storage.googleapis.com/pcf-workshop/autoscale2.png)

2. CPUの閾値の設定

![image](https://storage.googleapis.com/pcf-workshop/autoscale3.png)

設定が完了したら`ENABLE`になっていることを確認します。

![image](https://storage.googleapis.com/pcf-workshop/autoscale4.png)

30秒に一度CPU利用率がチェックされ、閾値を下回るとインスタンスが一つずつ減っていくのでその様子を下記のコマンドで確認します。

```console
cf app hello-redis-<STUDENT_ID> | grep "%"
#0   running   2018-04-24T07:07:36Z   0.3%   297.1M of 1G   138.1M of 1G
#1   running   2018-04-26T06:49:18Z   0.0%   240.8M of 1G   138.1M of 1G
#2   running   2018-04-26T06:49:18Z   0.0%   40K of 1G      8K of 1G
```

数秒に一度実行すると徐々にスケールインする様子がわかります。

**ここまで完了したら進捗シートにチェックをしてください。**
