# Git Rescue Lab Workflow

## 1. Bisect Finding

Using `git bisect`, I found that commit `c99fb4209e6fb6e5ed2789893fdb2f893d61d6c6` was the first bad commit. It changed the BULK20 condition so orders with exactly 5 items did not receive the intended 20% discount.

## 2. Branching Strategy

For a team of four, I would recommend GitHub Flow. It keeps the main branch stable while allowing each developer to work on a short-lived feature branch and open a pull request for review. This is relatively simple for a small team and avoids the extra complexity of Git Flow while still providing code review and controlled integration.

## 3. Removing the Secret from Git History

Removing `.env` from tracking does not remove it from previous commits. To fully remove the secret, I would need to rewrite the repository's history using a history-rewriting tool such as `git filter-repo` or an equivalent tool, then force-push the rewritten history. Any exposed credentials should also be revoked or rotated.

This assignment did not require rewriting the entire history because the credentials were explicitly fake and the exercise is focused on practicing Git tracking, `.gitignore`, branching, merging, rebasing, and history concepts.

## 4. Rewriting History

It was acceptable to rewrite the commit message because the commits being rebased had not been shared with teammates. Rewriting history changes commit hashes, so doing this to commits that teammates had already pulled could cause their local histories to diverge and create unnecessary conflicts. Once commits have been shared and others may depend on them, they generally should not be rewritten.
