# Session 2: `ssh`

The following tasks will help you setup `ssh` and familiarize yourself with `ssh` keys.

## Task 1: create an ssh key pair

In this exercise you'll create an ssh key and upload it to GitHub so you can use it download ("pull"), and upload ("push") changes to git repositories.

**Before doing step 1, check if you already have a key pair generated. Run:**
```
ls ~/.ssh/id_ed25519
```
**If a file path is returned, skip step 1. Otherwise, if you get "No such file or directory", do step 1.**

1. Go to [this help article on GitHub](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent), select which operating system you are using for your terminal (either Mac or Linux), then scroll down to "Generating a new SSH key" and follow the instructions there. Stop when you get to "Adding your SSH key to the ssh-agent".

2. The keygen command should have created a public and private key pair in your `~/.ssh` directory. This is a hidden directory that contains your ssh keys, configuration settings, and other files related to ssh. Verify that the keys exist by typing:
```
ls ~/.ssh
```
You should see at least two files in there, one called `id_ed25519` and one called `id_ed25519.pub`. (You may also have other `id_*` files in there, if you had created key pairs previously.)

The `id_ed25519` file is your **private** key. **Do not share your private key with anyone!** It remains on your computer, and only on your computer. The `id_ed25519.pub` file is your **public** key. You can share the contents of your public key with other servers you want to log in to.

----
*Note: you need a GitHub account to do the following steps. If you do not have a GitHub account, sign up for one by going to `github.com` and following the onscreen instructions. Once you have an account you can proceed.*
----

3. Now let's upload your public key to GitHub so that you can use your ssh keypairs when interacting with GitHub. Follow the instructions here: [Adding a new SSH key to your GitHub account](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account?platform=linux#adding-a-new-ssh-key-to-your-account).

4. To test whether you were successful run
```
ssh -T git@github.com
```
You may be prompted with something like:
```
> The authenticity of host 'github.com (IP ADDRESS)' can't be established.
> ED25519 key fingerprint is SHA256:+DiY3wvvV6TuJJhbpZisF/zLDA0zPMSvHdkr4UvCOqU.
> Are you sure you want to continue connecting (yes/no)?
```
If so, type `yes`. You should then get
```
Hi USERNAME! You've successfully authenticated, but GitHub does not provide shell access.
```
where `USERNAME` is your username.

5. Now that you have an ssh key pair setup with github, you can use `ssh` URLs for cloning git repositories. Try it! Click the green `<> Code` button in the upper right of this repository, then click the `SSH` tab. Copy the URL there. Go to your terminal app and type `git clone ` in your terminal, then paste in the URL you copied from the website. Hit `<enter>`. The repo should clone. Type `ls` to verify it worked. You should see a `ospo-workshop-computing101-2` directory.


## Task 2: Modify ssh config

The standard way to log into a server using `ssh` is to type:
```
ssh USERNAME@HOSTNAME
```
where `USERNAME` is your username *on the server* (which may not be the same as on your computer) and `HOSTNAME` is the address of the server. For example, above we ran `ssh -T git@github.com`. There, `git` was the `USERNAME` and `github.com` was the `HOSTNAME`. This can be shortened somewhat if your username on the server is the same as on your computer; in that case you can just type `ssh HOSTNAME`. Even then, typing hostnames out can be tedious and can be difficult to remember or type. 

You can simplify what you need to type to login to a server by modifying the `~/.ssh/config` file. The syntax for the `config` file is to have blocks of text for each server that you want to set configurations for. In general, the syntax is:

```
Host HOST_ALIAS
    HostName HOSTNAME
    User USERNAME
    ETC.
```

Here, `HOST_ALIAS` is the string you want to type on the command line to instruct `ssh` to login in to a specific server, `HOSTNAME` is the full server address, `USERNAME` is your username on that server, and `ETC.` is any other commands you want to give to modify what `ssh` uses when it logs in. For example, another common command to set is `IdentityFile`, which can be used to tell `ssh` to use a specific key for a server (this is needed in the case that you have multiple ssh keys).

To illustrate how this works, open `~/.ssh/config` using your favorite text editor and add the following:
```
Host github
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519
```
then save and exit the file. Now, on the command line, type:
```
ssh -T github
```
You should get the same "You've successfully authenticated" message you got above. Notice that you did not need to type out `git@github.com`. This is because your `config` file instructed `ssh` to login to `github.com` using the user `git` when it's given host name `github`.

If you want to do add more servers, add new blocks of text to your config file, each beginning with `Host`. Wild cards can also be used for `Host` to match multiple servers. For example, if you want to some settings to apply to all servers, create a section called `Host *`.

## Task 3: Setting up an ssh agent

If you protected your `ssh` key with a passphrase when you ran `ssh-keygen` (it's best practice to do so), then every time you use your ssh keys you'll need to reenter your passphrase. This can get annoying, especially when you use the keys often; e.g. to push code to GitHub. You can make it so that you only need to enter your passphrase once per session by turning on an *ssh agent*. To do that, run the following:
```
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
```
When you run the `ssh-add` command you will be prompted for your passphrase for that key. After you enter it, you will not need to enter your passphrase again for the rest of your session.

-----

*Optional:* Now that you have a `~/.ssh/config` file setup and you know how to turn on an ssh-agent, you may want to enable ssh agent forwarding. This will cause your ssh-agent to be "forwarded" to a server that you login to, so that it can unlock your ssh keys on your behalf, without needing separate keys stored on the server. This is particularly useful, if you want to pull/push from/to GitHub from a server using the ssh key you created on your laptop. To enable forwarding for all hosts, add the following to your `~/.ssh/config` file:

```
Host *
    ForwardAgent yes
```
