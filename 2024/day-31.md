**Author:** Vũ

---

# Commit your code in style with git.emoji 🎉

Have you ever get bored with the same old commit messages, all starting with “update” or “fix” or “feat”? Do you know that emojis are the latest advancement in human communication? They can convey emotions and ideas in a single character and make your messages alive! Think about it, even when aliens come to Earth, they will quickly understand the meaning of these little characters! 😎 New year, new you! Let’s leave boring commits in the past and embrace the magic of emojis to make every commit a celebration!

![git.emoji favicon](https://olivernguyen.io/images/favicon.png)

https://olivernguyen.io/w/git.emoji/

![git.emoji preview](https://iolivernguyen.github.io/images/ogx.png)

---

Have you ever get bored with the same old commit messages, all starting with “update” or “fix” or “feat”? Do you know that emojis are the latest advancement in human communication? They can convey emotions and ideas in a single character and make your messages alive! Think about it, even when aliens come to Earth, they will quickly understand the meaning of these little characters! 😎

New year, new you! Let’s leave boring commits in the past and embrace the magic of emojis to make every commit a celebration! 🥳✨

### Introduction

[git.emoji](https://github.com/connectlyai/git.emoji) is a tool written in Go that allows you to use emojis in your commit messages. It provides a simple command-line interface to help you select the right emoji for your commit. Whether you are fixing a bug, adding a new feature, or updating documentation, you can now express yourself with emojis!

### Installation

To install `git.emoji`, you need to have Go installed on your machine. You can download the latest version of Go from the [official website](https://golang.org/dl/). Once you have Go installed, you can install `git.emoji` using the following command:

```bash
go install github.com/connectlyai/git.emoji@latest
```

Now you should have `git.emoji` installed on your machine, usually at `~/go/bin/git.emoji`. Make sure to add this directory to your `PATH` so you can use `git.emoji` from anywhere.

Let’s check if it’s installed correctly:

```bash
git.emoji --version
```

And try running `git.emoji`:

```bash
git.emoji
```

### Setup

To start using `git.emoji`, you need to set up the git hooks in your repository. You can do this by running the following command in your repository:

```bash
git.emoji setup-hooks
```

This command will set up the necessary git hooks to use `git.emoji` for your commits. Now you can start committing with emojis!

### Usage

#### Your first commit with emoji

Let’s start with a simple commit:

```bash
git commit
```

This command will open a prompt for you to select an emoji for your commit message. You can choose from a list of emojis and add a message to describe your commit. Once you have selected an emoji and added a message, the commit will be made with the emoji and message.

For example, if you want to commit a new feature, you can use input the number `1` or the abbreviation `ft` to select the **Features** category and the `💻` emoji. Each category has a list of emojis that you can choose from to describe your commit. To use the next emoji instead, you can enter `1a` or `ft1` to select the `✨` emoji, and so on.

#### Commit with emoji from the command line

In addition to using the interactive prompt, you can also commit with an emoji directly from the command line. You can use the `-<abbr>` flag to select the emoji for your commit. For example, to commit a new feature, you can use the following command:

```bash
git.emoji commit -ft -m "add awesome new feature"
```

This way, you can quickly commit your changes with emojis without going through the interactive prompt.

#### Use `git.emoji` as a wrapper for `git`

If you don’t like to type `git.emoji` every time you want to commit, you can alias `git.emoji` as a wrapper for `git`. Add the following line to your shell configuration file (e.g., `.bashrc`, `.zshrc`, `.bash_profile`):

```bash
alias git="git.emoji"
```

Now you can use `git` as you normally would, and it will automatically use `git.emoji` for your commits.

#### It works with rebase, merge, and other git commands

Not just committing, whenever you use `git` commands that involve committing, such as `rebase`, `merge`, or `cherry-pick`, if there is a commit message without emoji, `git.emoji` will prompt you to select an emoji for the commit message. This way, you can ensure that all your commits are consistent and expressive.

#### Customize your emojis

You don’t have to use the default emojis provided by `git.emoji`. You can always use your favourite emojis, either by input the emoji directly while commiting your changes or customize the emojis in your local [`emoji.config`](https://github.com/connectlyai/git.emoji/blob/main/emoji.config) file.

Currently, it supports two locations for the `emoji.config` file:

- `<YOUR_REPOSITORY>/emoji.config`
- `<YOUR_REPOSITORY>/.git/emoji.config`

With the first way, you can customize and share the emojis with your team in the repository. With the second way, you can customize the emojis for your local repository only.

To start customizing your emojis, you can run this command:

```bash
git.emoji init-config
```

This will ask you where you want to save the `emoji.config` file:

```text
Where do you want to save emoji.config?
1) <YOUR_REPOSITORY>/emoji.config
2) <YOUR_REPOSITORY>/.git/emoji.config
```

The [config file](https://github.com/connectlyai/git.emoji/blob/main/emoji.config) will look like this:

```ini
[Features]
abbr = ft
emojis = 💻,✨,🧩

[Fixes]
abbr = fx
emojis = 🐛,🚑,🔧
```

Just edit the file, add your favourite emojis, create any categories you like, and save the file. Now you can use these emojis in your commits!

### How it works

`git.emoji` works by setting up git hooks in your repository to intercept the commit messages. When you make a commit, the git hook will call `git.emoji` to prompt you to select an emoji for your commit message. Once you have selected an emoji, `git.emoji` will add the emoji to your commit message and make the commit.

#### Under the hooks

The git hooks are set up in the `.git/hooks` directory of your repository. If you look in this directory, you will see a bunch of hook files:

```bash
ls .git/hooks
```

Whenever git runs a command that triggers a hook, it will execute the corresponding hook script. For example, when you make a commit, git will run the `commit-msg` hook. This is where `git.emoji` comes in. It provides a script that you can use as the `commit-msg` hook to prompt you to select an emoji for your commit message.

By default, they are all empty sample shell scripts. After running:

```bash
git.emoji setup-hooks
```

the `commit-msg` hook in your repository will be updated to include the `git.emoji` script. This script will call `git.emoji` to prompt you to select an emoji for your commit message:

```sh
#!/bin/sh
git.emoji hook-commit-msg "$@"
```

So you get the idea. Whenever you make a commit, the `commit-msg` hook will call `git.emoji`, which will prompt you to select an emoji for your commit message. Once you have selected an emoji, `git.emoji` will add the emoji to your commit message by editing the file `.git/COMMIT_EDITMSG`.

Another related hook is `prepare-commit-msg`, which is called before the commit message editor is opened. This is where `git.emoji` can automatically add an emoji to your commit message when you are doing a rebase, merge, or cherry-pick.

### Conclusion

[git.emoji](https://github.com/connectlyai/git.emoji) brings a fresh and expressive way to commit your code by adding emojis to your messages. It enhances communication within your team by making commit intent clear and fun, whether you’re fixing bugs, adding features, or releasing updates. With its seamless integration into Git hooks and a customizable emoji configuration, it’s a practical tool to elevate your workflow.

So why stick to boring commit messages when you can make them vibrant and meaningful? Start your New Year and upgrade your commits with emojis! 🎉