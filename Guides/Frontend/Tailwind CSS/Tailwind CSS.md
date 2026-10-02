# Tailwind CSS
## Usage
Styling is done be adding class names to JSX elements. The body in _app/layout.tsx_ in Next.js projects can be given a background and text color:
```ts
export default function RootLayout({ children }: LayoutProps<"/">) {
	return (
		<html
		lang="en"
		className={`${geistSans.variable} ${geistMono.variable} min-h-screen bg-background text-foreground`}
>		
			<body>
				<AuthSessionProvider>
					<NotificationProvider>
						<NavBar />
						<Notification />
						{children}
					</NotificationProvider>
				</AuthSessionProvider>
			</body>
		</html>
	);
}
```

### layout.tsx file setup
```css
@import "tailwindcss";

:root {
  --background: #ffffff;
  --foreground: #171717;
}

@theme inline {
  --color-background: var(--background);
  --color-foreground: var(--foreground);
  --font-sans: var(--font-geist-sans);
  --font-mono: var(--font-geist-mono);
}

@media (prefers-color-scheme: dark) {
  :root {
    --background: #0a0a0a;
    --foreground: #ededed;
  }
}

body {
  background: var(--background);
  color: var(--foreground);
  font-family: Arial, Helvetica, sans-serif;
}
```
The _@import "tailwindcss"_ line loads all of Tailwind's utilities. The _--background_ and _--foreground_ CSS variables are defined in _:root_ and automatically switch to dark-mode values when the user's operating system prefers a dark colour scheme. The _@theme inline_ block maps those variables into Tailwind's colour palette so that classes like _bg-background_ and _text-foreground_ resolve to the correct values.
## Related
- [[Tailwind CSS setup with Next.js]]