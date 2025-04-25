# Coding Guidelines

## HTML

#### 原則

- W3C標準準拠: W3C勧告に沿った、有効なHTMLコードを記述すること。
- セマンティクス: 要素の持つ意味を正しく理解し、適切なセマンティック要素を選択・使用すること。意味を持たない、あるいは内容の伴わない冗長な `div` 要素などの使用は避ける。
- アクセシビリティ: スクリーンリーダー利用者やキーボード操作者を含む全てのユーザーがコンテンツを利用できるよう、代替テキスト (`alt`) の指定、フォームコントロールとラベルの関連付け、適切な見出し構造など、基本的なアクセシビリティ要件を満たすこと。
- パフォーマンス: 不必要な要素やマークアップを避け、画像の最適化や遅延ロード (`loading="lazy"`) の活用など、読み込み速度の向上に配慮すること。

#### 準拠確認
上記原則に沿っているかは、主にプロジェクトに導入されている静的解析ツール [markuplint](https://markuplint.dev/ja/) によって確認されます。markuplint の出力する Lint エラーや警告は、修正すべき事項として扱ってください。

## 💡 Tips

### 画面外要素の不要な処理停止

#### 原則
ループアニメーションやスクロールイベントなど、要素が画面外にある間も継続的にリソースを消費する処理は、原則として要素が画面内に表示されている間のみ実行するようにします。

#### 理由
画面外で要素が動き続けたり、イベント監視が行われたりすると、ユーザーに見えない部分で不要なCPUリソースを消費し、ページのパフォーマンス低下につながる可能性があるためです。

#### 対応方法
`Intersection Observer API` を活用し、要素の画面内への出入りを検知して処理（クラスの追加/削除、イベントリスナーの追加/削除など）を制御します。

#### 実装例 (アニメーション制御)

要素が画面に入ったら特定のクラスを付与し、出たらクラスを削除する例。

```ts
const target = document.querySelector('.js-hoge') as HTMLElement;

const io = new IntersectionObserver((entries) => {
  entries.forEach((entry) => {
    if (entry.isIntersecting) {
      // 要素が画面に入った場合
      target.classList.add("is-loop"); // アニメーションを開始するクラスを追加
    }
    else {
      // 要素が画面から出た場合
      target.classList.remove("is-loop"); // アニメーションを停止するクラスを削除
    }
  });
});

// 対象要素の監視を開始
io.observe(target);
```

#### 実装例 (スクロールイベント制御)

要素が画面に入ったらスクロールイベントリスナーを追加し、出たら削除する例。

```ts
const target = document.querySelector('.js-hoge') as HTMLElement;

/**
 * @description スクロール時に実行する処理
 */
const onScroll = (): void => {
    // ここにスクロールイベントで実行したい処理を記述...
    console.log("Scrolling inside target element");
}

const io = new IntersectionObserver((entries) => {
  entries.forEach((entry) => {
    if (entry.isIntersecting) {
      // 要素が画面に入った場合
      // スクロールイベントリスナーを追加
      target.addEventListener("scroll", onScroll);
      console.log("Scroll listener added");
    }
    else {
      // 要素が画面から出た場合
      // スクロールイベントリスナーを削除
      target.removeEventListener("scroll", onScroll);
      console.log("Scroll listener removed");
    }
  });
});

// 対象要素の監視を開始
io.observe(target);
```

---

### タッチデバイスにおける `:hover` の挙動制御

#### 問題点
標準的な `:hover` 疑似クラスは、スマートフォンやタブレットなどのタッチデバイスでは、要素をタップした際にホバー時のスタイルが適用され、再度タップするまでその状態が維持されてしまう（いわゆる「hover stuck」問題）。これにより、意図しないスタイルの残存やユーザーエクスペリエンスの低下を招く可能性があります。

#### 対応方法
ホバー操作が可能なデバイスにのみホバー時のスタイルを適用するために、CSSの `@media (any-hover: hover)` メディアクエリを使用します。

#### 実装例

```css
/* タッチデバイスなど、ホバー操作が主でないデバイスではこのスタイルは適用されない */
@media (any-hover: hover) {
  .hoge:hover {
    opacity: 0.5;
    transition: opacity 0.3s ease;
  }
}
```

---

### 狭小スクリーン（375px未満）におけるビューポートの固定

#### 目的
画面幅が非常に狭小な端末（例: 375px未満）に対するきめ細やかなレスポンシブ対応は、実装コストが増大する傾向があります。そのため、特定のブレークポイント未満の画面幅ではビューポートの表示幅を固定することで、対応範囲を限定し開発効率を高めます。

#### 対応方法
JavaScriptを用いて、`window.outerWidth` が特定の幅（例: 375px）を下回る場合に、ビューポートの `content` プロパティを固定値（例: `"width=375"`）に書き換えます。

#### 実装例

```ts
const viewport = document.querySelector('meta[name="viewport"]');

/**
 * @description ウィンドウ幅に応じてビューポートの表示幅を調整・固定する関数
 * 375px以下のウィンドウ幅では、ビューポートをwidth=375に固定します。
 */
export const viewportFix = (): void => {
  // 現在のウィンドウの外部幅を取得
  const outerWidth = window.outerWidth;

  // 375pxより大きい場合は通常のレスポンシブ設定、そうでなければ375pxに固定
  const value = outerWidth > 375 ? "width=device-width,initial-scale=1" : "width=375";

  // 現在のviewport設定と異なる場合のみ更新
  if (viewport?.getAttribute("content") !== value) {
    viewport?.setAttribute("content", value);
    console.log(`Viewport set to: ${value}`); // 確認用ログ
  }
};

// ページロード時およびウィンドウリサイズ時にviewportFix関数を実行
window.addEventListener('resize', viewportFix);
window.addEventListener('load', viewportFix);
```