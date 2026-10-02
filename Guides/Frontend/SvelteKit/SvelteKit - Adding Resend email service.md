# SvelteKit - Adding Resend email service
Install the _resend_ package with `bun add resend`
1. in your `+page.server.ts` file that needs the email service import the _api key_ from your `.env` file
```ts
import { fail } from '@sveltejs/kit';
import { RESEND_API_KEY } from '$app/env/private';
import { Resend } from 'resend';

import type { Actions } from './$types';

const resend = new Resend(RESEND_API_KEY);

export const actions: Actions = {
	default: async ({ request }) => {
		const formData = await request.formData();
		...
	}
}
```
2. After you have validated your form inputs send the actual email:
```ts
export const actions: Actions = {
	default: async ({ request }) => {
	...
		const name = nimi as string;
		const senderEmail = email as string;
		const message = viesti as string;
		
		const { data, error } = await resend.emails.send({
			from: RESEND_FROM_EMAIL,
			to: CONTACT_TO_EMAIL,
			replyTo: senderEmail,
			subject: `Yhteydenotto verkkosivulta - ${name}`,
			text: `${message}
			
			------
			Lähettäjä:
			${name}
			${senderEmail}
			`
		});
		
		if (error) {
			console.error('email error', error);
		
			return fail(500, {
				sendError: 'Viestin lähetys epäonnistui.'
			});
		}
		
			console.log('DATA', data);
		
		return {
			success: true
		};
	}
};
```

