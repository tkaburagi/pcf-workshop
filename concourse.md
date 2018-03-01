# Concourseによる継続的デリバリ

今回は下記の環境のConcourseサーバーを利用します。下記にアクセスできることは確認してください。
http://ci.pcflab.jp/

## バージョン1のアプリケーションのデプロイ
[バージョン1のアプリケーション](https://github.com/tkaburagi1214/pcf-workshop/blob/master/demo-bug-1.0.0-BUILD-SNAPSHOT.jar)を`cf push`します。
``` console
cf push pcfapp-<STUDENT_ID> -p demo-bug-1.0.0-BUILD-SNAPSHOT.jar 
```
![](https://github.com/tkaburagi1214/pcf-workshop/blob/master/image/Screen%20Shot%200030-03-01%20at%2012.00.52%20PM.png)

## Concourseパイプラインの作成

`fly`コマンドを使ってConcourseにログインします。
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



## アプリケーションの修正

## Concourseパイプラインの実行

## Concourseパイプラインの修正 (単体テスト実行の追加)
