# GitHub - Multiple GitHub accounts on same computer
Almost same instructions than in [[GitHub - Adding SSH keys]] but we can't go with the default filename
1. Run ´ssh-keygen -t ed25519 -C "name-for-that-another-account" -f ~/.ssh/id_ed25519_some-kind-of-tag-in-the-end´
2. Run `cat ~/.ssh/id_ed25519_some-kind-of-tag-in-the-end.pub`and copy the value
3. Paste it in the *Key* field in the *ssh* section on GitHub settings
4. Add the key

## SSH config file
Then we create or modify the _ssh config_ file in the terminal with command `nano ~/.ssh/config`:
```sh
Host github.com 
	AddKeysToAgent yes
	IgnoreUnknown UseKeychain
	IdentityFile ~/.ssh/id_ed25519
```
It looks like this for now at least for me. Then we add under that:
```sh
Host github-client
	HostName github.com
	User git
	IdentityFile ~/.ssh/id_ed25519_some-kind-of-tag-in-the-end
	IdentitiesOnly yes
```
So the whole file would be:
```sh
Host github.com 
	AddKeysToAgent yes
	IgnoreUnknown UseKeychain
	IdentityFile ~/.ssh/id_ed25519
	
Host github-client
	HostName github.com
	User git
	IdentityFile ~/.ssh/id_ed25519_some-kind-of-tag-in-the-end
	IdentitiesOnly yes
```
- Save with `Ctrl+O`
- Press `Enter`
- `CTRL+X` to exit nano

5. Test the client-connection in the terminal `ssh -T git@github-client`. If its not working do the steps again a bit sharper

## Cloning with right alias
When we clone a repository with ssh we need to be sure we use our _github-client_-alias and not the original address
6. Make sure you clone with the `git clone git@github-client:reponame`and NOT with `git clone git@github.com:reponame`
7. Clone all the repos in the same directory now with this account

### Adding the repo in the gitconfig
You don't need to remember to change the alias if you add the github repo in the *gitconfig* file itself `nano ~/.gitconfig`:
```sh
[url "git@github-client:your-another-account/"] 
	insteadOf = git@github.com:your-another-account/
```

## Own git-indentity for this account
We need a new file so `nano ~/.gitconfig-client`:
```sh
[user]
	name = Your Name 
	email = yourname@email.me
```

Then we add this to the main-gitconfig. So we run ´nano ~/.gitconfig´ and add under the current stuff
```sh
[includeIf "gitdir:~/workspace/asiakkaat/"] 
	path = ~/.gitconfig-client
```

## Related
- [[GitHub]]
- [[GitHub - Adding SSH keys]]