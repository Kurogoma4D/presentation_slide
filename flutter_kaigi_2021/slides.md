---
theme: simple_dark_theme
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

# Dartコードのバックグラウンド実行

任意のコードをバックグラウンドで実行することができるパッケージの一例

- <span class="text-green-300">workmanager</span> https://pub.dev/packages/workmanager
  - Android WorkManager のラッパー
- <span class="text-green-300">background_locator</span> https://pub.dev/packages/background_locator
  - フォアグラウンド・バックグラウンドで位置情報の継続的な取得ができる
- <span class="text-green-300">flutter_isolate</span> https://pub.dev/packages/flutter_isolate
  - 後述する Isolate を実現する
  - Isolate 自体は Pure Dart でもできるが、普通に使うとプラットフォーム固有の処理をつかった plugin が使えない
    - このパッケージは plugin が使える Isolate を扱う

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
layout: two-cols
---

# Flutterの仕組み

<Gap :size="80"/>

Framework / Engine / Embedder の3層

今回出てくるAPIは主にEmbedder

::right::

<img class="mx-auto my-0" width="450" alt="architecture" src="/images/archdiagram.png" />

https://flutter.dev/docs/resources/architectural-overview#architectural-layers

---

# Flutterの仕組み

既存のネイティブアプリにFlutterを組み込むAdd-to-App ([Androidでのコード例](https://flutter.dev/docs/development/add-to-app/android/add-flutter-screen#initial-route-with-a-cached-engine))

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
  }
}
```

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

```dart{all|5|8-12}
import 'dart:isolate';

/// Root Isolate
void main() {
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
layout: two-cols
---

# ネイティブとの連携

とても分かりやすい公式ドキュメント

https://flutter.dev/docs/development/platform-integration/platform-channels

基本的にやることは

1. `MethodChannel` を名前付きでインスタンス化
1. `invokeMethod()` メソッドを呼ぶ
   - 引数でメソッド名、任意のデータ（引数）を渡す
2. `setMethodCallHandler()` を使ってメソッド呼び出しを検知する

::right::

<img style="margin: auto 0 auto 32px; object-fit: contain; height: 100%;" alt="Dart Executer" src="/images/platform_channels.png" />
---
layout: center
---
# バックグラウンド実行を実際にやってみた

---
layout: pip
image: /images/demo.gif
---

# サンプルアプリ
- Android前提
- **Foreground Serviceを使ったタイマー型カウンターアプリ**
  - 1秒ごとにコンソールにカウントが出力される
  - サンプルでは分かりやすいように画面にも出すようにしている
- メイン画面の機能
  - タイマーを起動する
  - タイマーを停止する


---

<img class="w-full h-full bg-transparent" src="/images/background_execution_all.svg" />

---
layout: pip
image: /images/background_execution_all.svg
---

main.dart
```dart
class _MyHomePageState extends State<MyHomePage> {
  @override
  void initState() {
    super.initState();
    TimerManager.initialize();
  }

  @override
  Widget build(BuildContext context) {
    ...
    ElevatedButton(
      onPressed: () => TimerManager.startTimer(timerCallback),
        child: const Text('Start'),
    ),
    ElevatedButton(
      onPressed: () => TimerManager.stopTimer(),
        child: const Text('Stop'),
    ),
    ...
  }
}

```

---
layout: pip
image: /images/background_execution_all.svg
---

timer_manager.dart
```dart{all|5-11}
class TimerManager {

  static const _channel = MethodChannel('dev.krgm4d/timer_manager');

  static Future<void> initialize() async {
    final callback = PluginUtilities.getCallbackHandle(callbackDispatcher);

    await _channel.invokeMethod(
        'TimerManager.initializeService', [callback.toRawHandle()]);
  }

  static Future<void> startTimer(void Function(int time) callback) async {
    final handle = PluginUtilities.getCallbackHandle(callback);

    await _channel.invokeMethod(
      'TimerManager.startTimer',
      [handle.toRawHandle()],
    );
  }
}

```

---
layout: pip
image: /images/background_execution_all.svg
---

MainActivity.kt
```kotlin{all|11-14}
override fun configureFlutterEngine(flutterEngine: FlutterEngine) {
    super.configureFlutterEngine(flutterEngine)

    MethodChannel(
        flutterEngine.dartExecutor.binaryMessenger,
        "dev.krgm4d/timer_manager"
    ).setMethodCallHandler { call, result ->
        run {
            val args = call.arguments<ArrayList<*>>()
            when (call.method) {
                "TimerManager.initializeService" -> {
                    initializeService(context, args)
                    result.success(true)
                }
                "TimerManager.startTimer" -> startTimer(context, args, result)
                "TimerManager.stopTimer" -> stopTimer(context, result)
                else -> result.notImplemented()
            }
        }
    }
}

```

---
layout: pip
image: /images/background_execution_all.svg
---

MainActivity.kt `companion object`
```kotlin
  @JvmStatic
  private fun initializeService(context: Context, args: ArrayList<*>?) {
      Log.d(TAG, "Initializing TimerService")
      // Callback Dispatcher のアドレス*を SharedPreference に保存する
      val callbackHandle = args!![0] as Long
      context.getSharedPreferences(SHARED_PREFERENCES_KEY, Context.MODE_PRIVATE).edit()
          .putLong(
              CALLBACK_DISPATCHER_HANDLE_KEY, callbackHandle
          ).apply()
  }

```

\* Flutterの内部では `CallbackHandle` というクラスで扱われる値の<br>ナマの数値

便宜上アドレスと表現するが、<br>厳密にメモリ上のアドレスというわけではない

---
layout: pip
image: /images/background_execution_all.svg
---

callback_dispatcher.dart
```dart
void callbackDispatcher() {
  const _backgroundChannel =
      MethodChannel('dev.krgm4d/timer_manager_background');
  WidgetsFlutterBinding.ensureInitialized();
  ...
}
```

---
layout: pip
image: /images/background_execution_all.svg
---

callback_dispatcher.dart
```dart
void callbackDispatcher() {
  ...
  _backgroundChannel.setMethodCallHandler((call) async {
    final List<dynamic> args = call.arguments;

    // バックグラウンドで実行する任意のDartコードのアドレスを取得
    // （図では [A] ）
    final callback = PluginUtilities.getCallbackFromHandle(
      CallbackHandle.fromRawHandle(args[0]),
    );

    // バックグランドでカウントした数値を取得
    final time = args[1] as int;

    // [A] を実行
    callback(time);
  });

  // 初期化が完了したことをプラットフォーム側に通知する
  _backgroundChannel.invokeMethod('TimerService.initialized');
}
```

---
layout: pip
image: /images/background_execution_all.svg
---

timer_manager.dart
```dart{all|12-19}
class TimerManager {

  static const _channel = MethodChannel('dev.krgm4d/timer_manager');

  static Future<void> initialize() async {
    final callback = PluginUtilities.getCallbackHandle(callbackDispatcher);

    await _channel.invokeMethod(
        'TimerManager.initializeService', [callback.toRawHandle()]);
  }

  static Future<void> startTimer(void Function(int time) callback) async {
    final handle = PluginUtilities.getCallbackHandle(callback);

    await _channel.invokeMethod(
      'TimerManager.startTimer',
      [handle.toRawHandle()],
    );
  }
}

```

---
layout: pip
image: /images/background_execution_all.svg
---

MainActivity.kt
```kotlin{all|15}
override fun configureFlutterEngine(flutterEngine: FlutterEngine) {
    super.configureFlutterEngine(flutterEngine)

    MethodChannel(
        flutterEngine.dartExecutor.binaryMessenger,
        "dev.krgm4d/timer_manager"
    ).setMethodCallHandler { call, result ->
        run {
            val args = call.arguments<ArrayList<*>>()
            when (call.method) {
                "TimerManager.initializeService" -> {
                    initializeService(context, args)
                    result.success(true)
                }
                "TimerManager.startTimer" -> startTimer(context, args, result)
                "TimerManager.stopTimer" -> stopTimer(context, result)
                else -> result.notImplemented()
            }
        }
    }
}

```

---
layout: pip
image: /images/background_execution_all.svg
---

MainActivity.kt `companion object`
```kotlin
  @JvmStatic
  private fun startTimer(
      context: Context,
      args: ArrayList<*>?,
      result: MethodChannel.Result
  ) {
      // [A] のアドレスを引数から取り出し SharedPreference に保存する
      val callbackHandle = args!![0] as Long
      context.getSharedPreferences(
        SHARED_PREFERENCES_KEY,Context.MODE_PRIVATE).edit()
          .putLong(
              CALLBACK_HANDLE_KEY, callbackHandle
          ).apply()

      if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
          // Foreground Service を起動
          context.startForegroundService(
            Intent(context, IsolateHolderService::class.java)
          )
      }
      result.success(true)
  }

```

---

IsolateHolderService.kt
```kotlin
class IsolateHolderService : Service(), MethodChannel.MethodCallHandler {
    private lateinit var mBackgroundChannel: MethodChannel

    // onCreate から呼ばれる
    private fun startIsolateHolderService(context: Context) {
        val callbackHandle = context.getSharedPreferences(MainActivity.SHARED_PREFERENCES_KEY,Context.MODE_PRIVATE)
                .getLong(MainActivity.CALLBACK_DISPATCHER_HANDLE_KEY, 0)

        // CallbackDispatcherのアドレスからコールバック情報を取得する
        val callbackInfo =
            FlutterCallbackInformation.lookupCallbackInformation(callbackHandle)
        ...
    }
}
```

---

IsolateHolderService.kt
```kotlin
class IsolateHolderService : Service(), MethodChannel.MethodCallHandler {
    private lateinit var mBackgroundChannel: MethodChannel

    // onCreate から呼ばれる
    private fun startIsolateHolderService(context: Context) {
        ...
        // CallbackDispatcherをエントリーポイントとして Flutter Engine を起動
        sBackgroundFlutterEngine = FlutterEngine(context)
        sBackgroundFlutterEngine!!.dartExecutor.executeDartCallback(DartExecutor.DartCallback(
            context.assets,
            FlutterInjector.instance().flutterLoader().findAppBundlePath(),
            callbackInfo
        ))

        // CallbackDispatcherから初期化が完了した通知を受け取れるようにする
        mBackgroundChannel = MethodChannel(..., "dev.krgm4d/timer_manager_background")
        mBackgroundChannel.setMethodCallHandler(this)
    }
}
```

---
layout: pip
image: /images/background_execution_all.svg
---

IsolateHolderService.kt
```kotlin
  // Foreground Serviceを起動させたときに呼ばれる
  override fun onStartCommand(intent: Intent?, flags: Int, startId: Int): Int {
    // Foreground Service を使うための準備（ここでは省略）

    // [A] のアドレスを取得
    callbackHandle = this.getSharedPreferences(MainActivity.SHARED_PREFERENCES_KEY, Context.MODE_PRIVATE)
        .getLong(MainActivity.CALLBACK_HANDLE_KEY, 0)
    ...
  }

```

---
layout: pip
image: /images/background_execution_all.svg
---

IsolateHolderService.kt
```kotlin
  // Foreground Serviceを起動させたときに呼ばれる
  override fun onStartCommand(intent: Intent?, flags: Int, startId: Int): Int {
    ...
    runnableTask = object : Runnable {
        var count = 0
        override fun run() {
            if (sServiceStarted.get()) {
                // 1秒間隔でこのスコープ内が定期実行される
                // countをインクリメントし、CallbackDispatcherで受け取ることができる
                // MethodCallを行う
                count++
                mBackgroundChannel.invokeMethod("", listOf(callbackHandle, count))
            }
            handler?.postDelayed(this, 1000)
        }
    }
    handler?.post(runnableTask!!)
    ...
  }

```

---
layout: pip
image: /images/background_execution_all.svg
---

callback_dispatcher.dart
```dart
void callbackDispatcher() {
  ...
  _backgroundChannel.setMethodCallHandler((call) async {
    final List<dynamic> args = call.arguments;

    // バックグラウンドで実行する任意のDartコードのアドレスを取得
    // （図では [A] ）
    final callback = PluginUtilities.getCallbackFromHandle(
      CallbackHandle.fromRawHandle(args[0]),
    );

    // バックグランドでカウントした数値を取得
    final time = args[1] as int;

    // [A] を実行
    callback(time);
  });
  ...
}
```

---
layout: pip
image: /images/background_execution_all.svg
---

main.dart
```dart
void timerCallback(int time) => debugPrint('time: $time');
```

🎉 🙌 🎉

---

# まとめ

Dartコードをバックグラウンド実行するためには

1. ①`main()`の代わりになる関数、②実行したい関数を静的な形で用意する
2. プラットフォーム側から①を立ち上げる
3. プラットフォーム側で任意のアクションをしたとき、②を`MethodChannel`経由で呼び出す

<Gap :size="60" />

あくまでFlutterはプラットフォームより上のレイヤー、プラットフォームとの連携がキモ

---

# 参考文献

- 原典
  - [Executing Dart in the Background with Flutter Plugins and Geofencing](https://medium.com/flutter/executing-dart-in-the-background-with-flutter-plugins-and-geofencing-2b3e40a1a124)
- 公式ドキュメントたち
  - [Flutter architectural overview](https://flutter.dev/docs/resources/architectural-overview#architectural-layers)
  - [Adding a Flutter screen to an Android app](https://flutter.dev/docs/development/add-to-app/android/add-flutter-screen?tab=cached-engine-with-initial-route-kotlin-tab#initial-route-with-a-cached-engine)
  - [Writing custom platform-specific code](https://flutter.dev/docs/development/platform-integration/platform-channels)
- おまけ
  - [FlutterでIsolateを用いた並列処理をするべきシーンとそのやり方](https://medium.com/flutter-jp/isolate-a3f6eab488b5)
  - [flutter_isolateは何をしているのか](https://www.rm48.net/post/flutter_isolate%E3%81%AF%E4%BD%95%E3%82%92%E3%81%97%E3%81%A6%E3%81%84%E3%82%8B%E3%81%AE%E3%81%8B)