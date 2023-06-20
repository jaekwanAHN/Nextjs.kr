# Returning response body in Middleware

> Note: In Next.js v13.0.0 you can now respond to Middleware directly by returning a `NextResponse`. For more information, see [Producing a Response](https://nextjs.org/docs/advanced-features/middleware#producing-a-response).

#### Why This Error Occurred

[Middleware](https://nextjs.org/docs/advanced-features/middleware) can no longer produce a response body as of `v12.2+`.

#### Possible Ways to Fix It

Migrate to using `rewrite`/`redirect` to Pages/API Routes handling a response.

#### Explanation

To respect the differences in client-side and server-side navigation, and to help ensure that developers do not build insecure Middleware, Middleware can no longer produce a response body. This ensures that Middleware is only used to `rewrite`, `redirect`, or modify the incoming request (e.g. [setting cookies](https://nextjs.org/docs/advanced-features/middleware#using-cookies)).

The following patterns will no longer work:

```js
new Response('a text value')
new Response(streamOrBuffer)
new Response(JSON.stringify(obj), { headers: 'application/json' })
NextResponse.json()
```

### How to upgrade

For cases where Middleware is used to respond (such as authorization), you should migrate to use `rewrite`/`redirect` to pages that show an authorization error, login forms, or to an API Route.

#### Before

```typescript
// pages/_middleware.ts
import { NextResponse } from 'next/server'
import type { NextRequest } from 'next/server'
import { isAuthValid } from './lib/auth'

export function middleware(request: NextRequest) {
  // Example function to validate auth
  if (isAuthValid(request)) {
    return NextResponse.next()
  }

  return NextResponse.json({ message: 'Auth required' }, { status: 401 })
}
```

#### After

```typescript
// middleware.ts
import { NextResponse } from 'next/server'
import type { NextRequest } from 'next/server'
import { isAuthValid } from './lib/auth'

export function middleware(request: NextRequest) {
  // Example function to validate auth
  if (isAuthValid(request)) {
    return NextResponse.next()
  }

  request.nextUrl.searchParams.set('from', request.nextUrl.pathname)
  request.nextUrl.pathname = '/login'

  return NextResponse.redirect(request.nextUrl)
}
```