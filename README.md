# Dot repo

This is a repo to help automate the installation and management of your configuration files in your home directory, typically referred to as "dot files" because many of them start with `.`, e.g. `.bashrc`.

Your files are not simply copied to your home directory, but instead they are symlinked in. This means changing those files in your home directory will automatically be reflected in your "dot files git repo", making tracking, committing and pushing those changes much easier.

# Getting started

To use this, you will have your own git repo that contains your personal dot files. Your dot files repo will be contain all _your_ files, and this "dot core repo" is added to your dot files repo as a submodule. This way you are not tied to any of my personal dot files (https://github.com/andyneff/dot_files.git), but you get all the install and uninstall functionality with bug fixes and all additional features as they are added.

The very first time you set up your dot files repo, you should create an empty directory (I suggest `~/.dot`) and then run the `new_repo.bsh` script. Here's how to do that:

1. `mkdir ~/.dot`
2. `cd ~/.dot`
3. `bash <(curl -L https://raw.githubusercontent.com/andyneff/dot_core/main/new_repo.bsh)`

And that is it for getting started. You will have a `files` directory where you place your configuration files, in the same directory structure to mimic how you want them installed in your home directory. E.g.:

- `files/.bashrc` -> `~/.bashrc`
- `files/.ssh/config` -> `~/.ssh/config`

This "dot files repo" works best as a public repo. That way you do not need to have an ssh key setup to do your initial clone, and the install script can walk you through setting that up for you.

Remember not to include any secret information in there (like your ssh private keys or license keys) even if they are encrypted. It's bad security practice to make an encrypted ssh key public or even store it in a private repo on the clouds (github.com, bitbucket.org, gitlab.com, etc...). The install script instead guides you through setting up your own ssh key for each computer.

# Features

- Installs files in any directory structure you want in your home directory
- Handles your git server (typically github) ssh key generation for you. This way you do _not_ commit a private ssh key in any git repo (which is best security practice)
- Use symlinks so that changes are reflected in the original repo
    - Symlink feature requires admin privileges on Windows 10/11. If you do not have them you can set it up to copy instead.
- Backs up existing files before they are replaced
    - Restores using the backup files on uninstall
    - Continues to backup every ~unique~ version of a file that is replace. For example, an external process may decide to undo the symlink and change a file (outside of dot files' control). This is a safety mechanism to ensure that things are never lost by accident.

## Advanced features

- Supports multiple git repos of your dot files, so you have have some public and some private (license keys, etc...)
- Download simple files (like `jq`, etc...) from a URL and put them in your `~/bin` directory
- For everything else, there is a custom install/uninstall steps to support scenarios more complex than a symlink. This is a simple bash file where you can either write all your custom steps in bash, or call your own scripts in whatever language you prefer.
- Simple `--config` for install to automatically trigger `skip_if_env` checks

# Typical Installation

Once you have your dot files repo setup, make sure your `README.md` reads:

1. `git clone https://{your git server}/{your git repo}.git ~/.dot`
  - For example: `git clone https://github.com/andyneff/dot_files.git ~/.dot`
2. `~/.dot/install.bsh`
    - The script will walk you trough creating your ssh key on your computer, if you haven't, and cloning the submodules, and then finally installing all your dot files.
    - Optional `~/.dot/install.bsh --config`

# Updating

1. `cd ~/.dot`
1. `git pull origin main`
1. `./install.bsh`

# Uninstall

If the installation script errors for some reason, and cannot be resumed, or you just want it to remove all the symlinks for you, you can run

- `~/.dot/uninstall.bsh`
    - Note: This will only remove symlinks for files that are still in your dot_repos.
    - This will also call all `unsetup` functions in your `custom.bsh` files.
    - Any files that were backed up during installation, will be restored during uninstallation.

# Feature Explanations

## Install script

The `install.bsh` script in your "personal dot files repo" is designed and intended to be run every time you add new files. Run your `install.bsh` script every time you:

- Setup a new computer
- Add a new file in your `files` dir
- Pull from your latest changes

The `new_repo.bsh` will create a small `install.bsh` script that will call the `dot_core/install_common.bsh` script for you. It is intended you not modify `install.bsh`, but instead add custom steps to your `custom.bsh` file. This maintains an easy upgrade path should any bug fixes/feature changes require your `install.bsh` script to be updated.

The install script has the following steps:

0. `install.bsh` will make sure the `dot_core` submodule is initialized and checked out. The rest of the steps are controlled by the `dot_core/install_common.bsh`
1. It will check for known dependencies and error out if they cannot be found, like `find`
2. It will make sure `dot_core`'s submodules are initialized and checked out.
3. It will check to see if your SSH key is generated. Default `~/.ssh/id_ed25519` (`DOT_GIT_SERVER_SSH_KEY`)
  - If you key is not generated yet, it will start the generation process for you.
    - It will pause and wait for you to upload the new ssh key to your git server (e.g. github.com). Do this now if you have any private submodules in your dot_files.
  - It will make sure you use a passphrase, if you do not the key will be deleted and exit so you can try again
    - There is no option to bypass this by design, you'd have to create the key yourself.
4. All additional submodules will now be checked out. Until now all submodules have used `https` because they are public and did not require a git ssh key. Now that the ssh key is setup and ready to use
5. Advanced feature: `DOT_*_DOWNLOAD_URLS` files are downloaded and installed for you.
6. Finally all the files in the `files` directories are symlinked over.
  - Backups are made for preexisting files
  - The `custom.bsh` `setup` function is called after each `files` directory is processed.
7. The permissions of your `DOT_DIR` (`~/.dot`) is set to `DOT_DIR_PERMISSIONS`. Default 700

## Environment Variables

TODO: Compile and upload doc, link here

- `DOT_DIR` - the location of your person dot repo, traditionally: `~/.dot`

All environment variables can be overwritten by setting them in your personal `dot.env` file, in the root of your dot files repo.

# Advanced Feature Explanation

## dot.env

All of the variables set in the "dot repo" `dot.env` files, can be overridden by simply setting them in your person dot files repo very own `dot.env` file. For example:

```bash
DOT_GIT_SSH_KEY_TYPE=rsa
```

No fancy bash expressions needed. This allows you can permanently set that value just for your dot files, and never have to modify the "dot repo". However, this is a bash source file, so any bash expressions are allowed, such as:

```
if [ "$(hostname)" = "cat" ]; then
  # The computer named cat must use RSA cause I made this up
  DOT_GIT_SSH_KEY_TYPE=rsa
fi
```

## custom.bsh

Currently, the install script automatically symlinks, in the exact same directory structure as your `files` directory. Sometimes you need more, you need to edit files, and insert a line. Any custom actions can be added to your `custom.bsh` file

- `setup` - If you define a function called `setup` in `custom.bsh`, it will be called after the `files` have been installed.
- `unsetup` - If there are any changes that have to be undone when `uninstall.bsh` is called, they can be added to a function called `unsetup`

**Note:** This is an advanced feature indented for people who are comfortable writing their own setup code. The default is to write in `bash`, but you can all your own scripts here and use whatever language you prefer

Polyglot example:

```bash
function setup()
{
  perl "${DOT_DIR}/custom_setup.pl"
}
```

## additional_repos

Either for modularity, or for the ability to have a private repo to store information you would rather keep to yourself (like server names or license keys), you can add as many submodules as you want in the `additional_repos` directory. `additional_repos` are cloned after you ssh key is set up, so the process is smoothed out and automated

Each submodule should mirror the same layout as the original "dot files repo"

- `files` - directory to store all your configuration files
- (optional) `dot.env` - An environment files for you to override environment variables
- (optional) `custom.bsh` - A custom script containing a `setup` and `unsetup` function to customize any steps that are not strictly symlinks

Remember, you still shouldn't add information like your ssh private keys, even if they are encrypted. Unlike a license key, a private ssh key gives access to other servers, and you are inherently trusting the git server with these keys, which is bad practice for the ssh key ecosystem.

# FAQ

1. Why is generating an ssh key part of install?
    - When setting up a new computer, in order to get ssh protocol working for git (`git@...`), a key needs to be added to the git server, but this will happen after the initial clone. The following use pattern solves this chicken/egg problem gracefully:
    1. You clone your main dot files repo using https, since it can be public
    1. The install script will make sure you have an ssh key, if you do not already, it will assist you in generating it
    1. It will print out the public key to the screen, so you can go to your git server and add it, before continuing
    1. Now the install script will recursively checkout all your submodules so that can use both `https` and `git@` protocols in your submodules.
    1. Then the rest of the install script runs.
1. I keep getting `You are not running with admin rights, mklink will probably fail` warning on windows.
    - On Windows 10/11, you need to call `install.bsh` in bash running with admin privileges due to symlink limitations (Right click "Git Bash" and select "More" -> "Run as Administrator")
    - If you are unable to run with admin privileges, you will have to run: `FORCE_LN=1 ~/.dot/install.bsh`
        - However, this will copy the files instead of symlinking them, making committing changing back to the repo harder
        - Note this can be accomplished editing the `dot.env` file in your dot_repos with something like:

              if [ "$(hostname)" = "problem computer name here" ]; then
                FORCE_LN=1
              fi
