# Resend - Email service
<https://resend.com/>

## Create an API key
1. Log in in the _Resend homepage_ 
2. Click _API keys_ on the sidebar
3. Click *Create API key* button and copy the key for example in your projects _.env_ file
4. So your _.env_ file should have at least 3 variables now
```env
RESEND_API_KEY=re_adsdad...
RESEND_FROM_EMAIL=Address Resend sends from <onboarding@resend.dev>
CONTACT_TO_EMAIL=where the contact-form messages arrive
```

## Adding custom domain
1. Choose _Domains_ from the sidebar
2. Click *Add domain* and add `mydomain.com` there
3. Resend shows then *DNS* records that needs to be added to your hostingservice (for example in Cloudflare)

### Adding Resend to Cloudflare
In Resend dashboard
1. Choose _Domains_ from the sidebar
2. Click *Add domain* and add `mydomain.com` there
3. Resend shows then *DNS* records that needs to be added to your hostingservice (for example in Cloudflare)

Then in Cloudflare
4. Choose *Domains* in the sidebar
5. Then *DNS* --> *Records* and add those records from resend
6. *Click* _I've already added the records_ and for Resend to check the records

Then in your project and in Cloudflare change the `.env` variables to match your domain
```env
RESEND_FROM_EMAIL=something email@mydomain.com
CONTACT_TO_EMAIL=email@mydomain.com
```

7. If its not working go to _Asikassivujen etusivu_ and _Kirjaudu hallintapaneeliin_
8. Choose _DNS-vyöhykkeen muokkaus_ --> _yourdomain_
9. Add add the records from *Resend*

## Related
- [[SvelteKit]]
- [[SvelteKit - Adding Resend email service]]
- [[Adding custom domain to Cloudflare]]
