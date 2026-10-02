# SvelteKit
https://svelte.dev/docs/kit/introduction

## Creating a project
### Current folder
1. `bunx sv create .` creates project to the current folder (for example if you already cloned a git repository)
### New folder
1. `bunx sv create PROJECT_NAME` creates project to the PROJECT_NAME folder

2. Which template? --> *SvelteKit minimal*
3. Add type checking with TypeScript? --> Yes 
4. What would you like to add to your project? --> prettier, eslint, vitest, playwright are some safe choices
5. vitest: What do you want to use vitest for? --> unit testing, component testing
6. `bun dev` to go.

## .env variables
1. Create `.env`or `.env.local`file and add your variables there
```env
API_KEY=4qrea....
```

2. Update the `vite.config.ts`file:
```ts
sveltekit({
	compilerOptions: {
		...
		experimental: {
			explicitEnvironmentVariables: true
	}
})
```


3. Create `src/env.ts` file and define your variables there also (just the name)
```ts
import { defineEnvVars } from '@sveltejs/kit/env';

export const variables = defineEnvVars({
	API_KEY: {},
});
```

4. Now the keys can be imported with `import { API_KEY } from '$app/env/private'`

## Related
- [[SvelteKit - Images]]
- [[Resend - Email service]]