# Dot files

This is a repo to help automate the installation and management of your configuration files in your home directory, typically referred to as "dot files" because many of them start with `.`, e.g. `.bashrc`. 

Essentially you have a directory structure in a `files` dir in your `dot_repos` submodules, and that exact same structure will be mimiced onto your home directory.

These files are not simply copied to your home directory, but instead of symlinked in. This means changing those files in your home directory will automatically be reflected in this git repo, making committing and pushing those changes easier.

# Installation

1. `git clone https://github.com/andyneff/dot.git ~/.dot`
2. `~/.dot/install.bsh`
    - The script will walk you trough creating your ssh key on your computer, if you haven't, and cloning the submodules, and then finally installing all your dot files.
    - On Windows 10, this needs to be done in bash running with admin privileges due to symlink limitations (Right click "Git Bash" and select "More" -> "Run as Administrator")
    - If you are unable to run with admin privileges, you will have to run: `FORCE_LN=1 ~/.dot/install.bsh`
        - However, this will copy the files instead of symlinking them, making committing changing back to the repo harder
        - Note this can be accomplished editing the `dot.env` file in your dot_repos with something like:

              if [ "$(hostname)" = "problem computer name here" ]; then
                FORCE_LN=1
              fi

# Uninstall

If the installation script errors for some reason, and cannot be resumed, or you just want it to remove all the symlinks for you, you can run

- `~/.dot/uninstall.bsh`
    - Note: This will only remove symlinks for files that are still in your dot_repos.
    - This will also call all `unsetup` functions in your `custom.bsh` files.
    - Any files that were backed up during installation, will be replaces during uninstallation.

# Forking

The general idea is for you to fork this repo, but still be able to get the updates:

1. Fork it
1. `git clone https://github.com/yourfork/dot.git ~/.dot` (Note, `~/.dot` can be anywhere you want)
1. `cd ~/.dot`
1. `git rm dot_repos`
1. `git submodule add git@github.com:username/dot_files.git ./dot_repos/50_my_files`
    - Name `50_my_files` whatever you want, but it must be in the `dot_repos` directory.
    - Multiple dot repos are processed in alphabetical order)
1. (optional) Change the url in the git clone command above ⬆ in `README.md`
1. In your submodules you _must_ have
    1. `files` directory - This is where you files mimic the directory structure of your home dir. For example:
        - `./files/.bashrc` for `~/.bashrc`
        - `./files/.ssh/config` for `~/.ssh/config`
        - **Note**: It is not considered secure to add your private ssh keys to your submodule, even with a passphrase.
1. In your submodules you _may_ have 
    1. `dot.env` - This file is where you can override the default environment variables set it the main `dot.env` file. For example:
        - `DOT_GIT_SSH_KEY_TYPE=ed25519`
    1. `custom.bsh` - file where you can define a `setup` and `unsetup` function that is called when installing/uninstalling. This is where you can handle all sorts of custom actions that are not "just symlink this file in", like appending lines to config files, etc...
    1. Any other files you want, that `custom.bsh` can use, or licenses, read me, etc...

This will allow you to make a minimal commit to your fork, which replace my submodules with yours. Then every time this main fork of the main submodule is updated, all you have to do is rebase/merge those new changes, and still easily maintain all of _your_ files and your custom scripts.

# FAQ

1. Why generate the ssh key?
    - When setting up a new computer, in order to get ssh protocol working for git, a key needs to be added to the git server, but this will happen after the initial clone.
    1. Clone main repo using https, since it can be public
    1. The `install.bsh` script will make sure you have an ssh key, if not, generate it
    1. It will make sure you use a passphrase, cause you should
    1. It will print out the public key to the screen, so you can go to your git server and add it, before continuing
    1. Now the install script will recursively checkout all your submodules so that can use both `https` and `git@` protocols in your submodules.
    1. Then the rest of the install script runs.
