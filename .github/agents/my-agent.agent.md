---
name: github-fork-maintenance
description: Maintain a forked GitHub repository over the long term (sync with upstream, move custom changes onto a new upstream release, resolve conflicts, keep the fork's own changes small and isolated, and document how the fork differs from upstream). Use ONLY when the repo is a fork of another GitHub project, for example "my fork is behind upstream", "pull in upstream changes", "rebase my patches onto the new release", "sync my fork", "keep my fork up to date". Do not use for ordinary non-fork repos or general git questions. APPROVAL REQUIRED - before loading or applying this skill, ask the user whether to use it and wait for a clear yes; if they say no or don't answer, do not use it.
---
 
# GitHub fork maintenance
 
Keeps a forked GitHub project in sync with its upstream for a small team (one or two maintainers) with as little ongoing work as possible. A fork gets expensive when your changes overlap with what upstream keeps changing, so every practice below either shrinks that overlap or makes each sync cheaper. When two options are close, pick the one that creates less future work.
 
## 1. Approval gate (do this first)
 
This skill rewrites branches and pushes to the user's fork, so it only runs with their explicit go-ahead.
 
- Before running commands, changing files, or reading further, ask something like: "This looks like a forked-repo task. Want me to use the fork-maintenance skill for it?"
- Wait for a clear yes. If the answer is no, or there is no answer, stop using this skill and help without it.
- Ask again in each new conversation; an earlier yes does not carry over.
## 2. Scope check
 
Continue only if the repository is a fork of another GitHub project. Check without changing anything: `gh repo view --json isFork,parent`, or look at the remotes, and ask the user if it is still unclear. If it is not a fork, say so and stop using this skill.
 
Also confirm the fork is meant to keep following upstream. If it has become its own project, the syncing advice below does not apply; tell the user rather than forcing it.
 
## 3. The core rule: own files, minimal hooks
 
This is the most important practice for a fork kept by one or two people, so it gets its own section on top of item 1 below. Keep your modifications in your own files, and use the fewest possible lines in the original code to call them. Then move to a new upstream release whenever it suits you (rebase onto release branches), because there is almost nothing left to conflict.
 
Why it works: a conflict only happens where your edit and upstream's edit land on the same lines. Code in files upstream does not know exist cannot conflict. The few hook lines can, but they are tiny, easy to re-apply, and rarely touched by upstream.
 
Apply it whenever you make or review a change in the fork:
 
1. **Start with "can this live in a new file?"** Before editing any upstream file, check whether the feature can be a new file, module, class, or folder that you own (for example a `custom/` directory, or a filename with your own suffix).
2. **Reduce every edit to a hook.** A hook is one call, import, registration, or config entry that hands off to your file. If a change needs real logic inside an upstream function, move the logic into your file and leave only the call behind.
3. **Wrap or extend instead of editing in place.** Prefer a subclass, wrapper, decorator, plugin or extension point, event listener, config override, or build-system include over changing upstream code directly.
4. **Do not move, rename, reformat, or reorder upstream code**, even to make a hook look neater.
5. **Mark and record each hook.** Add a `FORK-PATCH: <why>` comment at the hook and list it in `FORK.md` (file, area, what it calls).
6. **Check the diff before finishing.** Run `git diff --stat <upstream-base>..HEAD`. Upstream-owned files should show only small hook edits. If a hook is more than a few lines, or an upstream file shows a large diff, look for a way to move that code into your own files, and tell the user if it cannot be avoided.
7. **If upstream offers no place to attach a small hook,** either accept the larger patch and note it in `FORK.md` as a conflict hotspot, or propose a small upstream PR that adds an extension point (see item 7 below).
## 4. Best practices
 
### Shape the fork so there is little to maintain
 
This is the biggest lever. Most sync pain comes from touching upstream files, not from the sync commands.
 
1. **Own files, minimal hooks.** Put custom code in new files or folders you own (a `custom/` directory, or a suffixed name like `Makefile_<you>`), and touch upstream files only with the fewest lines needed to call into your code. Fewer touched upstream files means fewer conflicts. See section 3 for how to apply this.
2. **Leave the rest of upstream alone.** No reformatting, renames, reordering, spelling fixes, or "cleanup" in upstream code. Each one is a future conflict that buys the fork nothing.
3. **Prefer upstream's own extension points** (config files, settings, variables, plugin mechanisms) over patching code.
4. **Mark every edit to an upstream file** with a short comment such as `FORK-PATCH: <why>`, so each one is easy to grep for and re-apply after a sync.
5. **Never hand-merge generated files.** Keep regenerated output in its own commit; on conflict, rerun the generator instead of resolving by hand.
6. **Put your CI changes in your own workflow files.** Upstream workflows often need secrets a fork does not have; editing them also creates conflicts.
### Shrink the fork by contributing back
 
7. If a customization keeps conflicting, offer upstream a small PR that adds the hook or config option it needs; the fork's patch then shrinks to configuration.
8. If a feature is generally useful, upstream it, one feature or fix per PR. Every merged patch is one fewer to carry. Upstream may decline; that is fine, just keep the patch small.
9. Before a large divergence, check upstream issues and PRs or ask the maintainers; the change may already be planned.
### Branch layout (keep it simple)
 
If the repo already has a layout, keep it. Do not reorganize branches unless asked.
 
10. **Keep your changes separable from upstream.** A common layout: one branch mirrors upstream with no commits of its own (fast-forward only), and a patch branch holds upstream release plus your commits. If the repo keeps patches on the default branch, the same rebase approach applies to it.
11. **Follow releases, not every upstream commit.** Base the patch branch on an upstream release tag or release branch and skip unreleased code. If upstream has no releases, use a recent known-good commit. Sync when there is a reason (new release, security fix, a feature you need), not on every upstream push.
12. **Do not mint competing version numbers.** Keep upstream's tags in the fork. If you tag your own builds, mark them as derived from the upstream version (upstream `1.2.3` becomes `1.2.3+<name>`), so they cannot be mistaken for upstream's.
### Syncing
 
13. **Remotes:** `origin` is the fork, `upstream` is the original (read-only use). Run `git fetch upstream --tags`.
14. **Mirror branch:** use GitHub's "Sync fork" button, `gh repo sync <owner>/<fork> -b <branch>`, or `git merge --ff-only upstream/<branch>`. Only use these on a branch with no commits of your own. `gh repo sync` fails when there are conflicts.
15. **Patch branch:** default to rebasing your commits onto the new release: `git rebase --onto <new-tag> <old-tag> <patch-branch>`. Rebase keeps a short linear patch stack, makes it obvious what is yours, and makes patches easy to upstream or drop. Merging upstream instead is also valid, especially when maintaining several release lines that need history that is never rewritten. Pick one approach per repo and stick to it; switching back and forth makes it hard to tell where the fork branched from upstream. In GitHub Desktop, Branch > Rebase Current Branch rebases onto a branch you pick; for a release tag, run `git rebase --onto` in the local clone instead (Desktop works on the same clone and shows the result).
16. **Sync lazily, but not never.** The longer between syncs, the more painful each one is. Choose a cadence (for example, each upstream release). If the fork is far behind, step through releases one at a time rather than jumping to the newest in one go.
17. **Preview conflicts first.** `git merge-tree --write-tree HEAD <upstream-ref>` (Git 2.38+) shows conflicts without touching the working tree. Compare the files you changed (`git diff --name-only <base>..HEAD`) with the files upstream changed to see where trouble will be.
18. **Enable `git rerere`** (`git config rerere.enabled true`) so a conflict resolved once is replayed automatically in later rebases. The record is local to the clone, and `git gc` prunes old records (resolved ones after 60 days by default, setting `gc.rerereResolved`), so raise it if syncs are months apart.
19. **Drop patches upstream already has.** Before rebasing, run `git cherry -v <new-tag>` on the patch branch: lines starting with `-` are already upstream and will be dropped. Remove them from the fork notes too.
20. **Verify after each sync.** Build and run the project's own tests, the same ones upstream runs. Upstream changes can break the build or tests without any textual git conflict. After a rebase, `git range-diff <old-base>..<backup> <new-base>..<branch>` confirms your patches did not change unintentionally.
### Safety when rewriting history
 
21. Create a backup ref before rebasing (`git branch backup/<branch>-<date>`).
22. Leave the push to the user. Their changes go out through the GitHub Desktop app, which shows a **Force push origin** button after a rebase. Tell them which branches need pushing and that rewritten ones need a force push because history changed. Push from the command line only if they ask, and then use `--force-with-lease`, never bare `--force`.
23. Never `reset --hard` to upstream or discard commits on a branch that holds the user's changes; that deletes fork-only commits.
### Automation (only if the user wants it)
 
24. Keep it light for a one-or-two person fork. GitHub disables Actions on forks by default, scheduled workflows stay off until the owner enables them, and in public repos scheduled workflows switch themselves off after 60 days without repository activity. A quiet fork's cron sync can silently stop, so do not rely on one. If you automate, automate the safe part (fast-forwarding the mirror branch, or opening an issue or PR when a new upstream release appears) and leave rebasing to a person. If a rebase is automated, have it open a PR on conflict instead of force-pushing.
### Document the fork
 
25. Keep a short `FORK.md` (or a README section) with: why the fork exists; the upstream version the patches sit on; a list of patches (what, which files, why, upstream PR link and status); and the exact sync commands used. Update it on every sync. The next sync (maybe months later, maybe by the second maintainer) depends on it to know what to carry, drop, or upstream. Propose adding it if it is missing.
### Licensing
 
26. Keep upstream's LICENSE, copyright headers, and NOTICE files intact. Some licenses (Apache-2.0, for example) require modified files to carry a notice that you changed them, and some forbid removing copyright notices. Check the license before publishing or distributing the fork.
## 5. Routine for "sync my fork"
 
1. **Orient (read-only).** `git remote -v`, current branch, `git status`, `git fetch upstream --tags`. Identify upstream's default branch and latest release, the version the patches currently sit on, how many commits are the fork's own, and which upstream files the fork touches.
2. **Tell the user in a few lines:** how far behind the fork is, the target release, how many patches, and where conflicts are likely (use the preview from item 17). Suggest the plan, following the repo's existing merge or rebase convention.
3. **Update.** Back up, bring the mirror branch forward, rebase or merge the patch branch onto the target (rerere on, regenerate generated files instead of hand-merging), then build and test.
4. **Clean up.** Drop patches that landed upstream, update the base version and patch list in `FORK.md`, and note any patch that would be worth sending upstream.
5. **Hand off the push.** List the branches the user should push from GitHub Desktop, and which of them need **Force push origin**.
6. **Report briefly:** what changed, what was dropped, which conflicts needed a decision and how they were resolved.
Keep tooling proportionate. For one or two maintainers, plain git plus a short `FORK.md` beats a heavy setup.
 
