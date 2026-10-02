# Next.js - Error handling
## Form validation errors

React 19 _useActionState_ hook is made exactly for this purpose ([[React - Hooks#useActionState]])

### Server side
The Server Action needs to change its signature. Instead of taking only _formData_, it now also receives _prevState_ as its first argument. This is the previous state returned by the action.

#### Form state
First we create the empty form state (_src/app/actions/blogs.ts_):
```ts
type BlogFromState = {
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
};
```

#### Form action
And then we use the form state in the form action:
```ts
export const createBlog = async (
	prevState: BlogFromState,
	formData: FormData,
): Promise<BlogFromState> => {
	const session = await auth();
	if (!session) {
		redirect("/login");
	}
	
	const title = formData.get("title") as string;
	const author = formData.get("author") as string;
	const url = formData.get("url") as string;
	
	const errors: BlogFromState["errors"] = {};
	
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
		};
	}

	await addBlog(title, author, url);

	revalidatePath("/blogs");
	redirect("/blogs");
};
```

On validation failure, it returns the form state with the errors and the field values so user doesn't need to fill the fields again. On success, it still calls _redirect_ as before.

### Client side
The form itself becomes a Client Component so it can use the useActionState hook. First we make the hook to have _initialState_ (_src/app/blogs/new/page.tsx_):
```ts
const initialState = {
	errors: {},
	values: {
		title: "",
		author: "",
		url: "",
	},
};
```

And we add the errors under the inputs and also defaultValues to the inputs:
```ts
export default function NewBlog() {
const [state, formAction] = useActionState(createBlog, initialState);

	return (
		<div>
			<h2>Create new blog</h2>
			<form action={formAction}>
				<div>
					<label>
						Title
						<input
						type="text"
						name="title"
						defaultValue={state.values?.title}
						/>
					</label>
					{state.errors?.title && (
					<p style={{ color: "red " }}>{state.errors.title}</p>
					)}
				</div>
				<div>
					<label>
						Author
						<input
						type="text"
						name="author"
						defaultValue={state.values?.author}
						/>
					</label>
					{state.errors?.author && (
					<p style={{ color: "red " }}>{state.errors.author}</p>
					)}
				</div>
				<div>
					<label>
						Url
						<input type="text" name="url" defaultValue={state.values?.url} />
					</label>
					{state.errors?.url && (
					<p style={{ color: "red " }}>{state.errors.url}</p>
					)}
				</div>
			<button type="submit">Create</button>
			</form>
		</div>
	);
}
```

## Relations
- [[Next.js]]
- [[React - Hooks]]
- [[Drizzle ORM with Neon]]