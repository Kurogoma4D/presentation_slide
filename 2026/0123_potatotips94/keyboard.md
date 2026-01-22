# InputMethodService と Jetpack Compose を使ったキーボードUI実装ガイド

このドキュメントでは、Android の InputMethodService を実装しながら、Jetpack Compose を使ってキーボードのUIを構築する方法について、Scanji プロジェクトの実装を基に解説します。

## 1. 概要

Android でカスタムキーボードを作成する際、通常は InputMethodService を継承して実装します。しかし、InputMethodService は従来の View ベースの UI を想定しており、Jetpack Compose を直接使用することはできません。この問題を解決するため、以下の要素を組み合わせます：

- **InputMethodService**: キーボードサービスの基本実装
- **AbstractComposeView**: Compose UI を Android View として扱うブリッジ
- **Lifecycle/ViewModel/SavedState**: Compose で必要な Android アーキテクチャコンポーネント

## 2. AndroidManifest.xml の設定

まず、InputMethodService をシステムに登録する必要があります。

### 必要な権限とサービス宣言

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android">

    <!-- カメラを使用する場合 -->
    <uses-permission android:name="android.permission.CAMERA" />
    <uses-feature android:name="android.hardware.camera" android:required="false" />

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

### Input Method メタデータ（res/xml/method.xml）

```xml
<?xml version="1.0" encoding="utf-8"?>
<input-method xmlns:android="http://schemas.android.com/apk/res/android">
    <subtype
        android:imeSubtypeMode="keyboard"
        android:imeSubtypeLocale="ja_JP"
        android:label="Scanji OCR Keyboard" />
</input-method>
```

## 3. InputMethodService の実装

InputMethodService を Jetpack Compose で使用するためには、Lifecycle、ViewModel、SavedState をサポートする必要があります。

### IMEService.kt の実装

```kotlin
package dev.krgm4d.scanji

import android.inputmethodservice.InputMethodService
import android.view.View
import android.view.inputmethod.EditorInfo
import androidx.lifecycle.Lifecycle
import androidx.lifecycle.LifecycleOwner
import androidx.lifecycle.LifecycleRegistry
import androidx.lifecycle.ViewModelStore
import androidx.lifecycle.ViewModelStoreOwner
import androidx.lifecycle.setViewTreeLifecycleOwner
import androidx.savedstate.SavedStateRegistry
import androidx.savedstate.SavedStateRegistryController
import androidx.savedstate.SavedStateRegistryOwner
import androidx.savedstate.setViewTreeSavedStateRegistryOwner

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
    private val savedStateRegistryController = SavedStateRegistryController.create(this)
    override val savedStateRegistry: SavedStateRegistry
        get() = savedStateRegistryController.savedStateRegistry

    override fun onCreate() {
        super.onCreate()
        // SavedState の復元
        savedStateRegistryController.performRestore(null)
        // Lifecycle を CREATED に設定
        lifecycleRegistry.currentState = Lifecycle.State.CREATED

        // Window の DecorView に Lifecycle Owner を設定
        window?.window?.decorView?.let { decorView ->
            decorView.setViewTreeLifecycleOwner(this)
            decorView.setViewTreeSavedStateRegistryOwner(this)
        }
    }

    override fun onCreateInputView(): View {
        // Input View 作成時に STARTED に設定
        lifecycleRegistry.currentState = Lifecycle.State.STARTED

        return ComposeKeyboardView(
            context = this,
            lifecycleOwner = this,
            viewModelStoreOwner = this,
            savedStateRegistryOwner = this
        ).apply {
            // テキスト入力のコールバック設定
            onCommitText = { text ->
                commitText(text)
            }
        }
    }

    private fun commitText(text: String) {
        val inputConnection = currentInputConnection ?: return

        when (text) {
            "\b" -> {
                // バックスペース処理
                inputConnection.deleteSurroundingText(1, 0)
            }
            else -> {
                // 通常のテキスト入力
                inputConnection.commitText(text, 1)
            }
        }
    }

    override fun onStartInputView(info: EditorInfo?, restarting: Boolean) {
        super.onStartInputView(info, restarting)
        // キーボードが表示されたら RESUMED に設定
        lifecycleRegistry.currentState = Lifecycle.State.RESUMED
    }

    override fun onFinishInputView(finishingInput: Boolean) {
        super.onFinishInputView(finishingInput)
        // キーボードが非表示になったら STARTED に戻す
        lifecycleRegistry.currentState = Lifecycle.State.STARTED
    }

    override fun onDestroy() {
        super.onDestroy()
        // サービス破棄時に DESTROYED に設定し、ViewModel をクリア
        lifecycleRegistry.currentState = Lifecycle.State.DESTROYED
        _viewModelStore.clear()
    }
}
```

### 重要なポイント

1. **3つのインターフェースの実装**
   - `LifecycleOwner`: Compose が Lifecycle を追跡できるようにする
   - `ViewModelStoreOwner`: ViewModel の保持と管理
   - `SavedStateRegistryOwner`: 状態の保存と復元

2. **Lifecycle の状態遷移**
   - `onCreate()`: CREATED
   - `onCreateInputView()`: STARTED
   - `onStartInputView()`: RESUMED
   - `onFinishInputView()`: STARTED に戻す
   - `onDestroy()`: DESTROYED

3. **InputConnection による入力処理**
   - `currentInputConnection` を使用してテキストフィールドに文字を送信
   - `commitText()` でテキスト入力
   - `deleteSurroundingText()` でバックスペース処理

## 4. ComposeKeyboardView の実装

AbstractComposeView を継承して、Compose UI を View として提供します。

### ComposeKeyboardView.kt

```kotlin
package dev.krgm4d.scanji

import android.content.Context
import android.view.View
import androidx.compose.runtime.Composable
import androidx.compose.ui.platform.AbstractComposeView
import androidx.compose.ui.platform.ViewCompositionStrategy
import androidx.lifecycle.LifecycleOwner
import androidx.lifecycle.ViewModelStoreOwner
import androidx.lifecycle.setViewTreeLifecycleOwner
import androidx.lifecycle.viewmodel.compose.viewModel
import androidx.savedstate.SavedStateRegistryOwner
import androidx.savedstate.setViewTreeSavedStateRegistryOwner
import dev.krgm4d.scanji.ui.KeyboardScreen
import dev.krgm4d.scanji.viewmodel.KeyboardViewModel

class ComposeKeyboardView(
    context: Context,
    private val lifecycleOwner: LifecycleOwner,
    private val viewModelStoreOwner: ViewModelStoreOwner,
    private val savedStateRegistryOwner: SavedStateRegistryOwner
) : AbstractComposeView(context) {

    var onCommitText: ((String) -> Unit)? = null

    init {
        // View が Window から detach されたときに Composition を破棄
        setViewCompositionStrategy(ViewCompositionStrategy.DisposeOnDetachedFromWindow)
    }

    override fun onAttachedToWindow() {
        // 親 View 階層すべてに LifecycleOwner を設定
        var currentParent = parent
        while (currentParent != null) {
            if (currentParent is View) {
                currentParent.setViewTreeLifecycleOwner(lifecycleOwner)
                currentParent.setViewTreeSavedStateRegistryOwner(savedStateRegistryOwner)
            }
            currentParent = currentParent.parent
        }

        setViewTreeLifecycleOwner(lifecycleOwner)
        setViewTreeSavedStateRegistryOwner(savedStateRegistryOwner)

        super.onAttachedToWindow()
    }

    @Composable
    override fun Content() {
        // ViewModelStoreOwner を指定して ViewModel を取得
        val viewModel: KeyboardViewModel = viewModel(viewModelStoreOwner = viewModelStoreOwner)
        viewModel.onCommitText = onCommitText

        // Compose UI のルート
        KeyboardScreen(viewModel = viewModel)
    }
}
```

### 重要なポイント

1. **AbstractComposeView の継承**
   - `Content()` をオーバーライドして Compose UI を定義

2. **ViewCompositionStrategy の設定**
   - `DisposeOnDetachedFromWindow` を使用して、View が detach されたときに Composition を破棄
   - これによりメモリリークを防止

3. **Lifecycle Owner の伝播**
   - `onAttachedToWindow()` で親 View 階層全体に LifecycleOwner を設定
   - InputMethodService の Window 構造に対応するため必要

4. **ViewModel の取得**
   - `viewModel(viewModelStoreOwner = ...)` で明示的に ViewModelStoreOwner を指定
   - IMEService が持つ ViewModelStore を使用

## 5. Jetpack Compose UI の実装

通常の Compose アプリと同様に UI を構築できます。

### KeyboardScreen.kt

```kotlin
package dev.krgm4d.scanji.ui

import androidx.compose.foundation.layout.*
import androidx.compose.runtime.Composable
import androidx.compose.runtime.collectAsState
import androidx.compose.runtime.getValue
import androidx.compose.ui.Modifier
import dev.krgm4d.scanji.viewmodel.KeyboardViewModel

@Composable
fun KeyboardScreen(
    viewModel: KeyboardViewModel,
    modifier: Modifier = Modifier
) {
    val ocrResults by viewModel.ocrResults.collectAsState()

    Column(
        modifier = modifier.fillMaxSize()
    ) {
        // カメラプレビュー + OCR 結果のオーバーレイ
        Box(
            modifier = Modifier
                .fillMaxWidth()
                .weight(1f)
        ) {
            CameraPreview(
                modifier = Modifier.fillMaxSize(),
                onTextDetected = { results ->
                    viewModel.updateOcrResults(results)
                }
            )
            TextOverlay(
                ocrResults = ocrResults,
                modifier = Modifier.fillMaxSize(),
                onTextSelected = { text ->
                    viewModel.onTextSelected(text)
                }
            )
        }

        // キーボードキー
        KeyboardKeys(
            onKeyClick = { key ->
                viewModel.onKeyClick(key)
            },
            modifier = Modifier.fillMaxWidth()
        )
    }
}
```

### ViewModel による状態管理

```kotlin
package dev.krgm4d.scanji.viewmodel

import androidx.lifecycle.ViewModel
import dev.krgm4d.scanji.ocr.OcrResult
import kotlinx.coroutines.flow.MutableStateFlow
import kotlinx.coroutines.flow.StateFlow
import kotlinx.coroutines.flow.asStateFlow

class KeyboardViewModel : ViewModel() {
    // OCR 結果の状態管理
    private val _ocrResults = MutableStateFlow<List<OcrResult>>(emptyList())
    val ocrResults: StateFlow<List<OcrResult>> = _ocrResults.asStateFlow()

    // テキスト入力コールバック（IMEService から注入）
    var onCommitText: ((String) -> Unit)? = null

    fun updateOcrResults(results: List<OcrResult>) {
        _ocrResults.value = results
    }

    fun onTextSelected(text: String) {
        // OCR で認識したテキストが選択されたとき
        onCommitText?.invoke(text)
    }

    fun onKeyClick(key: String) {
        // キーボードキーがクリックされたとき
        when (key) {
            "\b" -> onCommitText?.invoke(key) // バックスペース
            else -> onCommitText?.invoke(key) // 通常の入力
        }
    }
}
```

## 6. アーキテクチャ図

```
┌─────────────────────────────────────────────┐
│           IMEService                        │
│  (InputMethodService + Lifecycle +          │
│   ViewModelStore + SavedStateRegistry)      │
└──────────────┬──────────────────────────────┘
               │ onCreateInputView()
               │ returns View
               ▼
┌─────────────────────────────────────────────┐
│       ComposeKeyboardView                   │
│     (AbstractComposeView)                   │
│  - Lifecycle Owner の設定                    │
│  - ViewCompositionStrategy の管理            │
└──────────────┬──────────────────────────────┘
               │ Content()
               │ @Composable
               ▼
┌─────────────────────────────────────────────┐
│         KeyboardScreen                      │
│      (Jetpack Compose UI)                   │
│  - CameraPreview                            │
│  - TextOverlay (OCR 結果)                   │
│  - KeyboardKeys                             │
└──────────────┬──────────────────────────────┘
               │
               ▼
┌─────────────────────────────────────────────┐
│       KeyboardViewModel                     │
│  - OCR 結果の状態管理                        │
│  - onCommitText コールバック                 │
└──────────────┬──────────────────────────────┘
               │ onCommitText?.invoke(text)
               ▼
┌─────────────────────────────────────────────┐
│   IMEService.commitText(text)               │
│  - currentInputConnection.commitText()      │
└─────────────────────────────────────────────┘
```

## 7. まとめ

InputMethodService で Jetpack Compose を使用するには：

1. **InputMethodService に Lifecycle サポートを追加**
   - LifecycleOwner, ViewModelStoreOwner, SavedStateRegistryOwner を実装
   - サービスのライフサイクルに合わせて Lifecycle 状態を管理

2. **AbstractComposeView でブリッジを作成**
   - Compose UI を Android View として提供
   - 親 View 階層に Lifecycle Owner を伝播

3. **通常の Compose UI を構築**
   - @Composable 関数で UI を定義
   - ViewModel で状態管理
   - コールバックで InputConnection に入力を送信

この構造により、最新の Jetpack Compose を使いながら、Android の InputMethodService の機能をフルに活用できます。
