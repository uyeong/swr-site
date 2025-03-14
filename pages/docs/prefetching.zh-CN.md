# 预请求数据

## 顶级页面数据

有很多方法可以为 SWR 预请求数据。对于顶级请求，强烈推荐 [`rel="preload"`](https://developer.mozilla.org/en-US/docs/Web/HTML/Preloading_content)

```html
<link rel="preload" href="/api/data" as="fetch" crossorigin="anonymous">
```

只需把它放在HTML的 `<head>` 里即可。简单、快速、原生。

它将在 HTML 加载时预请求数据，甚至是在 JavaScript 开始下载之前。使用相同 URL 的所有传入 fetch 请求都将重用结果(当然包括SWR)。

## 手动预请求

SWR provides the `preload` API to prefetch the resources programmatically and store the results in the cache. `preload` accepts `key` and `fetcher` as the arguments.

You can call `preload` even outside of React.

```jsx
import { useState } from 'react'
import useSWR, { preload } from 'swr'

const fetcher = (url) => fetch(url).then((res) => res.json())

// Preload the resource before rendering the User component below,
// this prevents potential waterfalls in your application.
// You can also start preloading when hovering the button or link, too.
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

Within React rendering tree, `preload` is also available to use in event handlers or effects.

```jsx
function App({ userId }) {
  const [show, setShow] = useState(false)

  // preload in effects
  useEffect(() => {
    preload('/api/user?id=' + userId, fetcher)
  }, [useId])

  return (
    <div>
      <button
        onClick={() => setShow(true)}
        {/* preload in event callbacks */}
        onHover={() => preload('/api/user?id=' + userId, fetcher)}
      >
        Show User
      </button>
      {show ? <User /> : null}
    </div>
  )
}
```

配合 Next.js 的 [页面预加载](https://nextjs.org/docs/api-reference/next/router#routerprefetch)，你将能立即加载下一页和数据。

In Suspense mode, you should utilize `preload` to avoid waterfall problems.

```jsx
import useSWR, { preload } from 'swr'

// should call before rendering
preload('/api/user', fetcher);
preload('/api/movies', fetcher);

const Page = () => {
  // The below useSWR hooks will suspend the rendering, but the requests to `/api/user` and `/api/movies` have started by `preload` already,
  // so the waterfall problem doesn't happen.
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

## 数据预填充

如果你想在 SWR 缓存中预填充已经存在的数据，你可以使用 `fallbackData` 选项，例如：

```jsx
useSWR('/api/data', fetcher, { fallbackData: prefetchedData })
```

当 SWR 还没有获取此次数据的时候， 这个 hook 将返回 `prefetchedData` 作为 fallback 。 

你也可以为所有的 SWR hooks 和不同的 key 配置 `<SWRConfig>` 和它的 `fallback` 选项。 查看 [Next.js SSG 和 SSR](/docs/with-nextjs) 了解更多详细信息。
