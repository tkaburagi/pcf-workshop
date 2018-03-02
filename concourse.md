# Concourseによる継続的デリバリ

今回は下記の環境のConcourseサーバーを利用します。下記にアクセスできることは確認してください。
http://ci.pcflab.jp/

## バージョン1のアプリケーションのデプロイ

[pcf-workshop-app](https://github.com/tkaburagi1214/pcf-workshop-app)のレポジトリをforkします。
下記画像の右上のボタンです。

![](https://github.com/tkaburagi1214/pcf-workshop/blob/master/image/Screen%20Shot%200030-03-02%20at%2010.05.01%20AM.png)


[バージョン1のアプリケーション](https://github.com/tkaburagi1214/pcf-workshop/blob/master/demo-bug-1.0.0-BUILD-SNAPSHOT.jar)を`cf push`します。
``` console
cf push pcfapp-<STUDENT_ID> -p demo-bug-1.0.0-BUILD-SNAPSHOT.jar 
```
![](https://github.com/tkaburagi1214/pcf-workshop/blob/master/image/Screen%20Shot%200030-03-01%20at%2012.00.52%20PM.png)

## Concourseパイプラインの作成

`fly`コマンドを使ってConcourseにログインします。パスワードとユーザネームは事前に配布されます。
``` console
fly login -t ci -c http://35.201.185.239 -n handson
```

CDのパイプラインを記述します。ConcourseではパイプラインはすべてYamlで定義します。
`pipeline.yml`というファイルを作成して下記をコピペしてください。`<カッコ内>`の値は自分の環境に合わせて置き換えてください。

``` yaml
---
resources:
  - name: pcfapp
    type: git
    source:
      uri: <GIT_URI>
      branch: <GIT_BRANCH>
    check_every: 10s
  - name: cf
    type: cf
    source:
      api: api.sys.pcflab.jp
      username: <STUDENT_ID>
      password: <YOUR_PASSWD>
      organization: <ORG_ID>
      space: <SPACE_ID>
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
      - name: repo
      caches:
      - path: repo/m2   
      run:
        path: bash
        args:
        - -c
        - |
          set -e
          cd repo
          rm -rf ~/.m2
          ln -fs $(pwd)/m2 ~/.m2
          mvn test
```

上記のファイルをConcourseサーバにアップロードすることでパイプラインが生成されます。

``` console
fly -t ci set-pipeline -p simple-pipeline -c pipeline.yml
```

アップロードが成功したらWeb GUIで確認してみます。
ブラウザで`http://ci.pcflab.jp/`へアクセスします。配布されたパスワードとユーザ名を使ってログインしてください。

Web GUIからパイプラインを実行します。

パイプラインが緑色に変化すれば処理が成功しています。

## アプリケーションの修正
次にアプリケーションを修正し、gitにコミット、Concourseからその更新情報を取得し、アプリケーションをテストし、アップグレードという流れを実施します。

まずはConcourseのパイプラインにアプリケーションをビルドとアップグレードするための処理を追加します。
``` yaml
---
resources:
  - name: pcfapp
    type: git
    source:
      uri: <GIT_URI>
      branch: <GIT_BRANCH>
    check_every: 10s
  - name: cf
    type: cf
    source:
      api: api.sys.pcflab.jp
      username: <STUDENT_ID>
      password: <YOUR_PASSWD>
      organization: <ORG_ID>
      space: <SPACE_ID>
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
      - name: repo
      caches:
      - path: repo/m2   
      run:
        path: bash
        args:
        - -c
        - |
          set -e
          cd repo
          rm -rf ~/.m2
          ln -fs $(pwd)/m2 ~/.m2
          mvn test
  - name: build-and-artifact
    serial_groups:[ version ]
    plan:
      - get: pcfdapp
        passed: [ unit-test ]
        trigger: true
      - task: build
        file: pcfdemoapp/ci/tasks/build.yml

```

アプリケーションのソースコードを下記のように変更してください。
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

## Concourseパイプラインの実行

## Concourseパイプラインの修正 (単体テスト実行の追加)
