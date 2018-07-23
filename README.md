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

Repo Setup
----------

As a `ci` user on the CI server:

```sh
cd ~/
mkdir -p git/src
mkdir src
cd git/src
git init --bare git-ci.git
cd ../../src
git clone ../git/src/git-ci.git
```

On your local machine, hookup the `git-ci` repo to the remote on the CI machine:

```sh
cd src/git-ci
git remote add ci 'ssh://ci/~/git/src/git-ci.git'
git push ci --all
```

Back on the server, pull down the changes:

```sh
cd src/git-ci
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

Directory `bin` contains extra `git` commands which help with using this tool.

-   `git-ci` is the main test executor.
-   `git-replay` allows replaying an existing git commit.
