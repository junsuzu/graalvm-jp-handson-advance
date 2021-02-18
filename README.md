# Oracle GraalVM Enterprise ハンズオン演習 (Advance編)

## ＜目的と対象＞：
このハンズオン演習は、下記Oracle GraalVM Enterpriseハンズオン演習のアドバンス編になります。  
[Oracle GraalVM Enterprise ハンズオン演習 (Basic編)](https://github.com/junsuzu/graalvm-jp-handson-basic/)

Basic編では、次世代Polyglot(多言語プログラミング）対応実行環境であるOracle GraalVM Enterprise版の導入と操作手順を学びましたが、この演習ではGraalVMをCloud Native環境で応用するためのより高度な内容となります。演習を通して以下の項目をマスターすることを目的としています。  
* GraalVMのNative Image機能でMicroservicesを作る
* GraalVMのPGOチューニング
* GraalVM  

このハンズオン演習の対象は上記Basic編を習得済みであることは望ましいが、必須ではありません。  

※この内容はOracle主催のOracle GraalVM Enterprise Virtual Hands-On Lab(Advance)の演習部分にあたります。  
参加者はこちらの内容に沿って事前環境セットアップおよび当日演習を実施して頂けます。また単独でGraalVMのアドバンス編演習としてもご利用頂けます。  
<br/>

## ＜前提環境／事前準備＞
* ハンズオンの内容はWindows10でWSL(Windows Subsystem for Linux)を利用し、LinuxディストリビューションのUbuntu20.04をインストールした環境を前提に進みます。  
* Ubuntu以外のLinuxおよびMacOS、Windowsもサポートされます。
* Docker Client: Docker Engine Community 20.10.1 
* Docker Server: Docker Engine Community 20.10.2 (Docker Desktop) 
* IntelliJ IDEA Community 2020.3

※「Oracle GraalVM Enterprise Virtual Hands-On Lab」の参加者は基本的に事前セットアップ済みの環境でハンズオン演習を実施して頂きます。ただし、演習が不要な方は、演習部分を視聴のみして頂くことも可能です。  
<br/>


## ＜演習内容＞

* **[演習 1: GraalVMとMicronautによるマイクロサービス作成](#演習-1-GraalVMとMicronautによるマイクロサービス作成)**
   * [1.1: Micronautアプリケーションの作成と起動](#11-Micronautアプリケーションの作成と起動)
   * [1.2: GraalVMを使用してNative Imageを作成](#12-GraalVMを使用してNative-Imageを作成)
   * [1.3: GraalVMとDockerでNative Imageを作成](#13-GraalVMとDockerでNative-Imageを作成)
   

* **[演習 2: GraalVMとSpringBootによるマイクロサービス作成](#演習-2-GraalVMとSpringBootによるマイクロサービス作成)**
   * [2.1: SpringフレームワークでRESTfulのWebサービスを作成](#21-SpringフレームワークでRESTfulのWebサービスを作成)
   * [2.2: Springアプリケーションからnative imageを生成](#22-Springアプリケーションからnative-imageを生成)
   * [2.3: native imageをベースにDockerコンテナを生成](#23-native-imageをベースにDockerコンテナを生成)
<br/>
<br/>


# 演習-1-GraalVMとMicronautによるマイクロサービス作成

この演習では、以下の内容を実施します。  
* Micronautアプリケーションの導入と稼働確認
* GraalVMでMicronautアプリのnative imageの作成と稼働確認
* native imageをベースにDockerイメージを作成し、Dockerコンテナによるマイクロサービスの稼働を確認

この演習を実施するためには、以下のソフトを導入します。  
* Micronaut ([SDKmanによるインストール](https://micronaut.io/download.html))
* GraalVM EE 20.1.0 ([ハンズオン演習 Basic編参照](https://github.com/junsuzu/graalvm-jp-handson-basic/))
* Docker CE([Install Docker Engine on Ubuntu](https://docs.docker.com/engine/install/ubuntu/))
* Docker DeskTop([Install Docker Engine on Ubuntu](https://docs.docker.com/engine/install/ubuntu/))

</br>

# 1.1-Micronautアプリケーションの作成と起動

(1) Micronautアプリケーションの作成  

任意のディレクトリーを作成し、その配下にMicronaut CLI(Command Line Interface)を利用してMicronautプロジェクトexample.micronaut.completeを作成します。
  >```sh
  >$ mn create-app example.micronaut.complete
  >```

![Download Picture 4](images/GraalVMadvance04.JPG)

上記コマンドより、completeフォルダが作成されていることをご確認ください。
<br/>

(2)IntelliJ IDEAから上記(1)で作成したアプリケーションをカスタマイズし、Micronautコントローラーを追加します。  
IntelliJ IDEAからcompleteフォルダーを開き、src>main>java>example.microanutパッケージの配下に新規JAVAソースファイルHelloController.javaを作成します。HelloControllerはHTTPリクエストに対して、"Hello World"という文字列をリターンします。

![Download Picture 5](images/GraalVMadvance05.JPG)

既存のApplicationおよび新規作成のHelloControllerのソースはそれぞれ下記をご確認ください。IntelliJ IDEA上ソースを作成したら、Ctrl+Sで保存します。
  
src/main/java/example/micronaut/Application.java
```java
package example.micronaut;

import io.micronaut.runtime.Micronaut;

public class Application {

    public static void main(String[] args) {
        Micronaut.run(Application.class);
    }
}
```  
src/main/java/example/micronaut/HelloController.java
```java
package example.micronaut;

import io.micronaut.http.MediaType;
import io.micronaut.http.annotation.Controller;
import io.micronaut.http.annotation.Get;
import io.micronaut.http.annotation.Produces;

@Controller("/hello")  //①
public class HelloController {
    @Get //②
    @Produces(MediaType.TEXT_PLAIN)  //③
    public String index() {
        return "Hello World"; //④
    }
}
```
* ① @Controller アノテーションがコントローラーを定義し、/helloというリクエスト・パスに対応します。  
* ② @Get アノテーションは下記index メソッドをすべてのHTTP Getリクエストに対応するようにマッピングします。
* ③ デフォルトではMicronautアプリのリスポンスのContentTypeはapplicaiton/jasonです。ここではJSONオブジェクトの代わりにStringをリターンしますので、text/plain を明示的に指定します。
* ④　"Hello World"　をリターンします。

(4)IntelliJ IDEA上Javaソースを保存したら、UbuntuターミナルからMicronautアプリケーションをbuildした後、起動します。以降のコマンドはすべてcomplete配下で実行してください。

  >```sh
  >$ ./gradlew run -t
  >```
出力結果をご確認ください。

  >```
  >linuser@JUNSUZU-JP:~/work/complete$ java -jar ./build/libs/complete-0.1-all.jar
>15:55:55.420 [main] INFO  io.micronaut.runtime.Micronaut - Startup completed in 5915ms. Server Running: http://localhost:8080
  >```
Microanutアプリケーションの起動時間をメモっておきます。

別ターミナルを立ち上げ、アプリケーションにアクセスしてみましょう。"Hello World"がレスポンスとして表示されることを確認します。

```
$ curl http://localhost:8080/hello
Hello World
```
アプリケーションをCtrl+Cで停止します。  
<br/>

# 1.2: GraalVMを使用してNative Imageを作成

(1) Basic編演習で導入したGraalVMのバージョンを再確認します。(GraalVMのバージョンは20.1.0以上であれば問題ありません。)

  >```sh
  >$ java -version  
  >```

  >```sh
  >java version "1.8.0_251"
  >Java(TM) SE Runtime Environment (build 1.8.0_251-b08)
  >Java HotSpot(TM) 64-Bit Server VM GraalVM EE 20.1.0 (build 25.251-b08-jvmci-20.1-b02, mixed mode)  
  >
<br/>

(2)GraalVMを使用し、MicronautサンプルアプリケーションのNative Imageを作成します。  


  >```sh
  >$ ./gradlew nativeImage  
  >```
<br/>
mavenでビルドする場合、下記コマンドを使用します。

  >```sh
  >$ ./mvn package -Dpackaging=native-image  
  >```
<br/>

環境によってNative Imageビルドに少し時間がかかります。下図のようにビルド成功のメッセージを確認します。  

![Download Picture 1](images/GraalVMadvance01.JPG)

gradleを利用しビルドした結果、build/native-image/配下にapplicationという名前のNative Imageが作成されていることが確認できます。  
mavenを利用した場合、./target配下となります。  

(3)作成したMicronautアプリケーションのNative Imageを動かしてみましょう。  
* gradleの場合
  >```sh
  >$ ./build/native-image/application  
  >```
* mavenの場合
  >```sh
  >$ ./target/application  
  >```

アプリケーション起動した結果、8080番ポートでMicronautアプリケーションのサービスが短い時間で起動していることが確認できます。
```
$ ./build/native-image/application
13:22:50.338 [main] INFO  io.micronaut.runtime.Micronaut - Startup completed in 571ms. Server Running: http://localhost:8080
```
native imageの起動時間と上記演習1.1で通常のJavaアプリケーションの起動時間と比較し、native image起動の速さを確認します。  

別ターミナルを立ち上げ、起動中のサービスに対してリクエストを送ってみます。レスポンスのHello Worldが表示されることを確認します。
  >```sh
  >$ curl localhost:8080/hello  
  >Hello World
  >```
<br/>

アプリケーションをCtrl+Cで停止します。  
<br/>

# 1.3: GraalVMとDockerでNative Imageを作成

(1)Dockerデーモンを起動します。本演習の環境ではWindowsのDocker DesktopをDockerデーモンとして2375番ポートでオープンします。Ubuntu側のDockerクライアントはそのデーモンに接続します。下記はDocker Desktopのsetting画面です。

![Download Picture 2](images/GraalVMadvance02.JPG)

(2)complete配下のbuild.gradleを修正し、下記定義を追加し、ファイルを保存します。  

  >```sh
  >  dockerfileNative {
  >  baseImage = "gcr.io/distroless/cc-debian10"
  >  }
  >
  >  nativeImage {
  >  args("-H:+StaticExecutableWithDynamicLibC")
  >  }
  >```

(3)Docker内でnative imageを作成します。complete配下で以下のコマンドを実行します。

  >```sh
  >$ ./gradlew dockerBuildNative
  >```
<br/>
以下のようにDockerのビルドが正常に終了していることを確認します。

```
$ ./gradlew dockerBuildNative

> Task :dockerfileNative
Dockerfile written to: /home/linuser/work/complete/build/docker/DockerfileNative

> Task :dockerBuildNative
Building image using context '/home/linuser/work/complete'.
Using Dockerfile '/home/linuser/work/complete/build/docker/DockerfileNative'
Using images 'complete'.
Step 1/10 : FROM ghcr.io/graalvm/graalvm-ce:java8-21.0.0 AS graalvm
 ---> b16c825a27d5
Step 2/10 : RUN gu install native-image
 ---> Running in 12b7541f509d
Downloading: Component catalog from www.graalvm.org
Processing Component: Native Image
Downloading: Component native-image: Native Image  from github.com
Installing new component: Native Image (org.graalvm.native-image, version 21.0.0)
Refreshed alternative links in /usr/bin/
Removing intermediate container 12b7541f509d
 ---> fbb638ac0eec
Step 3/10 : WORKDIR /home/app
 ---> Running in 6d6e83ccde66
Removing intermediate container 6d6e83ccde66
 ---> 026389920310
Step 4/10 : COPY build/layers/libs /home/app/libs
 ---> f578c3bf7b3c
Step 5/10 : COPY build/layers/resources /home/app/resources
 ---> 6357d0efca7d
Step 6/10 : COPY build/layers/application.jar /home/app/application.jar
 ---> 8bea9004a758
Step 7/10 : RUN native-image -H:+StaticExecutableWithDynamicLibC -H:Class=example.micronaut.Application -H:Name=application --no-fallback -cp /home/app/libs/*.jar:/home/app/resources:/home/app/application.jar
 ---> Running in ea875cb029c9
[application:20]    classlist:   7,182.37 ms,  1.54 GB
[application:20]        (cap):     881.12 ms,  2.15 GB
[application:20]        setup:   3,000.94 ms,  2.15 GB
[application:20]     (clinit):   1,006.71 ms,  2.53 GB
[application:20]   (typeflow):  41,681.63 ms,  2.53 GB
[application:20]    (objects):  27,847.59 ms,  2.53 GB
[application:20]   (features):   2,351.08 ms,  2.53 GB
[application:20]     analysis:  75,210.73 ms,  2.53 GB
[application:20]     universe:   2,866.95 ms,  2.53 GB
[application:20]      (parse):  11,273.21 ms,  2.35 GB
[application:20]     (inline):  15,611.99 ms,  2.75 GB
[application:20]    (compile):  66,151.20 ms,  2.90 GB
[application:20]      compile:  95,776.74 ms,  2.90 GB
[application:20]        image:   8,211.37 ms,  2.77 GB
[application:20]        write:   2,427.36 ms,  2.77 GB
[application:20]      [total]: 195,062.43 ms,  2.77 GB
Removing intermediate container ea875cb029c9
 ---> f5a92c52b9a2
Step 8/10 : FROM gcr.io/distroless/cc-debian10
 ---> fd0fdc7125b0
Step 9/10 : COPY --from=graalvm /home/app/application /app/application
 ---> d47cfb74a005
Step 10/10 : ENTRYPOINT ["/app/application"]
 ---> Running in 8e58749cb38f
Removing intermediate container 8e58749cb38f
 ---> 6ffb95f82e79
Successfully built 6ffb95f82e79
Successfully tagged complete:latest
Created image with ID '6ffb95f82e79'.

BUILD SUCCESSFUL in 3m 53s
6 actionable tasks: 2 executed, 4 up-to-date

```

(3)Dockerコンテナを起動します。
```
$ docker run -p 8080:8080 complete
 __  __ _                                  _
|  \/  (_) ___ _ __ ___  _ __   __ _ _   _| |_
| |\/| | |/ __| '__/ _ \| '_ \ / _` | | | | __|
| |  | | | (__| | | (_) | | | | (_| | |_| | |_
|_|  |_|_|\___|_|  \___/|_| |_|\__,_|\__,_|\__|
  Micronaut (v2.3.0)

02:21:32.936 [main] INFO  io.micronaut.runtime.Micronaut - Startup completed in 173ms. Server Running: http://d447523ef3ea:8080
```
Dockerコンテナーの起動時間と上記演習1.1、1.2の結果と比較します。  

別ターミナルを立ち上げ、起動中のDockerサービスに対してリクエストを送ってみます。レスポンスのHello Worldが表示されることを確認します。
  >```sh
  >$ curl localhost:8080/hello  
  >Hello World
  >```
<br/>

Docker コンテナとDocker Imageを確認できます。
```
$ docker ps -l
CONTAINER ID   IMAGE      COMMAND              CREATED         STATUS                            PORTS     NAMES
d447523ef3ea   complete   "/app/application"   8 minutes ago   Exited (130) About a minute ago             hopeful_lehmann
```
```
$ docker images
REPOSITORY                      TAG            IMAGE ID       CREATED          SIZE
complete                        latest         6ffb95f82e79   11 minutes ago   78.5MB
<none>                          <none>         f5a92c52b9a2   11 minutes ago   1.39GB
ghcr.io/graalvm/graalvm-ce      java8-21.0.0   b16c825a27d5   10 days ago      1.29GB
frolvlad/alpine-glibc           alpine-3.12    d955957758ab   3 weeks ago      17.9MB
hello-world                     latest         bf756fb1ae65   13 months ago    13.3kB
gcr.io/distroless/cc-debian10   latest         fd0fdc7125b0   51 years ago     21.2MB
```

# 演習 2: GraalVMとSpringBootによるマイクロサービス作成
この演習では、以下の内容を実施します。  
* Springフレームワークを利用し、RESTfulのWebサービスを作成
* GraalVMでSpringBootアプリのnative imageの作成と稼働確認
* native imageをベースにDockerイメージを作成し、Dockerコンテナによるマイクロサービスの稼働を確認

</br>

# 2.1 SpringフレームワークでRESTfulのWebサービスを作成

この演習の中でHTTPリクエストに対しJSONオブジェクト（Hello World !）をリターンする簡単なSpringアプリケーションを作成します。通常Springアプリケーションの作成はSpring Initializrの利用をお勧めですが、この演習ではサンプルソースコードをダウンロードし、カスタマイズするアプローチで進めます。

(1) Springサンプルソースのダウンロード  

Githubよりソースをダウンロードします。ダウンロード後completeディレクトリー配下に移動します。以降のコマンド実行はすべてcomplete配下で行います。

  >```sh
  >git clone https://github.com/spring-guides/gs-rest-service
  >cd gs-rest-service/complete
  >```

(2)IntelliJ IDEAなどのIDEを使ってcompleteフォルダーを開き、中にあるJavaソースやpom.xmlを編集します。（もしくは直接エディターで編集）  
* Plain Java Objectクラスを作成(確認)します。src/main/java/com/example/restservice/Greeting.java　　
```java
package com.example.restservice;

public class Greeting {

	private final long id;
	private final String content;

	public Greeting(long id, String content) {
		this.id = id;
		this.content = content;
	}

	public long getId() {
		return id;
	}

	public String getContent() {
		return content;
	}
}
```
* HTTPリクエストをハンドリングするResource Controllerを作成（確認）します。（オプション：レスポンスの文字列を適宜に変更します。）src/main/java/com/example/restservice/GreetingController.java
```java
package com.example.restservice;

import java.util.concurrent.atomic.AtomicLong;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class GreetingController {

	private static final String template = "Hello, %s!";
	private final AtomicLong counter = new AtomicLong();

	@GetMapping("/greeting")
	public Greeting greeting(@RequestParam(value = "name", defaultValue = "World") String name) {
		return new Greeting(counter.incrementAndGet(), String.format(template, name));
	}
}
```
(3)Mavenを使い実行可能なJARファイルにビルドし、実行します。
 >```sh
 >./mvnw clean package
 >```
```
[INFO]
[INFO] Results:
[INFO]
[INFO] Tests run: 2, Failures: 0, Errors: 0, Skipped: 0
[INFO]
[INFO]
[INFO] --- maven-jar-plugin:3.2.0:jar (default-jar) @ rest-service ---
[INFO] Building jar: /home/linuser/work2/gs-rest-service/complete/target/rest-service-0.0.1-SNAPSHOT.jar
[INFO]
[INFO] --- spring-boot-maven-plugin:2.4.2:repackage (repackage) @ rest-service ---
[INFO] Replacing main artifact with repackaged archive
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  42.415 s
[INFO] Finished at: 2021-02-18T12:23:28+09:00
[INFO] ------------------------------------------------------------------------
linuser@JUNSUZU-JP:~/work2/gs-rest-service/complete$
```

正常ビルド完了後、JARファイルを実行します。Web Serviceの起動時間を確認します。
>```sh
 >java -jar target/rest-service-0.0.1-SNAPSHOT.jar
 >```

```
2021-02-18 12:26:24.325  INFO 1376 --- [           main] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2021-02-18 12:26:24.328  INFO 1376 --- [           main] w.s.c.ServletWebServerApplicationContext : Root WebApplicationContext: initialization completed in 3107 ms
2021-02-18 12:26:24.903  INFO 1376 --- [           main] o.s.s.concurrent.ThreadPoolTaskExecutor  : Initializing ExecutorService 'applicationTaskExecutor'
2021-02-18 12:26:25.555  INFO 1376 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
2021-02-18 12:26:25.600  INFO 1376 --- [           main] c.e.restservice.RestServiceApplication   : Started RestServiceApplication in 5.848 seconds (JVM running for 7.438)
```
(4)別ターミナルを立ち上げ、以下のコマンドを実行し、HTTPリクエストからレスポンスが正常にリターンされることを確認します。
```sh
linuser@JUNSUZU-JP:~$ curl http://localhost:8080/greeting
{"id":1,"content":"Hello, World!"}
```
# 2.2 Springアプリケーションからnative imageを生成
GraalVMからnative-image-maven-pluginが提供され、mavenコマンドによってSpringアプリケーションをnative imageにビルドすることが可能です。

(1)pom.xmlを編集します。  

profileタグの中にGraalVM提供のnative-image-maven-pluginおよびSpring提供のspring-boot-maven-plugin両方を指定します。以下の内容をpom.xmlに追加します。  
(※-Dspring.native.remove-yaml-support=true と -Dspring.spel.ignore=true はフットプリントを削減するためのパラメータです。）

 ```
<profiles>
  <profile>
    <id>native</id>
    <build>
      <plugins>
        <plugin>
          <groupId>org.graalvm.nativeimage</groupId>
          <artifactId>native-image-maven-plugin</artifactId>
          <version>20.3.0</version>
          <configuration>
            <mainClass>com.example.restservice.RestServiceApplication</mainClass>
            <buildArgs>-Dspring.native.remove-yaml-support=true -Dspring.spel.ignore=true</buildArgs>
          </configuration>
          <executions>
            <execution>
              <goals>
                <goal>native-image</goal>
              </goals>
              <phase>package</phase>
            </execution>
          </executions>
        </plugin>
        <plugin>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
      </plugins>
    </build>
  </profile>
</profiles>
```
さらに、dependencyタグにSpring提供のspring-graalvm-native を追加します。
```
<dependencies>
    <!-- ...... -->
    <dependency>
        <groupId>org.springframework.experimental</groupId>
        <artifactId>spring-graalvm-native</artifactId>
        <version>0.8.5</version>
    </dependency>
</dependencies>
```
また、以下のリポジトリー定義を追加します。
```
<repositories>
    <repository>
        <id>spring-milestone</id>
        <name>Spring milestone</name>
        <url>https://repo.spring.io/milestone</url>
    </repository>
</repositories>
```
```
<pluginRepositories>
    <pluginRepository>
        <id>spring-milestone</id>
        <name>Spring milestone</name>
        <url>https://repo.spring.io/milestone</url>
    </pluginRepository>
</pluginRepositories>
```
propertiesタグにテストをスキップするよう定義します。
```
	<properties>
		<java.version>1.8</java.version>
		<skipTests>true</skipTests>
	</properties>
```
(2)GreetingController.javaを適宜編集し、レスポンスでリターンされる文字列を変更します。例：  
```
defaultValue = "World with Native Image"
```
(3)以下のコマンドでSpringアプリケーションをnative imageへビルドします。  
```
mvn -Pnative clean package
```
※-Pnativeを指定することにより、profileタグ内の定義が有効になりなす。  
native imageが正常にビルドされることを確認します。
```
Number of types dynamically registered for reflective access: #1452
[com.example.restservice.restserviceapplication:1934]     (clinit):     930.18 ms,  3.56 GB
[com.example.restservice.restserviceapplication:1934]   (typeflow):  21,747.29 ms,  3.56 GB
[com.example.restservice.restserviceapplication:1934]    (objects):  24,425.65 ms,  3.56 GB
[com.example.restservice.restserviceapplication:1934]   (features):   1,747.57 ms,  3.56 GB
[com.example.restservice.restserviceapplication:1934]     analysis:  50,500.21 ms,  3.56 GB
[com.example.restservice.restserviceapplication:1934]     universe:   2,914.62 ms,  3.56 GB
[com.example.restservice.restserviceapplication:1934]      (parse):  26,267.71 ms,  4.76 GB
[com.example.restservice.restserviceapplication:1934]     (inline):   6,773.49 ms,  5.20 GB
[com.example.restservice.restserviceapplication:1934]    (compile): 127,892.37 ms,  6.67 GB
[com.example.restservice.restserviceapplication:1934]      compile: 164,133.60 ms,  6.67 GB
[com.example.restservice.restserviceapplication:1934]        image:   4,841.53 ms,  6.67 GB
[com.example.restservice.restserviceapplication:1934]        write:   3,685.89 ms,  6.67 GB
[com.example.restservice.restserviceapplication:1934]      [total]: 247,545.92 ms,  6.67 GB
[INFO]
[INFO] --- spring-boot-maven-plugin:2.4.2:repackage (repackage) @ rest-service ---
[INFO] Replacing main artifact with repackaged archive
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  04:18 min
[INFO] Finished at: 2021-02-18T14:41:39+09:00
[INFO] ------------------------------------------------------------------------
linuser@JUNSUZU-JP:~/work2/gs-rest-service/complete$
```
target配下にnative image "com.example.restservice.restserviceapplication"が生成されたことを確認します。

(4)以下のコマンドでnative imageを実行します。
```
target/com.example.restservice.restserviceapplication
```
Springアプリケーションの起動時間を確認し、演習2.1の結果と比較します。
```
INFO: Initializing Spring embedded WebApplicationContext
2021-02-18 14:47:38.331  INFO 2161 --- [           main] w.s.c.ServletWebServerApplicationContext : Root WebApplicationContext: initialization completed in 36 ms
2021-02-18 14:47:38.342  INFO 2161 --- [           main] o.s.s.concurrent.ThreadPoolTaskExecutor  : Initializing ExecutorService 'applicationTaskExecutor'
Feb 18, 2021 2:47:38 PM org.apache.coyote.AbstractProtocol start
INFO: Starting ProtocolHandler ["http-nio-8080"]
2021-02-18 14:47:38.507  INFO 2161 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
2021-02-18 14:47:38.508  INFO 2161 --- [           main] c.e.restservice.RestServiceApplication   : Started RestServiceApplication in 0.234 seconds (JVM running for 0.236)
```
(5)別ターミナルを立ち上げ、以下のコマンドを実行し、HTTPリクエストからレスポンスが正常にリターンされることを確認します。
```sh
$ curl http://localhost:8080/greeting
{"id":1,"content":"Hello, World with Native Image!"}
```
# 2.3 native imageをベースにDockerコンテナを生成
Spring BootがCloud Native Buildpackを提供し、MavenおよびGradleプラグインからdockerイメージを直接ビルドする機能をサポートします。
この機能により、Dockerfileなしに、簡単なコマンドおよびプラグイン定義の編集だけで、容易にdockerイメージをビルド可能です。  

(1)演習2.2のpom.xmlをさらに編集を加えます。  
Buildpackを利用するため、buildタグの内容が以下になるように編集します。  
Springから提供されるspring-boot-maven-pluginおよび使用するBuildpackのイメージ(paketobuildpacks/builder:tiny)を指定します。また、BP_BOOT_NATIVE_IMAGE 環境変数の値をtrueに指定します。
```
  <build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
				<configuration>
					<image>
						<builder>paketobuildpacks/builder:tiny</builder>
						<env>
							<BP_BOOT_NATIVE_IMAGE>true</BP_BOOT_NATIVE_IMAGE>
							<BP_BOOT_NATIVE_IMAGE_BUILD_ARGUMENTS>
								-Dspring.native.remove-yaml-support=true
								-Dspring.spel.ignore=true
							</BP_BOOT_NATIVE_IMAGE_BUILD_ARGUMENTS>
						</env>
					</image>
				</configuration>
			</plugin>
		</plugins>
	</build>
```
(2)native imageを含むdockerイメージをビルドします。  
```
mvn spring-boot:build-image
```




<br/>

お疲れ様でした！
<br/>
ここまでは、Oracle GraalVM Enterprise ハンズオン演習 (Basic編)の内容はすべて終了しました。この演習では以下の項目について学びました。
 
* GraalVMの導入
* GraalVM JITコンパイラでJavaクラスを実行
* GraalVM AOTコンパイラでNative Imageの生成と実行
* Polyglot 多言語プログラミングと実行 

