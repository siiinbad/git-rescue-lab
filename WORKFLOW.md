Git Rescue Lab Master Guide
1. Verification: Finding the First Bad Commit

After running git bisect, Git identifies the exact commit that introduced the regression.

For this lab, the first bad commit was:

c99fb4209e6fb6e5ed2789893fdb2f893d61d6c6


Its original commit message was:

asdf


The commit modified pricing.js and introduced the BULK20 pricing regression. Specifically, the condition incorrectly required more than five items instead of allowing orders containing exactly five items.

The correct condition is:

items.length >= 5


The expected behavior is that an order with five or more items receives the 20% BULK20 discount.

2. Branching Strategy: Why GitHub Flow?

For a small team of four developers, I would recommend GitHub Flow.

GitHub Flow offers a simple balance between structure and agility:

main should remain stable and deployable.

Developers create short-lived feature or bug-fix branches from main.

Each change is submitted through a Pull Request.

Pull Requests provide opportunities for code review, discussion, and automated testing.

Once approved, the branch can be merged into main.

This approach is relatively simple for a small team and avoids the additional complexity of Git Flow while still providing a controlled development and review process.

3. Removing Secrets from Git History

Adding .env to .gitignore prevents the file from being tracked in future commits, but it does not remove the file from commits where it was already committed.

To completely remove the file and its contents from the repository's history, I would need to rewrite the repository history.

A modern approach is to use git filter-repo.

Installation
pip install git-filter-repo

Remove .env from all historical commits
git filter-repo --path .env --invert-paths


If the repository has already been pushed to GitHub, the rewritten history would then need to be force-pushed:

git push origin --force --all
git push origin --force --tags


Any credentials that were actually exposed should also be revoked or rotated, because deleting them from Git history does not guarantee that nobody previously obtained them.

Verification

The history can be checked with:

git log --all --full-history -- .env


If there is no output, the .env path is no longer present in the reachable history being examined.

Why the assignment did not require this

This assignment only requires removing .env from tracking and adding it to .gitignore.

The credentials in the exercise are fake, so completely rewriting the repository's history would add unnecessary complexity and could interfere with the Git history students are required to demonstrate.

4. Rewriting History with Interactive Rebase

Interactive rebase can be used to modify commits that have not yet been shared with teammates.

For example:

git rebase -i HEAD~3


The editor displays commits using commands such as:

pick abc1234 Some commit
pick def5678 Another commit
pick ghi9012 Another commit


To change a commit message, change pick to reword:

reword abc1234 Some commit


After saving and exiting, Git prompts for the new commit message.

For this assignment, the unhelpful message:

asdf


was changed to:

Fix BULK20 discount calculation

Verification

Check the resulting history with:

git log --oneline


The commit hash changes after a rebase because Git creates a new version of the rewritten commit.

Why this is acceptable locally

The commits being rewritten had not been pushed for teammates to use. Therefore, changing the commit history did not disrupt anyone else's local repository.

Once teammates have pulled commits, rewriting those shared commits can cause their histories to diverge and require unnecessary recovery work.

5. Git Merge vs. Git Rebase
Git Merge

git merge combines two lines of development and normally creates a merge commit.

For example:

git checkout main
git merge feature/holiday-sale


A merge preserves the existing branch history and records the point where the branches were brought together.

Git Rebase

git rebase moves a sequence of commits onto a different base commit.

For example:

git checkout feature
git rebase main


Rebase can create a cleaner, more linear history, but it rewrites commits and therefore changes their hashes.

Because of this, rewriting shared public history should generally be avoided.

6. Merge Conflict Resolution

When Git encounters conflicting changes during a merge, it marks the affected files.

First check the repository status:

git status


Git will identify the conflicted files.

Open the conflicted file and look for conflict markers:

<<<<<<< HEAD
Current branch changes
=======
Incoming branch changes
>>>>>>> feature/holiday-sale


The developer must manually determine how the changes should be combined.

After editing the file:

Remove the conflict markers.

Keep the correct code from both sides where appropriate.

Save the file.

Run the tests.

Stage the resolved file.

For example:

git add pricing.js


For a merge, complete the merge with:

git commit


For a rebase, continue with:

git rebase --continue

7. Conflict Resolution in This Lab

The conflict occurred in pricing.js because main contained the BULK20 logic while the holiday-sale branch added the HOLIDAY25 discount.

The correct combined implementation supports all three discount codes:

function calculateTotal(items, discountCode) {
  let subtotal = items.reduce((sum, item) => sum + item.price * item.qty, 0);
  let discount = 0;

  if (discountCode === 'SAVE10') {
    discount = subtotal * 0.10;
  } else if (discountCode === 'HOLIDAY25') {
    discount = subtotal * 0.25;
  } else if (discountCode === 'BULK20' && items.length >= 5) {
    discount = subtotal * 0.20;
  }

  return subtotal - discount;
}

module.exports = { calculateTotal };


The important bug fix is:

items.length >= 5


rather than:

items.length > 5


The final test suite should produce:

PASS: no discount
PASS: 10% off with SAVE10
PASS: 20% off with BULK20 (5+ items)
PASS: 25% off with HOLIDAY25

8. Useful Git Commands
View commit history
git log --oneline --graph --decorate --all

Check repository status
git status

View a specific commit
git show <commit-hash>

Start a bisect
git bisect start
git bisect bad
git bisect good <known-good-commit>


Or automate the tests:

git bisect run node test.js

End a bisect
git bisect reset

View branches
git branch -a

Merge a feature branch
git merge feature/holiday-sale

Create a release tag
git tag v1.0

Push the branch
git push -u origin main

Push tags
git push --tags

9. Final Lab Checklist

Before submitting the repository, verify:

 git bisect identified the first bad commit.

 The bad commit message was reworded using interactive rebase.

 feature/holiday-sale was merged.

 The pricing.js merge conflict was resolved manually.

 SAVE10 works.

 BULK20 works for five or more items.

 HOLIDAY25 works.

 .env is no longer tracked.

 .env is listed in .gitignore.

 WORKFLOW.md documents the required answers.

 The final commit is tagged v1.0.

 The final branch has been pushed to GitHub.

 The v1.0 tag has been pushed.

 The repository has a clean working tree.