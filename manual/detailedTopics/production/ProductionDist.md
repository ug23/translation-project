<!--- Copyright (C) 2009-2013 Typesafe Inc. <http://www.typesafe.com> -->
<!--
# Creating a standalone version of your application
-->
# スタンドアロンで実行可能なアプリケーションのビルド

<!--
## Using the dist task
-->
## dist タスクを使う

The simplest way to deploy a Play application is to retrieve the source (typically via a git workflow) on the server and to use either `activator start` or `activator stage` to start it in place.

<!--
However, you sometimes need to build a binary version of your application and deploy it to the server without any dependency on Play itself. You can do this with the `dist` task.
-->
その他に、Play のフレームワークに依存しないアプリケーションのバイナリをビルドしたいこともあるでしょう。その場合は、 `dist` タスクが利用できます。

<!--
In the Play console, simply type `dist`:
-->
Play コンソールで、 `dist` を入力してみましょう。

```bash
[my-first-app] $ dist
```

[[images/dist.png]]

This produces a ZIP file containing all JAR files needed to run your application in the `target/universal` folder of your application. Alternatively you can run `activator dist` directly from your OS shell prompt, which does the same thing:

```bash
$ activator dist
```

<!--
> For Windows users a start script will be produced with a .bat file extension. Use this file when running a Play application on Windows.
-->
> Windows ユーザー向けの起動スクリプトは拡張子 .bat のファイルとして提供されています。Windows 上で Play アプリケーションを実行する場合は、このファイルを使用してください。
>
>
<!--
> For Unix users, zip files do not retain Unix file permissions so when the file is expanded the start script will be required to be set as an executable:
-->
Unix ユーザーのみなさん、zip ファイルは Unix ファイルパーミッションを保持しないので、ファイルを展開したらこの起動スクリプトに実行権限を付与する必要があるでしょう。
>
> ```bash
> $ chmod +x /path/to/bin/<project-name>
> ```
>
>
> ```bash
> $ chmod +x /path/to/bin/<project-name>
> ```
>
<!--
> Alternatively a tar.gz file can be produced instead. Tar files retain permissions. Invoke the `universal:package-zip-tarball` task instead of the `dist` task:
-->
> この代わりに tar.gz ファイルを作ることができます。このファイルはパーミッションを保持します。`dist` タスクの代わりに `universal:package-zip-tarball` タスクを実行してください:
>
> ```bash
> play universal:package-zip-tarball
> ```
>
> ```bash
> activator universal:package-zip-tarball
> ```

By default, the dist task will include the API documentation in the generated package. If this is not necessary, it can be avoided by including this line in `build.sbt`:

```scala
doc in Compile <<= target.map(_ / "none")
```
For builds with sub-projects, the statement above has to be applied to all sub-project definitions.

<!--
## The Native Packager
-->
## Native Packager

<!--
Play uses the [SBT Native Packager plugin](http://www.scala-sbt.org/sbt-native-packager/). The native packager plugin declares the `dist` task to create a zip file. Invoking the `dist` task is directly equivalent to invoking the following:
-->
Play は [SBT Native Packager プラグイン](http://www.scala-sbt.org/sbt-native-packager/) を使っています。この native packager プラグインは zip ファイルを作る `dist` タスクを宣言しています。`dist` タスクを実行することは、以下を実行することとまったく等価です:

```bash
$ activator universal:package-bin
```

<!--
Many other types of archive can be generated including:
-->
この他にも、以下を含む多くの種類のアーカイブを生成することができます。

* tar.gz
* OS X disk images
* Microsoft Installer (MSI)
* RPMs
* Debian packages
* System V / init.d and Upstart services in RPM/Debian packages

<!--
Please consult the [documentation](http://www.scala-sbt.org/sbt-native-packager) on the native packager for more information.
-->
もっと詳しい情報は、native packager の [ドキュメント](http://www.scala-sbt.org/sbt-native-packager) をご覧ください。

### Build a server distribution

The sbt-native-packager plugins provides and `java_server` archetype which enables the following features

* System V or Upstart startup scripts
* [Default folders](http://www.scala-sbt.org/sbt-native-packager/GettingStartedServers/MyFirstProject.html#default-mappings)

A full documentation can be found in the [documentation](http://www.scala-sbt.org/sbt-native-packager/GettingStartedServers/index.html).

The `java_server` archetype is enabled by default, but depending on which package you want to build you have to add a few settings. 

#### Minimal Debian settings

```scala
import com.typesafe.sbt.SbtNativePackager._
import NativePackagerKeys._

maintainer in Linux := "First Lastname <first.last@example.com>"

packageSummary in Linux := "My custom package summary"

packageDescription := "My longer package description"
```

Build your package with

```bash
play debian:packageBin
```

#### Minimal RPM settings

```scala
import com.typesafe.sbt.SbtNativePackager._
import NativePackagerKeys._

maintainer in Linux := "First Lastname <first.last@example.com>"

packageSummary in Linux := "My custom package summary"

packageDescription := "My longer package description"

rpmRelease := "1"

rpmVendor := "example.com"

rpmUrl := Some("http://github.com/example/server")

rpmLicense := Some("Apache v2")
```

```bash
play rpm:packageBin
```

> There will be some error logging. This is rpm logging on stderr instead of stdout !

#### Play PID Configuration 

Play manages its own PID, which is described in the [[Production configuration|ProductionConfiguration]].  In order to tell the startup script where to place the PID file put a file `etc-default` inside `src/templates/` folder and add the following content

```bash
-Dpidfile.path=/var/run/${{app_name}}/play.pid
# Add all other startup settings here, too
```

For a full list of replacements take a closer look at the [documentation](http://www.scala-sbt.org/sbt-native-packager/GettingStartedServers/AddingConfiguration.html).


<!--
## Publishing to a Maven (or Ivy) repository
-->
## Maven (または Ivy) レポジトリへパブリッシュする

<!--
You can also publish your application to a Maven repository. This publishes both the JAR file containing your application and the corresponding POM file.
-->
アプリケーションを Maven レポジトリへパブリッシュすることもできます。パブリッシュされるのは、アプリケーションの JAR と、POM ファイルの二つです。

<!--
You have to configure the repository you want to publish to, in your `build.sbt` file:
-->
`build.sbt` にパブリッシュしたいリポジトリを設定する必要があります:

```scala
 publishTo := Some(
   "My resolver" at "http://mycompany.com/repo"
 ),
 
 credentials += Credentials(
   "Repo", "http://mycompany.com/repo", "admin", "admin123"
 )
```

<!--
Then in the Play console, use the `publish` task:
-->
設定を記述したら、Play コンソールで `publish` タスクを実行します。

```bash
[my-first-app] $ publish
```

<!--
> Check the [sbt documentation](http://www.scala-sbt.org/release/docs/index.html) to get more information about the resolvers and credentials definition.
-->
リゾルバと認証情報の定義に関するさらに詳しい情報については [sbt ドキュメント](http://www.scala-sbt.org/release/docs/index.html) を確認してください。

## Using the SBT assembly plugin

Though not officially supported, the SBT assembly plugin may be used to package and run Play applications.  This will produce one jar as an output artifact, and allow you to execute it directly using the `java` command.

To use this, add a dependency on the plugin to your `project/plugins.sbt` file:

```scala
addSbtPlugin("com.eed3si9n" % "sbt-assembly" % "0.11.2")
```

Now add the following configuration to your `build.sbt`:

```scala
import AssemblyKeys._

assemblySettings

mainClass in assembly := Some("play.core.server.NettyServer")

fullClasspath in assembly += Attributed.blank(PlayKeys.playPackageAssets.value)
```

Now you can build the artifact by running `activator assembly`, and run your application by running:

```
$ java -jar target/scala-2.XX/<yourprojectname>-assembly-<version>.jar
```

You'll need to substitute in the right project name, version and scala version, of course.

<!--
> **Next:** [[Production configuration|ProductionConfiguration]]
-->
> **Next:** [[本番環境の設定|ProductionConfiguration]]
