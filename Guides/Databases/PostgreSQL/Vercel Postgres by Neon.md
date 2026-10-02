# Vercel Postgres By Neon
## Create database

1. Go to your Vercel dashboard
2. Choose the project where you want to add the database
3. Navigate to the _Storage_ tab
4. Click _Create Database_ --> _Neon_ 
5. Select the closest region that you can find and click _Continue_
6. Give your database a name on the _Resource Name_ field and click _Create_
7. Wait for the creation and click _Connect_ on the next window (Install Integration)

To use the same database on local development, copy the connection string from the Vercel dashboard and create a _.env.local_ file in the root of your project:

```env
DATABASE_URL=postgresql://neondb_........
```

Remember to add the _.env.local_ file to your _.gitignore_ file!

## Related
- [[Deploy a project to Vercel]]
- [[Drizzle ORM with Neon]]
- [[Next.js]]
