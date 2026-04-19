---
layout: post
title: gt, a stacked PRs CLI
tag:
- ruby
- git
- github
- side-project
- claude
---

I wanted to see what it feels like to build a DIY [Graphite](https://graphite.dev) clone. So I spent a weekend building [`gt`](https://rubygems.org/gems/gt-cli), a CLI for stacked PRs on GitHub.

```sh
$ gt create new-user-mailer
? Include untracked file '.session.key'? (You chose: no)
? Include untracked file 'app/views/admin_mailer/new_user.text.erb'? (You chose: yes)
? Include untracked file 'logfile'? (You chose: no)
...
✓ Creating branch new-user-mailer
Opening PR for new-user-mailer → postmark

https://github.com/grodowski/undercover-ci/pull/807
✓ Updating stack comments
```

It also posts a stack comment on each PR:

![gt stack comment on GitHub PR](/assets/images/posts/gt-stack-comment.png)

### Squash merges

When GitHub squash-merges a PR the resulting commit SHA doesn't exist in your local history, so a naive rebase breaks. `gt` stores a fork-point (the parent branch tip at the time you branched) in git config and uses `git rebase --onto` to skip already-merged commits and replay only yours. All state in `.git/config`.

### What it does

`gt create` stages your changes, creates a new branch from the current one, commits, pushes, and opens a PR targeting the parent branch.

```sh
gt create auth -m "add authentication"
# ... make changes ...
gt create profile -m "add profile page"
# ... make changes ...
gt create settings -m "add settings"
```

`gt log` shows the stack:

```sh
gt log
# main
#   └─ auth
#      └─ profile
#         └─ settings *
```

`gt sync` pulls the latest `main` and rebases the whole stack on top of it. `gt restack` does the same after a PR gets merged and prompts to clean up the branch.

Navigation:

```sh
gt up      # toward the tip
gt down    # toward main
gt top     # jump to tip
```

All commands:

```sh
$ gt help
Usage: gt <command> [options]

Commands:
  create <name> -m <msg> [-p]  Create a new stacked branch and PR
  log (ls)                     Show the current stack
  restack                      Rebase the stack onto updated parents
                               (prompts to delete if bottom PR was merged)
  modify (m) [-m <msg>] [-p]   Amend the current branch and restack
  sync                         Pull main and restack

Navigation:
  up                           Move up one level in the stack
  down                         Move down one level in the stack
  top                          Jump to the top of the stack

Options for restack:
  --continue                   Continue after resolving conflicts
  --abort                      Abort an in-progress restack
```

### Building with Claude

I designed the commands, the stack model, and the edge cases. Claude did the implementation. Under the hood it leans on `gh` for all GitHub operations and [cli-ui](https://github.com/Shopify/cli-ui) for the interactive prompts. I used the [undercover-claude](https://github.com/grodowski/undercover-claude) skill to keep test coverage tight as features landed. How much that actually matters is the subject of the [follow-up post](/2026/04/17/claude-code-undercover-skill).

```sh
gem install gt-cli
```
