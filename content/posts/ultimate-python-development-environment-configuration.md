---
title: "Ultimate Python development environment configuration"
date: 2022-08-24
tags: ["python"]
draft: false
---

*Disclaimer: **I’m not saying** `virtualenv` methods are bad.*

## The Old way

For years, I've used Python for a lot of different projects. My setup was simple: use `virtualenv` and work locally.

But this approach came with a few pain points. I'd often forget to `source venv/bin/activate`, which meant I'd end up running `pip install my-package` on my global system instead.

I also had poor secrets management, setting them unencrypted in a shell script loaded at program start.

As bad as this:

```
export DB_NAME="db_name"
export DB_USER="user"
export DB_PASS="pass"
export DB_HOST=172.17.0.2
export DB_PORT=3306
```

Even though some great code editors, like VSCode, can handle Python interpreters inside a virtual environment, I felt my workflow automation still wasn't as complete as it could be.

## Direnv

How do we make sure we never forget to load a virtual environment, or accidentally install a package globally while working on a project?

## What is direnv and how it works

From the official documentation:

> *direnv is an extension for your shell. It augments existing shells with a new feature that can load and unload environment variables depending on the current directory. Before each prompt, direnv checks for the existence of a .envrc file (and optionally a .env file) in the current and parent directories. If the file exists (and is authorized), it is loaded into a bash sub-shell and all exported variables are then captured by direnv and then made available to the current shell.*

> *It supports hooks for all the common shells like bash, zsh, tcsh and fish. This allows project-specific environment variables without cluttering the ~/.profile file.*

> *Because direnv is compiled into a single static executable, it is fast enough to be unnoticeable on each prompt. It is also language-agnostic and can be used to build solutions similar to rbenv, pyenv and phpenv.*

## direnv installation

`direnv` is available as a package in almost all distributions.

## Package installation

For a global system installation:

```
λ ~/ sudo apt-get install direnv
```

## Manual installation

For a custom installation, take a look at this [script](https://direnv.net/install.sh) hosted by the official direnv documentation.

```
λ ~/ curl -sfL https://direnv.net/install.sh | bash
```

## Shell configuration

Once `direnv` is installed, you need to configure your `$SHELL` to hook it into your default shell. It supports `bash`, `zsh`, `fish`, `tcsh`, and `elvish`.

*e.g. with zsh*

```
λ ~/ echo 'eval "$(direnv hook zsh)"' >> ~/.zshrc
λ ~/ source ~/.zshrc
```

## How python works with direnv

We can load a Python virtual environment thanks to `direnv`. To do so, we need to specify the `layout` command in a `.envrc` file located at our project root.

Let's create our project directory first.

```
λ ~/ mkdir project && cd project
```

Create our `.envrc` file.

```
λ ~/project/ echo 'layout python3' > .envrc
```

You'll notice an error message like this:

```
direnv: error project/.envrc is blocked. Run `direnv allow` to approve its content
```

This is a security measure to block default content in the file. You can allow it with:

```
λ ~/project/ direnv allow
direnv: loading ~/project/.envrc 
direnv: export +VIRTUAL_ENV ~PATH
```

The output tells us the virtual environment was created automatically. You won't see any prompt modification like `virtualenv` gives you with `source venv/bin/activate`, that's normal.

Quick check of our Python binary:

```
λ ~/project/ which python
project/.direnv/python-3.10.4/bin/python
```

You can then work as you normally would with a `venv`, installing your Python dependencies as usual.

```
# install a single package
λ ~/project/ pip install django
# or install your project dependencies
λ ~/project/ pip install -r requirements.txt
```

*Note: we'll stop using `pip` in favor of `poetry`, see the Poetry section below for more details.*

Just to be clear: `direnv` automatically loads your virtual environment when you move into your project directory, and automatically deactivates it as soon as you move out.

```
λ ~/project/ cd 
direnv: unloading
```

That's enough for common use if you don't need a specific Python version beyond what's already installed on your system. But sometimes you'll want to work with specific versions, which is where `pyenv` comes in.

## Pyenv

`pyenv` lets you easily switch between multiple versions of Python. It's simple, unobtrusive, and follows the Unix tradition of single-purpose tools that do one thing well. It lets you set a Python version per project, and has been supported by `direnv` since `2.21.0`.

## Installation and Configuration

Get `pyenv`:

```
λ ~/ curl -L https://pyenv.run | bash
```

Configure your $SHELL.

*e.g. with zsh*

```
λ ~/ echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.zshrc
λ ~/ echo 'command -v pyenv >/dev/null || export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.zshrc
λ ~/ echo 'eval "$(pyenv init -)"' >> ~/.zshrc
```

Check installation:

```
λ ~/ pyenv --version
pyenv 2.3.1
```

And install the latest Python version available at the time of writing, `3.10.5`:

```
λ ~/ pyenv install 3.10.5
```

## Use pyenv with direnv

Now that `pyenv` is installed, let's hook it up with our project and `direnv`.

```
λ ~/ cd project/
λ ~/ echo 'layout pyenv 3.10.5' > .envrc
λ ~/ direnv allow
```

Check our Python version and interpreter:

```
λ ~/project/ python --version && which python
Python 3.10.5
~/project/.direnv/python-3.10.5/bin/python
```

That’s it.

## Poetry

## What is Poetry

Poetry is a tool for dependency management and packaging in Python. It allows you to declare the libraries your project depends on and it will manage (install/update) them for you.

Poetry effectively replaces `pip` and brings several useful advantages, such as:

- one configuration file for all dependencies and their configs
- *can create and manage virtual environments* (true, but we manage with `direnv` instead)
- automatically resolves dependencies of installed plugins

## Poetry installation

The Poetry documentation provides installation guidelines, including a section for **osx/linux/bashonwindows** distributions:

```
λ ~/ curl -sSL https://install.python-poetry.org | python3 -
Retrieving Poetry metadata
# Welcome to Poetry!
This will download and install the latest version of Poetry,
a dependency and package manager for Python.
It will add the `poetry` command to Poetry's bin directory, located at:
$HOME/.local/bin
You can uninstall at any time by executing this script with the --uninstall option,
and these changes will be reverted.
Installing Poetry (1.1.13): Done
Poetry (1.1.13) is installed now. Great!
You can test that everything is set up by executing:
`poetry --version`
```

You can check your installation and `poetry` version:

```
λ ~/ poetry --version 
Poetry version 1.1.13
```

## Poetry shell Configuration

You can enable tab completion for your shell. [This documentation](https://python-poetry.org/docs/#enable-tab-completion-for-bash-fish-or-zsh) has a guide for each shell.

*e.g. with ZSH and Oh-my-zsh*

```
λ ~/ mkdir $ZSH_CUSTOM/plugins/poetry
poetry completions zsh > $ZSH_CUSTOM/plugins/poetry/_poetry
```

For oh-my-zsh, you then need to enable the poetry plugin in your ~/.zshrc:

```
plugins(
	poetry
	...
	)
```

## Poetry usage

According to the `poetry` documentation, you can use it for a new project and/or an existing one.

## New project

```
λ ~/ poetry new poetry-demo
```

This will create the poetry-demo directory with the following content:

```
poetry-demo
├── pyproject.toml
├── README.md
├── poetry_demo
│   └── __init__.py
└── tests
    └── __init__.py
```

## Existing project

Instead of creating a new project, Poetry can be used to 'initialize' a pre-populated directory. To interactively create a `pyproject.toml` file in an existing directory:

```
λ ~/ cd project
λ ~/project/ poetry init
```

## Add dependencies

As simple as:

```
λ ~/project/ poetry add django
Using version ^4.0.5 for Django
Updating dependencies
Resolving dependencies... (0.9s)
Writing lock file
Package operations: 3 installs, 0 updates, 0 removals
  • Installing asgiref (3.5.2)
  • Installing sqlparse (0.4.2)
  • Installing django (4.0.5)
```

It automatically finds a suitable version constraint and installs the package along with its sub-dependencies.

Find the details in your `toml` config file:

```
λ ~/project/ cat pyproject.toml 
[tool.poetry]
name = "project"
version = "0.1.0"
description = ""
authors = ["lebarondecharlus <git@lebaron.sh>"]
[tool.poetry.dependencies]
python = "^3.10"
Django = "^4.0.5"
[tool.poetry.dev-dependencies]
[build-system]
requires = ["poetry-core>=1.0.0"]
build-backend = "poetry.core.masonry.api"
```

## Link Poetry with direnv

From the `poetry` documentation:

> *By default, poetry creates a virtual environment in {cache-dir}/virtualenvs ({cache-dir}\virtualenvs on Windows). You can change the cache-dir value by editing the poetry config. Additionally, you can use the virtualenvs.in-project configuration variable to create virtual environment within your project directory.*

What we want here is to tell `poetry` not to configure its own environment, but to use the `direnv` configuration we already set up.

```
# .envrc
layout pyenv 3.10.5
# POETRY
if [[ ! -f pyproject.toml ]]; then
  log_status 'No pyproject.toml found. Will initialize poetry in no-interactive mode'
  poetry init -n -q
  poetry run pip install -U pip wheel setuptools
fi  
poetry run echo >> /dev/null
local VENV=$(dirname $(poetry run which python))
export VIRTUAL_ENV=$(echo "$VENV" | rev | cut -d'/' -f2- | rev)
export POETRY_ACTIVE=1
PATH_add "$VENV"
if [ ! -L .venv ]; then
  ln -ns $VIRTUAL_ENV .venv
fi
```

Let's allow `direnv` and check that we're using the right environment.

For our Django example, we should be able to find the dependency inside the `.direnv` folder.

```
λ ~/project/ find .direnv -name 'django'  
.direnv/python-3.10.5/lib/python3.10/site-packages/django
.direnv/python-3.10.5/lib/python3.10/site-packages/django/forms/jinja2/django
.direnv/python-3.10.5/lib/python3.10/site-packages/django/forms/templates/django
```

## GPG

Remember the secrets management problem? Now that our virtual environments are set up, how do we work with secrets without writing them directly into our project or repository?

Before we can use a password manager like `pass`, we need a `gpg` key ID.

## Generate a key

Let's generate one first.

```
λ ~/ gpg --full-generate-key
Please select what kind of key you want:
   (1) RSA and RSA (default)
   (2) DSA and Elgamal
   (3) DSA (sign only)
   (4) RSA (sign only)
  (14) Existing key from card
Your selection? 1
RSA keys may be between 1024 and 4096 bits long.
What keysize do you want? (3072) 
Requested keysize is 3072 bits
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0) 
Key does not expire at all
Is this correct? (y/N) y
Real name: Kader Ovski
Email address: me@kaderovski.com
Comment: 
You selected this USER-ID:
    "Kader Ovski <me@kaderovski.com>"
Change (N)ame, (C)omment, (E)mail or (O)kay/(Q)uit? O
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.
gpg: key 0854057891EFB8F0 marked as ultimately trusted
pub   rsa3072 2022-06-10 [SC]
      DC3EC748A8D97169F47C16690854057891EFB8F0
uid                      Kader Ovski <me@kaderovski.com>
sub   rsa3072 2022-06-10 [E]
```

## Get key ID

We can then list our new key:

```
λ ~/ gpg --list-keys
gpg: checking the trustdb
gpg: marginals needed: 3  completes needed: 1  trust model: pgp
gpg: depth: 0  valid:   1  signed:   0  trust: 0-, 0q, 0n, 0m, 0f, 1u
$HOME/.gnupg/pubring.kbx
------------------------
pub   rsa3072 2022-06-10 [SC]
      DC3EC748A8D97169F47C16690854057891EFB8F0
uid           [ultimate] Kader Ovski <me@kaderovski.com>
sub   rsa3072 2022-06-10 [E]
```

## Pass

`pass` is the standard Unix password manager.

> *With pass, each password lives inside of a gpg encrypted file whose filename is the title of the website or resource that requires the password. These encrypted files may be organized into meaningful folder hierarchies, copied from computer to computer, and, in general, manipulated using standard command line file management utilities.*

## Pass installation

`pass` is available in almost all distributions and can be installed like this:

```
λ ~/ sudo apt-get install pass
```

## Pass usage

Now that we have `pass` installed on our system **and** a valid `gpg_key_id`, we can initialize our password manager with:

```
λ ~/ pass init DC3EC748A8D97169F47C16690854057891EFB8F0
mkdir: created directory '$HOME/.password-store/'
Password store initialized for DC3EC748A8D97169F47C16690854057891EFB8F0
```

Let's insert our first password:

```
λ ~/ pass insert kaderovski.com/blog/some_secret
mkdir: created directory '$HOME/.password-store/kaderovski.com'
mkdir: created directory '$HOME/.password-store/kaderovski.com/blog'
Enter password for kaderovski.com/blog/some_secret: 
Retype password for kaderovski.com/blog/some_secret:
```

Our password is now stored in `pass`:

```
λ ~/ pass
Password Store
`-- kaderovski.com
    `-- blog
        `-- some_secret
```

You can retrieve your secret by entering your GPG passphrase:

```
λ ~/ pass show kaderovski.com/blog/some_secret
my_secret
```

## Tomb

Due to how `pass` is structured, file and directory names aren't encrypted in the password store. `pass-tomb` offers a convenient solution: it puts your password store inside a Tomb, keeping your password tree encrypted when you're not using it.

It uses the same GPG key to encrypt both the passwords and the tomb, so you don't need to manage an extra key or secret. You can also configure `pass-tomb` to automatically close the store after a given time.

## Tomb installation

You can install it as a package on your distribution:

```
λ ~/ sudo apt install pass-extension-tomb
```

## Tomb usage

Create a new password tomb:

*e.g.*

```
> pass tomb DC3EC748A8D97169F47C16690854057891EFB8F0
 (*) Your password tomb has been created and opened in ~/.password-store.
 (*) Password store initialized for DC3EC748A8D97169F47C16690854057891EFB8F0
  .  Your tomb is: ~/.password.tomb
  .  Your tomb key is: ~/.password.key.tomb
  .  You can now use pass as usual.
  .  When finished, close the password tomb using 'pass close'.
```

You can now use the best part of `pass-tomb`:

- open your tomb

```
λ ~/ pass open
 (*) Your password tomb has been opened in $HOME/.password-store/.
  .  You can now use pass as usual.
  .  When finished, close the password tomb using 'pass close'.
```

- close your tomb

```
λ ~/ pass close
 (*) Your password tomb has been closed.
  .  Your passwords remain present in $HOME/.password.tomb.
```

Let's check what actually happens with the `open` and `close` commands.

When the `tomb` is closed, the `pass` command and our password path look like this:

```
λ ~/ pass
Password Store
└── 2
λ ~/ tree .password-store/            
.password-store/
└── 2
1 directory, 0 files
```

We can't see our key:value (path/password-name) elements in the output or on the filesystem.

When we open the `tomb`, we decrypt our password database:

```
λ ~/ pass ; tree .password-store/
Password Store
├── kaderovski.com
│   └── blog
│       └── some_secret.gpg
λ ~/ tree .password-store/            
.password-store/
├── kaderovski.com
│   └── blog
│       └── some_secret.gpg
```

## Secrets with direnv and pass

We can now use our secrets in our Python code thanks to `direnv` and `pass`.

We need to slightly modify the `.envrc` in our project directory to check whether the `tomb` is open. I do this by checking if I can `stat` `$HOME/.password-store/.gpg-id`, which only succeeds when the `tomb` is open. If it fails, the `tomb` is closed and needs to be opened.

```
# .envrc
layout pyenv 3.10.5
# POETRY
if [[ ! -f pyproject.toml ]]; then
  log_status 'No pyproject.toml found. Will initialize poetry in no-interactive mode'
  poetry init -n -q
  poetry run pip install -U pip wheel setuptools
fi  
poetry run echo >> /dev/null
local VENV=$(dirname $(poetry run which python))
export VIRTUAL_ENV=$(echo "$VENV" | rev | cut -d'/' -f2- | rev)
export POETRY_ACTIVE=1
PATH_add "$VENV"
if [ ! -L .venv ]; then
  ln -ns $VIRTUAL_ENV .venv
fi
# CHEKING IF TOMB IS OPEN
if ! stat $HOME/.password-store/.gpg-id > /dev/null 2>&1 ; then
    # IF NOT OPEN IT
    pass open
fi
# THEN PUT SECRET IN ENV
export MY_SECRET=$(pass show kaderovski.com/blog/some_secret)
```

We can now reload our `direnv`.

```
λ ~/project/ direnv reload
direnv: loading ~/project/.envrc
 (*) Your password tomb has been opened in /home/$USER/.password-store/.
  .  You can now use pass as usual.
  .  When finished, close the password tomb using 'pass close'.
direnv: export +MY_SECRET +VIRTUAL_ENV ~PATH
```

Notice `+MY_SECRET` in the output, telling us it's now accessible through our environment.

We can try to reach our secret:

```
λ ~/project/ echo $MY_SECRET
my_secret
```

What really matters here is that **we are not storing plaintext secrets** in our code, project, or repo. All our secrets are reachable through our virtual environment and can be used in your Python code like this:

```
>>> import os
>>> secret=os.environ.get('MY_SECRET')
>>> print(secret)
my_secret
```

## Direnv advanced configurations

In more complex projects, you may need commands, external dependencies, third-party binaries, or third-party configuration. Let's see how we can manage these with `direnv`.

## Check for commands dependencies

In your `.envrc` file, you can make sure the commands your project needs are actually available.

You can do it like this:

```
# .envrc
layout pyenv 3.10.5
# POETRY
if [[ ! -f pyproject.toml ]]; then
  log_status 'No pyproject.toml found. Will initialize poetry in no-interactive mode'
  poetry init -n -q
  poetry run pip install -U pip wheel setuptools
fi
poetry run echo >> /dev/null
local VENV=$(dirname $(poetry run which python))
export VIRTUAL_ENV=$(echo "$VENV" | rev | cut -d'/' -f2- | rev)
export POETRY_ACTIVE=1
PATH_add "$VENV"
if [ ! -L .venv ]; then
  ln -ns $VIRTUAL_ENV .venv
fi
# CHEKING IF TOMB IS OPEN
if ! stat $HOME/.password-store/.gpg-id > /dev/null 2>&1 ; then
    # IF NOT OPEN IT
    pass open
fi
# THEN PUT SECRET IN ENV
export MY_SECRET=$(pass show kaderovski.com/blog/some_secret)
# CHECKING COMMANDS DEPENDENCIES
DIRENV_CMD_DEPENDENCIES="unzip tar mkdir curl chmod rm"
for mandatory_cmd in ${DIRENV_CMD_DEPENDENCIES}; do
   if [ -z "$(which ${mandatory_cmd})" ]; then
        echo "===> Mandatory command not found: ${mandatory_cmd}"
        exit 1
   fi
done
```

## Third-party binaries

We can also set things up to use specific third-party binaries, such as `packer`, `terraform`, `vault`, and so on.

We'll tell `direnv` to extend `PATH` so our third-party binaries live under the `.direnv/bin` path.

Let’s grab our `.envrc` file again:

```
# .envrc
layout pyenv 3.10.5
# POETRY
if [[ ! -f pyproject.toml ]]; then
  log_status 'No pyproject.toml found. Will initialize poetry in no-interactive mode'
  poetry init -n -q
  poetry run pip install -U pip wheel setuptools
fi
poetry run echo >> /dev/null
local VENV=$(dirname $(poetry run which python))
export VIRTUAL_ENV=$(echo "$VENV" | rev | cut -d'/' -f2- | rev)
export POETRY_ACTIVE=1
PATH_add "$VENV"
if [ ! -L .venv ]; then
  ln -ns $VIRTUAL_ENV .venv
fi
# CHEKING IF TOMB IS OPEN
if ! stat $HOME/.password-store/.gpg-id > /dev/null 2>&1 ; then
    # IF NOT OPEN IT
    pass open
fi
# THEN PUT SECRET IN ENV
export MY_SECRET=$(pass show kaderovski.com/blog/some_secret)
# CHECKING COMMANDS DEPENDENCIES
DIRENV_CMD_DEPENDENCIES="unzip tar mkdir curl chmod rm"
for mandatory_cmd in ${DIRENV_CMD_DEPENDENCIES}; do
   if [ -z "$(which ${mandatory_cmd})" ]; then
        echo "===> Mandatory command not found: ${mandatory_cmd}"
        exit 1
   fi
done
export DIRENV_TMP_DIR="${PWD}/.direnv"
export DIRENV_BIN_DIR="${DIRENV_TMP_DIR}/bin"
if [ ! -e "${DIRENV_BIN_DIR}" ]; then
   mkdir -p "${DIRENV_BIN_DIR}"
fi
export PATH="${DIRENV_BIN_DIR}:${PATH}"
```

Now let's install a binary, we'll try it with HashiCorp's `packer`:

```
# .envrc
layout pyenv 3.10.5
# POETRY
if [[ ! -f pyproject.toml ]]; then
  log_status 'No pyproject.toml found. Will initialize poetry in no-interactive mode'
  poetry init -n -q
  poetry run pip install -U pip wheel setuptools
fi
poetry run echo >> /dev/null
local VENV=$(dirname $(poetry run which python))
export VIRTUAL_ENV=$(echo "$VENV" | rev | cut -d'/' -f2- | rev)
export POETRY_ACTIVE=1
PATH_add "$VENV"
if [ ! -L .venv ]; then
  ln -ns $VIRTUAL_ENV .venv
fi
# CHEKING IF TOMB IS OPEN
if ! stat $HOME/.password-store/.gpg-id > /dev/null 2>&1 ; then
    # IF NOT OPEN IT
    pass open
fi
# THEN PUT SECRET IN ENV
export MY_SECRET=$(pass show kaderovski.com/blog/some_secret)
# CHECKING COMMANDS DEPENDENCIES
DIRENV_CMD_DEPENDENCIES="unzip tar mkdir curl chmod rm"
for mandatory_cmd in ${DIRENV_CMD_DEPENDENCIES}; do
   if [ -z "$(which ${mandatory_cmd})" ]; then
        echo "===> Mandatory command not found: ${mandatory_cmd}"
        exit 1
   fi
done
export DIRENV_TMP_DIR="${PWD}/.direnv"
export DIRENV_BIN_DIR="${DIRENV_TMP_DIR}/bin"
if [ ! -e "${DIRENV_BIN_DIR}" ]; then
   mkdir -p "${DIRENV_BIN_DIR}"
fi
export PATH="${DIRENV_BIN_DIR}:${PATH}"
# PACKER INSTALLATION
PACKER_VERSION="1.8.1"
PACKER_ARCH="linux_amd64"
PACKER_PKG_NAME="packer_${PACKER_VERSION}_${PACKER_ARCH}.zip"
PACKER_PKG_URL="https://releases.hashicorp.com/packer/${PACKER_VERSION}/${PACKER_PKG_NAME}"
PACKER_PKG_PATH="${DIRENV_TMP_DIR}/${PACKER_PKG_NAME}"
if [ ! -e "${DIRENV_BIN_DIR}/packer" ]; then
   echo "===> Getting packer:${PACKER_VERSION}:${PACKER_ARCH} (can take a while to execute)"
   curl -s -L "${PACKER_PKG_URL}" -o "${PACKER_PKG_PATH}"
   unzip ${PACKER_PKG_PATH} -d ${DIRENV_BIN_DIR}
   chmod 700 ${DIRENV_BIN_DIR}/packer
   rm -f ${PACKER_PKG_PATH}
fi
```

Let's try out our new configuration:

```
λ ~/project direnv allow
direnv: loading ~/project/.envrc                                                                    
===> Getting packer:1.8.1:linux_amd64 (can take a while to execute)
Archive:  ~/project/.direnv/packer_1.8.1_linux_amd64.zip
  inflating: ~/project/.direnv/bin/packer  
direnv: export +MY_SECRET +VIRTUAL_ENV ~PATH
```

Now that `packer` is installed in our `PATH`, let's locate it and run it:

```
λ ~/project/ which packer
~/project/.direnv/bin/packer
λ ~/ packer --version
1.8.1
```

If you need to change the `packer` version in the future, just remove the current `packer` binary and update the `PACKER_VERSION` variable to rebuild your `direnv` setup.

## Third-party configs

Of course, if our third-party tools need custom configuration, we can specify it directly in our `.envrc`. But to keep things clear and organized, we can also split our config into separate subfiles.

By adding this block at the end of our `.envrc` file:

```
# .envrc 
# […]
# ADDONS
ENV_ADDONS=".env.packer .env.custom_config"
for addon in ${ENV_ADDONS}; do
   if [ -e "${PWD}/${addon}" ]; then
       source ${PWD}/${addon}
   fi
done
```

And then create your custom addon config files:

```
# .env.packer
export PACKER_VAR1=VAR1_VALUE
export PACKER_VAR2=VAR2_VALUE
# .env.custom_config
export CUSTOM_VAR1=CUSTOM1_VALUE
export CUSTOM_VAR2=CUSTOM2_VALUE
```

Don’t forget to reload:

```
λ ~/project/ direnv reload
```

## Final template

Here is a final `.envrc` template you can grab and adapt to your needs:

```
# .envrc
layout pyenv 3.10.5
# POETRY
if [[ ! -f pyproject.toml ]]; then
  log_status 'No pyproject.toml found. Will initialize poetry in no-interactive mode'
  poetry init -n -q
  poetry run pip install -U pip wheel setuptools
fi
poetry run echo >> /dev/null
local VENV=$(dirname $(poetry run which python))
export VIRTUAL_ENV=$(echo "$VENV" | rev | cut -d'/' -f2- | rev)
export POETRY_ACTIVE=1
PATH_add "$VENV"
if [ ! -L .venv ]; then
  ln -ns $VIRTUAL_ENV .venv
fi
# CHEKING IF TOMB IS OPEN
if ! stat $HOME/.password-store/.gpg-id > /dev/null 2>&1 ; then
    # IF NOT OPEN IT
    pass open
fi
# THEN PUT SECRET IN ENV
export MY_SECRET=$(pass show kaderovski.com/blog/some_secret)
# CHECKING COMMANDS DEPENDENCIES
DIRENV_CMD_DEPENDENCIES="unzip tar mkdir curl chmod rm"
for mandatory_cmd in ${DIRENV_CMD_DEPENDENCIES}; do
   if [ -z "$(which ${mandatory_cmd})" ]; then
        echo "===> Mandatory command not found: ${mandatory_cmd}"
        exit 1
   fi
done
export DIRENV_TMP_DIR="${PWD}/.direnv"
export DIRENV_BIN_DIR="${DIRENV_TMP_DIR}/bin"
if [ ! -e "${DIRENV_BIN_DIR}" ]; then
   mkdir -p "${DIRENV_BIN_DIR}"
fi
export PATH="${DIRENV_BIN_DIR}:${PATH}"
# PACKER INSTALLATION
PACKER_VERSION="1.8.1"
PACKER_ARCH="linux_amd64"
PACKER_PKG_NAME="packer_${PACKER_VERSION}_${PACKER_ARCH}.zip"
PACKER_PKG_URL="https://releases.hashicorp.com/packer/${PACKER_VERSION}/${PACKER_PKG_NAME}"
PACKER_PKG_PATH="${DIRENV_TMP_DIR}/${PACKER_PKG_NAME}"
if [ ! -e "${DIRENV_BIN_DIR}/packer" ]; then
   echo "===> Getting packer:${PACKER_VERSION}:${PACKER_ARCH} (can take a while to execute)"
   curl -s -L "${PACKER_PKG_URL}" -o "${PACKER_PKG_PATH}"
   unzip ${PACKER_PKG_PATH} -d ${DIRENV_BIN_DIR}
   chmod 700 ${DIRENV_BIN_DIR}/packer
   rm -f ${PACKER_PKG_PATH}
fi
# ADDONS
ENV_ADDONS=".env.packer .env.custom_config"
for addon in ${ENV_ADDONS}; do
   if [ -e "${PWD}/${addon}" ]; then
       source ${PWD}/${addon}
   fi
done
```

## Conclusion

Yes, this requires a bit of knowledge and some configuration to get comfortable with this workflow, but keep in mind that everything ends up automated and much easier to version control.

For production, you can arrange to share your GPG key, or manage multiple GPG keys within the same `pass` store (*e.g. one GPG ID per team member*), then version your `pass` store and make it accessible through git.

All dependencies, with the exact same package versions, will be deployed exactly as in your development environment.

Keep it simple.

A starter template is available [here](https://gist.github.com/lebarondecharlus/f91f4c29a5f655920d4c65a62eb275b0).

