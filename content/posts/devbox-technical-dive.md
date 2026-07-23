---
title: "Streamlining Development Workflows with Devbox: A Technical Dive"
date: 2025-02-04
tags: ["devops"]
image: "/images/devbox-technical-dive/01-cover.png"
draft: false
---

Achieving efficiency and maintaining consistency across development environments are two of the most persistent challenges developers face. My search for tools that streamline the development process while keeping work environments reproducible and manageable led me to Devbox: a command-line utility that changes the way developers create and manage isolated environments, a bit like a Swiss Army knife for the modern developer.

## Introducing Devbox

Devbox lets developers spin up isolated shells customized for specific project requirements. It works through a configuration file, devbox.json, which lists the packages a project needs. Devbox then installs those tools into the environment, similar to how a package manager like yarn works, but at the operating system level, handling packages you'd normally install via brew or apt-get.

### The Advantages of Adopting Devbox

Devbox addresses several common development hurdles, with benefits including:

- **Ensuring Consistent Development Environments**

By declaring required tools in a devbox.json file and running `devbox shell`, Devbox guarantees identical shell environments for every team member. This approach kills the infamous "it works on my machine" problem by standardizing tool versions across the board.

- **Isolation Without Compromising Performance**

One of Devbox's key strengths is creating isolated environments on your laptop without the overhead of extra virtualization layers, and without sacrificing performance, a common drawback of solutions like Docker containers.

- **Clean Experimentation**

Because environments are isolated, you can experiment with new tools or versions without touching your main working setup. Tools can be added or removed just by updating the `devbox.json` file, keeping your workspace clutter-free.

- **Resolving Version Conflicts with Ease**

Devbox makes it easy to work on multiple projects with conflicting dependencies. You create a separate environment for each project, keeping compatibility and stability intact without having to juggle different tool versions by hand.

- **Enhanced Portability and Flexibility**

Devbox doesn't just standardize development environments across teams, it also makes them portable. Whether you're moving from local development to a devcontainer in VSCode, generating a Dockerfile for production, or setting up a cloud-based remote environment, Devbox keeps things consistent throughout.

Let's go!

![Let's go!](/images/devbox-technical-dive/02-lets-go.gif)

## Getting Started with Devbox

Devbox leverages the Nix package manager to manage dependencies. If Nix is not present on your system, Devbox will facilitate its installation.

![Wait, what's Nix?](/images/devbox-technical-dive/03-nix.gif)

### Installation Process

To install Devbox on a Linux system, run the command below (if you're on another system, see [this documentation](https://www.jetify.com/docs/devbox/installing-devbox) instead; the rest of the article applies exactly the same):

```
$ curl -fsSL https://get.jetify.com/devbox | bash
```

Verify the installation with:

```
$ devbox version
```

## Creating and Configuring a New Environment

After installing Devbox, navigate to your project directory and run `devbox init` to generate a `devbox.json` file. This file can be manually edited to define required packages or updated through the Devbox CLI as packages are added.

```
$ devbox init
```

## Package Installation

For instance, to install Node.js and Go:

```
$ devbox add nodejs go
```

Devbox utilizes Nix packages for installation, ensuring precise control over package versions and dependencies.

Check the output:

```
$ devbox add go
Info: Adding package "go@latest" to devbox.json
Info: Installing the following packages to the nix store: go@latest
```

### **Search for Packages and Versions**

You can search for available packages just as easily:

```
$ devbox search nodejs
```

Specific package versions can be specified as follows:

```
$ devbox add nodejs@22.12.0 go
```

This command updates the `devbox.json` file to reflect the specified versions:

```json
{
 "packages": {
 "nodejs": "22.12.0",
 "go": "latest"
 }
}
```

### **Remove a package**

Just as simple:

```
$ devbox rm go
```

## Activating the Environment

With the environment configured, use `devbox shell` to access the isolated shell, where installed tools are available for use.

```
$ devbox shell

Info: Ensuring packages are installed.
✓ Computed the Devbox environment.
Starting a devbox shell...
```

Inside the shell, we can now verify that our packages are present:

```
$ go version && node --version

go version go1.23.4 linux/amd64
v23.6.1
```

It's worth checking where the binaries actually live, to get a sense of how Nix operates behind the scenes in Devbox.

```
$ which go
/tmp/t/.devbox/nix/profile/default/bin/go
```

## Scripting

One of Devbox's more powerful features is scripting, which takes automation a step further.

Devbox scripting lets developers define and run shell commands directly from their `devbox.json` configuration file. It's not just about running individual commands; it's about orchestrating a series of actions that kick off as soon as your development environment is ready. Whether that means initializing settings, starting servers, or running custom build steps, Devbox scripts tie all of that together.

### **Configuration**

To use Devbox scripting, add your scripts to the `devbox.json` file. Each script needs a unique name and either a single command or an array of commands to execute.

Let's revisit our previous example and look at two things in particular.

```json
{
  "$schema": "https://raw.githubusercontent.com/jetify-com/devbox/0.13.7/.schema/devbox.schema.json",
  "packages": [
    "nodejs@latest",
    "go@latest"
  ],
  "shell": {
    "init_hook": [
      "echo 'Welcome to devbox!' > /dev/null"
    ],
    "scripts": {
      "test": [
        "echo \"Error: no test specified\" && exit 1"
      ]
    }
  }
}
```

- `init_hook`: a special script that runs every time the Devbox shell starts.
- `scripts`: where your custom automations live.

### Executing Scripts

Running a script is as simple as invoking `devbox run <script_name>`. This fires up your Devbox shell, executes the `init_hook`, and then runs the specified script.

For instance:

```
$ devbox run test

Error: no test specified
Error: error running script "test" in Devbox: exit status 1
```

That matches exactly the script we defined in devbox.json:

```json
"scripts": {
      "test": [
        "echo \"Error: no test specified\" && exit 1"
      ]
    }
```

### Running Ad-hoc Commands

Devbox also supports running one-off commands within the shell, even if they're not predefined in your scripts. This lets you run any shell command on the fly, flags included:

```
$ devbox run echo "Hello Medium !"
```

### Custom Environment Variables

You can enhance scripts with custom environment variables using the `--env` flag, letting them run with settings tailored to the task at hand:

```
$ devbox run --env FOO=bar echo $FOO
```

For loading environment variables from a file, the `--env-file` flag comes in handy:

```
$ devbox run --env-file .env.devbox echo $FOO
```

## Sharing and Collaboration

Devbox environments can be easily shared by committing the `devbox.json` file to your project’s repository, allowing team members to replicate the environment with a single command.

## Conclusion

Devbox makes it trivial to spin up isolated, reproducible development environments, which is a genuine win for anyone trying to boost productivity and keep consistency across projects.

Its tight integration with Nix gives you precise control over dependencies, which alone makes it worth a spot in any modern development team's toolkit.
