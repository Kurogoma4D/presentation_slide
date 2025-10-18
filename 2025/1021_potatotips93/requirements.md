- Deferred Deep Linkとは
  - AppLinks / Universal Linksなどで実現したディープリンクのうち、アプリ未インストール時でも体験が継続されるもの
  - （図で説明）
- 過去にはFirebase Dynamic Linksを導入することで解決していたが、サービス終了につきAppsFlyer、Adjust等のサービスを導入するか、自前で実装するかの択になった
- 小規模なサービスには前述のサービス導入をためらうので、自前でどう実装できるのか知りたい
- 調べてみた
  - Android / iOS両方調べたが、今回はAndroid枠の登壇なのでAndroidの話
- AndroidではInstall Referrer APIというものがある
  - Google Playからアプリをインストールした際のリファラー情報（どこ経由でGoogle Playにアクセスしたか）をアプリ上で取得できる
  - GAで使う `utm_source` とか
- リファラー情報は形式としてはクエリパラメータ
  - つまりリファラー情報としてアプリ側で処理したい情報を入れれば実現できる
- やってみた
  - 自作のScrapToというアプリ
  - https://play.google.com/store/apps/details?id=dev.krgm4d.scrapto
- Install Referrer APIを使って、`utm_source=landing_page` だったら「ダウンロードありがとうございます！」というトーストを表示する

```kotlin
val referrerClient = InstallReferrerClient.newBuilder(context).build()
referrerClient.startConnection(object : InstallReferrerStateListener {
    override fun onInstallReferrerSetupFinished(responseCode: Int) {
        when (responseCode) {
            InstallReferrerClient.InstallReferrerResponse.OK -> {
                try {
                    val response: ReferrerDetails = referrerClient.installReferrer
                    val referrerUrl = response.installReferrer
                    // リファラーURLからutm_sourceを確認
                    if (referrerUrl.contains("utm_source=landing_page")) {
                        // ランディングページからのインストールであることを確認
                        Toast.makeText(
                            context,
                            "ダウンロードありがとうございます!",
                            Toast.LENGTH_LONG
                        ).show()
                    }
                    // チェック済みフラグを立てる
                    prefs.edit().putBoolean(referrerCheckedKey, true).apply()
                }
                ... // エラー処理
            }
            ... // リファラー情報が取得できなかった場合
        }
    }
```

- これを応用すれば、Deep Linkを実現できる！
- 注意点
  - デバッグ難易度がちょっと高い
  - 多分インストール後必ずしもすぐに取得できるわけではない
    - 調べた感じ、数分とかかかる場合もありそう？
