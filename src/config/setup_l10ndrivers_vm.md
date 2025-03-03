# Setting up a Linux Virtual Machine for Webdashboards

This document describes how to configure a virtual machine based on Linux Ubuntu 18.04 LTS to manage tools like Langchecker, Bedrock (mozilla.org). The assumption is that you’ve already installed a software to manage virtual machines, like VMWare Fusion on Mac, and installed Linux. The virtual machine should have at least 40 GB of disk, 2 cores and 4 GB or memory.

## Update the system

First of all, make sure to update the installed packages. Search for the terminal and type:

```BASH
sudo sh -c "apt update;apt dist-upgrade -y;apt autoremove -y"
```

The password for `sudo` is the same you use to login to the system with your user. Once finished, reboot the virtual machine to make sure that all packages are updated.

If you’re using VMWare, make also sure to install VMWare Tools:

```BASH
sudo apt install open-vm-tools open-vm-tools-desktop -y
```

Restart the virtual machine at the end to make sure all packages are up to date.

## Configure Git and repositories

Before starting the setup you need to make sure to setup Git to use SSH on this VM.

Install Git:

```BASH
sudo apt install git -y
```

Generate a new SSH key for this VM (replace the email address placeholder), make sure to setup a passphrase when asked:

```BASH
ssh-keygen -t rsa -b 4096 -f ~/.ssh/github_ssh -C "YOUR_EMAIL_ADDRESS"
```

File will be stored in `~/.ssh/github_ssh`. The next step is to [add this key](https://help.github.com/articles/adding-a-new-ssh-key-to-your-github-account/#platform-linux) to your GitHub account.

```BASH
cat ~/.ssh/github_ssh.pub
```

Add a new key in [your profile](https://github.com/settings/keys) and copy and paste the test displayed by the previous command, starting with `ssh-rsa` and finishing with the email address you provided.

At this point you’re ready to test your connection to GitHub (you will need to accept the certificate by typing `yes`)

```BASH
ssh -T git@github.com
The authenticity of host 'github.com (192.30.253.112)' can't be established.
RSA key fingerprint is SHA256:nThbg6kXUpJWGl7E1IGOCspRomTxdCARLviKw6E5SY8.
Are you sure you want to continue connecting (yes/no)? yes
Hi USERNAME! You've successfully authenticated, but GitHub does not provide shell access.
```

It will ask for your passphrase, you can set the system to unlock this key whenever you’re logged in by selecting the checkbox under the input field.

You should also tweak your Git configuration with these commands (replace placeholders with your email address and name, they will be used for commit messages):

```BASH
git config --global user.email "YOUR_EMAIL_ADDRESS"
git config --global user.name "YOUR_NAME"
git config --global push.default simple
```

## Run the setup scripts

At this point you can run this command to download and execute a setup script. Note that this script must be run with `sudo`, since it needs to install several packages on the system.

```BASH
sudo sh -c "wget -O - https://raw.githubusercontent.com/mozilla-l10n/vm_scripts/master/setup_vm/setup_vm.sh | bash"
```

Then run this script as standard user to clone all requested repositories:

```BASH
~/mozilla/git/scripts/setup_vm/setup_repositories.sh
```

The entire procedure will take several minutes, depending on the speed of your Internet connection. During the process you will be asked, possibly more than once, to **create a token on GitHub for Composer**: follow the link to generate one and copy and paste it in the terminal.

All projects are now cloned using https and the original repository as `origin` (e.g. mozilla-l10n for langchecker), so you won’t be able to commit any changes. **If you plan to make updates and commit them**, make sure to create forks of the following repositories in your account:
* langchecker: https://github.com/mozilla-l10n/langchecker
* webdashboard: https://github.com/mozilla-l10n/webdashboard
* bedrock: https://github.com/mozilla/bedrock

Then run the following script providing your GitHub username (e.g. `flodolo`): it will setup the original repository as a remote called `upstream`, and your fork as `origin`. It will also switch relevant l10n repositories, like mozilla_org, to ssh to allow you to make direct commits.

```BASH
~/mozilla/git/scripts/setup_vm/setup_remotes.sh GITHUB_USERNAME
```

Make sure to restart the terminal to enable the new command aliases. It’s recommended to use *Terminator*, installed as part of the setup script: it’s an improved terminal with support for tabs and split windows.
