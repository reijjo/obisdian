# Authentication with NextAuth
[[#Install]] | [[#Database setup]] | [[#NextAuth configuration]] | [[#How the session works]] | [[#Auth API route]] | [[#Client-side session access]] | [[#Login]] | [[#Server-side session access]] | [[#Protecting routes and actions]] | [[#Environment variables]] | [[#Auth flow recap]] | [[#Related]]

## Install

First we need to install the needed packages:
```sh
bun add next-auth@beta bcryptjs && bun add -d @types/bcryptjs
```

## Database setup

### Users table
And we need to create _users_ table for our database (this example is done by using [[Drizzle ORM with Neon]]):
```ts
export const users = pgTable("users", {
	id: serial("id").primaryKey(),
	username: text("username").notNull().unique(),
	name: text("name").notNull(),
	passwordHash: text("password_hash").notNull().default(""),
});
```

## NextAuth configuration
Create NextAuth configuration file in _src/auth.ts_:
```ts
import NextAuth from "next-auth";
import Credentials from "next-auth/providers/credentials";
import { db } from "./db";
import { eq } from "drizzle-orm";
import { users } from "./db/schema";
import bcrypt from "bcryptjs";

export const { handlers, auth, signIn, signOut } = NextAuth({
	providers: [
		Credentials({
			credentials: {
				username: { label: "Username", type: "text" },
				password: { label: "Password", type: "password" },
			},
			async authorize(credentials) {
				if (!credentials?.username || !credentials?.password) {
					return null;
			}
	
			const user = await db.query.users.findFirst({
				where: eq(users.username, credentials.username as string),
			});
	
			if (!user || !user.passwordHash) {
				return null;
			}
	
			const isValid = await bcrypt.compare(
				credentials.password as string,
				user.passwordHash,
			);
	
			if (!isValid) {
				return null;
			}
	
			return {
				id: String(user.id),
				name: user.name,
				email: user.username,
			};
		},
	}),
	],
	pages: {
		signIn: "/login",
	},
	session: {
		strategy: "jwt",
	},
});
```

### Credentials provider
NextAuth supports many different authentication providers such as _Google_, _GItHub_ etc. Here we use the _Credentials provider_, which lets users log in with a username and password.

#### Authorize function
The _authorize_ function is the core of the Credentials provider. It receives the values the user typed in the login form as _credentials_. It then looks up the user in the database by username and compares the password if the submitted password matches the stored hash.  If it fails for any reason, it returns _null_, which causes NextAuth to reject the login attempt.

When authentication succeeds, _authorize_ returns a plain object that NextAuth encodes into the JWT session token:
```ts
{
  id: String(user.id),
  name: user.name,
  email: user.username,
}
```
NextAuth's built-in session type has three fields: _id_, _name_, and _email_. We do not have an email address in our schema, so we reuse the _email_ field to store the username instead (can be done with custom TypeScript declarations).

### Custom sign-in page
The _pages: { signIn: "/login" }_ option tells NextAuth to use our own login page at _/login_ instead of the built-in NextAuth sign-in page. Whenever NextAuth needs to redirect an unauthenticated user to log in, it will send them to _/login_.

### Session strategy
The _session: { strategy: "jwt" }_ option tells NextAuth to store session data in a signed JSON Web Token in a cookie, rather than in a server-side session store.

## How the session works
After a successful login, NextAuth creates a session and stores it as a signed JWT in an HTTP-only cookie in the browser. The cookie is sent automatically with every subsequent request. On the server, NextAuth verifies the signature and reads the token to know who the user is, without touching a database.

The token contains whatever the _authorize_ function returned: in our case _id_, _name_, and _email_ (which we used to store the username). These values are available through _useSession_ on the client and through _auth()_ on the server.

## Auth API route
NextAuth requires an API route that handles all authentication requests (sign in, sign out, session checks). We create the file _src/app/api/auth/[...nextauth]/route.ts_:
```ts
import { handlers } from "@/auth";

export const { GET, POST } = handlers;
```

The [...nextauth] part is a Next.js catch-all route segment that matches any path under _/api/auth/_, for example _/api/auth/signin_, _/api/auth/signout_, and _/api/auth/session_. NextAuth intercepts all of these and handles them automatically.

## Client-side session access
### SessionProvider
Some parts of our UI need to have access to the session on the client side. The navigation bar is the clearest example: it could eg. show the logged-in user's name and a logout button when a session exists, and a login link when there is none.

NextAuth's client-side hooks like _useSession_ share session data through React context, which requires a provider component to sit above any component that uses it. NextAuth ships its own SessionProvider for exactly this purpose. Because it uses React context it must be a Client Component, and since we cannot place a Client Component directly in the root layout, we create a thin wrapper _src/app/components/SessionProvider.tsx_:

```ts
"use client";

import { SessionProvider } from "next-auth/react";
import { ReactNode } from "react";

type AuthSessionProviderProps = {
	children: ReactNode;
};

export default function AuthSessionProvider({
	children,
}: AuthSessionProviderProps) {
	return <SessionProvider>{children}</SessionProvider>;
}
```

Then we update the _layout.tsx_ to wrap the app inside _AuthSessionProvider_:
```ts
export default function RootLayout({ children }: LayoutProps<"/">) {
return (
	<html lang="en" className={`${geistSans.variable} ${geistMono.variable}`}>
		<body>
			<AuthSessionProvider>
				<NavBar />
				{children}
			</AuthSessionProvider>
		</body>
	</html>
);
}
```

### useSession hook
The navigation bar needs to render differently depending on whether a user is logged in. Because the logout button calls _signOut()_ in an _onClick_ handler, the component must be a Client Component. It gets hold to the session via the _useSession_ hook, which pulls the value from the React context provided by _SessionProvider_. We place the component in _src/app/components/NavBar.tsx_:
```ts
"use client";

import { signOut, useSession } from "next-auth/react";
import Link from "next/link";

export default function NavBar() {
const { data: session } = useSession();

return (
	<nav>
		<Link href="/">home</Link>
		{" | "}
		<Link href="/blogs">blogs</Link
		{" | "}
		<Link href="/users">users</Link>
		{" | "}
		{session ? (
			<>
			<Link href="/blogs/new">create blog</Link>
			{" | "}
			<em>{session.user?.name} logged in</em>{" "}
			<button onClick={() => signOut()}>logout</button>
			</>
		) : (
			<Link href="/login">login</Link>
		)}
	</nav>
);
}
```

## Login
### Login page
The login page lives at _src/app/login/page.tsx_. It is a Client Component because it handles form submission and manages local state for error messages:
```ts
"use client";

import { signIn } from "next-auth/react";
import { useRouter } from "next/router";
import { useState, SubmitEvent } from "react";

export default function LoginPage() {
	const router = useRouter();
	const [error, setError] = useState("");

	const handleSubmit = async (e: SubmitEvent<HTMLFormElement>) => {
		e.preventDefault();
		const formData = new FormData(e.currentTarget);
		
		const result = await signIn("credentials", {
			username: formData.get("username"),
			password: formData.get("password"),
			redirect: false,
		});
		
		if (result?.error) {
			setError("Invalid username or password");
		} else {
			router.push("/");
		}
	};
	
	return (
		<div>
			<h2>Login</h2>
			{error && <p style={{ color: "red" }}>{error}</p>}
			<form onSubmit={handleSubmit}>
				<div>
					<label>
						Username
						<input type="text" name="username" required />
					</label>
				</div>
				<div>
					<label>
						Password
						<input type="password" name="password" required />
					</label>
				</div>
				<button type="submit">Login</button>
			</form>
		</div>
	);
}
```
The form calls signIn from _next-auth/react_ with _redirect: false_, which means NextAuth will return the result as an object instead of automatically redirecting the browser. If authentication fails, _result.error_ will be set and we display an error message. If it succeeds, we use the Next.js router to navigate to the home page.

## Server-side session access
For Server Components and Server Actions we cannot use _useSession_, since hooks only run in the browser. Instead Auth.js provides the _auth_ function, which reads the session from the request headers on the server side. We wrap this in a helper function _src/app/services/session.ts_:
```ts
import { auth } from "@/auth";
import { db } from "@/db";
import { users } from "@/db/schema";
import { eq } from "drizzle-orm";

export const getCurrentUser = async () => {
	const session = await auth();
	if (!session?.user?.email) {
		return null;
	}
	
	return db.query.users.findFirst({
		where: eq(users.username, session.user.email),
	});
};
```

## Protecting routes and actions
### Protecting a Server Action
Only logged users can create blogs so lets update the _src/app/actions/blogs.ts_:
```ts
"use server";

import { revalidatePath } from "next/cache";
import { redirect } from "next/navigation";
import { addBlog, addLike } from "../services/blogs";
import { auth } from "@/auth";

export const createBlog = async (formData: FormData) => {
	const session = await auth()
	if (!session) {
		redirect("/login")
	}
	
	const title = formData.get("title") as string;
	const author = formData.get("author") as string;
	const url = formData.get("url") as string;
	
	await addBlog(title, author, url);
	
	revalidatePath("/blogs");
	redirect("/blogs");
};
```

## Environment variables
NextAuth requires a secret key to sign the JWT session token, so we can generate one also in terminal:
```sh
echo "$(openssl rand -base64 32)"
```
And copy the output in the _.env.local_ file:
```env
DATABASE_URL=postgresql://...

AUTH_SECRET=...
```
Also add the variables in _.env.local_ file and also add _AUTH_URL_ to Vercel. _AUTH_URL_ is the public URL of the deployed app, for example _https://your-app.vercel.app_
- Choose your project
- Settings --> Environments --> Production --> Environment Variables --> _Add Environment Variable_

## Auth flow recap
### Sign-in flow
The user fills in the login form on _/login_. The form's _onSubmit_ handler calls _signIn("credentials", { redirect: false, ... }), which_ is a function provided by NextAuth. Under the hood, NextAuth's _signIn_ makes a _POST_ request to the catch-all API route at _/api/auth/callback/credentials_. The route handler is defined in _app/api/auth/[...nextauth]/route.ts_, where _handlers_ exported from _auth.ts_ registers the GET and POST handlers for Next.js.

When the POST arrives, NextAuth's internal logic takes over and calls the _authorize_ function from _auth.ts_. That function looks up the user in the database by username and uses _bcrypt.compare_ to verify the password against the stored hash. If the credentials are valid, _authorize_ returns a user object with _id_, _name_, and _email_ (where we store the username). NextAuth takes that object, encodes it into a JWT signed with _AUTH_SECRET_, and sets it as an HTTP-only cookie in the response. An HTTP-only cookie cannot be read by JavaScript in the browser, only sent automatically with each subsequent request, which protects the token from XSS attacks. The login page then uses _router.push("/")_ to navigate the user to the home page.

### Client-side session flow
On the next render, the browser sends the cookie with every request. _AuthSessionProvider_ (which wraps the whole app in the root layout) reads the cookie, verifies the JWT, and makes the session data available through React context. Any Client Component that calls _useSession()_ receives the session from that context without any additional network request.

### Server-side session flow
When a Server Component or Server Action needs to know who is logged in, it calls _auth()._ NextAuth reads the signed JWT from the incoming request headers and returns the decoded session. The helper _getCurrentUser_ wraps this call and additionally fetches the full user record from the database so that the rest of the application code gets a proper user object.

### Protected actions 
Server Actions that require authentication call _auth()_ at the top and redirect to _/login_ if the session is missing. Only after confirming a valid session is any database write performed.

## Register
### Server action
We need a Server Action that hashes the password and inserts the new user in the database in _src/app/actions/users.ts_:
```ts
"use server";

import { db } from "@/db";
import { users } from "@/db/schema";
import bcrypt from "bcryptjs";
import { redirect } from "next/navigation";

export const registerUser = async (formData: FormData) => {
	const username = (formData.get("username") as string)?.trim();
	const name = (formData.get("name") as string)?.trim();
	const password = formData.get("password") as string;
	
	const passwordHash = await bcrypt.hash(password, 10);
	
	await db.insert(users).values({ username, name, passwordHash });
	
	redirect("/login");
};
```
### Register page
_src/app/register/page.tsx_:
```ts
import { registerUser } from "../actions/users";

export default function RegisterPage() {
	return (
		<div>
			<h2>Register</h2>
			<form action={registerUser}>
				<div>
					<label>
						Username
						<input type="text" name="username" required />
					</label>
				</div>{" "}
				<div>
					<label>
						Name
						<input type="text" name="name" required />
					</label>
				</div>{" "}
				<div>
					<label>
						Password
						<input type="password" name="password" required />
					</label>
				</div>
				<button type="submit">Register</button>
			</form>
		</div>
	);
}
```
Remember to add the link to register page to the Nav component.
## Related
- [[Deploy a project to Vercel]]
- [[Drizzle ORM with Neon]]
- [[Next.js]]