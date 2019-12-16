# Expo + Next.js Router + React Navigation 🥳

A set of hooks that copy the the `react-navigation` API that you're already using with an Expo app, and make it work with `next/router`.

This aims to solve the challenge with managing navigation with the awesome [expo/next-js integration](https://docs.expo.io/versions/latest/guides/using-nextjs/).

## Install

```es6
yarn add expo-next-react-navigation
```

## Table of contents

- [Set up](#set-up)
- Hooks
  - [`useRouting`](#userouting)
  - [`useFocusEffect`](#useFocusEffect)
- Components
  - [`Link`](#link)

## Set up

**Step 0. Install next with expo:**

- Init: `expo init` (or `npx create-next-app`)

- Install: `yarn add @expo/next-adapter`

- Configure: `yarn next-expo`

- Start: `yarn next dev`

_I recommend becoming more familiar `next`'s architecture with `expo`. Follow the docs on the [Expo docs](https://docs.expo.io/versions/latest/guides/using-nextjs/) or see [this article](https://dev.to/evanbacon/next-js-expo-and-react-native-for-web-3kd9) by Evan Bacon if you're curious._

**Step 1. Edit/create next.config.js**

```bash
yarn add next-compose-plugins next-fonts next-images next-transpile-modules
```

Next, edit `next.config.js` to look something like this:

```es6
const { withExpo } = require('@expo/next-adapter')
const withFonts = require('next-fonts')
const withImages = require('next-images')
const withTM = require('next-transpile-modules')
const withPlugins = require('next-compose-plugins')

module.exports = withPlugins(
  [
    [
      withTM,
      {
        transpileModules: ['expo-next-react-navigation'],
      },
    ],
    withFonts,
    withImages,
    [withExpo, { projectRoot: __dirname }],
  ],
  {
    // ...
  }
)
```

**All done! Run `yarn next dev` & open [http://localhost:3000](http://localhost:3000)** 👻

- Take a look at the [next tutorial](https://nextjs.org/learn/basics/create-dynamic-pages) for creating pages.

_You can add other packages that need transpiling to the `transpileModules` array. See [this post](https://forums.expo.io/t/next-js-expo-web-syntaxerror-unexpected-token-export-with-npm-module/31127) for details._

## Usage

Replace the following instances in your code after installation and setup:

### `useNavigation` 👉 `useRouting`

```diff
-import { useNavigation } from 'react-navigation-hooks'
+import { useRouting } from 'expo-next-react-navigation'
```

### `useLayoutEffect`

```diff
-import { useLayoutEffect } from 'react-navigation-hooks'
+import { useLayoutEffect } from 'expo-next-react-navigation'
```

### `<TouchableOpacity />` 👉 `<Link />`

```diff
-import { TouchableOpacity } from 'react-native'
+import { Link } from 'expo-next-react-navigation'

-<TouchableOpacity onPress={() => navigate({ routeName: 'chat' })}>
-  <Text>Go</Text>
- </TouchableOpacity>
+<Link routeName="chat" params={{ roomId: 'hey!' }}>
+  <Text>Go</Text>
+</Link>
```

## `useRouting`

React hook that wraps `useNavigation` (from react-navigation) hook and `useRouter` (from next-router).

It follows the [same API](https://reactnavigation.org/docs/en/next/use-navigation.html) as `useNavigation`.

### Dynamic routing

```
import { useRouting } from 'expo-next-react-navigation`

export default () => {
  const { navigation, push, getParam, goBack } = useRouting()

  // ...
}
```

#### `navigate`

Only argument is a dictionary with these values

- `routeName`: string, required
- `params`: optional dictionary
- `web`: Optional dictionary with added values for web, following the API from `next/router`'s `Router.push` [function](https://nextjs.org/docs#with-url-object-1).
- `path`: (optional) Fulfills the same value as `pathname` from `next/router`, overriding the `routeName` field. If you set this to `/cars`, it will navigate to `/cars` instead of the `routeName` field. As a result, it will load the file located at `pages/cars.js`.
- `as`: (optional) If set, the browser will show this value in the address bar. Useful if you want to show a pretty/custom URL in the address bar that doesn't match the actual path. Unlike the `path` field, this does not affect which route you actually go to.

**Example:** Navigate to a user

```es6
export default function Home() {
  const { navigate } = useRouting()

  // goes to yourdomain.com/user?id=chris
  const onPress = () => navigate({ routeName: 'user', params: { id: 'chris' } })

  // goes to `yourdomain.com/user/chris`
  const navigateCleanLink = () =>
    navigate({
      routeName: 'user',
      params: { id: 'chris' },
      web: { as: `/user/chris` },
    })

  // 'profile' path overrides 'user' on web, so it uses the pages/profile.js file
  // shows up as yourdomain.com/@chris
  // ...even though it navigates to yourdomain.com/profile?id=chris?color=blue`
  const navigateCleanLinkWithParam = () =>
    navigate({
      routeName: 'user',
      params: { id: 'chris', color: 'blue' }, // accessed with getParam in the next screen
      web: { as: `/@chris`, path: 'profile' },
    })
}
```

This follows the next pattern of [dynamic routing](https://nextjs.org/learn/basics/clean-urls-with-dynamic-routing). You'll need to create a `pages/user/[id].tsx` file.

**Thoughts on the `web` field:**

`web` can help provide cleaner urls (`user/mike` instead of `user?id=mike`).

Also, navigation patterns on mobile can be different than web.

For instance, imagine you have a tab navigator, and one tab has a nested stack navigator with an inbox screen and a chat room screen. If you navigate from a notifications tab to this tab, and a chat room is already open, you probably want that chat room to stay open on mobile. Only if you press the tab button a second time will it pop back to the inbox screen.

This may not be the case on `web`. Web navigation patterns on web may lead you to want to open the inbox directly, instead of the open chat screen. This example could look something like this:

```es6
navigate({
  routeName: 'inboxStack',
  web: {
    path: 'inbox',
  },
})
```

I've also considered letting the `web` field take a `dynamic` parameter like this `chat/:roomId`:

```es6
// goes to `yourdomain.com/chat/chris` and still passes `chris` as a `roomId` param
const navigateCleanLink = () =>
  navigate({
    routeName: 'chat',
    params: { roomId: 'chris' },
    web: { dynamic: `chat/:roomId` },
  })

// goes to yourdomain.com/chat?roomId=chris
const onPress = () =>
  navigate({
    routeName: 'chat',
    params: { roomId: 'chris' },
  })
```

But that's not added yet. For now, the same is achieved by doing this:

```es6
const roomId = 'chris'
const navigateCleanLink = () =>
  navigate({
    routeName: 'chat',
    params: { roomId },
    web: { path: `chat/${roomId}` },
  })
```

#### `getParam`

[Same API](https://reactnavigation.org/docs/en/navigation-prop.html#getparam-get-a-specific-param-value-with-a-fallback) as `getParam` from react-navigation.

Similar to `query` from `next/router`, except that it's a function to grab the values.

**pages/user/[id].tsx**

Imagine you navigated to `yourdomain.com/user/chris` on web using the example above.

```es6
export default function User() {
  const { getParam } = useRouting()

  const id = getParam('id') // chris

  // do something with the id
}
```

## `useFocusEffect`

See [react navigation docs](https://reactnavigation.org/docs/en/next/use-focus-effect.html#docsNav). On web, it simply replaces the focus effect with a normal effect hook. On mobile, it is the exact react navigation hook.

Make sure to use [useCallback](https://reactjs.org/docs/hooks-reference.html#usecallback) as seen in the example.

```es6
import { useFocusEffect } from 'expo-next-react-navigation'

export default ({ userId }) => {
  useFocusEffect(
    useCallback(() => {
      const unsubscribe = API.subscribe(userId, user => setUser(user))

      return () => unsubscribe()
    }, [userId])
  )

  return <Profile userId={userId} />
}
```

## `Link`

The following will use the `chat` route in react navigation.

However, it will use the `pages/room.tsx` file for nextjs. Also, it will show up as `domain.com/messages` in the address bar.

Optionally accepts a `nextLinkProps` prop dictionary and `touchableOpacityProps` dictionary as well.

```es6
export default function Button() {
  return (
    <Link
      routeName="chat"
      params={{ roomId: '12' }}
      web={{
        path: '/room',
        as: 'messages',
      }}
    >
      <Text>Chat in room 12</Text>
    </Link>
  )
}
```

**Required props**:

- `routeName`: string, see `useRouting().navigate` docs.

**Optional props**

- `web`: dictionary, see `useRouting().navigate` docs.

- `touchableOpacityProps`: extends React Native's `TouchableOpacity` props.

- `nextLinkProps`: extends `next/router`'s [Link props](https://nextjs.org/docs#with-link).

## Other shout outs

### `nextjs-progressbar`

I think this is an awesome package for adding a loading progress bar to your `next` pages. It's super easy. Check it out.

Link: [https://www.npmjs.com/package/nextjs-progressbar](https://www.npmjs.com/package/nextjs-progressbar)

```es6
yarn add nextjs-progressbar
```

**pages/\_app.js**

```es6
import React from 'react'
import App from 'next/app'
import NextNprogress from 'nextjs-progressbar'

class MyApp extends App {
  render() {
    const { Component, pageProps } = this.props
    return (
      <>
        <NextNProgress
          color="#29D"
          startPosition="0.3"
          stopDelayMs="200"
          height="3"
        />
        <Component {...pageProps} />
      </>
    )
  }
}

export default MyApp
```
