## 簡単なアプリケーションをデプロイ (Java編)

[Spring Boot](http://projects.spring.io/spring-boot/)を使ってとても簡単なWebアプリケーションを作成 & デプロイしましょう。

### プロジェクトの作成

アプリケーションを`git clone`します。
任意のディレクトリで以下のコマンドを実行してください。

``` console
$ mkdir pcf-workshop
$ cd pcf-workshop
$ git clone https://github.com/tkaburagi/hello-cf
$ cd hello-cf
```

クローンしたプロジェクトのソースを少しだけ修正します。任意のエディタ、IDEで
`src/main/java/com/example/HelloCfApplication.java`を開いてください。

``` java
package com.example;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
// ここから追加
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;
// ここまで

@SpringBootApplication
@RestController // 追加
public class HelloCfApplication {

        // ここから追加
        @GetMapping("/") 
        String hello() {
                return "Hello World!";
        }
        // ここまで

        public static void main(String[] args) {
                SpringApplication.run(HelloCfApplication.class, args);
        }
}
```

ソースコードを修正したら、アプリケーションをビルドします。

``` console
$ cd hello-cf
$ ./mvnw package -Dmaven.test.skip=true
```

> HTTP Proxyがある場合、`.mvn/jvm.config`に次の設定を行ってください
> 
> ```
> -Dhttp.proxyHost=proxy.example.com 
> -Dhttp.proxyPort=8080
> -Dhttps.proxyHost=proxy.example.com 
> -Dhttps.proxyPort=8080 
> -Dhttp.proxyUser=username 
> -Dhttp.proxyPassword=password

まずはローカルでアプリケーションを実行してみましょう。

``` console
$ java -jar target/hello-cf-0.0.1-SNAPSHOT.jar
```

[http://localhost:8080](http://localhost:8080)にアクセスしてください。

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/02020313-0016-ba4e-e2bd-f4ddd5db40f0.png)

Hello World!が表示されれば成功です。

**ここまで完了したら進捗シートにチェックをしてください。**

### アプリケーションをPivotal Cloud FoundryにPush

ビルドしたアプリケーションをPivotal Cloud FoundryにPushしましょう。
`cf`コマンドを使えばとても簡単です。以下のコマンドを実行してください。

``` console
$ cf push hello-<STUDENT_ID> -p target/hello-cf-0.0.1-SNAPSHOT.jar
```

`<STUDENT_ID>`は配布されたIDに置換してください。


``` console
$ cf push hello-<STUDENT_ID> -p target/hello-cf-0.0.1-SNAPSHOT.jar
Creating app hello-tmaki in org tmaki / space development as ****@gmail.com...
OK

Creating route hello-tmaki.cfapps.io...
OK

Binding hello-tmaki.cfapps.io to hello-tmaki...
OK

Uploading hello-tmaki...
Uploading app files from: target/hello-cf-0.0.1-SNAPSHOT.jar
Uploading 490.1K, 88 files
Done uploading               
OK

Starting app hello-tmaki in org tmaki / space development as ****@gmail.com...
Downloading nodejs_buildpack...
Downloading staticfile_buildpack...
Downloading ruby_buildpack...
Downloading java_buildpack...
Downloading go_buildpack...
Downloading python_buildpack...
Downloading php_buildpack...
Downloading liberty_buildpack...
Downloading binary_buildpack...
Downloaded staticfile_buildpack
Downloaded go_buildpack
Downloaded binary_buildpack
Downloaded nodejs_buildpack
Downloaded java_buildpack
Downloaded ruby_buildpack
Downloaded liberty_buildpack
Downloaded php_buildpack
Downloaded python_buildpack
Creating container
Successfully created container
Downloading app package...
Downloaded app package (11.4M)
Staging...
-----> Java Buildpack Version: v3.6 | https://github.com/cloudfoundry/java-buildpack.git#5194155
-----> Downloading Open Jdk JRE 1.8.0_73 from https://download.run.pivotal.io/openjdk/trusty/x86_64/openjdk-1.8.0_73.tar.gz (1.4s)
       Expanding Open Jdk JRE to .java-buildpack/open_jdk_jre (1.0s)
-----> Downloading Open JDK Like Memory Calculator 2.0.1_RELEASE from https://download.run.pivotal.io/memory-calculator/trusty/x86_64/memory-calculator-2.0.1_RELEASE.tar.gz (0.1s)
       Memory Settings: -Xms768M -Xss1M -Xmx768M -XX:MaxMetaspaceSize=104857K -XX:MetaspaceSize=104857K
-----> Downloading Spring Auto Reconfiguration 1.10.0_RELEASE from https://download.run.pivotal.io/auto-reconfiguration/auto-reconfiguration-1.10.0_RELEASE.jar (0.2s)
Exit status 0
Staging complete
Uploading droplet, build artifacts cache...
Uploading droplet...
Uploading build artifacts cache...
Uploaded build artifacts cache (44.7M)
Uploaded droplet (56.4M)
Uploading complete

0 of 1 instances running, 1 starting
0 of 1 instances running, 1 starting
0 of 1 instances running, 1 starting
0 of 1 instances running, 1 starting
0 of 1 instances running, 1 starting
0 of 1 instances running, 1 starting
0 of 1 instances running, 1 starting
0 of 1 instances running, 1 starting
0 of 1 instances running, 1 starting
0 of 1 instances running, 1 starting
1 of 1 instances running

App started


OK

App hello-tmaki was started using this command `CALCULATED_MEMORY=$($PWD/.java-buildpack/open_jdk_jre/bin/java-buildpack-memory-calculator-2.0.1_RELEASE -memorySizes=metaspace:64m.. -memoryWeights=heap:75,metaspace:10,native:10,stack:5 -memoryInitials=heap:100%,metaspace:100% -totMemory=$MEMORY_LIMIT) && JAVA_OPTS="-Djava.io.tmpdir=$TMPDIR -XX:OnOutOfMemoryError=$PWD/.java-buildpack/open_jdk_jre/bin/killjava.sh $CALCULATED_MEMORY" && SERVER_PORT=$PORT eval exec $PWD/.java-buildpack/open_jdk_jre/bin/java $JAVA_OPTS -cp $PWD/.:$PWD/.java-buildpack/spring_auto_reconfiguration/spring_auto_reconfiguration-1.10.0_RELEASE.jar org.springframework.boot.loader.JarLauncher`

Showing health and status for app hello-tmaki in org tmaki / space development as ****@gmail.com...
OK

requested state: started
instances: 1/1
usage: 1G x 1 instances
urls: hello-tmaki.cfapps.io
last uploaded: Thu Mar 17 07:34:03 UTC 2016
stack: cflinuxfs2
buildpack: java-buildpack=v3.6-https://github.com/cloudfoundry/java-buildpack.git#5194155 java-main open-jdk-like-jre=1.8.0_73 open-jdk-like-memory-calculator=2.0.1_RELEASE spring-auto-reconfiguration=1.10.0_RELEASE

     state     since                    cpu     memory         disk           details   
#0   running   2016-03-17 04:35:50 PM   10.3%   117.9M of 1G   135.8M of 1G
````

これでデプロイに成功しました。
`cf apps`でデプロイされているアプリケーションの一覧を取得できます。

``` console
$ cf apps
Getting apps in org tmaki / space development as ****@gmail.com...
OK

name          requested state   instances   memory   disk   urls   
hello-tmaki   started           1/1         1G       1G     hello-tmaki.cfapps.io <---
```

`urls`の列に出力されている値がアプリケーションのURLです。この場合は[http://hello-tmaki.apps.pcflab.jp](http://hello-tmaki.apps.pcflab.jp)です。

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/79860de1-4846-c922-8583-787e77a185d2.png)

Pivotal Cloud Foundry上にデプロイされたアプリケーションにもアクセスできました。

[http://hello-<STUDENT_ID>.apps.pcflab.jp/env](http://hello-tmaki.cfapps.io/env)にアクセスすると環境変数やプロパティを確認できます。


**ここまで完了したら進捗シートにチェックをしてください。**

Spring Boot 1.5では`/env`をはじめとするSpring Boot Actuatorエンドポイントの多くがデフォルトに認可制御されるようになりました。
そのため、`/env`にアクセスしても401エラーが返ります。

この認可制御を無効にするにはプロパティ`management.security.enabled`を`false`に設定する必要があります。
Cloud Foundryでは`cf set-env`で環境変数を設定でき、環境変数でこのプロパティを設定可能です。
 
```
cf set-env hello-<STUDENT_ID> management.security.enabled false
cf restart hello-<STUDENT_ID>
```
もちろん、プロダクション用時には適切な認可設定が必要です。


![image](https://qiita-image-store.s3.amazonaws.com/0/1852/dfafa4f4-0ec1-e568-ba5e-3d5cfaa799a5.png)

[Pivotal Cloud Foundry Apps Manager](https://login.sys.pcflab.jp/login)を見てみましょう。
![image](https://qiita-image-store.s3.amazonaws.com/0/1852/4fd1565a-9bad-63a2-9382-6e7f59363b10.png)

「Space development」をクリックしてください。`development`というスペースにデプロイされているアプリケーションの一覧を確認できます。`hello-<STUDENT_ID>`が表示されています。

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/a0684f30-3757-2cfe-0580-c19b3598a499.png)

`hello-<STUDENT_ID>`をクリックしてください。

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/58280398-6b1b-1d41-7581-62ba76f24d06.png)

アプリケーションの情報が確認できます。

直近のログは`cf logs <App> --recent`で確認できます。

``` console
$ cf logs hello-<STUDENT_ID> --recent
Connected, dumping recent logs for app hello-tmaki in org tmaki / space development as ****@gmail.com...

2016-03-23T14:35:03.24+0900 [STG/0]      OUT Downloading ruby_buildpack...
...
2016-03-23T14:35:03.60+0900 [STG/0]      OUT Creating container
2016-03-23T14:35:04.27+0900 [STG/0]      OUT Successfully created container
2016-03-23T14:35:04.27+0900 [STG/0]      OUT Downloading app package...
2016-03-23T14:35:05.14+0900 [STG/0]      OUT Downloaded app package (11.8M)
2016-03-23T14:35:05.14+0900 [STG/0]      OUT Downloading build artifacts cache...
2016-03-23T14:35:06.87+0900 [STG/0]      OUT Downloaded build artifacts cache (44.7M)
2016-03-23T14:35:06.87+0900 [STG/0]      OUT Staging...
2016-03-23T14:35:07.52+0900 [APP/0]      OUT 2016-03-23 05:35:07.500  INFO 14 --- [       Thread-3] ationConfigEmbeddedWebApplicationContext : Closing org.springframework.boot.context.embedded.AnnotationConfigEmbeddedWebApplicationContext@5cdaddaf: startup date [Fri Mar 18 19:57:58 UTC 2016]; root of context hierarchy
2016-03-23T14:35:07.59+0900 [APP/0]      OUT 2016-03-23 05:35:07.590  INFO 14 --- [       Thread-3] o.s.c.support.DefaultLifecycleProcessor  : Stopping beans in phase 0
2016-03-23T14:35:07.68+0900 [APP/0]      OUT 2016-03-23 05:35:07.685  INFO 14 --- [       Thread-3] o.s.j.e.a.AnnotationMBeanExporter        : Unregistering JMX-exposed beans on shutdown
2016-03-23T14:35:07.94+0900 [APP/0]      OUT Exit status 143
2016-03-23T14:35:08.46+0900 [STG/0]      OUT -----> Java Buildpack Version: v3.6 | https://github.com/cloudfoundry/java-buildpack.git#5194155
2016-03-23T14:35:08.64+0900 [STG/0]      OUT -----> Downloading Open Jdk JRE 1.8.0_73 from https://download.run.pivotal.io/openjdk/trusty/x86_64/openjdk-1.8.0_73.tar.gz (found in cache)
2016-03-23T14:35:09.70+0900 [STG/0]      OUT        Expanding Open Jdk JRE to .java-buildpack/open_jdk_jre (1.0s)
2016-03-23T14:35:09.72+0900 [STG/0]      OUT -----> Downloading Open JDK Like Memory Calculator 2.0.1_RELEASE from https://download.run.pivotal.io/memory-calculator/trusty/x86_64/memory-calculator-2.0.1_RELEASE.tar.gz (found in cache)
2016-03-23T14:35:09.75+0900 [STG/0]      OUT        Memory Settings: -Xms768M -XX:MetaspaceSize=104857K -Xss1M -Xmx768M -XX:MaxMetaspaceSize=104857K
2016-03-23T14:35:09.76+0900 [STG/0]      OUT -----> Downloading Spring Auto Reconfiguration 1.10.0_RELEASE from https://download.run.pivotal.io/auto-reconfiguration/auto-reconfiguration-1.10.0_RELEASE.jar (found in cache)
2016-03-23T14:35:18.57+0900 [STG/0]      OUT Exit status 0
2016-03-23T14:35:18.57+0900 [STG/0]      OUT Staging complete
2016-03-23T14:35:18.57+0900 [STG/0]      OUT Uploading droplet, build artifacts cache...
2016-03-23T14:35:18.57+0900 [STG/0]      OUT Uploading droplet...
2016-03-23T14:35:18.57+0900 [STG/0]      OUT Uploading build artifacts cache...
2016-03-23T14:35:19.79+0900 [STG/0]      OUT Uploaded build artifacts cache (44.7M)
2016-03-23T14:35:41.65+0900 [STG/0]      OUT Uploaded droplet (56.7M)
2016-03-23T14:35:41.66+0900 [STG/0]      OUT Uploading complete
2016-03-23T14:35:42.65+0900 [CELL/0]     OUT Creating container
2016-03-23T14:35:43.22+0900 [CELL/0]     OUT Successfully created container
2016-03-23T14:35:46.20+0900 [CELL/0]     OUT Starting health monitoring of container
2016-03-23T14:35:47.88+0900 [APP/0]      OUT   .   ____          _            __ _ _
2016-03-23T14:35:47.88+0900 [APP/0]      OUT  /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
2016-03-23T14:35:47.88+0900 [APP/0]      OUT ( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
2016-03-23T14:35:47.88+0900 [APP/0]      OUT  \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
2016-03-23T14:35:47.88+0900 [APP/0]      OUT   '  |____| .__|_| |_|_| |_\__, | / / / /
2016-03-23T14:35:47.88+0900 [APP/0]      OUT  =========|_|==============|___/=/_/_/_/
2016-03-23T14:35:47.89+0900 [APP/0]      OUT  :: Spring Boot ::        (v1.3.3.RELEASE)
2016-03-23T14:35:48.08+0900 [APP/0]      OUT 2016-03-23 05:35:48.081  INFO 17 --- [           main] pertySourceApplicationContextInitializer : Adding 'cloud' PropertySource to ApplicationContext
2016-03-23T14:35:48.13+0900 [APP/0]      OUT 2016-03-23 05:35:48.138  INFO 17 --- [           main] nfigurationApplicationContextInitializer : Adding cloud service auto-reconfiguration to ApplicationContext
2016-03-23T14:35:48.15+0900 [APP/0]      OUT 2016-03-23 05:35:48.152  INFO 17 --- [           main] com.example.HelloCfApplication           : Starting HelloCfApplication on f8e414fd07r with PID 17 (/home/vcap/app started by vcap in /home/vcap/app)
2016-03-23T14:35:48.15+0900 [APP/0]      OUT 2016-03-23 05:35:48.152  INFO 17 --- [           main] com.example.HelloCfApplication           : The following profiles are active: cloud
2016-03-23T14:35:48.21+0900 [APP/0]      OUT 2016-03-23 05:35:48.212  INFO 17 --- [           main] ationConfigEmbeddedWebApplicationContext : Refreshing org.springframework.boot.context.embedded.AnnotationConfigEmbeddedWebApplicationContext@e17c606: startup date [Wed Mar 23 05:35:48 UTC 2016]; root of context hierarchy
...
2016-03-23T14:35:52.78+0900 [APP/0]      OUT 2016-03-23 05:35:52.783  INFO 17 --- [           main] s.b.c.e.t.TomcatEmbeddedServletContainer : Tomcat started on port(s): 8080 (http)
2016-03-23T14:35:52.79+0900 [APP/0]      OUT 2016-03-23 05:35:52.790  INFO 17 --- [           main] com.example.HelloCfApplication           : Started HelloCfApplication in 5.935 seconds (JVM running for 6.547)
2016-03-23T14:35:53.24+0900 [CELL/0]     OUT Container became healthy

```

また`cf logs <App>`で今後流れるログを確認することができます(`tail -f`相当)。

### アプリケーションの削除

`cf delete`でアプリケーションを削除できます。

``` console
$ cf delete hello-<STUDENT_ID>

Really delete the app hello-tmaki?> y
Deleting app hello-tmaki in org tmaki / space development as ****@gmail.com...
OK
```

### Buildpackを指定する

`cf push`でアプリケーションをアップロードした後、ステージングとよばれるフェーズでランタイム(JREやサーバーなど)を追加し実行可能なDropletという形式を作成します。Dropletを作るためのBuildpackとよばれる仕組みが使われます。

アップロードしたファイル群(アーティファクト)から自動で適用すべきBuildpackが判断され、Cloud Foundryに他言語対応はここで行われています。

利用可能なBuildpack一覧は`cf buildpacks`で取得できます。

``` console
$ cf buildpacks
Getting buildpacks...

buildpack                    position   enabled   locked   filename
staticfile_buildpack         1          true      false    staticfile_buildpack-cached-v1.4.12.zip
java_buildpack               2          true      false    java-buildpack-offline-v3.18.zip
ruby_buildpack               3          true      false    ruby_buildpack-cached-v1.6.46.zip
nodejs_buildpack             4          true      false    nodejs_buildpack-cached-v1.6.5.zip
go_buildpack                 5          true      false    go_buildpack-cached-v1.8.6.zip
python_buildpack             6          true      false    python_buildpack-cached-v1.5.22.zip
php_buildpack                7          true      false    php_buildpack-cached-v4.3.39.zip
dotnet_core_buildpack        8          true      false    dotnet-core_buildpack-cached-v1.0.23.zip
dotnet_core_buildpack_beta   9          true      false    dotnet-core_buildpack-cached-v1.0.0.zip
binary_buildpack             10         true      false    binary_buildpack-cached-v1.0.14.zip
```

デフォルトでは、`cf push`でアーティファクトをアップロードした後、利用可能なBuildpackを全てダウンロードし、優先順(`position`順)にチェックし、対象のBuildpackを特定しDroplet(実行可能な形式)を作成します。

今回の場合は、jarファイルからjava_buildpackが検知され、かつjarの内部に`lib/spring-boot-.*.jar`が存在することからSpring Boot用のDropletが作成されます。

Buildpackは`-b`で明示的に指定できます。明示することで自動検出のための時間を短縮できます。

``` console
$ cf push hello-<STUDENT_ID> -p target/hello-cf-0.0.1-SNAPSHOT.jar -b java_buildpack_offline
Updating app hello-tmaki in org tmaki / space development as ****@gmail.com...
OK

Uploading hello...
Uploading app files from: target/hello-cf-0.0.1-SNAPSHOT.jar
Uploading 490.1K, 88 files
Done uploading               
OK

Stopping app hello in org tmaki / space development as ****@gmail.com...
OK

Starting app hello in org tmaki / space development as ****@gmail.com...
Downloading java_buildpack...
Downloaded java_buildpack
Creating container
Successfully created container
Downloading app package...
Downloaded app package (11.4M)
Downloading build artifacts cache...
Downloaded build artifacts cache (44.7M)
Staging...
-----> Java Buildpack Version: v3.6 | https://github.com/cloudfoundry/java-buildpack.git#5194155
-----> Downloading Open Jdk JRE 1.8.0_73 from https://download.run.pivotal.io/openjdk/trusty/x86_64/openjdk-1.8.0_73.tar.gz (found in cache)
       Expanding Open Jdk JRE to .java-buildpack/open_jdk_jre (1.0s)
-----> Downloading Open JDK Like Memory Calculator 2.0.1_RELEASE from https://download.run.pivotal.io/memory-calculator/trusty/x86_64/memory-calculator-2.0.1_RELEASE.tar.gz (found in cache)
       Memory Settings: -Xmx768M -XX:MaxMetaspaceSize=104857K -Xss1M -Xms768M -XX:MetaspaceSize=104857K
-----> Downloading Spring Auto Reconfiguration 1.10.0_RELEASE from https://download.run.pivotal.io/auto-reconfiguration/auto-reconfiguration-1.10.0_RELEASE.jar (found in cache)
Exit status 0
Staging complete
Uploading droplet, build artifacts cache...
Uploading droplet...
Uploading build artifacts cache...
Uploaded build artifacts cache (44.7M)
Uploaded droplet (56.4M)
Uploading complete

0 of 1 instances running, 1 starting
1 of 1 instances running

App started


OK

App hello was started using this command `CALCULATED_MEMORY=$($PWD/.java-buildpack/open_jdk_jre/bin/java-buildpack-memory-calculator-2.0.1_RELEASE -memorySizes=metaspace:64m.. -memoryWeights=heap:75,metaspace:10,native:10,stack:5 -memoryInitials=heap:100%,metaspace:100% -totMemory=$MEMORY_LIMIT) && JAVA_OPTS="-Djava.io.tmpdir=$TMPDIR -XX:OnOutOfMemoryError=$PWD/.java-buildpack/open_jdk_jre/bin/killjava.sh $CALCULATED_MEMORY" && SERVER_PORT=$PORT eval exec $PWD/.java-buildpack/open_jdk_jre/bin/java $JAVA_OPTS -cp $PWD/.:$PWD/.java-buildpack/spring_auto_reconfiguration/spring_auto_reconfiguration-1.10.0_RELEASE.jar org.springframework.boot.loader.JarLauncher`

Showing health and status for app hello in org tmaki / space development as ****@gmail.com...
OK

requested state: started
instances: 1/1
usage: 1G x 1 instances
urls: hello-tmaki.cfapps.io
last uploaded: Thu Mar 17 08:23:17 UTC 2016
stack: cflinuxfs2
buildpack: java_buildpack

     state     since                    cpu    memory         disk           details   
#0   running   2016-03-17 05:24:00 PM   0.0%   421.8M of 1G   135.8M of 1G
```

### Manifestファイルを作成

ここまで`cf`コマンドで指定してきたオプションは`manifest.yml`というyamlファイルに定義できます。

* `cf push hello-<STUDENT_ID> -p target/hello-cf-0.0.1-SNAPSHOT.jar -b java_buildpack`
* `cf set-env hello-<STUDENT_ID> management.security.enabled false`

を`manifest.yml`で表すと、

``` yaml
---
applications:
  - name: hello-<STUDENT_ID>
    path: target/hello-cf-0.0.1-SNAPSHOT.jar
    buildpack: java_buildpack_offline
    env:
      management.security.enabled: false
```

となります。
hello-cfのディレクトリにいることを確認し、上記のyamlを作成してください。
```console
$ vi manifest.yml
```

このManifestファイルがあれば実行コマンドは`cf push`だけで良いです。

``` console
$ cf push
Using manifest file /Users/makit/git/hello-cf/manifest.yml
(以下、略)
```

**ここまで完了したら進捗シートにチェックをしてください。**
