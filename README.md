# Dot repo

This is a repo to help automate the installation and management of your configuration files in your home directory, typically referred to as "dot files" because many of them start with `.`, e.g. `.bashrc`.

Your files are not simply copied to your home directory, but instead they are symlinked in. This means changing those files in your home directory will automatically be reflected in your "dot files git repo", making tracking, committing and pushing those changes much easier.

# Your personal dot files

To use this, you will be using two git repos. You will have your personal "dot files repo" which will be all yours. And this "dot repo" is added to your "dot files repo" as a  submodule. This way you are not tied to any of my personal dot files (https://github.com/andyneff/dot_files.git), but you  get all the install and uninstall functionality with bug fixes for free!

# Features

- Installs files in any directory structure you want in your home directory
- Use symlinks so that changes are reflected in the original repo
    - Symlink feature requires admin privileges on Windows 10
- Custom install/uninstall steps to support scenarios more complex than a symlink
- Backs up existing files before they are replaced
    - Restores using the backup files on uninstall
    - Continues to backup every ~unique~ version of a file (in case it is replaced/changed outside of dot files). This is safety mechanism so that things are never lost by accident.

# Getting started

The very first time you set up your "dot files repo", you should create an empty directory (I suggest `~/.dot`) and then run the `new_repo.bsh` script. Here's how to do that:

1. `mkdir ~/.dot`
2. `cd ~/.dot`
3. `bash <(curl -L https://raw.githubusercontent.com/andyneff/dot/main/new_repo.bsh)`

And that is it for getting started. You will have a `files` directory where you place your configuration files, in the same directory structure to mimic how you want them installed in your home directory. E.g.:

- `files/.bashrc` -> `~/.bashrc`
- `files/.ssh/config` -> `~/.ssh/config`

This "dot files repo" works best as a public repo. That way you do not need to have an ssh key setup to do your initial clone, and the install script can walk you through setting that up for you.

Remember not to include any secret information in there (like your ssh private keys or license keys) even if they are encrypted.

# Installation

1. `git clone https://github.com/andyneff/dot.git ~/.dot`
2. `~/.dot/install.bsh`
    - The script will walk you trough creating your ssh key on your computer, if you haven't, and cloning the submodules, and then finally installing all your dot files.

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

# Advanced

## custom.bsh

TODO

## dot.env

TODO

## additional_repos

Either for modularity, or for the ability to have a private repo to store information you would rather keep to yourself (like server names or license keys), you can add as many submodules as you want in the `additional_repos` directory. `additional_repos` are cloned after you ssh key is set up, so the process is smoothed out and automated

Each submodule should mirror the same layout as the original "dot files repo"

- `files` - directory to store all your configuration files
- (optional) `dot.env` - An environment files for you to override environment variables
- (optional) `custom.bsh` - A custom script containing a `setup` and `unsetup` function to customize any steps that are not strictly symlinks

Remember, you still shouldn't add information like your ssh private keys, even if they are encrypted. Unlike a license key, a private ssh key gives access to other servers, and you are inherently trusting the git server with these keys, which is bad practice for the ssh key ecosystem.

# FAQ

1. Why is generating an ssh key part of install?
    - When setting up a new computer, in order to get ssh protocol working for git (`git@...`), a key needs to be added to the git server, but this will happen after the initial clone.
    1. Clone main repo using https, since it can be public
    1. The `install.bsh` script will make sure you have an ssh key, if not, generate it
    1. It will make sure you use a passphrase, cause you should
    1. It will print out the public key to the screen, so you can go to your git server and add it, before continuing
    1. Now the install script will recursively checkout all your submodules so that can use both `https` and `git@` protocols in your submodules.
    1. Then the rest of the install script runs.
1. I keep getting `You are not running with admin rights, mklink will probably fail` warning on windows.
    - On Windows 10, you need to call `install.bsh` in bash running with admin privileges due to symlink limitations (Right click "Git Bash" and select "More" -> "Run as Administrator")
    - If you are unable to run with admin privileges, you will have to run: `FORCE_LN=1 ~/.dot/install.bsh`
        - However, this will copy the files instead of symlinking them, making committing changing back to the repo harder
        - Note this can be accomplished editing the `dot.env` file in your dot_repos with something like:

              if [ "$(hostname)" = "problem computer name here" ]; then
                FORCE_LN=1
              fi
