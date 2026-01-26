This is a [Next.js](https://nextjs.org) project bootstrapped with [`create-next-app`](https://nextjs.org/docs/app/api-reference/cli/create-next-app).

## Getting Started

First, run the development server:

```bash
npm run dev
# or
yarn dev
# or
pnpm dev
# or
bun dev
```

Open [http://localhost:3000](http://localhost:3000) with your browser to see the result.

## Memory Leak Issue

Use Node.js version **24.13.0**.

Build the project:

```shell
npm run build
```

Run production build with `--inspect` option:

```shell
NODE_OPTIONS=--inspect ./node_modules/.bin/next start
```

Follow http://localhost:3000/ open Node.js DevTools and make an initial memory snapshot (do GC before). Then, to emulate request cancellation, you can run the snippet in the browser console:

```js
const ATTEMPT_COUNT = 50
const ATTEMPT_TIMEOUT_MS = 10
const REQUEST_COUNT = 10
const REQUEST_TIMEOUT_MS = 500

for await (const attemptIndex of Array.from({ length: ATTEMPT_COUNT }).keys()) {
  for (const requestIndex of Array.from({ length: REQUEST_COUNT }).keys()) {
    fetch('http://localhost:3000/', {
      signal: AbortSignal.timeout(REQUEST_TIMEOUT_MS),
    })
  }

  await new Promise((resolve) => {
    setTimeout(resolve, ATTEMPT_TIMEOUT_MS)
  })
}
```

Make another snapshot after it (call GC before) and compare them. You should see a similar picture:

![Screenshot 2026-01-26 at 17.37.02.png](Screenshot%202026-01-26%20at%2017.37.02.png)

`Node / zlib_memory` chunks won't dissapper, they will appear more and more during testing. If you disable compression in Next.js config with the option `compress` `false`, the issue is gone.
