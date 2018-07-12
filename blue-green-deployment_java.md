## Blue-Greenデプロイ

[Blue-Greenデプロイ](http://martinfowler.com/bliki/BlueGreenDeployment.html)はBlueとGreenとよばれる2つの環境を用意して、ルーティングを切り替えることによりダウンタイムリスクを低減させる手法です。

Cloud Foundryでは`cf map-route`、`umnap-route`コマンドによりルーティングの設定を行うことでBlue-Greenデプロイを実現できます。

はじめに作成した`hello-cf`アプリケーションを使ってBlue-Greenデプロイを試しましょう。
`hello-cf`のディレクトリの直下に以下のファイルを作成します。ファイル名は`manifest.yml`としてください。

``` yaml
---
applications:
  - name: hello-<STUDENT_ID>
    path: target/hello-cf-0.0.1-SNAPSHOT.jar
    buildpack: java_buildpack_offline
```

`HelloCfApplication.java`の`hello`メソッドを少しだけ修正してください。

``` java
    @GetMapping("/")
    String hello() {
        return "Hello World! V1"; // V1を追加
    }
```

これをビルドしてpushします。これはBlueに相当します。

``` console
$ ./mvnw package -Dmaven.test.skip=true
$ cf push
```

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/16d05b51-d04c-61a9-18e2-5ced07c21034.png)


現在のアプリケーションのバージョンを確認するために、`curl`を定期的に実行しましょう。

``` console
$ while true; do curl -s http://hello-<STUDENT_ID>.apps.pcflab.jp.io; echo; sleep 1;done
```

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/b47937f2-9d32-23fd-e386-de7018394372.png)

**ここまで完了したら進捗シートにチェックをしてください。**

`HelloCfApplication.java`の`hello`メソッドを少しだけ修正してください。

``` java
    @GetMapping("/")
    String hello() {
        return "Hello World! V2"; // V2に変更
    }
```

ビルドしてpushします。これはGreenに相当します。

``` console
$ ./mvnw package -Dmaven.test.skip=true
$ cf push hello-<STUDENT_ID>-green # manifest内のapplication nameをoverride
```

`cf apps`の結果は以下のようになります。

``` console
$ cf apps
Getting apps in org tmaki / space development as ****@gmail.com...
OK

name                requested state   instances   memory   disk   urls   
hello-tmaki         started           1/1         1G       1G     hello-tmaki.cfapps.io   
hello-tmaki-green   started           1/1         1G       1G     hello-tmaki-green.cfapps.io 
```

新バージョンにアクセスして、正常に動作していることを確認してください。

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/6d89019e-59cd-4718-47c5-6115a97f29f3.png)

`cf map-route hello-<STUDENT_ID>-green <Domain> -n <Hostname>`で`hello-<STUDENT_ID>.apps.pcflab.jp`へのリクエストが`hello-<STUDENT_ID>-green`にルーティングされるようにします。

``` console
$ cf map-route hello-<STUDENT_ID>-green apps.pcflab.jp -n hello-<STUDENT_ID>
Creating route hello-tmaki.cfapps.io for org tmaki / space development as ****@gmail.com...
OK
Route hello-tmaki.cfapps.io already exists
Adding route hello-tmaki.cfapps.io to app hello-tmaki-green in org tmaki / space development as ****@gmail.com...
OK
```

2つのアプリケーションに対して`hello-<STUDENT_ID>.apps.pcflab.jp`でアクセスできるため、`curl`の結果は次のように`V1`と`V2`の両方が表示されるようになります。

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/56c6deb5-1327-dd2f-f069-7cf0a0af6192.png)


`map-route`とは反対の`unmap-route`コマンドで`hello-<STUDENT_ID>`へのルーティングを外します。

``` console
$ cf unmap-route hello-<STUDENT_ID> apps.pcflab.jp -n hello-<STUDENT_ID>
Removing route hello-tmaki.cfapps.io from app hello-tmaki in org tmaki / space development as ****@gmail.com...
OK
```

これで`V2`のみが出力されるようになりました。


![image](https://qiita-image-store.s3.amazonaws.com/0/1852/d3924903-283e-c923-1e9d-d1e28c648964.png)


`V2`に問題がなければ、旧バージョンを削除し、`hello-<STUDENT_ID>-green.apps.pcflab.jp`のルーティングも削除します。

``` console
$ cf delete hello-<STUDENT_ID>
$ cf unmap-route hello-<STUDENT_ID>-green apps.pcflab.jp -n hello-<STUDENT_ID>-green
```

`hello-tmaki.apps.pcflab.jp`にアクセスし続けていましたが、404エラーなどが発生することなく`V1`から`V2`へ移行することができました。

``` console
$ cf apps
Getting apps in org tmaki / space development as ****@gmail.com...
OK

name                requested state   instances   memory   disk   urls   
hello-tmaki-green   started           1/1         1G       1G     hello-tmaki.cfapps.io
```

アプリケーション名を`hello-tmaki`に戻しましょう。

``` console
$ cf rename hello-<STUDENT_ID>-green hello-<STUDENT_ID>
```

これで元の通りです。

``` console
$ cf apps
Getting apps in org tmaki / space development as ****@gmail.com...
OK

name          requested state   instances   memory   disk   urls   
hello-tmaki   started           1/1         1G       1G     hello-tmaki.cfapps.io
```


`cf delete hello-<STUDENT_ID>`する前に、もし新バージョン(green)で問題が発覚すれば、旧バージョン(blue)に切り戻しすればよいです。

``` console
$ cf map-route hello-<STUDENT_ID> apps.pcflab.jp -n hello-<STUDENT_ID>
$ cf unmap-route hello-<STUDENT_ID>-green apps.pcflab.jp -n hello-<STUDENT_ID>
```

を行えば`V1`に戻ります。

**ここまで完了したら進捗シートにチェックをしてください。**
