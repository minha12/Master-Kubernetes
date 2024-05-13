# Creating an SSH key on a Mac for use with GitHub

## Check for Existing SSH Keys
Before you create a new SSH key, check if there are already any SSH keys on your machine, which you might wish to use:
```
ls -al ~/.ssh
```
This shows a list of files in your .ssh directory, if it exists. Look for files named `id_rsa.pub`, `id_ecdsa.pub`, `id_ed25519.pub`, etc. These are public key files (if they exist).

## Generate a New SSH Key
If you don’t have an SSH key or want to create a new one specifically for GitHub, use the following command. Replace "your_email@example.com" with your GitHub email address:
```
ssh-keygen -t ed25519 -C "your_email@example.com"
```

- `-t ed25519`: Specifies the type of key to create, `ed25519` is preferred for its security properties.
- `-C`: Adds a label to the key, usually your email.
When prompted to "Enter a file in which to save the key," press Enter to accept the default file location (usually `~/.ssh/id_ed25519`).

## Start the SSH Agent
To make it easier to use your SSH key, you can add it to the SSH agent, which manages your SSH keys and remembers your passphrase:
```
eval "$(ssh-agent -s)"
```

```
ssh-add ~/.ssh/id_ed25519
```

If you saved your key with a different name, or in a different location, replace `~/.ssh/id_ed25519` with the path and name of your private key file.

## Add the SSH Key to Your GitHub Account
1. Copy the SSH public key to your clipboard. To copy the contents of the public key file to your clipboard use:

```
pbcopy < ~/.ssh/id_ed25519.pub
```

2. If you used a different path or key name, be sure to change the path accordingly.

3. Go to GitHub and sign in.

4. Click on your profile photo, then click Settings.

5. In the user settings sidebar, click SSH and GPG keys.

6. Click on New SSH key or Add SSH key.

7. In the "Title" field, add a descriptive label for the new key, such as "My MacBook Pro".

8. Paste your key into the "Key" field (the key should already be in your clipboard).

9. Click Add SSH key.

## Test Your SSH Connection
Test your SSH connection to make sure it’s working correctly:
```
ssh -T git@github.com
```

You may receive a warning like this:
```
The authenticity of host 'github.com (IP ADDRESS)' can't be established.
RSA key fingerprint is SHA256:xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx.
Are you sure you want to continue connecting (yes/no/[fingerprint])?
```

Type yes, and then press Enter.

If everything is set up correctly, you will see a message like:
```
Hi username! You've successfully authenticated, but GitHub does not provide shell access.
```

This confirms that your SSH key has been successfully added to your GitHub account and that you can communicate with GitHub securely using SSH.

## Troubleshoot

If you are prompted for your username and password when you attempt to push to GitHub, even though you have already set up SSH authentication, it's likely because your local Git repository is configured to use HTTPS URLs for the remote repository instead of SSH URLs. SSH keys will only work with SSH URLs.

### Check Current Remote URL
First, check what the current remote URL is for your repository. Open your terminal, navigate to your repository, and run:
```
git remote -v
```
This command will show you the URLs that Git has stored for the remote repository, typically named `origin`. If the URLs start with `https://`, then Git is set up to use HTTPS.

### Switch to SSH
If you find that the remote URL is an HTTPS URL, you can switch it to use SSH. You will need the SSH URL of your GitHub repository, which looks like git@github.com:username/repository.git.

To change the URL, use the git remote set-url command:
```
git remote set-url origin git@github.com:username/repository.git
```

### Verify the Change
To ensure that the remote URL has been changed, run:
```
git remote -v
```

Now, the output should show the SSH URL beginning with git@github.com.

### Try Pushing Again
Now that your repository is set up to use SSH, try pushing again:
```
git push
```

This time, it should not ask for your username and password if everything is set up correctly with SSH.

