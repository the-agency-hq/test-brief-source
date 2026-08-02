---
paths:
  - "**/*"
---

# Worktrees

Create Git worktrees in the `.worktrees` directory at the root of the project:

```bash
git worktree add .worktrees/<branch-name> -b <branch-name>
```

One fixed location means every worktree for a project is discoverable from the project itself — `git worktree list`
and `ls .worktrees/` agree — rather than scattered across sibling directories whose names have to be remembered.

Remove a worktree with `git worktree remove` rather than deleting the directory, so Git's own bookkeeping is
updated with it.
