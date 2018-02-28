# PCF環境へのログイン

Pivotal Cloud Foundryの環境へログインします。

* APIエンドポイント: api.sys.pcflab.jp
* ユーザ名: 講師より配布
* パスワード: 講師より配布

``` console
cf login -a api.sys.pcflab.jp  -o handson-<STUDENT_ID> -s development--skip-ssl-validation
```

ログインに成功したら下記のコマンドで確認し、正しく表示されればログイン成功です。

``` console
cf target
api endpoint:   https://api.sys.pcflab.jp
api version:    2.98.0
user:           admin
org:            handson-instructor
space:          development
```

今回は"Hands-on-*"というorganizationと"development", "staging", "production"というspaceを利用します。
