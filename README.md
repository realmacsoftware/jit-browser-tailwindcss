# @mhsdesign/jit-browser-tailwindcss
Still in Development

Client side api to generate css via tailwind jit in the browser - Dynamic tailwindcss!

![image](https://user-images.githubusercontent.com/85400359/157231070-2de5d2ad-c852-40db-92dd-09d7171990bb.png)

### Tailwinds CDN: https://cdn.tailwindcss.com
(Keep in mind its not only pure tailwind but also dom observing etc)
with tailwind Version 3.1.8
- 372kB
- 98.9kB brotli
- 99.4kB gzip

### This library: https://unpkg.com/@mhsdesign/jit-browser-tailwindcss
(pure Tailwind API)
with tailwind Version 3.1.8
- 246kB
- 74.3kB gzip

## Implementation

### Current  implementation

This library is inspired/extracted from the library [monaco-tailwindcss](https://github.com/remcohaszing/monaco-tailwindcss)
see [core code link](https://github.com/remcohaszing/monaco-tailwindcss/blob/main/src/tailwindcss.worker.ts#L176)

As you might see in the code [core code example](https://github.com/mhsdesign/jit-browser-tailwindcss/blob/d3b726e7122ff1d296ae50db17030a1962be36c8/src/index.ts#L17-L34) we uses tailwind internals to create a postcss plugin via `createTailwindcssPlugin`. This is not api but works for now ;) And produces highly optimized bundle sizes.

### Previous / Other implementations

Previously i implemented this based on various other projects by mocking `fs.readFileSync` and having some kind of virtual file store.

This turned out to be not so efficient in terms of bundle size ;)

<details>
<summary>To be continued ...</summary>

Also mocking `fs.readFileSync` had to be done in some postcss source files and this required the whole postcss package to be prebundled. If the developer wants to use post css too it would result postcss being in the bundle twice.

See packages which are implemented like this:

- [@mhsdesign/jit-browser-tailwindcss:@legacy](https://github.com/mhsdesign/jit-browser-tailwindcss/tree/legacy) 501kB (resource) - [core code link](https://github.com/mhsdesign/jit-browser-tailwindcss/blob/3604924a3d2245b64ee359edc5f19b7106943a2a/src/jitBrowserTailwindcss.js#L23-L27)

- [beyondcode/tailwindcss-jit-cdn](https://github.com/beyondcode/tailwindcss-jit-cdn) 778kB (resource) - [core code link](https://github.com/beyondcode/tailwindcss-jit-cdn/blob/main/src/observer.js#L40-L52)

- [tailwindlabs/play.tailwindcss.com](https://github.com/tailwindlabs/play.tailwindcss.com/) - [core code link](https://github.com/tailwindlabs/play.tailwindcss.com/blob/01c39f107a7c514b4a84ec1385926748ae5a0ef0/src/workers/processCss.js#L238-L249)

The advantage here being that it uses the official API and doesnt rely much on internals.

</details>

## Supported Features of Tailwind Css
everything ;) - plugins, custom Tailwind config, custom Tailwind base css, variants ...

## Usage:

### npm

```shell
npm install @mhsdesign/jit-browser-tailwindcss
```

### cdn

```
https://unpkg.com/@mhsdesign/jit-browser-tailwindcss
```

### api to generate styles from content

```ts
/**
 * The entry point to retrieve 'tailwindcss'
 *
 * @param options {@link TailwindcssOptions}
 * @example
 * const tailwindConfig: TailwindConfig = {
 *   theme: {
 *     extend: {
 *       colors: {
 *         marcherry: 'red',
 *       },
 *     },
 *   },
 * };
 * const tailwindCss = tailwindcssFactory({ tailwindConfig });
 */
export function createTailwindcss(
  options?: TailwindcssOptions,
): Tailwindcss;

export interface TailwindcssOptions {
  /**
   * The tailwind configuration to use.
   */
  tailwindConfig?: TailwindConfig;
}

export interface Tailwindcss {
  /**
   * Update the current Tailwind configuration.
   *
   * @param tailwindConfig The new Tailwind configuration.
   */
  setTailwindConfig: (tailwindConfig: TailwindConfig) => void;

  /**
   * Generate styles using Tailwindcss.
   *
   * This generates CSS using the Tailwind JIT compiler. It uses the Tailwind configuration that has
   * previously been passed to {@link createTailwindcss}.
   *
   * @param css The CSS to process. Only one CSS file can be processed at a time.
   * @param content All content that contains CSS classes to extract.
   * @returns The CSS generated by the Tailwind JIT compiler. It has been optimized for the given
   * content.
   * @example
   * tailwindcss.generateStylesFromContent(
   *   css,
   *   [myHtmlCode]
   * )
   */
  generateStylesFromContent: (css: string, content: (Content | string)[]) => Promise<string>;
}

/**
 * Contains the content of CSS classes to extract.
 * With optional "extension" key, which might be relevant
 * to properly extract css classed based on the content language.
 */
export interface Content {
  content: string;
  extension?: string;
}
```

### lower level api to create tailwind post css plugin

```ts
/**
 * Lower level API to create a PostCSS Tailwindcss Plugin
 * @internal might change in the future
 * @example
 * const processor = postcss([createTailwindcssPlugin({ tailwindConfig, content })]);
 * const { css } = await processor.process(css, { from: undefined });
 */
export function createTailwindcssPlugin(
  options: TailwindCssPluginOptions
): AcceptedPlugin;

export interface TailwindCssPluginOptions {
  /**
   * The tailwind configuration to use.
   */
  tailwindConfig?: TailwindConfig;
  /**
   * All content that contains CSS classes to extract.
   */
   content: (Content | string)[];
}
```

## Examples:

see `examples/*`
- esbuild-demo
- esbuild-demo-worker
- script-tag


## Use cases
this plugin was developed to make dynamic html content elements from a CMS usable with tailwind classes. In that case one should already have a tailwind build and css file at hand - any further css can then be generated via this package. To have the least amount of css duplication, one should disable the normalize css and also use it without the `@base` include:

```ts
const tailwind = createTailwindcss({
    tailwindConfig: {
        // disable normalize css
        corePlugins: { preflight: false }
    }
})

const css = await tailwind.generateStylesFromContent(`
    /* without the "@tailwind base;" */
    @tailwind components;
    @tailwind utilities;
`, [htmlContent])
```

## Development

build the package
```sh
npm run build
```

test each example (will be served with esbuild)

```sh
npm run example1
npm run example2
npm run example3
```
