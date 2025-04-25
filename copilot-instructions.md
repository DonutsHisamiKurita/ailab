# コーディングガイドライン

## アニメーション

- 画面外の要素のループアニメーションやスクロールイベントは基本的に削除してください。

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