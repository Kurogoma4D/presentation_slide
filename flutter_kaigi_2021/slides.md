---
theme: ../simple_dark_theme
highlighter: shiki
# show line numbers in code blocks
lineNumbers: false
info: |
  ## Slidev Starter Template
  Presentation slides for developers.

  Learn more at [Sli.dev](https://sli.dev)
drawings:
  persist: false
---

# FlutterのバックグラウンドDartコード実行を理解する

The magic of combination of dart and platform.

---

# 誰？

<AvaterRow />

- <span class="text-green-300">Cross-platform Mobile App Engineer</span> on CBcloud株式会社
  - We are hiring!!!
- Twitter: @Krgm4D
- https://scrapbox.io/kurogoma4d-lab/

<Gap :size="80" />

あんまり役に立たない開発が好きです

---

# なんか導入

バックグラウンド実行を理解するのに必要な概念
- Dart VMの起動
- Isolate
- プラットフォームとDartコードの連携

---
layout: center
---

# Flutterの仕組み

---

# Flutterの仕組み

<img class="mx-auto my-0" width="450" alt="architecture" src="/images/archdiagram.png" />

https://flutter.dev/docs/resources/architectural-overview#architectural-layers

---

# Flutterの仕組み

既存のネイティブアプリにFlutterを組み込むAdd-to-App

```kotlin{all|5-12}
class MyApplication : Application() {
  lateinit var flutterEngine : FlutterEngine
  override fun onCreate() {
    super.onCreate()
    // Instantiate a FlutterEngine.
    flutterEngine = FlutterEngine(this)
    // Configure an initial route.
    flutterEngine.navigationChannel.setInitialRoute("your/route/here");
    // Start executing Dart code to pre-warm the FlutterEngine.
    flutterEngine.dartExecutor.executeDartEntrypoint(
      DartExecutor.DartEntrypoint.createDefault()
    )
    // Cache the FlutterEngine to be used by FlutterActivity or FlutterFragment.
    FlutterEngineCache
      .getInstance()
      .put("my_engine_id", flutterEngine)
  }
}
```

[Androidでのコード例](https://flutter.dev/docs/development/add-to-app/android/add-flutter-screen#initial-route-with-a-cached-engine)

---
layout: two-cols
---
# Flutterの仕組み

<div class="mt-32">Embedderに用意された</div>
<p>Flutterを起動するためのクラス</p>

<Gap :size="16"/>

<a style="word-wrap: break-word;" href="https://api.flutter.dev/javadoc/io/flutter/embedding/engine/dart/DartExecutor.html#executeDartCallback-io.flutter.embedding.engine.dart.DartExecutor.DartCallback-">https://api.flutter.dev/javadoc/io/flutter/embedding/engine/dart/DartExecutor.html#executeDartCallback-io.flutter.embedding.engine.dart.DartExecutor.DartCallback-</a>

::right::

<img style="margin: auto 0 auto 32px; object-fit: contain; height: 100%;" alt="Dart Executer" src="/images/dart_executer.png" />

---
layout: center
---

# Isolate

---


# Isolate

- 簡単なサンプル
- Isolateとは
  - 並列処理ができる
  - （ただし厳密にプラットフォームの別スレッドを使うわけではない）
  - mainとのメモリ共有はできない
  - 感覚としてはスレッドだがどちらかというとプロセスっぽい
  - 重要なこと
    - main isolateとそれ以外のisolateがある
    - isolateのエントリーポイントはtop-level functionかstatic method

---
layout: center
---

# ネイティブとの連携

---

# ネイティブとの連携

- MethodChannel
- Dart → Platform
- Platform → Dart

---

# バックグラウンド実行

- 図を出す
- コードを追って説明