# Coding Guidelines

## 💡 Tips

- 画面外の要素のループアニメーションやスクロールイベントは基本的に削除してください。
    - 画面外でも要素が動き続けていると無駄にCPUを消費するため

```ts
const target = document.querySelector('.js-hoge') as HTMLElement;

const io = new IntersectionObserver((entries) => {
  entries.forEach((entry) => {
    if (entry.isIntersecting) {
      target.classList.add("is-loop");
    }
    else {
      target.classList.remove("is-loop");
    }
  });
});
io.observe(target);
```

```ts
const target = document.querySelector('.js-hoge') as HTMLElement;

const onScroll = (): void => {
    // 処理...
}

const io = new IntersectionObserver((entries) => {
  entries.forEach((entry) => {
    if (entry.isIntersecting) {
      target.addEventListener("scroll", onScroll)
    }
    else {
      target.removeEventListener("scroll", onScroll)
    }
  });
});
io.observe(target);
```

- `:hover`は`@media (any-hover: hover)`でhoverに対応しているか判別してください。
    - 通常の`hover`ではスマホやタブレット等のタッチデバイスでもタッチした際もhover時のスタイルが適用されて、再度タッチするまでhoverスタイルが残り続けてしまう。

```css
@media (any-hover: hover) {
  .hoge:hover {
    opacity: 0.5;
  }
}
```