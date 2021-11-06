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

# Flutterの仕組み

- プラットフォームからDart VMを起動するアレコレ

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

# ネイティブとの連携

- MethodChannel
- Dart → Platform
- Platform → Dart

---

# バックグラウンド実行

- 図を出す
- コードを追って説明