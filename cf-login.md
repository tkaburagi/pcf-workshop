# PCF環境へのログイン

Pivotal Cloud Foundryの環境へログインします。

* APIエンドポイント: api.sys.pcflab.jp
* ユーザ名: 講師より配布
* パスワード: 講師より配布

``` console
cf login -a api.sys.pcflab.jp --skip-ssl-validation
```

ログインに成功したら下記のコマンドで確認し、正しく表示されればログイン成功です。

``` console
cf target
```

今回は"Hands-on-*"というorganizationと"development", "staging", "production"というspaceを利用します。
