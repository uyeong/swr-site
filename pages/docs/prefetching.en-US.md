# Prefetching Data

## Top-Level Page Data

There’re many ways to prefetch the data for SWR. For top level requests, [`rel="preload"`](https://developer.mozilla.org/en-US/docs/Web/HTML/Preloading_content) is highly recommended:

```html
<link rel="preload" href="/api/data" as="fetch" crossorigin="anonymous">
```

Just put it inside your HTML `<head>`. It’s easy, fast and native.

It will prefetch the data when the HTML loads, even before JavaScript starts to download. All your incoming fetch requests with the same URL will reuse the result (including SWR, of course).

## Programmatically Prefetch

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

Together with techniques like [page prefetching](https://nextjs.org/docs/api-reference/next/router#routerprefetch) in Next.js, you will be able to load both next page and data instantly.

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

## Pre-fill Data

If you want to pre-fill existing data into the SWR cache, you can use the `fallbackData` option. For example:

```jsx
useSWR('/api/data', fetcher, { fallbackData: prefetchedData })
```

If SWR hasn't fetched the data yet, this hook will return `prefetchedData` as a fallback. 

You can also configure this for all SWR hooks and multiple keys with `<SWRConfig>` and the `fallback` option. Check [Next.js SSG and SSR](/docs/with-nextjs) for more details.
