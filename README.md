# Dot files

This is a repo to help automate the installation and management of your configuration files in your home directory, typically referred to as "dot files" because many of them start with `.`, e.g. `.bashrc`. 

Essentially you have a directory structure in a `files` dir in your `dot_repos` submodules, and that exact same structure will be mimics unto your home directory.

These files are not simply copied to your home directory, but instead of symlinked in. This means changing those files in your home directory will automatically be reflected in this git repo, making committing and pushing those changes easier.

# Installation

1. `git clone https://github.com/andyneff/dot.git ~/.dot`
2. `~/.dot/install.bsh`
    - The script will walk you trough creating your ssh key on your computer, if you haven't, and cloning the submodules, and then finally installing all your dot files.
    - On Windows 10, this needs to be done in bash running with admin privileges due to symlink limitations (Right click "Git Bash" and select "More" -> "Run as Administrator")
    - If you are unable to run with admin privileges, you will have to run: `FORCE_LN=1 ~/.dot/install.bsh`
        - However, this will copy the files instead of symlinking them, making committing changing back to the repo harder

# FAQ

1. Why do you not just use `.bashrc` directly?
    - I've deployed this on enough computers that I don't administer, to know some admins do annoying things, like write `.bashrc` and `.ssh/config` (usually on ssh login, but not always) for you. I have thus far been able to get away with this additive approach, and not have to do anything special to the git repo
    - Also, this allows me to to have computer specific things in the actually `.bashrc`/`.ssh/config`/etc... files, and not worry about "ok, I don't want to commit that specific file"
1. How can I use this?
    - The idea is...
    1. Fork this main repo
    2. Write add your own repos (public/private/a few of each, your choice) to the `dot_repos` dir, as submodules
        - Now all your dot files go in that submodule, including any custom scripts you may need
    3. Change the url in the git clone command above ⬆.
    4. Commit that change on your fork
    5. Now when even you want to update to my fork, you can update the main branch, and simply rebase onto that.
