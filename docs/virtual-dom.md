# 仮想 DOM と React

仮想 DOM とは、プログラム内で仮想的に作成された DOM のこと。

実際の DOM とは区別される。

React などの JS フレームワークは仮想 DOM の仕組みを用いてレンダリングと描画を最適化している。

## 比較ロジック

- Javascript などから情報を受け取る
- 変更を適用した仮想 DOM ツリーと適用する前の DOM ツリーを比較する
- 変更があればリアル DOM に反映する

## レンダリングと描画の違い

React においては変更差分を検出するために仮想 DOM を構築することをレンダリングと呼ぶ。

レンダリングした結果、実際の DOM に反映する必要があることが分かったら、変更差分を描画する。

レンダリングをレンダーフェース、描画をコミットフェーズという。

## 注意点

親コンポーネントがレンダリングされると、その中のすべての子コンポーネントを再帰的にレンダリングする。

そのため、レンダリングをスキップさせるようにする実装を意図的にする必要がある。

## React.memo

コンポーネントをメモ化するメソッド。
再レンダリングを防ぐために使用する。

使用例:

- 親のコンポーネントがレンダリングされたとき、子のコンポーネントはレンダリングさせないようにする
- props を変更していないときにレンダリングをスキップしたい

毎回レンダリングしていたら動作が遅くなる原因になるのでできるならスキップしたい。

`React.memo` によって props が等価であることを確認できればスキップするようにできる。

props の比較は javascript の Object.is()による比較を行っている。

そのため、オブジェクトの参照が別々の場合は同一の内容で合っても false を返す。

例えばコールバック関数をそのまま渡している場合。

```tsx title="parent.tsx"
type childProp = {
  onClick: () => void;
};

const Child = React.memo((props: childProp) => {
  console.log("child rendered");
  return (
    <div>
      再レンダリングさせたくないコンポーネント:
      <button onClick={props.onClick}>Child Button</button>
    </div>
  );
});

export default function Parent() {
  // highlight-next-line
  const onClick = () => console.log("clicked");
  const [count, setCount] = React.useState(0);
  return (
    <>
      <div>
        <p>clickedCount: {count}</p>
        <button onClick={() => setCount(count + 1)}>Click!</button>
      </div>
      <Child onClick={onClick} />
    </>
  );
}
```

この場合はメモ化していても再レンダリングされる。

レンダリングのたびに参照が異なる関数を作っているから。

この場合は`useCallback`を使用する。

```tsx title="parent.tsx"
type childProp = {
  onClick: () => void;
};

const Child = React.memo((props: childProp) => {
  console.log("child rendered");
  return (
    <div>
      再レンダリングさせたくないコンポーネント:
      <button onClick={props.onClick}>Child Button</button>
    </div>
  );
});

export default function Parent() {
  // 第2引数は依存している要素
  // highlight-next-line
  const onClick = React.useCallback(() => console.log("clicked"), []);
  const [count, setCount] = React.useState(0);
  return (
    <>
      <div>
        <p>clickedCount: {count}</p>
        <button onClick={() => setCount(count + 1)}>Click!</button>
      </div>
      <Child onClick={onClick} />
    </>
  );
}
```

`useCallback`はメモ化したコールバック関数を作れる。
これによって再レンダリングをスキップできる。

[code sandbox link](https://codesandbox.io/s/fervent-star-s13fp)
