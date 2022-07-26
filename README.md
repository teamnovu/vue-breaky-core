> **Warning**
> This package is no longer supported.

# vue-breaky-core

[![npm version][npm-version-src]][npm-version-href]
[![License][license-src]][license-href]

<!-- [![npm downloads][npm-downloads-src]][npm-downloads-href] -->
<!-- [![Circle CI][circle-ci-src]][circle-ci-href] -->
<!-- [![Codecov][codecov-src]][codecov-href] -->

> Show Tailwind CSS Breakpoints in Vuejs

[📖 **Release Notes**](./CHANGELOG.md)

## Intro

[DEMO](https://teamnovu.github.io/nuxt-breaky/)

breaky helps you create your responsive designs faster. breaky reads your defined breakpoints within your tailwind config and shows the currently active breakpoint based on your browser window width.

![Demo GIF of window resizing](./example/assets/img/resizing.gif 'Resizing Browser Window')
![Demo GIF of dragging](./example/assets/img/dragging.gif 'Dragging Card to Corners')
![Demo GIF of toggling dark mode](./example/assets/img/toggle-dark-mode.gif 'Toggling between Dark and Light Mode')

## Setup

NOTE: We highly recommend you to use one of the following packages:

- Nuxtjs: [nuxt-breaky](https://github.com/teamnovu/nuxt-breaky)
- Vuejs: [vue-breaky](https://github.com/teamnovu/vue-breaky)

These packages will help you install the breaky core.

If you only want to use the breaky core vue component and want to include it and the tailwind config yourself, feel free to do so but we won't guide your through the whole installation.

Just to get you started: Importing `@teamnovu/vue-breaky-core` imports the core component which you want to include on your page.

## Configuration

### Props

| Prop        	| Type    	| Default        	| Options                                                          | Description                                                              	|
|--------------	|----------	|----------------	|----------------------------------------------------------------- |--------------------------------------------------------------------------- |
| `breakpoints`	| `Object` 	| none, Required 	|                                                                  | The configured breakpoints (screens) as stored in the `tailwind.config.js` |
| `colorScheme` | `String`  | `auto`          | `'auto'` \| `'light'` \| `'dark'`                                | Switch between different color schemes                                     |
| `position`    | `String`  | `'bottomRight'` | `'topLeft'` \| `'topRight'` \| `'bottomLeft'` \| `'bottomRight'` | Breakys starting position                                                  |

## Development

1. Clone this repository
2. Install dependencies using `yarn install`
3. Start development server using `yarn dev`

### Release

1. `yarn release`
2. `npm publish`

## License

[MIT License](./LICENSE)

Copyright (c) teamnovu

<!-- Badges -->

[npm-version-src]: https://img.shields.io/npm/v/vue-breaky-core/latest.svg?style=flat-square
[npm-version-href]: https://github.com/teamnovu/vue-breaky-core/releases
[npm-downloads-src]: https://img.shields.io/npm/dt/vue-breaky-core.svg?style=flat-square
[npm-downloads-href]: https://github.com/teamnovu/vue-breaky-core/releases
[circle-ci-src]: https://img.shields.io/circleci/project/github/teamnovu/vue-breaky-core.svg?style=flat-square
[circle-ci-href]: https://circleci.com/gh/teamnovu/vue-breaky-core
[codecov-src]: https://img.shields.io/codecov/c/github/teamnovu/vue-breaky-core.svg?style=flat-square
[codecov-href]: https://codecov.io/gh/teamnovu/vue-breaky-core
[license-src]: https://img.shields.io/npm/l/vue-breaky-core.svg?style=flat-square
[license-href]: https://github.com/teamnovu/vue-breaky-core/blob/master/LICENSE
