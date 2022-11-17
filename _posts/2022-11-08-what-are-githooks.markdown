---
layout: post
title:  "What are Git hooks?"
date:   2022-11-08 20:07:52 +0200
categories: tech
permalink: /Git hooks/
---

**tl;dr**
In This post, I introduce the concept of Git hooks and use it to check that the commit message has certain style convention and it contains a ticket number. That's one of many examples of how Git hooks can improve the development workflow and save time and effort.

## Introduction

Imagine you have a team of great software engineers with different level of expertise. 
They are having issues working together as each person has her/his prefered way of writing and comitting code.

Alice joined the team recently and when she was browsing through some code, she found a line of code that she thought was useless.

She checked the commit message for that line and found the following message:

> Fixed the bug

There was no ticket number, no further info, no explanation or any references. The author of that line is not in the company anymore to follow up with.

She removed that line and ran all local tests successfully. 
She thought "the bug" was obviously fixed in some other way.

In the next release, a feature developed by another team was broken because they were depending on a util that was generated from the code removed by Alice before. Nobody would have known that. "The bug" mentioned in the commit was documented around that time in the system but who would be able to go search for a ticket that was "closed" any time around the commit date. And to admit it, we all know, we usually don't close our tickets on the same day we commit something and some tickets can be open for a long time. I personally had a ticket open for a couple of months and worked with a ticketing system with thousands of tickets created every single day! 

How can we solve this?

**Add a ticket number to each commit message!**

End of story.

(I know that not everyone writes useful tickets, but that problem needs a different post. Instead, let's focus on __enforcing__ a ticket number in every commit message.)

Now Alice, having had that bitter experience, documents every single bug she has on a ticket and makes sure every commit has a ticket number to make it easier to understand the reasoning behind it.

Her new colleague Bob has just joined the team and keeps forgetting the ticket number on the commit message.
Alice keeps bothering him with that.
He thinks she's not focusing on the essential work he is doing in his pull requests. She thinks he is not following "the rule" of how the team works. As you can imagine, the two are __not__ doing well together and the result is a frustrated team.

![A frustrated team](https://www.gomodus.com/hubfs/Modus-Engagement-business-people-1.png "a frustrated team")
*Image source: https://www.gomodus.com/blog/b2b-sales-leads*

It's time to rescue the team spirit and introduce Git hooks!

![Git Hooks](https://user-images.githubusercontent.com/773481/74965961-3f427880-5427-11ea-92b3-1a74c7e15db1.png "GitHooks"){: width="150" }

*Image source: https://github.com/butschster/LaravelGitHooks*

## What are Git hooks?

[Git hooks](https://githooks.com/) are scripts that Git executes before or after events such as commit, push, and receive. They are available in the standard Git installation, so you can use them right away.

Git hooks are a great way to improve developers' time and collaboration by automatically checking things like grammer, style and length of commit messages and it's a way to automatically check certain things were successful such as test suits and coverage checks or automatically pushing code to stage or production.

As mentioned before, Git hooks are available in any repository that was initialized with `git init` (so every repository).

The Git hooks are by default located in the `.git/hooks` subfolder, which comes with a list of samples written as shell scripts. They are very simple to follow and use.

![The default Git hooks](/assets/images/githooks-path-samples.png "Git hooks path")
*The default Git hooks available in any repository*

### Types of hooks

<!--- do we need this part?-->
There are two groups of Git hooks: *client-side* and *server-side*. 

*Client-side* hooks are triggered by operations such as committing and merging, while *server-side* hooks run on network operations such as receiving pushed commits. 
You can use these hooks for all sorts of use cases. [1](https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks) Examples include:

<!--- These scripts run before and after pushes to the server.-->

* pre-commit: Check the commit message for spelling errors.
* pre-receive: Enforce project coding standards.
* post-commit: Email/SMS team members of a new commit.
* post-receive: Push the code to production.

<!---todo add figure-->

Let's take a deeper look into the *client-side* hooks as in the figure below. Once some code is staged, a `pre-commit` hook is executed. This is mostly useful to check some tests are running successfully.
If the check was not successful, no commit will be possible before fixing that.
Afterwards, a `prepare-commit-msg` hook is run with the three parameters. This hook is mostly useful if you have automatically generated commits.
The next one is the `commit-msg` hook, which is the most common used one. This one can check the commit message written by a developer.
Once all those hooks are successful, a commit is possible.
Finally, a `post-commit-hook` is executed, which can be useful to send notifications.

@@ this figure needs to be described more or moved before the list of use cases@@

![Git commit hooks path](/assets/images/commit-hooks-final.drawio.png "Commit hooks")


## Get started with Git hooks

For simplicity, I'll just present two examples that I've worked with closely:

1. Modify `commit-msg` hook to enforce a check for a ticket number in the commit message.
2. Use existing tools to apply conventional commits in a Node.js project

### Example 1: A simple commit message check

Follow the next steps to activate your first Git hook:

1. Duplicate the `commit-msg.sample` and rename it to `commit-msg` (with no extension at all).
1. Give it execution permission using `chmod +x .git/hooks/commit-msg`
1. Modify the file with the code you need (see examples below)
1. Save your changes and you are ready to go! It will be picked up automatically by git next time you commit something.

Here is the `commit-msg.sample` code before modifications. It checks the duplicate signed-off-by (SOB) author lines in the commit message:

```bash
#!/bin/sh
#
# An example hook script to check the commit log message.
# Called by "git commit" with one argument, the name of the file
# that has the commit message.  The hook should exit with non-zero
# status after issuing an appropriate message if it wants to stop the
# commit.  The hook is allowed to edit the commit message file.
#
# To enable this hook, rename this file to "commit-msg".

# Uncomment the below to add a Signed-off-by line to the message.
# Doing this in a hook is a bad idea in general, but the prepare-commit-msg
# hook is more suited to it.
#
# SOB=$(git var GIT_AUTHOR_IDENT | sed -n 's/^\(.*>\).*$/Signed-off-by: \1/p')
# grep -qs "^$SOB" "$1" || echo "$SOB" >> "$1"

# This example catches duplicate Signed-off-by lines.

test "" = "$(grep '^Signed-off-by: ' "$1" |
	 sort | uniq -c | sed -e '/^[ 	]*1[ 	]/d')" || {
	echo >&2 Duplicate Signed-off-by lines.
	exit 1
}
```

Now, let's modify this hook to enforce having a reference ticket number in the commit message like this:

```ruby
#!/usr/bin/env ruby
message_file = ARGV[0]
message = File.read(message_file)

$regex = /\[ref: #(\d+)\]/

if !$regex.match(message)
  puts "[COMMIT-MSG] Your message is not formatted correctly"
  exit 1
end
```

After saving the file you will get the follwing error when your commit message doesn't contain the `[ref: #<number>]` pattern:

![Error when using bad commit message](/assets/images/example-error.png "Error when using bad commit message")

*An error when using bad commit message*

And here's a successful commit message that passed the hook:

![A successful commit message](/assets/images/example-commit-success.png "A successful commit message")

*A successful commit message*


**Tip:** If you want to ignore Git hooks, add the  `--no-verify` argument to your git command. However, this only applies to *client-side* hooks and should only used when absolutely necessary.

<!---
### Prohibt unwanted code
In this example, I'm making sure that nobody forgets debugging code that breaks.

I'll extend the `pre-commit` with this code:

```bash
FILES_PATTERN='\.(js|ts)(\..+)?$'
FORBIDDEN='debugger;'
git diff --cached --name-only | \
    grep -E $FILES_PATTERN | \
    GREP_COLOR='4;5;37;41' xargs grep --color --with-filename -n $FORBIDDEN && echo 'COMMIT REJECTED Found "$FORBIDDEN" references. Please remove them before commiting' && exit 1
```
--->

#### The limitations of plain Git hooks

Distributing the Git hooks to every developer is a typical problem because the hooks are located in the `.git` hidden folder which is usually not pushed to the remote repository. It will also get quickly challenging to handle all the regular expressions and adher to some commits convention.

### Example 2: Apply conventional commits in a Node.js project

To overcome the limitations of the simple process above, I'll demonstrate how we are using git hooks in a Node.js project.
#### Conventional Commits

A [conventional commit](https://www.conventionalcommits.org/en/v1.0.0/) is a specification for adding human and machine readable meaning to commit messages. It makes it easier to write a set of automated tools on top of that such as:

* Automatically generating CHANGELOGs.
* Automatically determining a semantic version bump (based on the types of commits landed).
* Triggering build and publish processes.

The structure of conventional commits messages looks like this:

```
<type>[optional scope]: <description>
[optional body]
[optional footer(s)]
```

So the example we've seen in the screenshots before can become something like this (an optional body was added here for demonstration)

```
feat(blog): Add limitation part.
Write the paragraph that explains the limitations of Git hooks.
[ref: #1]
```

There are already a couple of tools that help checking that the commit message has that pattern as demonstrated by [commitlint](#commitlint).

#### Commitlint

[commitlint](https://commitlint.js.org/#/) is an awesome package that helps you get commit messages with high quality and provides short feedback cycles by linting commit messages right when they are authored.

It is very easy to use by just installing `commitlint` as a `devDependencies` in your project and defining the configurations you need in a `commitlint.config.js` file in the root of your repository.

This is so straight forward in comparision to what we were trying to do with the plain Git hooks and parsing the params before.

To use *commitlint* to have a commit message with the following structure

```
type(scope?): subject
body?
footer?
```

We can use the simplest example `commitlint.config.js` file that extends the defeault convensions:

```js
module.exports = {
    extends: [
        "@commitlint/config-conventional"
    ],
}
```


```js
/**
 * List of possible commit scopes based on the folders
 * which is a good practice in a monorepo
 */
const scopes = [
  'workspace',
  'tooling',
  ...getFolders('./packages'),
];

const configuration = {
  /*
   * Resolve and load @commitlint/config-conventional from node_modules.
   * Referenced packages must be installed
   */
  extends: ['@commitlint/config-conventional'],
 
  parserPreset: {
    parserOpts: {
      issuePrefixes: ['ABC-', '#'],
    },
  },
  /*
   * Resolve and load @commitlint/format from node_modules.
   * Referenced package must be installed
   */
  formatter: '@commitlint/format',
  /*
   * Any rules defined here will override rules from @commitlint/config-conventional
   */
  rules: {
    'body-leading-blank': [1, 'always'],
    'footer-leading-blank': [1, 'always'],
    'header-max-length': [1, 'always', 72],
    'scope-case': [2, 'always', 'lower-case'],
    'scope-empty': [2, 'never'],
    'scope-enum': [2, 'always', scopes],
    'subject-case': [
      1,
      'always',
      ['sentence-case', 'start-case', 'pascal-case', 'upper-case'],
    ],
    'subject-empty': [2, 'never'],
    'subject-full-stop': [1, 'always', '.'],
    'type-case': [2, 'always', 'lower-case'],
    'type-empty': [2, 'never'],
    'type-enum': [2, 'always', ['feat', 'fix', 'docs', 'style', 'refactor', 'test', 'revert', 'release']],
  }
};

module.exports = configuration;
```

This examples enforces commit length, style and many others in just a basic config object.

#### Husky

[Husky](https://github.com/typicode/husky) is an npm package that improves commit messages and the management of the hooks in a Node.js project. It solves the issue of the hidden `.git/hooks` by "installing" the Git hooks from source code files located in the repository. 
It is a great option that scales and helps you define everything only once in one place for all.

Husky works within the `package.json` file by including an object that configures Husky to run certain scripts, and then Husky manages the script at specific points in the Git lifecycle. 

To get it working, you need to install `husky` as a `devDependencies` in your project.

Afterwards, run a command that will allow Git hooks to be enabled:

```bash
npx husky install
```

Next, you will want to adjust the `package.json` file. The last command is run after the installation process concludes:

// package.json
```json
{
  "scripts": {
    "prepare": "husky install"
  }
}
```

### Checking commit messages with Commitlint and Husky

Commitlint can be configured to run as a *husky* **pre-commit** hook for local setting. To do that you just need to run the following command once for the set up to happen:

```bash
npx husky add .husky/commit-msg 'npx commitlint --edit $1'
```


Commitlint can also be [configured with CI server](https://commitlint.js.org/#/guides-ci-setup) to ensure that all commits are linted correctly and never skipped with `--no-verify` argument.

The configuration in the example GitLab CI would look as simple as this:

```
lint:commit:
  stage: lint
  script:
    - echo "${CI_COMMIT_MESSAGE}" | npx commitlint
```

## Summary

TODO link and mention automatic release notes generation