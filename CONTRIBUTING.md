# Contributing to the COTI Documentation

Contributions are welcome. When sending a PR, please ensure:

- Your PR has no conflicts with `main`. If you have conflicts, please rebase.
- You are using signed commits.
- Commits in your PR are squashed (to help us keep a clean history).

## Standards

- Commit messages
  - Start with a present tense verb (if applied, this commit will...)
  - Keep your commit headline short but sufficient to understand the scope and focus of the change.
- Make sure images are optimized and compressed
- Keep images < 20kb whenever possible to keep the site fast and the repo small (try [trimage](https://trimage.org/))

## How to rebase a PR

```shell
git fetch
git rebase origin/main
# resolve conflicts
git push -f
```

It's okay to force push over your own PR branch. In fact, that's what we want so that the commits are clean.

## How to squash a PR

To squash multiple commits, use the following steps:

### 1. Rebase multiple commits into one

When squashing commits, you should always rebase from the main branch of the repo: `git rebase -i origin/main`.

After rebasing, you can squash the commits by changing `pick` into `squash` for all of your commits except the recent one:

```shell
pick 686e386 add doc structure
squash ee2eed7 add main content
squash 3eccd39 fix typos

# Rebase 287bc79..3eccd39 onto 287bc79 (3 commands)
# …
```

### 2. Fix your commit message

Next, comment or remove all of your commit messages and leave the one that describes your PR.

### 3. Force push to your branch

Lastly, force push the changes to your branch to have a clean history:

```shell
git push -f
```

It's easier to use `git --amend` while working on your PR first. Avoiding multiple commits is much cleaner and easier than fixing them.

## Style Guide

- **Bold** key terms or any terms the user must absolutely know when reading a doc.
- _Italicize_ terms for emphasis but only when necessary.
- Use markdown over HTML wherever possible.
- When writing an additional note, start it with **Note:**.
- Use a consistent voice. When writing documentation, we attempt to avoid first person plural "we" and opt for second person singular "you". Avoid passive voice.
