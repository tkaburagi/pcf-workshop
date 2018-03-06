## 事前準備

macOS, Linuxのいずれかを推奨します。

### Java SE Development Kit 8

[Java SE Development Kit 8 Downloads](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)からJDKをインストールしてください。

環境変数`JAVA_HOME`にインストールしたディレクトリを設定してください。

Macユーザーは

``` console
export JAVA_HOME=`/usr/libexec/java_home`
```

で設定できます

### Curlコマンド

Macの場合はインストール不要です。

Windowsの場合は、[Git](https://git-scm.com/)をインストールすれば同梱されます。

### watchコマンド

watchコマンドをインストールしてください。
``` console
brew install watch
```

### Cloud Foundry CLI

* [Windows 64 bit](https://cli.run.pivotal.io/stable?release=windows64&source=pws)
* [Windows 32 bit](https://cli.run.pivotal.io/stable?release=windows32&source=pws)
* [Mac OSX 64 bit](https://cli.run.pivotal.io/stable?release=macosx64&source=pws)
* [Linux 64 bit (.deb)](https://cli.run.pivotal.io/stable?release=debian64&source=pws)
* [Linux 32 bit (.deb)](https://cli.run.pivotal.io/stable?release=debian32&source=pws)
* [Linux 64 bit (.rpm)](https://cli.run.pivotal.io/stable?release=redhat64&source=pws)
* [Linux 32 bit (.rpm)](https://cli.run.pivotal.io/stable?release=redhat32&source=pws)

からインストーラーをダウンロードして`cf`コマンドをインストールしてください。

インストール後、`cf`にパスが通っていることを確認してください。

``` console
$ cf -v
cf version 6.15.0+fa1bfe2-2016-01-13
```

### Githubアカウントの作成とgit cliのインストール
[GithubにSign-up](https://github.com/join?source=header-home)してください。アカウントの作成は無償です。
次に、git cliをインストールします。[こちらのガイド](https://git-scm.com/book/ja/v2/%E4%BD%BF%E3%81%84%E5%A7%8B%E3%82%81%E3%82%8B-Git%E3%81%AE%E3%82%A4%E3%83%B3%E3%82%B9%E3%83%88%E3%83%BC%E3%83%AB)に従って各ご利用のOSのcliをインストールしてください。
``` console
git version
git version 2.14.3 (Apple Git-98)
```
