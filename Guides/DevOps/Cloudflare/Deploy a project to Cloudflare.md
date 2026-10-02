# Deploy a project to Cloudflare

## SvelteKit
<https://svelte.dev/docs/kit/adapter-cloudflare>
1. Install cloudflare adapter to your project: `bun add -d @sveltejs/adapter-cloudflare`
2. Add the adapter to `vite.config.ts` file
```ts
import { defineConfig } from 'vitest/config';
import { playwright } from '@vitest/browser-playwright';
import adapter from '@sveltejs/adapter-cloudflare';
import { sveltekit } from '@sveltejs/kit/vite';

export default defineConfig({
	plugins: [
	sveltekit({
	...
	
		adapter: adapter({
			config: undefined,
			platformProxy: {
				configPath: undefined,
				environment: undefined,
				persist: undefined
			},
			fallback: 'plaintext',
			routes: {
				include: ['/*'],
				exclude: ['<all>']
			}
			})
		})
],
...
```
3. Create *wrangler.jsonc* file with `bunx wrangler setup`
4.  Run `bunx wrangler types`and then run `bun run build` just to see that your build works on your project

### Cloudflare website
5. Click *Create App* on the dashboard
6. Connect your project to Cloudflare with your GitHub account
7. Add `bun run build` as your *Build command* in settings on your projects _Workers & Pages_ section

Then just push your project to GItHub and your project is there. You can find the online link to your page in the _Domains_ on your projects _Workers & Pages_ section. Click the "_Anyone with this URL can visit."  **Enable Access** button

## .env variables
1. Choose your _settings_ from your _Workers & Pages_ section
2. Add the secrets to _Runtime variables and secrets_ and to _Builds_ --> _Variables and secrets_

## Related
- [[GitHub]]
- [[SvelteKit]]