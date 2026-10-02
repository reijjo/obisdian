# Adding remote repository
## Creating the repo in GitHub
1. Make a new repository in <https://github.com/>
2. Give name to your repository
3. Short description also
4. Add README
5. Click *Create repository*

## Connecting the folder to GitHub
1. Run `git init` in the folder you want to connect to the repo you just created
2. Add _.gitignore_ and at least `.DS_Store` in that file
3. Then run `git add .` and check with `git status` that all good so far
4. Push files  `git commit -m 'here it is'` 
5. Rename the branch to _main_ `git branch -M main`
6. Then copy the _SSH URL_ from GitHub repository `git@github.com:username/folder.git`
7. Add connect it `git remote add origin git@github.com:username/folder.git
8. Run `git fetch origin` because of the README.md file that we made in the repo
9. Then push `git push -u origin main`
10. 

## Related
- [[GitHub]]