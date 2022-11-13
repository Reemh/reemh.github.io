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
Alice just recently joined the team and when she was browsing through some code and she found a line of code that she thought was useless.
She checked the commit message for that line and found the following message:

> Fixed the bug

There was no ticket number, no further info, no explanation or any references. The author of that line is not in the company anymore to follow up with him.
She removed that line and ran all local tests and everything was green for her. 
So she decided to remove that line of code because "the bug" obviously was fixed some other way as well.
In the next release a high profile team is suddenly blocked and nothing is working for them because they were depending on a util that was generated from a code that Alice touched before and something essential there broke.
The code was not directly used in the repository Alice used but was used in some edge case somewhere else. Nobody would have known that. "The bug" mentioned in the commit was something well documented around that time in the system but who would be able to go search for a ticket that was "closed" any time around the commit date.. And to admit it, we all know we usually don't close our tickets on the same day we commit something... and some tickets can be open for soooo long. I personally had a ticket open for a couple of months and worked with a ticketing system with thousands of tickets created every single day! 

## The solution
A ticket number!

End of story.

Ok now you might tell me but not everyone writes something useful in tickets. Now that problem needs a different post. Let's focus on __forcing__ a ticket number in every commit message. Now read that again.

TODO add image

Now say it out loud.

## The problem part 2
Now Alice, having had that bitter experience, documents every single bug she has on a ticket and makes sure every commit has a ticket number to make it easier to understand the reasoning behind it.

Bob just joined the team and he keeps forgetting that ticket number on the commit message.
Alice keeps bothering him with that.
He thinks she's not focusing on the essential work he does in his Pull Request. She thinks he's reluctant because he's not following "the rule" of how the team works. The guys are doing well together.

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

* pre-commit: Check the commit message for spelling errors.
* pre-receive: Enforce project coding standards.
* post-commit: Email/SMS team members of a new commit.
* post-receive: Push the code to production.

Ok wait, what is a commit and what is a receive?

If you know that, feels free to jump to the next section.

If not, then continue reading.





## Why should you care?
## How to use githooks to improve your workflow?
## Automatic release notes generation