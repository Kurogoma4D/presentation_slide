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
layout: center
---

# バックグラウンド実行を理解すると
<Gap :size="60"/>

<div class="flex space-x-4">
  <span class="p-8 border-green-600 border-4 bg-green-400 text-gray-900 rounded-2xl">プラグインが作れる</span>
  <span class="p-8 border-green-600 border-4 bg-green-400 text-gray-900 rounded-2xl">巷のプラグインの理解が進む</span>
  <span class="p-8 border-red-600 border-4 bg-red-400 text-gray-900 rounded-2xl">たのしい</span>
</div>

---

# バックグラウンド実行を理解するのに必要な概念

1. Flutterの仕組み
   - Flutterはどのように起動するのか
2. Isolate
   - Isolateの概念
   - サンプルコード
3. プラットフォームとDartコードの連携

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
layout: two-cols
---

# Isolate

並列処理ができる仕組み

```dart{all|13|16-20}
import 'dart:isolate';

/// Root Isolate
void main() {
  final receivePort = ReceivePort();
  final sendPort    = receivePort.sendPort;

  receivePort.listen((message) {
    print(message);
    receivePort.close();
  });

  await Isolate.spawn(anotherMain, sendPort);
}

/// Child Isolate
/// top-level function or static method
void anotherMain() {
  print('Hello from anotherMain!');
}
```

::right::

<Gap :size="80" />

- `main()` もIsolate
  - Root Isolate
- **Isolate間でメモリ共有はできない**
  - 通信手段はある
- Child Isolateとして実行できる関数
  - トップレベル関数
  - 任意のクラスの static メソッド
  - つまり**メモリから見て静的な関数**

詳しくは [monoさんの記事](https://medium.com/flutter-jp/isolate-a3f6eab488b5) がわかりやすいです

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