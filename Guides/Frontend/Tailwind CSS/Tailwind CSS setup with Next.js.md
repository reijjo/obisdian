# Tailwind CSS setup with Next.js
1. Install
```sh
bun add tailwindcss @tailwindcss/postcss postcss
```
2. Configure PostCSS Plugins
Create a _postcss.config.mjs_ file in the root of the project:
```js
const config = {
	plugins: {
		"@tailwindcss/postcss": {},
	},
};

export default config;
```
3. Import Tailwind CSS in the _src/app/globals.css_ file:
```css
@import "tailwindcss";
```
4. Start your project
```sh
bun dev
```
5. Try it for example in a header:
```ts
const Home = () => {
return (
<div>
	<div>
		<h2 className="text-3xl">notes app</h2>
...
```
6. If there is some kind of error, its probably a cache thing so:
```sh
rm -rf .next
bun dev
```
And it should be working.

## Related
- [[Next.js]]