# Adding custom domain to Cloudflare
1. Choose _Domains_ --> _Overview_ on the sidebar
2. Click *Add domain*
3. *Connect a domain* and add your domain to the _Domain name_ field
4. Click *Continue* and select the *Free plan* 
5. Then you see shitload of nameservers and then lets go to hostingservice specific instructions

## Domain/hostingservice specific instructions
- [[Hostingpalvelu]]

## Connect domain to the project
6. Select your project and choose the *Domains* tab
7. Click *Add Domain* button
8. Choose the domain you added earlier and don't add subdomain and  click *Add domain*

### In case of an error
I got this error `Hostname 'mydomain.com' already has externally managed DNS records (A, CNAME, etc). Delete them first or try a different hostname.` So we need to change the *DNS* --> *Records* of the page
1. So we choose *Domains* in the sidebar
2. Then *DNS* --> *Records*
3. Add we remove the 'mydomain.com' A-type from the list
4. Then we try to **Connect domain to the project** again

## Routing the www. prefix to the site
1. So we choose *Domains* in the sidebar
2. Then *DNS* --> *Records*
3. Remove the current **CNAME** record
4. Add a new record
```
Type: CNAME
Name: www 
Target: mydomain.com 
Proxy status: Proxied 🟠 
TTL: Auto
```
5. Then go to *Domains* --> *Rules* and click _Create a rule_
6. Choose the template *Redirect from WWW to root*
7. Click *Deploy* --> *Deploy rule* and after that you can try the `www.mydomain.com` address if it works

## Related
- [[Hostingpalvelu]]
- [[Resend - Email service]]