## Logging

デフォルトでは、ロギングの機能は、[clojure.tools.logging](https://github.com/clojure/tools.logging)ライブラリによって提供されます。この`clojure.tools.logging`ライブラリには、特定のロギング実装に処理を移譲するマクロがあります。Luminusで使用されるデフォルト実装は[log4j](https://logging.apache.org/log4j/2.x/)ライブラリです。

`clojure.tools.logging`には6つのログ・レベルがあり、Clojureのいかなるデータ構造でも直接ロギングすることができます。6つのログレベルは、`trace`、`debug`、`info`、`warn`、`error`、そして`fatal`です。

```clojure
(ns example
 (:require [clojure.tools.logging :as log]))

(log/info "Hello")
=>[2015-12-24 09:04:25,711][INFO][myapp.handler] Hello

(log/debug {:user {:id "Anonymous"}})
=>[2015-12-24 09:04:25,711][INFO][myapp.handler] {:user {:id "Anonymous"}}

(log/error (Exception. "I'm an error") "something bad happened")
=>[2015-12-24 09:43:47,193][ERROR][myapp.handler] something bad happened
  java.lang.Exception: I'm an error
    	at myapp.handler$init.invoke(handler.clj:21)
    	at myapp.core$start_http_server.invoke(core.clj:44)
    	at myapp.core$start_app.invoke(core.clj:61)
    	...
```

### Logging Configuration

デフォルトのロガーの設定は、`resources/log4j.properties`ファイルにあり、次のとおりです。

```
### Direct log4j properties to STDOUT ###
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.Target=System.out
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=[%d][%p][%c] %m%n

log4j.appender.R=org.apache.log4j.RollingFileAppender
log4j.appender.R.File=./log/<<name>>.log

log4j.appender.R.MaxFileSize=100KB
log4j.appender.R.MaxBackupIndex=20

log4j.appender.R.layout=org.apache.log4j.PatternLayout
log4j.appender.R.layout.ConversionPattern=[%d][%p][%c] %m%n

log4j.rootLogger=DEBUG, stdout, R
```

`LOG_CONFIG`環境変数にログ設定ファイルのパスをセットすれば、外部からログ設定を渡すことができます。例えば、`log4j-prod.properties`という本番環境用の設定ファイルを作成し、`/var/log/myapp.log`という場所にログを出力するといったこともできるでしょう。

```
log4j.appender.R=org.apache.log4j.RollingFileAppender
log4j.appender.R.File=/var/log/myapp.log

log4j.appender.R.MaxFileSize=100KB
log4j.appender.R.MaxBackupIndex=20

log4j.appender.R.layout=org.apache.log4j.PatternLayout
log4j.appender.R.layout.ConversionPattern=[%d][%p][%c] %m%n

log4j.rootLogger=INFO, R
```

このログ設定をアプリケーションに使用させるために、次のようなフラグを付けてアプリケーションを起動することができます。

```
java -Dlog_config="log4j-prod.properties" -jar myapp.jar
```
