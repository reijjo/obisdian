# React - React Context
<https://react.dev/learn/passing-data-deeply-with-context>

Context lets a parent component provide data to the entire tree below it. So goodbye prop-drilling through components.

This is an example of a notification system:
## Context
Let's create file _src/app/components/NotificationContext.tsx_
```ts
import { createContext, ReactNode, useContext, useState } from "react";

type NotificationType = "success" | "error";

type NotificationContextType = {
	message: string;
	type: NotificationType;
	showNotification: (message: string, type?: NotificationType) => void;
};

type NotificationProviderProps = {
	children: ReactNode;
};

const NotificationContext = createContext<NotificationContextType>({
	message: "",
	type: "success",
	showNotification: () => {},
});

export const NotificationProvider = ({
	children,
}: NotificationProviderProps) => {
	const [message, setMessage] = useState("");
	const [type, setType] = useState<NotificationType>("success");
	
	const showNotification = (
		msg: string,
		notifType: NotificationType = "success",
	) => {
		setMessage(msg);
		setType(notifType);
		setTimeout(() => setMessage(""), 5000);
	};
	
	return (
		<NotificationContext value={{ message, type, showNotification }}>
			{children}
		</NotificationContext>
	);
};

export const useNotification = () => useContext(NotificationContext);
```

The context holds _message_ and _type_ as state. The _showNotification_ function sets the message and schedules a setTimeout to clear it after 5 seconds.

## Component that uses the context
_src/app/components/Notification.tsx_
```ts
"use client";

import { CSSProperties } from "react";
import { useNotification } from "./NotificationContext";

export default function Notification() {
	const { message, type } = useNotification();
	
	if (!message) return null;
	
	const style: CSSProperties = {
		padding: "10px 16px",
		marginBottom: "10px",
		borderRadius: "4px",
		color: "white",
		backgroundColor: type === "success" ? "#16a34a" : "#dc2626",
	};
	
	return <div style={style}>{message}</div>;
}
```
When _message_ is empty the component returns _null_ and renders nothing. When a message is set, it renders a colored banner: green for _"success"_ and red for _"error"_.

To make the notification available throughout the app we wrap the layout with _NotificationProvider_ and place _Notification_ just below the navigation bar. So lets update the _src/app/layout.tsx_
```ts
export default function RootLayout({ children }: LayoutProps<"/">) {
	return (
		<html lang="en" className={`${geistSans.variable} ${geistMono.variable}`}>
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

## Usage
We need to update the actions that we return errors and success on the right format so let's update the _src/app/actions/blogs.ts_ file:
```ts
"use server";

import { revalidatePath } from "next/cache";
import { redirect } from "next/navigation";
import { addBlog, addLike } from "../services/blogs";
import { auth } from "@/auth";

type BlogFormState = {
	errors?: {
		title?: string;
		author?: string;
		url?: string;
	};
		values?: {
		title: string;
		author: string;
		url: string;
	};
	success: boolean;
};

  

export const createBlog = async (
	_prevState: BlogFormState,
	formData: FormData,
): Promise<BlogFormState> => {
	const session = await auth();
	if (!session) {
		redirect("/login");
	}

  

	const title = formData.get("title") as string;
	const author = formData.get("author") as string;
	const url = formData.get("url") as string;
	
	const errors: BlogFormState["errors"] = {};
	
	if (!title || title.length < 5) {
		errors.title = "Title must have at least 5 characters.";
	}

  

	if (!author || author.length < 5) {
		errors.author = "Author must have at least 5 characters.";
	}

  

	if (!url || url.length < 5) {
		errors.url = "Url must have at least 5 characters.";
	}

	if (Object.keys(errors).length > 0) {
		return {
			errors,
			values: {
			title,
			author,
			url,
			},
			success: false,
		};
	}

	await addBlog(title, author, url);	  
	
	revalidatePath("/blogs");
	
	return { errors: {}, success: true };
};
```
Then we use _useEffect_ to check the state in the _src/app/blogs/new/page.tsx_ file. If its a success it redirects to the blogs page and shows the notification:
```ts
"use client";

import { createBlog } from "@/app/actions/blogs";
import { useNotification } from "@/app/components/NotificationContext";
import { useRouter } from "next/navigation";
import { useActionState, useEffect } from "react";

const initialState = {
	errors: {},
	values: {
	title: "",
	author: "",
	url: "",
	},
	success: false,
};

export default function NewBlog() {
	const [state, formAction] = useActionState(createBlog, initialState);
	const { showNotification } = useNotification();
	const router = useRouter();

	useEffect(() => {
		if (state.success) {
			showNotification("blog created");
			router.push("/blogs");
		}
	}, [state, showNotification, router]);

	return (
		<div>
		<h2>Create new blog</h2>
		<form action={formAction}>
		...
	)
}
```