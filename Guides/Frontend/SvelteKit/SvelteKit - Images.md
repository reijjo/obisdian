# SvelteKit - Images
<https://svelte.dev/docs/kit/images>
Remember to change the format to _.avif_ or _.webp_ for better performance
## Enhanced-img
Install *@sveltejs/enhanced-img* plugin `bun add -d @sveltejs/enhanced-img`

Add it to `vite.config.ts`file:
```ts
import { enhancedImages } from '@sveltejs/enhanced-img'

export default defineConfig({
	plugins: [
	enhancedImages(),
	sveltekit({
		compilerOptions: {
			// Force runes mode for the project, except for libraries. Can be removed in svelte 6.
		runes: ({ filename }) => filename.split(/[/\\]/).includes('node_modules') ? undefined : true
		},
		adapter: adapter()
	})
],
...
)}
```

And use it rather than normal _<img />_ component like this:
```ts
<enhanced:img src="./path/to/your/image.jpg" alt="An alt text" />
```

## Related
- [[SvelteKit]]