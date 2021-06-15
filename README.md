1. `ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa_gh`
2. Add your ssh key to your [github account](https://github.com/settings/ssh/new)
3. `GIT_SSH_COMMAND="ssh -i ~/.ssh/id_rsa_gh -F /dev/null" git clone --recursive git@github.com:andyneff/dot.git ~/.dot`
4. `~/.dot/install.bsh` # Need admin in windows. If you can't get Admin, export `FORCE_LN=1`

Older versions of git (pre 2.1.0) need to use the `GIT_SSH` variable to point to a script with those arguments for step 3:

```
printf '#!/usr/bin/env bash\nssh -i ~/.ssh/id_rsa_gh -F /dev/null "${@}"\n' > ~/.tmp_git_ssh
chmod 755 ~/.tmp_git_ssh
GIT_SSH=~/.tmp_git_ssh git clone --recursive git@github.com:andyneff/dot.git ~/.dot
rm ~/.tmp_git_ssh
```
