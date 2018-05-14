# PCF環境へのログイン

Pivotal Cloud Foundryの環境へログインします。

* APIエンドポイント: api.sys.pcflab.jp
* ユーザ名: 講師より配布
* パスワード: 講師より配布 (Ry7CtDzF)

自分のPCにメモしてください。
転記後に削除します。
このパスワードは本日中のみ有効です。

``` console
cf login -a api.sys.pcflab.jp  -o handson-<STUDENT_ID> -s development --skip-ssl-validation
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

今回は"handson-<STUDENT_ID>"というorganizationと"development"というspaceを利用します。


**ここまで完了したら進捗シートにチェックをしてください。**
