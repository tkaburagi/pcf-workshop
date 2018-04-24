# Concourseによる継続的デリバリ

今回は下記の環境のConcourseサーバーを利用します。下記にアクセスできることは確認してください。
http://ci.pcflab.jp/

ログインができたら該当するOSのアイコンを押下し、flyコマンドをダウンロードします。
ダウンロードをしたら自身のOSに合わせて実行権限を付与し、パスを通します。
以下はMacの例です。
```console
chmod +x path/to/fly
mv path/to/fly /usr/local/bin
```
新しい端末を立ち上げ以下を実行しflyがインストールされていることを確認して下さい。

```console
fly --version
3.9.2
```
**ここまで完了したら進捗シートにチェックをしてください。**


## バージョン1のアプリケーションのデプロイ

[pcf-workshop-app](https://github.com/tkaburagi1214/pcf-workshop-app)のレポジトリをforkします。
下記画像の右上のボタンです。

![](https://github.com/tkaburagi1214/pcf-workshop/blob/master/image/Screen%20Shot%200030-03-02%20at%2010.05.01%20AM.png)

forkしたらローカルにクローンします。

``` console
git clone <YOUR_REPO_URL>
```

[バージョン1のアプリケーション](https://github.com/tkaburagi/pcf-workshop/blob/master/snapshots/demo-bug-1.0.0-BUILD-SNAPSHOT.jar)をダウンロードし、`cf push`します。`path/to`の部分はダウンロードをした自身のパスに置き換えてください。
``` console
cf push pcfapp-<STUDENT_ID> -p path/to/demo-bug-1.0.0-BUILD-SNAPSHOT.jar --hostname pcfapp-<STUDENT_ID> 
```

pushが成功したらアプリの動作を確認してみます。
``` console
curl http://pcfapp-<STUDENT_ID>.apps.pcflab.jp
```

![](https://github.com/tkaburagi1214/pcf-workshop/blob/master/image/Screen%20Shot%200030-03-01%20at%2012.00.52%20PM.png)

PVOTALとなっておりおかしいのでこれをPIVOTALと正しく表示されるようにConcourseを使って変更します。

**ここまで完了したら進捗シートにチェックをしてください。**

## Concourseパイプラインの作成

`fly`コマンドを使ってConcourseにログインします。パスワードとユーザネームは事前に配布されます。
``` console
fly login -t ci -c http://ci.pcflab.jp -n handson-<NUMBER>
```

CDのパイプラインを記述します。ConcourseではパイプラインはすべてYamlで定義します。
レポジトリ内に`ci`というディレクトリを作成し、`pipeline.yml`というファイルを作成して下記をコピペしてください。`<カッコ内>`の値は自分の環境に合わせて置き換えてください。`<GIT_URI>`は必ず先ほどFORKしたURIを指定してください。(FORK元のURI(https://github.com/tkaburagi1214/pcf-workshop-app)は指定しないでください。

```
cd pcf-workshop-app
mkdir ci
cd ci
```

``` yaml
---
resources:
  - name: pcfapp
    type: git
    source:
      uri: <GIT_URI>
      branch: master
    check_every: 10s
jobs:
- name: unit-test
  plan:
  - get: pcfapp
    trigger: true
  - task: mvn-test
    config:
      platform: linux
      image_resource:
        type: docker-image
        source:
          repository: maven
      inputs:
      - name: pcfapp
      caches:
      - path: pcfapp/m2   
      run:
        path: bash
        args:
        - -c
        - |
          set -e
          cd pcfapp
          rm -rf ~/.m2
          ln -fs $(pwd)/m2 ~/.m2
          mvn test
```

上記のファイルをConcourseサーバにアップロードすることでパイプラインが生成されます。

``` console
fly -t ci set-pipeline -p simple-pipeline -c pipeline.yml

```
![](https://github.com/tkaburagi1214/pcf-workshop/blob/master/image/Screen%20Shot%200030-03-02%20at%201.19.12%20PM.png)

アップロードが成功したらWeb GUIで確認してみます。
ブラウザで`http://ci.pcflab.jp/`へアクセスします。配布されたパスワードとユーザ名を使ってログインしてください。
![](https://github.com/tkaburagi1214/pcf-workshop/blob/master/image/Screen%20Shot%200030-03-02%20at%201.20.54%20PM.png)

CLIからパイプラインを実行します。

``` console
fly -t ci unpause-pipeline -p simple-pipeline
fly -t ci trigger-job -j simple-pipeline/unit-test
```

ブラウザに戻ってパイプラインを確認すると、パイプラインが実行状態に遷移ています。
![](https://github.com/tkaburagi1214/pcf-workshop/blob/master/image/ci-running.png)

ジョブが終了しパイプラインが緑色に変化すれば処理が成功しています。
![](https://github.com/tkaburagi1214/pcf-workshop/blob/master/image/ci-green.png)


パイプラインの`unit-test`のボックスをクリックすると実行内容のログが確認できます。
![](https://github.com/tkaburagi1214/pcf-workshop/blob/master/image/ci-log.png)

**ここまで完了したら進捗シートにチェックをしてください。**

## アプリケーションの修正
次にアプリケーションを修正し、gitにコミット、Concourseからその更新情報を取得し、アプリケーションをテストし、アップグレードという流れを実施します。

端末を開いて以下のコマンドを実行し、アプリケーションからのレスポンス内容を監視します。
``` console
while true;do curl -s https://pcfapp-<STUDENT_ID>.apps.pcflab.jp --insecure;echo;sleep 1;done
```

まずはConcourseのパイプラインにアプリケーションをビルドとアップグレードするための処理を追加します。
以下のテキストをコピーして`pipeline.yml`をすべて上書きします。

``` yaml
---
resources:
  - name: pcfapp
    type: git
    source:
      uri: <GIT_URI>
      branch: master
    check_every: 10s
  - name: deploy-to-cf
    type: cf
    source:
      api: api.sys.pcflab.jp
      username: <STUDENT_ID>
      password: <YOUR_PASSWD>
      organization: handson-<STUDENT_ID>
      space: development
      skip_cert_check: true
jobs:
- name: unit-test
  plan:
  - get: pcfapp
    trigger: true
  - task: mvn-test
    config:
      platform: linux
      image_resource:
        type: docker-image
        source:
          repository: maven
      inputs:
      - name: pcfapp
      caches:
      - path: pcfapp/m2   
      run:
        path: bash
        args:
        - -c
        - |
          set -e
          cd pcfapp
          rm -rf ~/.m2
          ln -fs $(pwd)/m2 ~/.m2
          mvn test
- name: build-and-deploy
  plan:
  - get: pcfapp
    passed: [ unit-test ]
    trigger: true
  - task: build
    config:
      platform: linux
      image_resource:
        type: docker-image
        source:
          repository: maven
      inputs:
      - name: pcfapp
      caches:
      - path: pcfapp/m2   
      run:
        path: bash
        args:
        - -c
        - |
          set -e
          cd pcfapp
          rm -rf ~/.m2
          ln -fs $(pwd)/m2 ~/.m2
          mvn clean package
  - put: deploy-to-cf
    params: 
      manifest: pcfapp/manifest.yml
      current_app_name: pcfapp-<STUDENT_ID>
```

Concourseのパイプラインを更新します。
``` console
fly -t ci set-pipeline -p simple-pipeline -c pipeline.yml
```

Webブラウザを更新して確認してください。パイプラインがアップデートされていることがわかります。
![](https://github.com/tkaburagi1214/pcf-workshop/blob/master/image/Screen%20Shot%200030-03-02%20at%201.40.26%20PM.png)


次にアプリケーションのソースコードを下記のように変更してください。
クローンしたレポジトリの下記のファイルです。
`src/main/java/com/example/demo/PcfdemoappApplication.java`

``` java
package com.example.demo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@SpringBootApplication
@RestController
public class PcfdemoappApplication {

	@RequestMapping("/")
	public String hello() {
		String java_ver = System.getProperty("java.vm.version");
		String app_ver = "3";
		String app_index = System.getenv("CF_INSTANCE_INDEX");
		String app_host = System.getenv("CF_INSTANCE_IP");
		return "MMMMMMMMMMMMMMMMMWMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMFMMMMMMM\n"+
				"M]                 ?WMMMMMM#     MMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMF    .M\n"+
				"M]    ........       .HMMMM#     MMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMF!`  .MMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMF    .M\n"+
				"M]    .MMMMMMMMNa.     MMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMM[    .MMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMF    .M\n"+
				"M]    .MMMMMMMMMMN     (MMMMHHHHHMMMMHHHHHHMMMMMMMMMMMMMMHHHHHMMMMMMMMMMMMHMMMMMMMMMMMMMM[    .HHHHHHHHMMMMMMMMMMYMMMMMMMHMMMMMMMMMF    .M\n"+
				"M]    .MMMMMMMMMMM{    .MMM#     MMMF      HMMMMMMMMMMMMF    .MMMMM#M`         .TMMMMMMMM[            .MMMMMM@M               ,MMMMF    .M\n"+
				"M]    .MMMMMMMMMMM`    .MMM#     MMMh..    .MMMMMMMMMMMM`   .MMMMB!      ...      ,HMMMMM[    .........MMMM@`     ........    .MMMMF    .M\n"+
				"M]    .MMMMMMMMMM^     JMMM#     MMMMMM]    4MMMMMMMMMM]    dMMMF    ..MMMMMMNJ     UMMMM[    .MMMMMMMMMMMF     .MMMMMMMMF    .MMMMF    .M\n"+
				"M]    .MMMMMMM=`      .MMMM#     MMMMMMM,    MMMMMMMMM#    .MMMF    .MMMMMMMMMMN.    MMMM[    .MMMMMMMMMMM`    MMMMMMMMMMF    .MMMMF    .M\n"+
				"M]    .MMM          .MMMMMM#     MMMMMMMN    (MMMMMMMM^   .MMMM%    dMMMMMMMMMMMb    (MMM[    .MMMMMMMMMM#    .MMMMMMMMMMF    .MMMMF    .M\n"+
				"M]    .MMM.  ....(MMMMMMMMM#     MMMMMMMM]    MMMMMMMF    JMMMM)    MMMMMMMMMMMM#    .MMM[    .MMMMMMMMMMF    (MMMMMMMMMMF    .MMMMF    .M\n"+
				"M]    .MMMMMMMMMMMMMMMMMMMM#     MMMMMMMMM,   .MMMMMM`   .MMMMM)    MMMMMMMMMMMM#    .MMM[    .MMMMMMMMMMF    (MMMMMMMMMMF    .MMMMF    .M\n"+
				"M]    .MMMMMMMMMMMMMMMMMMMM#     MMMMMMMMMb    dMMMMt   .MMMMMM)    MMMMMMMMMMMMF    -MMM[    .MMMMMMMMMM#    .MMMMMMMMMMF    .MMMMF    .M\n"+
				"M]    .MMMMMMMMMMMMMMMMMMMM#     MMMMMMMMMM[   .MMM#    (MMMMMMb    -MMMMMMMMMMM^    dMMM[    .MMMMMMMMMMM,    UMMMMMMMMMF    .MMMMF    .M\n"+
				"M]    .MMMMMMMMMMMMMMMMMMMM#     MMMMMMMMMMN.   (MM'   .MMMMMMMM,    -MMMMMMMM#'    .MMMM]    .MMMMMMMMMMMN.    ?HMMMMMMMF    .MMMMF    .M\n"+
				"M]    .MMMMMMMMMMMMMMMMMMMM#     MMMMMMMMMMMb    7^    MMMMMMMMMMx      ?MMM^      .MMMMMb       `````(MMMMN,         JMMF    .MMMMF    .M\n"+
				"M]    .MMMMMMMMMMMMMMMMMMMM#     MMMMMMMMMMMMb       .dMMMMMMMMMMMNJ.           ..MMMMMMMMN.          .MMMMMMN,.      JMMF    .MMMMF    .M\n"+
				"MNMMMMNMMMMMMMMMMMMMMMMMMMMNMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMNMMMMMMMMMMMMMMMMMMMMMMMNMMMMMMMNMMMMMMMMMMMNNMMMMMNMMMMMMMMMNMMMMNM\n"
				+"\r\n"
				+"Current app version = " + app_ver
				+"\r\n"
				+"Current java version = " + java_ver
				+"\r\n"
				+"App index = " + app_index
				+"\r\n"
				+"App host = " + app_host;
	}
	
	
	public static void main(String[] args) {
		SpringApplication.run(PcfdemoappApplication.class, args);
	}
}
```

また、`manifest.yml`内の`<STUDENT_ID>`を置き換えてください。
``` yaml
---
applications:
- name: pcfapp-<STUDENT_ID>
  memory: 1G
  host: pcfapp-<STUDENT_ID>
  domain: apps.pcflab.jp
  instances: 1
  path: target/demo-1.0.0-BUILD-SNAPSHOT.jar
  env:
   management.security.enabled: false
   endpoints.shutdown.enabled: true
```

## Concourseパイプラインの実行
アプリケーションを修正したらGithubにコミットします。GitにコミットするとConcourseから更新情報が取得され自動的にパイプラインが稼働します。`pcf-workshop-app`ディレクトリにいることを確認してください。

``` console
$ pwd 
path/to/pcf-workshop-app
$ git add .
$ git commit -m "Added I"
$ git push origin master
```

Web GUIでConcourseが実行中のステータスに遷移していることを確認してください。10秒ほど経過すると実行されるはずです。
`unit-test`が完了すると`build-and-artifact`に遷移し、ビルドとデプロイが実行されます。

![](https://github.com/tkaburagi1214/pcf-workshop/blob/master/image/ci-running-2.png)

すべてのタスクが実行されると成功の状態に遷移します。

![](https://github.com/tkaburagi1214/pcf-workshop/blob/master/image/ci-green-2.png)

curlコマンドのコンソールを見るとエラーが発生せずアプリケーションのアップグレードが完了していることがわかります。

![](https://github.com/tkaburagi1214/pcf-workshop/blob/master/image/Screen%20Shot%200030-03-02%20at%202.44.23%20PM.png)

**ここまで完了したら進捗シートにチェックをしてください。**
