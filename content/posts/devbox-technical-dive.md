---
title: "Streamlining Development Workflows with Devbox: A Technical Dive"
date: 2025-02-04
tags: ["devops"]
image: "/images/devbox-technical-dive/01-cover.png"
draft: false
---

Achieving efficiency and maintaining consistency across development environments are two of the most persistent challenges developers face. The search for tools that streamline the development process while keeping work environments reproducible and manageable led me to Devbox: a command-line utility that's changing the way developers create and manage isolated environments, a bit like a Swiss Army knife for the modern developer.

## Introducing Devbox

Devbox operates by allowing developers to spin up isolated shells, customized for specific project requirements. This is achieved through a configuration file, devbox.json, which specifies the necessary packages for a project. Devbox then ensures these tools are installed in the environment, similar to package managers like yarn but at the operating system level, managing packages typically installed via brew or apt-get.

### The Advantages of Adopting Devbox

Devbox addresses several common development hurdles, with benefits including:

- **Ensuring Consistent Development Environments**

By utilizing a devbox.json file to declare required tools and executing `devbox shell`, Devbox guarantees identical shell environments for all team members. This approach eradicates the infamous “it works on my machine” problem by standardizing tool versions across the board.

- **Isolation Without Compromising Performance**

A key feature of Devbox is its ability to create isolated environments on your laptop without the overhead of additional virtualization layers. This isolation is achieved without sacrificing performance, addressing a common drawback of other solutions like Docker containers.

- **Clean Experimentation**

The isolation provided by Devbox means that developers can experiment with new tools or versions without affecting their main working environment. Tools can be easily added or removed by updating the `devbox.json` file, ensuring a clutter-free workspace.

- **Resolving Version Conflicts with Ease**

Devbox simplifies working on multiple projects with conflicting dependencies. Separate environments can be created for each project, ensuring that compatibility and stability are maintained without the need to juggle different tool versions.

- **Enhanced Portability and Flexibility**

Devbox not only standardizes development environments across teams but also ensures portability. Whether transitioning from local development to a devcontainer in VSCode, generating a Dockerfile for production, or setting up a cloud-based remote development environment, Devbox maintains consistency and reliability.

Let’s go ?

## Getting Started with Devbox

Devbox leverages the Nix package manager to manage dependencies. If Nix is not present on your system, Devbox will facilitate its installation.

### Installation Process

To install Devbox on a Linux system (if you are using another system, refer to [this documentation](https://www.jetify.com/docs/devbox/installing-devbox), the rest of this article will work exactly the same), execute the following command:

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

You can easily search for packages:

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

Simply as that:

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

Inside our shell we can now verify if our packages are present:

```
$ go version && node --version

go version go1.23.4 linux/amd64
v23.6.1
```

An interesting thing is to look where the binaries are located to understand how nix is running behind devbox.

```
$ which go
/tmp/t/.devbox/nix/profile/default/bin/go
```

## Scripting

Devbox offers a powerful feature that significantly enhances automation: scripting.

Devbox scripting allows developers to define and execute shell commands within their `devbox.json` configuration file. This feature is not just about running individual commands; it’s about orchestrating a series of actions that kick off as soon as your development environment is ready. Whether it’s initializing settings, running servers, or executing custom build steps, Devbox scripts are the backbone of a seamless development setup.

### **Configuration**

To leverage Devbox scripting, you’ll need to add your scripts to the `devbox.json` file. Each script requires a unique name and either a single command or an array of commands to execute.

Let's look at our previous example and notice two things.

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
- `scripts`: your automations!

### Executing Scripts

Running a script is as simple as invoking `devbox run <script_name>`. This command fires up your Devbox shell, executes the `init_hook`, and then runs the specified script.

For instance:

```
$ devbox run test

Error: no test specified
Error: error running script "test" in Devbox: exit status 1
```

See? This is exactly our devbox.json script:

```json
"scripts": {
      "test": [
        "echo \"Error: no test specified\" && exit 1"
      ]
    }
```

### Running Ad-hoc Commands

Devbox also supports executing one-off commands within the shell, even if they’re not predefined in your scripts. This flexibility allows you to run any shell command on the fly, including those with flags:

```
$ devbox run echo "Hello Medium !"
```

### Custom Environment Variables

Enhancing scripts with custom environment variables is straightforward with the `--env` flag. This feature enables scripts to run with specific settings tailored to the task at hand:

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

Devbox represents a significant leap forward for developers seeking to enhance their productivity and ensure consistency across projects, by making it trivial to create isolated, reproducible development environments.

Its integration with Nix package management offers precise control over dependencies, making it an indispensable tool for modern development teams.
