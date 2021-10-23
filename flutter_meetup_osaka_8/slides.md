---
theme: default
highlighter: shiki
drawings:
  persist: false
fonts:
  sans: 'Kiwi Maru, Helvetica Neue'
---

# 実践 Overlay

Overlayを使ってみる

---

# 誰?

<div class="title" >
  <h3>Kurogoma4D</h3>
  <img class="w-24 h-24 rounded-3xl" src="https://pbs.twimg.com/profile_images/1289002876430213120/G2A564li_400x400.jpg"/>
</div>

- <span class="text-green-300">Cross-platform Mobile App Engineer</span> on CBcloud株式会社
- Twitter: @Krgm4D
- https://scrapbox.io/kurogoma4d-lab/

<div class="mt-20">あんまり役に立たない開発が好きです</div>

<style>
  h3 {
    line-height: 4em !important;
  }

  li {
    line-height: 2.5em !important;
  }

  .title {
    display: flex;
    justify-content: space-between;
  }
</style>

---
layout: center
---

# Overlay

---

<div grid="~ cols-2">
  <div>
    <p>package:in_app_notification</p>
    <Tweet id="1401858448170512388" scale="0.7"/>
  </div>

  <div>
    <p>Fleet風エディタ</p>
    <Tweet id="1424027276358733824" scale="0.6"/>
  </div>
</div>


---

# Overlayって何なんだ

#### 要するに
`WidgetsApp` で普段操作するUIの<span class="text-red-300">手前側にWidgetを表示する仕組み</span>

<style>
  h4 {
    margin-top: 4em !important;
  }
</style>

---

# 使い方

```dart {all|8-14|15|7|all}
class _ContentsState extends State<Contents> {
  OverlayEntry? entry;
  final random = math.Random();

  void tapped() {
    final percentage = random.nextInt(100);
    entry?.remove();
    entry = OverlayEntry(
      builder: (context) => Positioned(
        top: 32,
        left: MediaQuery.of(context).size.width * percentage / 100,
        child: Material(child: Text('$percentage')),
      ),
    );
    Navigator.of(context).overlay?.insert(entry!);
  }

  @override
  Widget build(BuildContext context) {
    // ボタンをタップして `tapped()` を呼び出すコード
  }
}
```

---

<iframe class="w-full h-full" src="https://dartpad.dev/embed-flutter.html?id=9527ad28dea871287beec45371c1557c&theme=dark&null_safety=true"></iframe>

---

# 外から状態を変更する

```dart {all|3|9-13}
class _ContentsState extends State<Contents> {
  OverlayEntry? entry;
  Offset mouse = Offset.zero;

  // ...

  @override
  Widget build(BuildContext context) {
    return MouseRegion(
      onHover: (event) {
        setState(() => mouse = event.localPosition);
        entry?.markNeedsBuild();
      },
      child: Scaffold(
        // ...
      ),
    );
  }
}
```

---

<iframe class="w-full h-full" src="https://dartpad.dev/embed-flutter.html?id=f1523f90cc2526c525b39e9fd62557f5&theme=dark&null_safety=true"></iframe>

---

# まとめ

- Overlayを使うには、`OverlayEntry` を生成して `Navigator` 経由で表示する
- `OverlayEntry` の `builder` 直下には `Positioned` を置くことができる
  - 位置やサイズの調整が可能
- 削除するには `remove()` メソッドを呼び出す
- Overlayの外から状態を更新するには、`markNeedsBuild()` メソッドを呼び出す

<style>
  li {
    line-height: 2.5em !important;
  }

  code {
    color: #FFE082;
  }
</style>

---
layout: two-cols
---

# ちなみに

#### https://pub.dev/packages/flutter_portal

<div class="mt-8">Provider/Riverpodで有名なRemi氏のパッケージ</div>
<p>Overlayを宣言的に扱うことができる</p>

`Positioned` を挟むことができなかったので `MouseRegion` の例は再現できず😢

::right::

```dart
class _ContentsState extends State<Contents> {
  final random = math.Random();
  int percentage = 0;
  bool showingOverlay = false;

  void tapped() => setState(() {
        percentage = random.nextInt(100);
        showingOverlay = true;
      });

  @override
  Widget build(BuildContext context) {
    return PortalEntry(
      visible: showingOverlay,
      childAnchor: Alignment.topCenter,
      portalAnchor: Alignment.topCenter,
      portal: Material(child: Text('$percentage')),
      child: Scaffold(
        body: // ...
      ),
    );
  }
}
```

<style>
  h4 {
    margin-top: 4em !important;
  }

  code {
    color: #FFE082;
  }
</style>
