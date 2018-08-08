Git CI
======

This is a suite of tools for making a CI server which handles Git repositories.

CI User Setup
-------------

Create a user which will be used for CI activities.
This assumes you have admin access to the server.

```sh
sudo useradd -m -s `which git-shell` ci
```

To change into that user, run:

```sh
sudo su -s `which bash` ci
```

Setup the user to your preferences.
Remember to add SSH access for users who you want to be able to admin the CI-server.

Repo Setup
----------

The following are run as the `ci` user created above on your CI server.

Setup `src` and `git/src` directories.

```sh
cd ~/
mkdir -p ~/git/src ~/src
```

Setup `src/git-ci` to track `git/src/git-ci.git`:

```sh
cd ~/git/src
git init --bare git-ci.git
cd ~/src
git clone ~/git/src/git-ci.git
```

Push the correct version of the `git-ci` repo to the server (on your local machine):

```sh
git remote add ci 'ssh://CI-SERVER/~/git/src/git-ci.git'
git push ci --all
```

Back on the server, pull down the changes:

```sh
cd ~/src/git-ci
git pull
```

Also symlink the `git-shell-commands` directory to enable remote `git-shell` access:

```sh
cd ~/
ln -s src/git-ci/git-shell-commands
```

And make sure that the `PATH` variable includes the `bin` directory for `git-ci` (add to your `~/.bashrc`):

```sh
export PATH="$PATH:/path/to/ci/src/git-ci/bin"
```

Git Tools
---------

### Shell Commands

Directory `bin` contains extra `git` commands which help with using this tool.

-   `git-ci` is the main test executor.
-   `git-replay` allows replaying an existing git commit.
-   `git-rebased-on` checks if the given pair of commits are related ancestrally.

### Interactive Server `git-shell`

Directory `git-shell-commands` contains helpful commands for admins of the CI server.

-   `help` lists the available commands.
-   `sync` syncs the checked out version of the CI repo with the bare version.
-   `bb` provides a simple interface to a running instance of Buildbot.

CI Servers
----------

### Buildbot

Directory `buildbot` contains instructions for setting up Buildbot and a sample config.
