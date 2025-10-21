---
theme: dracula
# background: https://source.unsplash.com/collection/94734566/1920x1080
title: Deferred Deep Linkに必要な技術を理解する
info: |
  ## potatotips 93
  LT session about some arts for Deferred Deep Link
drawings:
  persist: false
transition: fade-out
mdc: true
---

# Deferred Deep Linkに必要な技術を理解する

### Kurogoma4D

@ potatotips #93

<div class="flex items-center mt-12">
  <img class="h-20 rounded-xl" src="./images/kurogoma_chan_2.webp" />
  <div class="ml-12">
    株式会社Sun Asterisk - Lead Native-app Engineer<br>X: @Krgm4D
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

# Deferred Deep Linkとは？

AppLinks / Universal Linksで実現するディープリンクのうち、
**アプリ未インストール時でも体験が継続されるもの**

---
transition: fade-out
---

# 通常のDeep Link vs Deferred Deep Link

<div class="grid grid-cols-2 gap-8 mt-8">
  <div class="border-2 border-blue-500 p-6 rounded-lg">
    <div class="text-center font-bold text-xl mb-4 text-blue-400">通常のDeep Link</div>
    <div class="text-lg space-y-3">
      <div>1. リンクをタップ</div>
      <div>2. <span class="text-green-400">✓ アプリが開く</span></div>
      <div>3. <span class="text-green-400">✓ 指定画面に遷移</span></div>
    </div>
    <div class="mt-6 text-center text-sm opacity-75">
      アプリインストール済みが前提
    </div>
  </div>

  <div class="border-2 border-purple-500 p-6 rounded-lg">
    <div class="text-center font-bold text-xl mb-4 text-purple-400">Deferred Deep Link</div>
    <div class="text-lg space-y-3">
      <div>1. リンクをタップ</div>
      <div>2. <span class="text-yellow-400">→ Google Playへ</span></div>
      <div>3. <span class="text-yellow-400">→ アプリインストール</span></div>
      <div>4. <span class="text-green-400">✓ アプリ起動</span></div>
      <div>5. <span class="text-green-400">✓ 指定画面に遷移</span></div>
    </div>
    <div class="mt-6 text-center text-sm opacity-75">
      インストール後も体験が継続
    </div>
  </div>
</div>

---
transition: fade-out
---

# Firebase Dynamic Linksの終了

<div class="mt-12 space-y-8">
  <div class="text-xl">
    過去には<span class="text-yellow-400 font-bold">Firebase Dynamic Links</span>で簡単に実現できていた
  </div>

  <div class="text-xl text-red-400">
    しかし、2025年8月25日にサービス終了...
  </div>

  <div class="mt-12 text-lg">
    <div class="font-bold mb-4">代替手段は？</div>
    <div class="ml-8 space-y-3">
      <div>• AppsFlyer、Adjustなどの有料サービス</div>
      <div>• 自前で実装</div>
    </div>
  </div>

  <div class="mt-12 text-xl text-green-400">
    小規模サービスなら自前実装も検討したい！
  </div>
</div>

---
transition: fade-out
layout: center
---

# どうやって実装する？

Androidには**Install Referrer API**というものがある

---

# Install Referrer API

<div class="mt-8 space-y-8">
  <div class="text-lg">
    Google Playからアプリをインストールした際の<br>
    <span class="text-yellow-400 font-bold">リファラー情報</span>（どこ経由でGoogle Playにアクセスしたか）を取得できる
  </div>

  <div class="text-lg mt-8">
    よくあるパラメータ:
    <div class="ml-8 mt-4 space-y-2">
      <div>• <code>utm_source</code> - 参照元</div>
      <div>• <code>utm_medium</code> - メディア</div>
      <div>• <code>utm_campaign</code> - キャンペーン名</div>
    </div>
  </div>

  <div class="mt-12 text-xl text-green-400">
    形式はクエリパラメータ<br>
    → <span class="font-bold">任意の情報を渡せる！</span>
  </div>
</div>

---

# リファラー情報の流れ

<div class="mt-12">
  <div class="flex items-center justify-center space-x-4">
    <div class="bg-blue-600 p-6 rounded-lg text-center">
      <div class="font-bold mb-2">LP</div>
      <div class="text-sm">リンクをタップ</div>
    </div>
    <div class="text-3xl">→</div>
    <div class="bg-green-600 p-6 rounded-lg text-center">
      <div class="font-bold mb-2">Google Play</div>
      <div class="text-sm text-xs">
        <code>referrer=utm_source%3Dlanding_page</code>
      </div>
    </div>
    <div class="text-3xl">→</div>
    <div class="bg-purple-600 p-6 rounded-lg text-center">
      <div class="font-bold mb-2">アプリインストール</div>
      <div class="text-sm">リファラー情報を保持</div>
    </div>
    <div class="text-3xl">→</div>
    <div class="bg-yellow-600 p-6 rounded-lg text-center">
      <div class="font-bold mb-2">初回起動</div>
      <div class="text-sm">Install Referrer APIで取得</div>
    </div>
  </div>

  <div class="mt-12 text-center text-lg">
    リファラー情報に<span class="text-yellow-400 font-bold">遷移先情報</span>を含めれば<br>
    Deep Linkを実現できる！
  </div>
</div>

---

# 実装してみた

<div class="mt-8 space-y-6">
  <div class="text-lg">
    実装例: 自作アプリ<span class="text-yellow-400 font-bold">ScrapTo</span>で試してみた
  </div>

  <div class="text-sm opacity-75">
    https://play.google.com/store/apps/details?id=dev.krgm4d.scrapto
  </div>

  <div class="mt-8 text-lg">
    <div class="font-bold mb-4">要件:</div>
    <div class="ml-8">
      <code>utm_source=landing_page</code> からインストールした場合、<br>
      初回起動時に「ダウンロードありがとうございます！」というトーストを表示
    </div>
  </div>
</div>

---

# 実装コード (1/2)

```kotlin
// Install Referrer Clientの準備
val referrerClient = InstallReferrerClient.newBuilder(context).build()
referrerClient.startConnection(object : InstallReferrerStateListener {
    override fun onInstallReferrerSetupFinished(responseCode: Int) {
        when (responseCode) {
            InstallReferrerClient.InstallReferrerResponse.OK -> {
                try {
                    // リファラー情報を取得
                    val response: ReferrerDetails = referrerClient.installReferrer
                    val referrerUrl = response.installReferrer

                    // utm_sourceをチェック
                    if (referrerUrl.contains("utm_source=landing_page")) {
                        // 処理を実行
```

---

# 実装コード (2/2)

```kotlin
                    if (referrerUrl.contains("utm_source=landing_page")) {
                        Toast.makeText(
                            context,
                            "ダウンロードありがとうございます!",
                            Toast.LENGTH_LONG
                        ).show()
                    }

                    // チェック済みフラグを立てる（2回目以降は実行しない）
                    prefs.edit().putBoolean(referrerCheckedKey, true).apply()

                } catch (e: Exception) {
                    // エラー処理
                }
            }
        }
    }

    override fun onInstallReferrerServiceDisconnected() {
        // 再接続処理など
    }
})
```

---
transition: fade-out
---

# 注意点

<div class="mt-12 space-y-8">
  <div class="border-2 border-yellow-500 p-6 rounded-lg">
    <div class="font-bold text-xl mb-4 text-yellow-400">1. デバッグコストが高い</div>
    <div class="ml-4">
      手元で検証する場合、リンクを踏む→対応するapkをadbでインストール→起動という手順を踏む<br>
      これで確実なのかどうかあまりわかっていない
    </div>
  </div>

  <div class="border-2 border-yellow-500 p-6 rounded-lg">
    <div class="font-bold text-xl mb-4 text-yellow-400">2. 取得タイミングに注意</div>
    <div class="ml-4">
      インストール後すぐに取得できるとは限らない（多分）<br>
      また、インストール後に1度だけチェックするように制御する必要がある
    </div>
  </div>
</div>

---
layout: two-cols
---

<div class="flex flex-col items-center justify-center h-full">
  <div class="text-center font-bold text-xl mb-4">通常</div>
  <video controls src="./images/scrapto-normal.webm" class="h-100 rounded-3xl"></video>
</div>

::right::

<div class="flex flex-col items-center justify-center h-full">
  <div class="text-center font-bold text-xl mb-4">リファラー付き</div>
  <video controls src="./images/scrapto-referrer.webm" class="h-100 rounded-3xl"></video>
</div>

---
layout: center
---

# おまけ

Sun Asteriskをよろしくお願いします！

https://job.persona-ats.com/ja/sun-asterisk/jobs/735dd277-54f4-4ed8-97eb-a3738683df6b

<img src="./images/hire.png" class="h-50 rounded-xl" />
