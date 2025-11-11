---
theme: dracula
# background: https://source.unsplash.com/collection/94734566/1920x1080
title: Flutter is NOT DEAD.
info: |
  ## FlutterKaigi 2025
  Talk about development using Flutter.
drawings:
  persist: false
transition: fade-out
mdc: true
---

# Flutter is NOT DEAD.

### Kurogoma4D

@ FlutterKaigi 2025

<div class="abs-br m-6 flex gap-2">
  <a href="https://github.com/Kurogoma4D" target="_blank" alt="GitHub"
    class="text-xl slidev-icon-btn opacity-50 !border-none !hover:text-white">
    <carbon-logo-github />
  </a>
</div>

---
transition: fade-out
---

# $ whoami

<div class="flex items-center mt-12 mb-18">
  <img class="h-20 rounded-xl" src="./images/kurogoma_chan_2.webp" />
  <div class="ml-12">
    株式会社Sun Asterisk - Lead Native-app Engineer<br>X: @Krgm4D
  </div>
</div>

- 使ったことあるクロスプラットフォーム技術
  - Flutter, React Native(Expo), Xamarin Forms, ちょっとKMP, ちょっと.NET MAUI
- Jetpack ComposeやSwiftUIも好き
  - 宣言的UIの流れを作ったReact、Flutterに感謝
- 自作キーボード勢

---
transition: none
---

# 皆さんに質問です

<div class="mt-12 space-y-8">
  <div class="text-2xl">
    🙋 <span class="text-yellow-400 font-bold">普段Flutterで開発していますか？</span>
  </div>
</div>

---
transition: none
---

# 皆さんに質問です

<div class="mt-12 space-y-8">
  <div class="text-2xl">
    🙋 <span class="text-yellow-400 font-bold">普段Flutterで開発していますか？</span>
  </div>

  <div class="text-2xl mt-8">
    🙋 <span class="text-green-400 font-bold">ネイティブアプリ・ガワアプリをFlutterにリプレイスしたことはありますか？</span>
  </div>
</div>

---

# 皆さんに質問です

<div class="mt-12 space-y-8">
  <div class="text-2xl">
    🙋 <span class="text-yellow-400 font-bold">普段Flutterで開発していますか？</span>
  </div>

  <div class="text-2xl mt-8">
    🙋 <span class="text-green-400 font-bold">ネイティブアプリ・ガワアプリをFlutterにリプレイスしたことはありますか？</span>
  </div>

  <div class="text-2xl mt-8">
    🙋 <span class="text-red-400 font-bold">逆にFlutterアプリをネイティブアプリ・その他フレームワークにリプレイスした方は？</span>
  </div>
</div>


---
transition: fade-out
layout: center
---

# 今日は技術選定の話をします

<div class="text-2xl">
特に<span class="text-yellow-400 font-bold">「Flutterの採用が視野に入っているとき」</span>に
考えるべきことについて
</div>

---

# 昨今のFlutterを取り巻く声

<div class="mt-12 space-y-8">
  <img src="./images/flutter-is-dead.jpeg" class="h-80 mx-auto">
  <div class="mt-12 text-xl opacity-75">https://x.com/vibetushar/status/1939762052173897744</div>
</div>

---
layout: center
---

# Flutterに対する2つの大きな不安

<div class="mt-12 grid grid-cols-2 gap-8">
  <div class="border-2 border-blue-500 p-8 rounded-lg text-center">
    <div class="text-2xl font-bold text-blue-400">1</div>
    <div class="text-xl mt-4">アーキテクチャ上の特徴</div>
  </div>

  <div class="border-2 border-purple-500 p-8 rounded-lg text-center">
    <div class="text-2xl font-bold text-purple-400">2</div>
    <div class="text-xl mt-4">メンテナー問題</div>
  </div>
</div>

---
layout: center
---

# 不安その1

## アーキテクチャ上の特徴

---

# Flutterのアーキテクチャ上の特徴

<div class="mt-12 space-y-8">
  <div class="text-2xl">
    Flutterは<span class="text-red-400 font-bold">ネイティブのUI要素を描画しない</span>
  </div>

  <div class="text-center">
    <img class="h-70 mx-auto" src="./images/archdiagram.png" />
    https://docs.flutter.dev/resources/architectural-overview
  </div>
</div>

---

# Flutterのアーキテクチャ上の特徴

<div class="mt-12 space-y-8">
  <div class="text-2xl">
    Flutterは<span class="text-red-400 font-bold">ネイティブのUI要素を描画しない</span>
  </div>

  <div class="text-xl mt-8">
    つまり、プラットフォーム上のUIの変化についていけない
  </div>

  <div class="mt-12 text-2xl">
    <div class="mb-4">例えば</div>
    <div class="ml-8 space-y-2">
      <div>• iOSの新しいLiquid Glassデザイン</div>
      <div>• AndroidのMaterial Designの進化</div>
      <div>• その他新しいシステムコンポーネント</div>
    </div>
  </div>
</div>


---

# でも、ちょっと待って

<div class="mt-12 space-y-8">
  <div class="text-2xl text-yellow-400">
    プラットフォームのUIの変化は<span class="font-bold">誰もが望むものだろうか？</span>
  </div>

  <div class="mt-12 text-xl">
    • アプリ開発者や一部のユーザーにとっては嬉しいかもしれない<br>
    • 最新のプラットフォーム事情に追従できることは魅力的
  </div>
</div>

---

# 見方を変えてみる

<div class="h-100 text-2xl text-center flex items-center justify-center">
  Flutterは単なる<br>
  「iOS/Androidをターゲットにしたクロスプラットフォームフレームワーク」<br>
  ではない
</div>


---

# Flutterを

<div class="h-100 text-4xl text-center flex items-center justify-center">
  <span class="text-red-400 font-bold">プラットフォームへの統合が容易な<br>UIツールキット<br>
    <span class="text-2xl text-white">として見たらどうか？</span>
  </span>
</div>

---

# これは公式サイトにも書かれている

<div class="mt-12 space-y-8">
  <img src="./images/flutter-is-ui-toolkit.png" class="h-80 mx-auto">
  <div class="mt-12 text-xl opacity-75">https://flutter.dev/learn</div>
</div>

---

# Flutterを採用したとき

<div class="mt-12 space-y-8">
  <div class="text-2xl">
    OSの世界観をそのまま引き継がずに<span class="text-yellow-400 font-bold">プロダクト独自のUI</span>を構築することは<br>
    ほぼ既定路線である
  </div>

  <div class="mt-12 text-xl">
    <div class="font-bold mb-4">これは嬉しくない組織もあるが
      <span class="text-green-400 font-bold">逆に嬉しい組織もあるのでは？</span>
    </div>
  </div>

  <div class="mt-12 text-xl border-2 border-blue-500 p-6 rounded-lg">
    <div class="font-bold mb-2">例えば</div>
    あるWebサービス上で独自の世界観を築き上げている中で、<br>
    モバイルアプリを新規開発したいパターンなど
  </div>
</div>

---

# プラットフォームUIの採用について

<div class="mt-12 space-y-8">
  <div class="text-xl">ネイティブでの実装にしろ、React Native他フレームワークを使うにしろ</div>

  <div class="mt-8 text-2xl text-center border-2 border-purple-500 p-8 rounded-lg">
    AndroidでMaterial、<br>
    iOSでLiquid Glassを採用しない以上は<br>
    <span class="text-green-400 font-bold">UIに対する姿勢はどれも変わらない</span>
  </div>

  <div class="mt-12 text-xl">
    プラットフォームの世界観を重視するなら、
    <span class="text-blue-400 font-bold">SwiftUI</span>や<span class="text-blue-400 font-bold">Jetpack Compose</span>を選んだほうが<br>
    幸せになれるかもしれない
  </div>
</div>

---

# つまり

<div class="mt-12 space-y-8">
  <div class="text-2xl text-center border-2 border-yellow-500 p-8 rounded-lg">
    アーキテクチャの点で不安になるということは<br>
    <span class="text-yellow-400 font-bold text-3xl">現在の開発チームと技術が<br>ハマっていない</span><br>
    ということ
  </div>

  <div class="mt-12 text-xl">
    <div class="font-bold mb-4">重要なのは</div>
    <div class="ml-8">
      • OSの世界観を大事にしたいのか？<br>
      • プロダクト独自の世界観を構築したいのか？<br>
      • 複数プラットフォームで一貫したUIを提供したいのか？
    </div>
  </div>
</div>

---
layout: center
---

# 不安その2

## メンテナー問題

---

# メンテナー問題とは

<div class="mt-12 space-y-8">
  <div class="text-2xl">
    Flutterリポジトリには<span class="text-red-400 font-bold">大量のissue</span>がある
  </div>

  <div class="text-xl mt-8">
    • issueやPRを上げてもなかなか進まないという話はどうしても挙がる<br>
    • メンテナーが不足している？<br>
    • 本当に継続的にメンテナンスされているのか？
  </div>

  <div class="mt-12 text-2xl text-yellow-400">
    これに関しては<span class="font-bold">難しい問題</span>
  </div>
</div>

---

# 実際のところ

<div class="mt-12 space-y-8">
  <div class="text-2xl">
    <div class="ml-8 space-y-3">
      <div>• 優先度付けはされている</div>
      <div>• そのレポートもある</div>
      <div class="ml-8">• <a href="https://github.com/flutter/flutter/wiki/2025-Issue-Triage-Reports">https://github.com/flutter/flutter/wiki/2025-Issue-Triage-Reports</a></div>
      <div>• <span class="text-green-400 font-bold">対応される速度に差はある</span></div>
    </div>
  </div>

  <div class="mt-12 text-xl border-2 border-blue-500 p-6 rounded-lg">
    すべてのissueが同じ速度で対応されるわけではない<br>
    これは<span class="text-blue-400 font-bold">どのOSSプロジェクトでも同じ</span>
  </div>
</div>

---

# 我々にできること

<div class="mt-12 space-y-8">
  <div class="text-2xl text-center text-yellow-400 font-bold">
    コントリビュートすればいい
  </div>

  <div class="mt-12 text-xl">
      この辺はkojiさんのセッション<span class="text-green-400">「Flutterコントリビューションのススメ」</span>でもモチベーションや方法論について言及している<br>
  </div>

  <div class="mt-12 text-xl border-2 border-purple-500 p-6 rounded-lg">
    issueやフォーラムでの議論も大事だが、<br>
    <span class="text-purple-400 font-bold">実際にPRを提示するのも同じくらい大事</span>
  </div>
</div>

---

# Flutterのテストカバレッジ

<div class="mt-12 space-y-8">
  <div class="text-2xl">
    特にFlutter本体は<span class="text-green-400 font-bold">テストコードが充実</span>している
  </div>

  <div class="mt-8 text-xl">
    <div class="mb-4">• ユニットテスト</div>
    <div class="mb-4">• 統合テスト</div>
    <div class="mb-4">• E2Eテスト</div>
  </div>

  <div class="mt-12 text-2xl text-center border-2 border-green-500 p-8 rounded-lg">
    振る舞い的な正しさは<span class="text-green-400 font-bold">テストでかなり担保できる</span>
  </div>
</div>

---
transition: fade-out
---

# 自分たちでコントロールできる

<div class="mt-12 space-y-8">
  <div class="text-2xl">
    もしもFlutterに不具合があれば<br>
    <span class="text-yellow-400 font-bold">自分たちのコントロールできる範疇で対応可能</span>
  </div>

  <div class="mt-12 text-xl">
    今のところ<br>
    解消されていない不具合で<span class="text-red-400">致命的にアプリ開発ができない</span>みたいなものは無い（個人の感想）
  </div>

  <div class="mt-12 text-xl border-2 border-yellow-500 p-6 rounded-lg">
    <span class="text-yellow-400 font-bold">仮にそのようなものがあるとしたとき</span><br>
    どちらかと言うと<br>
    アーキテクチャ上やりたいことと噛み合っていない可能性がある
  </div>
</div>

---
layout: center
---

# Q. Flutterはどんなプロジェクトに向いてる？

---

# A. UIを自由自在に設計したいとき

<div class="mt-24 flex gap-8 items-center">
  <div class="text-center">
    <div class="inline-block border-4 border-blue-500 p-8 rounded-lg">
      <div class="text-3xl font-bold text-blue-400 mb-6">Flutter</div>
      <div class="text-xl">UIツール<br>キット</div>
      <div class="text-sm mt-2 opacity-75">自由なUI設計</div>
    </div>
  </div>

  <div class="text-center text-4xl text-yellow-400">+</div>

  <div class="text-center">
    <div class="inline-block border-4 border-green-500 p-8 rounded-lg">
      <div class="text-3xl font-bold text-green-400 mb-6">Dart</div>
      <div class="text-xl">ロジック<br>記述</div>
      <div class="text-sm mt-2 opacity-75">同じ言語で実装</div>
    </div>
  </div>

  <div class="text-center text-4xl text-yellow-400">=</div>

  <div class="text-center border-4 border-purple-500 p-8 rounded-lg">
    <div class="text-2xl font-bold text-purple-400">気楽にアプリ開発</div>
    <div class="text-lg mt-4">UI + ロジックを一貫して構築できる</div>
  </div>
</div>

---

# A. 組んだアプリを別のプラットフォームで展開したいとき

  <div class="mt-12 text-2xl">
    <div class="ml-8">
      • KMPやフルネイティブと比較すると<span class="text-red-400 font-bold">明確な強み</span><br><br>
      • React Nativeに関しては同じような構造なので同様のメリットを得られる<br><br>
    </div>
  </div>

---
transition: fade-out
layout: center
---

# 結論

---
transition: fade-out
---

# Flutter is NOT DEAD.

<div class="mt-12 space-y-8">
  <div class="text-3xl text-center text-yellow-400 font-bold">
    Flutterは別にまだ死んではいない
  </div>

  <div class="mt-8 text-xl border-2 border-green-500 p-8 rounded-lg">
    <div class="text-center">
      「このプロジェクトで<span class="text-green-400 font-bold">やりたいことがある</span>」<br>
      ↓<br>
      「だから<span class="text-green-400 font-bold">この技術を採用する</span>」
    </div>
  </div>

  <div class="mt-8 text-xl border-2 border-green-500 p-8 rounded-lg">
    <div class="text-xl text-center">
      更にFlutterに対して、コミュニティでやれることもまだまだある
    </div>
  </div>
</div>

---
transition: fade-out
layout: center
---

# FAQ

---
transition: fade-out
---

# Q. LLM（AI）エージェントはTypeScriptのほうが優れてますよね？

<div class="mt-12 space-y-8">
  <div class="text-2xl">
    A. <span class="text-yellow-400">そうかもしれない</span>
  </div>

  <div class="mt-8 text-xl">
    しかし、エージェントが犯す
    <span class="text-blue-400 font-bold">TypeScriptの記述ミス</span>と<span class="text-green-400 font-bold">Dartの記述ミス</span>に、
    <span class="text-red-400 font-bold">致命的な差はあるか？</span>
  </div>
</div>

---
transition: fade-out
---

# 結局やることは同じ

<div class="mt-12 space-y-8">
  <div class="text-2xl">
    エージェントがエラーを出したら
  </div>

  <div class="mt-8 text-xl">
    <div class="mb-4">1. エージェントに<span class="text-yellow-400 font-bold">追加のコンテキスト</span>を与えて修正してもらう</div>
    <div class="mb-4">または</div>
    <div class="mb-4">2. <span class="text-yellow-400 font-bold">自分の手で直す</span></div>
  </div>

  <div class="mt-12 text-4xl text-center border-2 border-purple-500 p-8 rounded-lg">
    <span class="text-purple-400 font-bold">そう、導くのはあなた</span>
  </div>
</div>

---
transition: fade-out
layout: center
---

# Thank you Flutter team!

---
transition: fade-out
layout: center
---

<div class="space-y-8">
  <img src="./images/sun-qr.png" class="h-80 mx-auto">
  <div class="mt-12 text-xl opacity-75">
    <a href="https://sun-asterisk.com/recruitment/mid-career/">https://sun-asterisk.com/recruitment/mid-career/</a>
  </div>
</div>
