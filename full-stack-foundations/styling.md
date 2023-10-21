## Asset Links

Each route file can export a `links` function returning an array of link objects for that route/page.

Then in the `root` file of the app use the Remix `Link` tag wihtin the head.
Remix will behind the scenes, inject all links from exported links function for that route and place them in the head.

```ts
import { type LinksFunction } from '@remix-run/node'
import { Links, LiveReload, Scripts } from '@remix-run/react'
import faviconAssetUrl from './assets/favicon.svg'
import { KCDShop } from './kcdshop.tsx'

export const links: LinksFunction = () => {
	return [{ rel: 'icon', type: 'image/svg+xml', href: faviconAssetUrl }]
}

export default function App() {
	return (
		<html lang="en">
			<head>
				<Links />
			</head>
			<body>
				<p>Hello World</p>
				<Scripts />
				<KCDShop />
				<LiveReload />
			</body>
		</html>
	)
}
```
## Asset Import Caching

Placing assets into an `assets` folder at the top level inside the `app` folder allows Remix to find and cache these assets which creates a file for each asset within `public/build/_assets`.

We then import the asset from the `app/assets` within components:

```ts
import faviconAssetUrl from './assets/favicon.svg'

export const links: LinksFunction = () => {
	return [{ rel: 'icon', type: 'image/svg+xml', href: faviconAssetUrl }]
}
```

The cached version will be used unless the file changes at which piont a new cache is created, replacing the old version.

## Custom Fonts & Global CSS

stylesheets, in this case within `app/styles` are also cached and added to the `public/build/_assets` folder...and the cache is updated only when the file changes. 

```ts
import fontStylesheetUrl from './styles/font.css'

export const links: LinksFunction = () => {
	return [
		{ rel: 'icon', type: 'image/svg+xml', href: faviconAssetUrl },
		{ rel: 'stylesheet', href: fontStylesheetUrl },
	]
}
```

## PostCSS & Tailwind

postcss.config.js
```ts
export default {
	plugins: {
		'tailwindcss/nesting': {},
		tailwindcss: {},
		autoprefixer: {},
	},
}
```

tailwind.config.ts
```ts
import { type Config } from 'tailwindcss'
import defaultTheme from 'tailwindcss/defaultTheme.js'

export default {
	content: ['./app/**/*.{ts,tsx,jsx,js}'],
	theme: {
		extend: {
			fontFamily: {
				sans: [
					'Nunito Sans',
					'Nunito Sans Fallback',
					...defaultTheme.fontFamily.sans,
				],
			},
		},
	},
} satisfies Config
```

> RESTART SERVER

app/styles/tailwind.css
```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

app/root.tsx
```ts
import tailwindStylesheetUrl from './styles/tailwind.css'

export const links: LinksFunction = () => {
	return [
		{ rel: 'icon', type: 'image/svg+xml', href: faviconAssetUrl },
		{ rel: 'stylesheet', href: fontStylesheetUrl },
		{ rel: 'stylesheet', href: tailwindStylesheetUrl },
	]
}
```

## Bundling CSS

```ts
import stylesheetUrl from './styles1.css' // <-- you use the URL in the links export
import './styles2.css' // <-- this will be bundled
```

```ts
import './styles2.css'
import './styles3.css'
import { cssBundleHref } from '@remix-run/css-bundle'
```

`cssBundleHref` is either a string or null so to handle this...as href is a string:

```ts
import './styles/global.css'

export const links: LinksFunction = () => {
	return [
		{ rel: 'icon', type: 'image/svg+xml', href: faviconAssetUrl },
		{ rel: 'stylesheet', href: fontStylesheetUrl },
		{ rel: 'stylesheet', href: tailwindStylesheetUrl },
		cssBundleHref ? { rel: 'stylesheet', href: cssBundleHref } : null,
	].filter(Boolean)
}
```