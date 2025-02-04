# postcss-pxtorem

**English** | [中文](./README-zh.md)

A plugin for [PostCSS](https://github.com/ai/postcss) that generates rem units from pixel units.

## ✨ New Features

- override any `postcss-pxtorem` options in css.
- ignore next line in css.
- **compatible with vite**.

## 🔧 Install

```bash
pnpm install postcss @minko-fe/postcss-pxtorem -D
```

## ✍️ Usage

> Pixels are the easiest unit to use (_opinion_). The only issue with them is that they don't let browsers change the default font size of 16. This script converts every px value to a rem from the properties you choose to allow the browser to set the font size.

### postcss.config.js

#### example

```js
// postcss.config.js
import pxtorom from '@minko-fe/postcss-pxtorem'

export default {
  plugins: [
    pxtorom({
      rootValue: 16,
      selectorBlackList: ['some-class'],
      propList: ['*'],
      atRules: ['media'],
      // ...
    }),
  ],
}
```

### options

| Name                | Type                                                         | Default         | Description                                                  |
| ------------------- | ------------------------------------------------------------ | --------------- | ------------------------------------------------------------ |
| rootValue           | `number` \| `((input: Input) => number)`                     | 16              | Represents the root element font size or returns the root element font size based on the [`input`](https://api.postcss.org/Input.html) parameter |
| unitToConvert       | `string`                                                     | `px`            | unit to convert, by default, it is px                        |
| unitPrecision       | `number`                                                     | 5               | The decimal numbers to allow the REM units to grow to.       |
| propList            | `string[]`                                                   | `['*']`         | The properties that can change from px to rem. Refer to: [propList](#propList) |
| selectorBlackList   | `(string \| RegExp)[]`                                       | []              | The selectors to ignore and leave as px. Refer to: [selectorBlackList](#selectorBlackList) |
| replace             | `boolean`                                                    | true            | Replaces rules containing rems instead of adding fallbacks.  |
| atRules             | `boolean` \| `string[]`                                      | false           | Allow px to be converted in at-rules. Refer to [At-rule](https://developer.mozilla.org/en-US/docs/Web/CSS/At-rule) |
| minPixelValue       | `number`                                                     | 0               | Set the minimum pixel value to replace.                      |
| exclude             | `string` \| `RegExp` \| `((filePath: string) => boolean) \| null` | /node_modules/i | The file path to ignore and leave as px. Refer to: [exclude](#exclude) |
| include             | `string` \| `RegExp` \| `((filePath: string) => boolean)` \| `null` | null            | The file path to convert px to rem, in contrast to `exclude`, have higher priority than `exclude`. Same rules as `exclude` |
| disable             | `boolean`                                                    | false           | disable plugin                                               |
| convertUnitOnEnd    | `ConvertUnit` \| `ConvertUnit[]` \| false \| null            | null            | convert unit when plugin process end                         |
| convertInMediaQuery | `boolean`                                                    | true            | convert unit in media queries                                |

#### propList

- Values need to be exact matches.
- Use wildcard `*` to enable all properties. Example: `['*']`
- Use `*` at the start or end of a word. (`['*position*']` will match `background-position-y`)
- Use `!` to not match a property. Example: `['*', '!letter-spacing']`
- Combine the "not" prefix with the other prefixes. Example: `['*', '!font*']`

#### selectorBlackList

- If value is string, it checks to see if selector contains the string.
  - `['body']` will match `.body-class`
- If value is regexp, it checks to see if the selector matches the regexp.
  - `[/^body$/]` will match `body` but not `.body`

#### exclude

- If value is string, it checks to see if file path contains the string.
  - `'exclude'` will match `\project\postcss-pxtorem\exclude\path`
- If value is regexp, it checks to see if file path matches the regexp.
  - `/exclude/i` will match `\project\postcss-pxtorem\exclude\path`
- If value is function, you can use exclude function to return a true and the file will be ignored.
  - the callback will pass the file path as a parameter, it should returns a Boolean result.
  - `function (file) { return file.includes('exclude') }`

## ✨ About new features

### ⚙️ Dynamically set plugin options in css

#### disable plugin

```css
/* pxtorem?disable=true */
.rule {
  font-size: 15px; // 15px
}
```

#### set rootValue

```css
/* pxtorem?rootValue=32 */
.rule {
  font-size: 30px; // 0.9375rem
}
```

🌰 The above is just a simple example, you can set any of the options supported by `postcss-pxtorem` in the css file

You may have seen that the css comment is very much like the browser url?😼.
That's right. For the specification, just refer to: [query-string](https://github.com/sindresorhus/query-string)

#### example

```css
/* pxtorem?disable=false&rootValue=32&propList[]=*&replace=false&selectorBlackList[]=/some-class/i */
```

### disable the next line in css file

```css
.rule {
  /* pxtorem-disable-next-line */
  font-size: 15px; // 15px
}
```

If you write `15PX` (as long as it's not `px`), the plugin also ignores it, because `unitToConvert` defaults to `px`
If you want to use `PX` to ignore and want the final unit to be `px`, you can:

```js
// postcss.config.js
import pxtorem from '@minko-fe/postcss-pxtorem'

export default {
  plugins: [
    pxtorem({
      convertUnitOnEnd: {
        sourceUnit: /[p|P][x|X]$/,
        targetUnit: 'px',
      },
    }),
  ],
}
```

## use with flexible js

### example

```ts
;(function () {
  if (typeof window === 'undefined') return

  const maxWidth = 1024
  const uiWidth = 375

  function resize() {
    let width = window.innerWidth

    if (width > window.screen.width) {
    } else {
      if (width >= maxWidth) {
        width = maxWidth
      }
      document.documentElement.style.fontSize = `${(width * 16) / uiWidth}px`
    }
  }

  resize()

  let timer: NodeJS.Timer
  const interval = 1000

  window.addEventListener('resize', () => {
    clearTimeout(timer)
    timer = setTimeout(resize, interval)
  })
  window.addEventListener('pageshow', (e) => {
    if (e.persisted) {
      clearTimeout(timer)
      timer = setTimeout(resize, interval)
      resize()
    }
  })
})()
```

## ❤️ Thanks

[postcss-pxtorem](https://github.com/cuth/postcss-pxtorem)

[@tcstory/postcss-px-to-viewport](https://github.com/tcstory/postcss-px-to-viewport)

## 👀 Related

A CSS post-processor that converts px to viewport: [postcss-pxtoviewport](https://github.com/hemengke1997/postcss-pxtoviewport)

## 💕 Support

**If this has helped you, please don't hesitate to give a STAR, thanks! 😎**
