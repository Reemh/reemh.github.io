---
layout: post
title:  "What are githooks"
date:   2022-11-08 20:07:52 +0200
categories: tech
permalink: /githooks/
---

# What are githooks?
## The problem
Imagine you have a team of great software engineers with different level of expertise. 
The are having issues working together as each person has her/his prefered way of writing and comitting code.

Alice just recently joined the team and when she was browsing through some code, she found a line of code that she thought was useless.

She checked the commit message for that line and found the following message:

> Fixed the bug

There was no ticket number, no further info, no explanation or any references. The author of that line is not in the company anymore to follow up with.

She removed that line after she ran all local tests that were green for her. 
She thought "the bug" was obviously fixed some other way.

In the next release, a feature developed by another team was broken because they were depending on a util that was generated from the code removed by Alice before. Nobody would have known that. "The bug" mentioned in the commit was documented around that time in the system but who would be able to go search for a ticket that was "closed" any time around the commit date. And to admit it, we all know, we usually don't close our tickets on the same day we commit something... and some tickets can be open for soooo long. I personally had a ticket open for a couple of months and worked with a ticketing system with thousands of tickets created every single day! 

## The solution
A ticket number!

End of story.

Ok now you might tell me but not everyone writes something useful in tickets. Now that problem needs a different post. Let's focus on __enforcing__ a ticket number in every commit message.

TODO add image


## The problem part 2
Now Alice, having had that bitter experience, documents every single bug she has on a ticket and makes sure every commit has a ticket number to make it easier to understand the reasoning behind it.

Bob has just joined the team. 
He keeps forgetting that ticket number on the commit message.
Alice keeps bothering him with that.
He thinks she's not focusing on the essential work he is doing in his Pull Request. She thinks he's not following "the rule" of how the team works. The guys are __not__ doing well together.

### The result
A frustrated team.

## The solution part 2
GitHooks!

Congratulations if you read that far. Now let's jump into the technical part.

## What are Githooks?
[Githooks](https://githooks.com/) (or Git Hooks) are scripts that Git executes before or after events such as: commit, push, and receive.

### Requirements
* An installed Git, nothing more to downloads.

### Examples

<!--- do we need this part?
-->
There are two groups of these hooks: *client-side* and *server-side*. 
*Client-side* hooks are triggered by operations such as committing and merging, 
while *server-side* hooks run on network operations such as receiving pushed commits. 
You can use these hooks for all sorts of reasons. [1](https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks)

<!--- These scripts run before and after pushes to the server.-->

* pre-commit: Check the commit message for spelling errors.
* pre-receive: Enforce project coding standards.
* post-commit: Email/SMS team members of a new commit.
* post-receive: Push the code to production.

<!---todo add figure-->

![Git commit hooks path](/home/reem/workspaces/reemh.github.io/myblog/assets/images/commit-hooks-final.drawio.png "Commit hooks")


## Why should you care?

Git hooks are a great way to improve developers' time and collaboration by automatically checking things like grammer, style and length of commit messages and it's a way to automatically check certain things were successful such as test suits and coverage checks or automatically pushing code to stage or production.

## How to use githooks to improve your workflow?

For simplicity, I'll just present two examples that I've worked with closely.

### Pre-commit to enforcing certain rules for a commit message


### Tip
If you want to ignore git hooks, add the  `--no-verify` argument to your git command. Please only do this only if you absolutely know what you are doing.

## The coding part (demo)
As mentioned before, the githooks are automatically in any repository that was initialized with `git init` (so every repository).

The git hooks are by default under `.git/hooks` subfolder, which by default contains a list of samples written as shell scripts. They are very simple to follow and use.

Here's the `commit-msg.sample` that checks the duplicate SOB author lines in the message:
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

1. Duplicate the `commit-msg.sample` and rename it to `commit-msg` (with no extension at all).
1. Give it execution permission using `chmod +x .git/hooks/commit-msg`
1. Modify the file with the code you need (see examples below)
1. Save your changes and you are ready to go! It will be picked up automatically by git.

Let's modify this hook to enforce having a reference ticket number in the commit message like this:
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

## The limitations of this approach
Those hooks exists in the `.git` repository which are usually not committed. Which means, every time you want to enforce something, you need to make sure that everyone has the same hooks.
This will also get quickly messy to handle all the regular expressions and making the possible patterns for the commit message more complicated.

### Husky
Since I'm writing a lot of **Node.js** code, my project uses [Husky](https://github.com/typicode/husky), an npm package that improves commit messages and the management of the hooks in a Node.js project.
It is a reasonable option that scales and helps you define everything only once for all.

### Commitlint
An awesome package that helps you get high commit message quality and short feedback cycles by linting commit messages right when they are authored is [commitlint](https://commitlint.js.org/#/).
It is very easy to use by just installing `commitlint` as a devDependencies in your project and defining the configurations you need in a `commitlint.config.js` file.
This is so straight forward in comparision to what we were trying to do with the plain git hooks and parsing the params above.

An example config would look like this
```js
// list of possible commit scopes based on the folders
const Scopes = [
  'workspace', // If it is related to the whole repository
  'tooling',
  ...getFolders('./packages'),
];

const Configuration = {
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
  },
  /*
   * Functions that return true if commitlint should ignore the given message.
   */
  ignores: [(commit) => commit === ''],
  /*
   * Whether commitlint uses the default ignore rules.
   */
  defaultIgnores: true,
  /*
   * Custom URL to show upon failure
   */
  helpUrl:
    'https://github.com/conventional-changelog/commitlint/#what-is-commitlint',
  /*
   * Custom prompt configs
   */
  prompt: {
    messages: {},
    questions: {
      type: {
        description: 'please input type:',
      },
    },
  },
};

module.exports = Configuration;

```
This examples enforces commit length, style and many others in just a basic config object. We will talk more about the `parserPreset` later.

This can be configured to run as a *husky* **pre-commit** hook for local setting and can be [configured with CI server](https://commitlint.js.org/#/guides-ci-setup) to ensure that all commits are linted correctly.

Here's how it looks like in GitLab CI
```
lint:commit:
  stage: lint
  script:
    - echo "${CI_COMMIT_MESSAGE}" | npx commitlint
```

### Conventional Commits
A [commit convention](https://www.conventionalcommits.org/en/v1.0.0/) is a specification for adding human and machine readable meaning to commit messages.
It makes it easier to write a set of automated tools on top of that such as:

* Automatically generating CHANGELOGs.
* Automatically determining a semantic version bump (based on the types of commits landed).
* Triggering build and publish processes.

The commit message structure looks like this
```
<type>[optional scope]: <description>
[optional body]
[optional footer(s)]
```

So when following that convention in combination with *commitlint* as mentioned above, we get the following structure:
```
type(scope?): subject
body?
footer?
```

So the example we've seen in the screenshots before can become something like this (an optional body was added here for demonstration)
```
feat(blog): Add limitation part.
Write the paragraph that explains what the limitations of git hooks are.
[ref: #1]
```

## Automatic release notes generation
