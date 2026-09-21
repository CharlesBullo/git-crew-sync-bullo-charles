1. What did the rejected push error message tell you, and why did it happen?
   
  ! [rejected]        feature/overtime-pay -> feature/overtime-pay (fetch first)
  error: failed to push some refs to 'https://github.com/CharlesBullo/git-crew-sync-bullo-charles.git'
  hint: Updates were rejected because the remote contains work that you do
  hint: not have locally. This is usually caused by another repository pushing
  hint: to the same ref. If you want to integrate the remote changes, use
  hint: 'git pull' before pushing again.
  hint: See the 'Note about fast-forwards' in 'git push --help' for details.

- The remote branch had commits my local branch didn't have the remote contains work that you do not have locally.
- The push was refused because it would not have been a fast-forward — my local tip was no longer an ancestor of the remote tip.

Why it happened: Git's default push policy refuses to overwrite remote history. Both clones started from the same base commit on feature/overtime-pay. 
Clone A pushed first, so the remote ref advanced to a new commit. Clone B still had the old base as its parent — its new commit was a sibling, not a 
descendant, of what was now on the remote. If Git had allowed the push, it would have silently discarded Clone A's commit. So the rejection is a safety 
feature, not a failure of the network or credentials.

It happened a second time in Task 4 for the same reason, just with the roles reversed — Clone A was now the stale one because Clone B had pushed the merge.

2. What's the actual difference between how you resolved Task 3 (merge) vs Task 4 (rebase)?
I ran git fetch origin and then git merge origin/feature/overtime-pay. Git tried to combine two divergent histories and hit a conflict inside calculatePay.
I resolved it by keeping both features — overtime logic from Clone A, and Math.round instead of Math.floor from Clone B — so the function applies overtime
first and rounds the result:

  function calculatePay(hours, rate) {
    if (hours <= 8) {
      return Math.floor(hours * rate);
    }
    return Math.floor(8 * rate + (hours - 8) * rate * 1.5);
  }

Because the branches had truly diverged, git merge created a new merge commit with two parents. Both original commits stay exactly as they were authored — same hashes, 
same messages. History keeps the diamond shape: the branches split and rejoined.

3. What one habit would have avoided both rejected pushes in this lab?

git pull --rebase (or at minimum git fetch immediately before every git push).
If I had synced with origin/feature/overtime-pay right before each push, my local branch would already have contained the remote's newest commit, and the push would have 
been a fast-forward — no rejection, no conflict resolution needed at that moment (or if there was a conflict, I'd have handled it at fetch-time instead of at push-time).

The mental model: a rejected push isn't a push problem. It's a staleness problem. The branch I was trying to push was based on an old snapshot of the remote. Pulling 
first keeps the local branch current, and push stops being an event where history has to be reconciled.

4. Which approach — merge or rebase — would you default to on a shared team branch, and why?
Shared branches are touched by multiple people. Any commit already pushed to a shared branch is by definition something a teammate may have based work on. Rebase
rewrites those commits' hashes, which means everyone else's local branches point to orphaned commits. They'd have to recover from a history rewrite (git pull
--rebase or a manual reset) — a coordination cost that isn't worth it for a cosmetic linear history.

Merge is additive and safe. It never changes existing commits, only adds a merge commit on top. That makes it the boring, predictable choice, which is what you 
want when a branch is a shared resource.
