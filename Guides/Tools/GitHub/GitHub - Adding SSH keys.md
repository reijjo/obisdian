# GitHub - Adding SSH keys
We need a SSH Key for cloning and pushing the projects
1. Go to your GitHub _settings_
2. In the sidebar choose _SSH and GPG keys_ in the _Access_ section
3. Click *New SSH key* button

Then we need to create a new ssh key in the terminal
4. Run: `ssh-keygen -t ed25519 -C "oma_sähköposti@email.com"`
5. Press Enter to accept the default file location (or specify a custom one if you already have a key there)
6. Optionally set a passphrase, or press Enter twice to skip it
7. Copy the public key. `cat ~/.ssh/id_ed25519.pub`

Then back to GitHub
8. Paste it into the *Key* field

And back to terminal
9. Test the connection with `ssh -T git@github.com`

## Related
- [[GitHub]]
- [[GitHub - Multiple GitHub accounts on same computer]]