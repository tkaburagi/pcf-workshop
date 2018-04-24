## スケールアウト/オートヒーリング (Java編)

本章では[バックエンドサービスRedisの利用](backend-service-redis_java.md)で作成したプロジェクトを利用します。

どちらのインスタンスにアクセスしているか分かるようにソースコードを修正しましょう。

``` java
    @GetMapping("/")
    String hello() {
        return greeter.hello() + " (" + System.getenv("CF_INSTANCE_INDEX") + ")"; // この行を変更
    }
```

インスタンス数1を指定して`cf push`します。

``` console
$ ./mvnw package -Dmaven.test.skip=true
$ cf push hello-redis-<STUDENT_ID> -p target/hello-redis-0.0.1-SNAPSHOT.jar --no-start
$ cf set-env hello-redis-<STUDENT_ID> JAVA_OPTS '-XX:ReservedCodeCacheSize=32M -XX:MaxDirectMemorySize=32M'
$ cf set-env hello-redis-<STUDENT_ID> JBP_CONFIG_OPEN_JDK_JRE '{ memory_calculator: { stack_threads: 30 } }'
$ cf scale -m 512m hello-redis-<STUDENT_ID>
```

別のターミナルを二つ立ち上げて次の2つのコマンドをそれぞれ実行しながら以後のステップ進むとわかりやすいです。

```
while true;do curl -s https://hello-redis-<STUDENT_ID>.apps.pcflab.jp/;echo;sleep 1;done
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

インスタンス数やメモリ数はApps Managerの管理画面からも変更できます。

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/fae8e9f5-10d9-9533-bcd7-620f6e912546.png)

Statusの表にインスタンスの情報が反映されます。

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/f705c419-76fc-6231-9f7f-355f951220c2.png)

Cloud Foundry内のRouterによってHTTPリクエストは2インスタンスに分散されますが、
インスタンス間でRedisキャッシュが共有されているため、出力結果は依然として同じです。

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/5f9e014c-e422-6882-ba82-3a66f4c4462b.png)

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

```
$ curl -X POST https://hello-redis-<STUDENT_ID>.apps.pcflab.jp/shutdown --insecure
{"message":"Shutting down, bye..."}
```

3つのインスタンスのうちのインスタンスがダウンしたため、アプリケーションにアクセスすると残りのインスタンスからのみレスポンスがあります。

Cloud Foundryは指定したインスタンス数を保つために自動でインスタンスを再起動させます。しばらくすると再び3つのインスタンスからレスポンスがあることを確認できるでしょう。

**ここまで完了したら進捗シートにチェックをしてください。**
