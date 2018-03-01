# Concourseによる継続的デリバリ

今回は下記の環境のConcourseサーバーを利用します。下記にアクセスできることは確認してください。
http://ci.pcflab.jp/

## バージョン1のアプリケーションのデプロイ
(バージョン1のアプリケーション)[https://github.com/tkaburagi1214/pcf-workshop/blob/master/demo-bug-1.0.0-BUILD-SNAPSHOT.jar]を`cf push`します。
``` console
cf push pcfapp-<STUDENT_ID> -p demo-bug-1.0.0-BUILD-SNAPSHOT.jar 
```
!()[https://github.com/tkaburagi1214/pcf-workshop/blob/master/image/Screen%20Shot%200030-03-01%20at%2012.00.52%20PM.png]

## Concourseパイプラインの作成

## アプリケーションの修正

## Concourseパイプラインの実行

## Concourseパイプラインの修正 (単体テスト実行の追加)
