# Drizzle ORM with Neon
https://orm.drizzle.team/

## Install
```sh
bun add drizzle-orm @neondatabase/serverless
```
- _drizzle-orm_ is the ORM itself
- _@neondatabase/serverless_ is the PostgreSQL driver for serverless environments like Vercel, where the database runs on Neon.
```sh
bun add -d drizzle-kit
```
- CLI tool for managing database migrations

## Setup
### Database Schema
First we create the _db/schema.ts_ file in the same folder where the _app_ folder is:
```ts
import { pgTable, serial, text, integer } from "drizzle-orm/pg-core";

  

export const blogs = pgTable("blogs", {
	id: serial("id").primaryKey(),
	title: text("title").notNull(),
	author: text("author").notNull(),
	url: text("url").notNull(),
	likes: integer().default(0).notNull(),
});
```
- _id_ has type *serial* which means it is an auto-increment integer, and it's marked as the *primary key* of the table
- _title_, _author_, _url_ has type *text* which corresponds to the SQL _TEXT_ type
- _likes_ has type *integer* which basically means it's a number and has a *default value* of 0

### Database connection
Next we create the database connection in the _db/index.ts_ file:
```ts
import { drizzle } from "drizzle-orm/neon-http";
import * as schema from "./schema";

export const db = drizzle(process.env.DATABASE_URL!, { schema });
```
The _drizzle_ function from _drizzle-orm/neon-http_ creates a serverless-friendly database connection using the connection string from our environment variable.

### Drizzle Kit configuration
Finally, we need a Drizzle configuration file for the migration so we create file _drizzle.config.ts_ in the project root:
```ts
import { defineConfig } from "drizzle-kit";
import * as dotenv from "dotenv";
dotenv.config({ path: ".env.local" });

export default defineConfig({
  schema: "./db/schema.ts",
  out: "./drizzle",
  dialect: "postgresql",
  dbCredentials: {
    url: process.env.DATABASE_URL!,
  },
});
```
We also need to install two more package:
```sh
bun add -d postgres dotenv
```

### Query logger
When something strange happens, it's useful to see the SQL that Drizzle sends to the database.
```ts
import { drizzle } from "drizzle-orm/neon-http"
import * as schema from "./schema"

export const db = drizzle(process.env.DATABASE_URL!, {
  schema,
  logger: true,
})
```

## Migrations
Now we have the schema so we can generate the migration and apply it to the database:
```sh
bunx drizzle-kit generate
bunx drizzle-kit migrate
```
The first command reads the schema and generates SQL migration files in the _drizzle_ directory. The second command applies those migrations to the database, creating the _blogs_ table.

### Migration problems (gets out of sync with db etc...)
Remove the Drizzle's migration tracking from the database and drop all the tables you have created:
```sql
DROP TABLE IF EXISTS drizzle.__drizzle_migrations CASCADE;
DROP TABLE IF EXISTS notes CASCADE;
```
Run these commands in the _Drizzle Studio_ or in the _Neon console_ on the _Vercel dashboard_

Second, remove the local migration history:
```sh
npx drizzle-kit drop
```
This command lets you select which migrations you want to remove

And last run the:
```sh
bunx drizzle-kit generate
bunx drizzle-kit migrate
```
And all should be fine.
## Drizzle Studio
Then we can verify that the table was created by running:
```sh
bunx drizzle-kit studio
```
This opens the Drizzle Studio, a visual database browser to address [https://local.drizzle.studio](https://local.drizzle.studio/) where we can see our table and its columns.

So add couple of blogs to the database so we can be sure everything is working.

## CRUD Operations
For example these can be in the _src/app/blogs/services/blogs.ts_ file or whatever is your folder structure

Remember that you need to _await_ the calls in the file you are calling the functions (_src/app/blogs/page.tsx)
```sh
import { getBlogs } from "../services/blogs";

export default async function Blogs() {
	const allBlogs = await getBlogs();

	return (
		...
	)
}
```

### Create
#### Add new
```ts
import { db } from "@/db";
import { blogs } from "@/db/schema";

export const addBlog = async (title: string, author: string, url: string) => {
	await db.insert(blogs).values({ title, author, url });
};
```

### Read
#### Find all
```ts
import { db } from "@/db";
import { blogs } from "@/db/schema";

export const getBlogs = () => {
	return db.query.blogs.findMany();
};
```

#### Find one
```ts
import { eq } from "drizzle-orm";
import { db } from "@/db";
import { blogs } from "@/db/schema";

export const getBlogById = (id: number) => {
	return db.query.blogs.findFirst({
		where: eq(blogs.id, id)
	})
};
```

### Update
```ts
export const addLike = async (id: number) => {
const blog = await getBlogById(id);

if (blog) {
	await db
		.update(blogs)
		.set({ likes: blog.likes + 1 })
		.where(eq(blogs.id, id));
	}
};
```

### Delete

## Relations between tables
Create an users table in the _db/schema.ts_ file where the other table also is:
```ts
export const users = pgTable("users", {
	id: serial("id").primaryKey(),
	username: text("username").notNull().unique(),
	name: text("name").notNull(),
});
```

And add a foreign key to the _blogs_ table referencing the _users_ table:
```ts
export const notes = pgTable("notes", { 
	id: serial("id").primaryKey(), 
	content: text("content").notNull(),
	important: boolean("important").notNull().default(false),
	userId: integer("user_id").notNull().references(() => users.id),
})
```
And run the _migrate_ and _generate_ commands.

Add these in the _schema.ts_ file:
```ts
import { relations } from "drizzle-orm";
...

export const usersRelations = relations(users, ({ many }) => ({
	blogs: many(blogs),
}));

export const blogsRelations = relations(blogs, ({ one }) => ({
	user: one(users, {
	fields: [blogs.userId],
	references: [users.id],
}),

}));
```
This tells Drizzle how the tables are connected so it can automatically join related data.
- The _usersRelations_ says that user can have many blogs
- The _blogsRelations_ says that a blog belongs to one user.

The _fields_ and _references_ properties are always required on the side that holds the foreign key column

## Auth
- [[Next.js - Authentication with NextAuth]]
## Related
- [[Vercel Postgres by Neon]]
- [[Next.js]]
