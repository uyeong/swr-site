# データのプリフェッチ

## トップレベルのページデータ

SWR のデータをプリフェッチする方法はたくさんあります。トップレベルのリクエストでは、 [`rel="preload"`](https://developer.mozilla.org/en-US/docs/Web/HTML/Preloading_content) を強く推奨します。

```html
<link rel="preload" href="/api/data" as="fetch" crossorigin="anonymous">
```

HTMLの `<head>` の中に入れるだけです。簡単、速い、そしてネイティブです。

JavaScriptのダウンロードが開始される前であっても、HTMLの読み込み時にデータをプリフェッチします。同じ URL で受信したすべてのフェッチリクエストは、その結果を再利用します（もちろん、 SWR を含む）。

## プログラムによるプリフェッチ

SWR は `preload` というデータをプログラマブルにプリフェッチして結果をキャッシュに保存する API を提供しています。`preload` は `key` と `fetcher` を引数として受け取ります。

`preload` は React の外からも呼ぶことが可能です。

```jsx
import { useState } from 'react'
import useSWR, { preload } from 'swr'

const fetcher = (url) => fetch(url).then((res) => res.json())

// User コンポーネントがレンダリングされる前にプリロードします
// これによりウォーターフォール問題の発生を避けられます
// また、ボタンやリンクにホバーされたタイミングでプリロードを開始することもできます
preload('/api/user', fetcher)

function User() {
  const { data } = useSWR('/api/user', fetcher)
  ...
}

export default function App() {
  const [show, setShow] = useState(false)
  return (
    <div>
      <button onClick={() => setShow(true)}>Show User</button>
      {show ? <User /> : null}
    </div>
  )
}
```

React のレンダリングツリー内においては, `preload` はイベントコールバックやエフェクトの中で利用可能です。

```jsx
function App({ userId }) {
  const [show, setShow] = useState(false)

  // エフェクトの中でプリロードする
  useEffect(() => {
    preload('/api/user?id=' + userId, fetcher)
  }, [useId])

  return (
    <div>
      <button
        onClick={() => setShow(true)}
        {/* イベントコールバックの中でプリロードする */}
        onHover={() => preload('/api/user?id=' + userId, fetcher)}
      >
        Show User
      </button>
      {show ? <User /> : null}
    </div>
  )
}
```

Next.js の [ページプリフェッチ](https://nextjs.org/docs/api-reference/next/router#routerprefetch) などの技術と合わせて、次のページとデータの両方を瞬時に読み込むことができるようになります。

サスペンスモードでは特に `preload` を利用してウォーターフォール問題を避けた方がいいでしょう。

```jsx
import useSWR, { preload } from 'swr'

// レンダリング前に呼ぶべき
preload('/api/user', fetcher);
preload('/api/movies', fetcher);

const Page = () => {
  // 下記の useSWR はレンダリングを中断しますが、`/api/user` と`/api/movies` に対するリクエストは `preload` によって開始されています
  // そのため、ウォーターフォール問題は起きません
  const { data: user } = useSWR('/api/user', fetcher, { suspense: true });
  const { data: movies } = useSWR('/api/movies', fetcher, { suspense: true });
  return (
    <div>
      <User user={user} />
      <Movies movies={movies} />
    </div>
  );
}
```

## 事前データ

もし既に存在しているデータを SWR にキャッシュしたい場合は、 `fallbackData` オプションを使うことができます。たとえば：

```jsx
useSWR('/api/data', fetcher, { fallbackData: prefetchedData })
```

もし SWR がまだデータを取得していない場合、このフックは初期値として `prefetchedData` を返します。

また `SWRConfig` と `fallback` オプションを使用して、すべての SWR フックと複数のキーに対してこれを構成することもできます。詳しくは、 [Next.js の SSG と SSR](/docs/with-nextjs) をチェックしてください。
