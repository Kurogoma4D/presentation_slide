---
theme: dracula
title: Jetpack Composeでキーボードを作る
info: |
  ## potatotips 94
  LT session about implementing keyboard UI with InputMethodService and Jetpack Compose
drawings:
  persist: false
transition: fade-out
mdc: true
---

# Jetpack Composeでキーボードを作る

### Kurogoma4D

@ potatotips #94

<div class="flex items-center mt-12">
  <img class="h-20 rounded-xl" src="./images/kurogoma_chan_3.webp" />
  <div class="ml-12">
    株式会社Sun Asterisk - Lead Native-app Engineer / DevRel<br>X: @Kurogoma4D<br><br>@Krgm4Dは凍結されました
  </div>
</div>

<div class="abs-br m-6 flex gap-2">
  <a href="https://github.com/Kurogoma4D" target="_blank" alt="GitHub"
    class="text-xl slidev-icon-btn opacity-50 !border-none !hover:text-white">
    <carbon-logo-github />
  </a>
</div>

---
transition: fade-out
layout: center
---

# カスタムキーボードを作りたい

でも、Jetpack Composeで作りたい...

---
transition: fade-out
---

# 問題: InputMethodServiceはViewベース

<div class="mt-12 space-y-8">
  <div class="text-xl">
    <span class="text-yellow-400 font-bold">InputMethodService</span>は従来のViewベースのUIを想定
  </div>

  <div class="text-xl text-red-400">
    Jetpack Composeを直接使用することはできない
  </div>

  <div class="mt-12 text-lg">
    <div class="font-bold mb-4">解決策</div>
    <div class="ml-8 space-y-3">
      <div>• <span class="text-yellow-400">AbstractComposeView</span> - Compose UI を View として扱うブリッジ</div>
      <div>• <span class="text-yellow-400">Lifecycle/ViewModel/SavedState</span> - Compose に必要なアーキテクチャコンポーネント</div>
    </div>
  </div>
</div>

---
transition: fade-out
---

# 実装の全体像

<div class="mt-8">

```
┌─────────────────────────────────────────────┐
│           IMEService                        │
│  (InputMethodService + Lifecycle +          │
│   ViewModelStore + SavedStateRegistry)      │
└──────────────┬──────────────────────────────┘
               │ onCreateInputView()
               ▼
┌─────────────────────────────────────────────┐
│       ComposeKeyboardView                   │
│     (AbstractComposeView)                   │
└──────────────┬──────────────────────────────┘
               │ Content()
               ▼
┌─────────────────────────────────────────────┐
│         KeyboardScreen                      │
│      (Jetpack Compose UI)                   │
└─────────────────────────────────────────────┘
```

</div>

---
transition: fade-out
---

# ステップ1: AndroidManifest.xml

<div class="mt-6">

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android">
    <application>
        <service
            android:name=".IMEService"
            android:exported="true"
            android:permission="android.permission.BIND_INPUT_METHOD">
            <intent-filter>
                <action android:name="android.view.InputMethod" />
            </intent-filter>
            <meta-data
                android:name="android.view.im"
                android:resource="@xml/method" />
        </service>
    </application>
</manifest>
```

</div>

<div class="mt-6 text-sm opacity-75">
  res/xml/method.xml でサブタイプ（言語、モード）を定義
</div>

---
transition: fade-out
---

# ステップ2: IMEServiceの実装 (1/2)

<div class="text-lg mb-4">
  <span class="text-yellow-400 font-bold">3つのインターフェース</span>を実装
</div>

```kotlin {all|3-5|8-9|12-13}
class IMEService : InputMethodService(),
    LifecycleOwner,
    ViewModelStoreOwner,
    SavedStateRegistryOwner {

    // Lifecycle 管理
    private val lifecycleRegistry = LifecycleRegistry(this)
    override val lifecycle: Lifecycle
        get() = lifecycleRegistry

    // ViewModel 管理
    private val _viewModelStore = ViewModelStore()
    override val viewModelStore: ViewModelStore
        get() = _viewModelStore

    // SavedState 管理
    private val savedStateRegistryController =
        SavedStateRegistryController.create(this)
    override val savedStateRegistry: SavedStateRegistry
        get() = savedStateRegistryController.savedStateRegistry
```

---
transition: fade-out
---

# ステップ2: IMEServiceの実装 (2/2)

<div class="text-lg mb-4">
  <span class="text-yellow-400 font-bold">Lifecycleの状態遷移</span>を適切に管理
</div>

```kotlin {all|2-3|6-7|11-12|15-16}
    override fun onCreate() {
        super.onCreate()
        lifecycleRegistry.currentState = Lifecycle.State.CREATED
    }

    override fun onCreateInputView(): View {
        lifecycleRegistry.currentState = Lifecycle.State.STARTED
        return ComposeKeyboardView(...)
    }

    override fun onStartInputView(info: EditorInfo?, restarting: Boolean) {
        lifecycleRegistry.currentState = Lifecycle.State.RESUMED
    }

    override fun onFinishInputView(finishingInput: Boolean) {
        lifecycleRegistry.currentState = Lifecycle.State.STARTED
    }
```

---
transition: fade-out
---

# ステップ3: ComposeKeyboardView (1/2)

<div class="text-lg mb-4">
  <span class="text-yellow-400 font-bold">AbstractComposeView</span>でCompose UIをViewに変換
</div>

```kotlin {all|6-8|10-20}
class ComposeKeyboardView(
    context: Context,
    private val lifecycleOwner: LifecycleOwner,
    // ...
) : AbstractComposeView(context) {
    init {
        setViewCompositionStrategy(
            ViewCompositionStrategy.DisposeOnDetachedFromWindow)
    }

    override fun onAttachedToWindow() {
        // 親View階層すべてにLifecycleOwnerを設定
        var currentParent = parent
        while (currentParent != null) {
            if (currentParent is View) {
                currentParent.setViewTreeLifecycleOwner(lifecycleOwner)
            }
            currentParent = currentParent.parent
        }
        super.onAttachedToWindow()
    }
```

---
transition: fade-out
---

# ステップ3: ComposeKeyboardView (2/2)

<div class="text-lg mb-4">
  <span class="text-yellow-400 font-bold">Content()</span>でCompose UIを定義
</div>

```kotlin {all}
    @Composable
    override fun Content() {
        val viewModel: KeyboardViewModel =
            viewModel(viewModelStoreOwner = viewModelStoreOwner)
        KeyboardScreen(viewModel = viewModel)
    }
}
```

<div class="mt-8 text-sm opacity-75">
  ViewModelStoreOwnerを明示的に指定することで、<br>
  IMEServiceが持つViewModelStoreを使用
</div>

---
transition: fade-out
---

# ステップ4: Compose UIの実装

<div class="text-lg mb-4">
  通常の<span class="text-yellow-400 font-bold">Compose UI</span>として実装可能
</div>

```kotlin
@Composable
fun KeyboardScreen(
    viewModel: KeyboardViewModel,
    modifier: Modifier = Modifier
) {
    val ocrResults by viewModel.ocrResults.collectAsState()

    Column(modifier = modifier.fillMaxSize()) {
        // カメラプレビュー + OCR結果のオーバーレイ
        Box(modifier = Modifier.weight(1f)) {
            CameraPreview(...)
            TextOverlay(ocrResults = ocrResults, ...)
        }

        // キーボードキー
        KeyboardKeys(onKeyClick = { viewModel.onKeyClick(it) })
    }
}
```

---
transition: fade-out
---

# テキスト入力の処理

<div class="text-lg mb-4">
  <span class="text-yellow-400 font-bold">InputConnection</span>でテキストフィールドに入力
</div>

```kotlin {all|2|4-7|9-12}
private fun commitText(text: String) {
    val inputConnection = currentInputConnection ?: return

    when (text) {
        "\b" -> {
            inputConnection.deleteSurroundingText(1, 0)
        }

        else -> {
            inputConnection.commitText(text, 1)
        }
    }
}
```

---
transition: fade-out
---

# 重要なポイント

<div class="mt-12 space-y-8">
  <div class="border-2 border-yellow-500 p-6 rounded-lg">
    <div class="font-bold text-xl mb-4 text-yellow-400">1. Lifecycle状態の管理</div>
    <div class="ml-4">
      onCreate → CREATED<br>
      onCreateInputView → STARTED<br>
      onStartInputView → RESUMED<br>
      onFinishInputView → STARTED<br>
      onDestroy → DESTROYED
    </div>
  </div>

  <div class="border-2 border-yellow-500 p-6 rounded-lg">
    <div class="font-bold text-xl mb-4 text-yellow-400">2. ViewCompositionStrategy</div>
    <div class="ml-4">
      DisposeOnDetachedFromWindowを使用してメモリリークを防止
    </div>
  </div>
</div>

---
transition: fade-out
---

# 重要なポイント (続き)

<div class="mt-12 space-y-8">
  <div class="border-2 border-yellow-500 p-6 rounded-lg">
    <div class="font-bold text-xl mb-4 text-yellow-400">3. LifecycleOwnerの伝播</div>
    <div class="ml-4">
      InputMethodServiceのWindow構造に対応するため、<br>
      親View階層全体にLifecycleOwnerを設定する必要がある
    </div>
  </div>

  <div class="border-2 border-yellow-500 p-6 rounded-lg">
    <div class="font-bold text-xl mb-4 text-yellow-400">4. ViewModelの取得</div>
    <div class="ml-4">
      viewModel(viewModelStoreOwner = ...)で<br>
      明示的にViewModelStoreOwnerを指定
    </div>
  </div>
</div>

---
transition: fade-out
layout: two-cols
---

<video controls src="./images/keyboard.webm" class="h-100 rounded-3xl"></video>

::right::

# デモ

https://github.com/Kurogoma4D/ComposeKeyboard

---
layout: center
---

# まとめ

<div class="mt-12 space-y-6 text-xl">
  <div>✓ InputMethodServiceに<span class="text-yellow-400">Lifecycleサポート</span>を追加</div>
  <div>✓ <span class="text-yellow-400">AbstractComposeView</span>でブリッジを作成</div>
  <div>✓ 通常のCompose UIとして実装可能</div>
  <div>✓ <span class="text-green-400">最新のJetpack Composeを活用できる</span></div>
</div>

<div class="mt-16 text-center text-2xl text-blue-400">
  Enjoy Composing! 🎨
</div>

---
layout: center
---

# おまけ

Sun Asteriskをよろしくお願いします！

https://job.persona-ats.com/ja/sun-asterisk/jobs/735dd277-54f4-4ed8-97eb-a3738683df6b

<img src="./images/hire.png" class="h-50 rounded-xl" />
