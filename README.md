# GIT-AUTOCOMMIT

git-autocommit periodically checks for working copy changes. If changes exit they are auto-committed (`git -A && git commit -m "<git-autocommit>"`).
By running 'git-autocommit -s' or by simply rerunning 'git-autocommit' after a previous abort via CTRL-C, a sequence of commits with a commit message of `<git-autocommit>` can be squashed into a single commit.

## Typical usage:

```
$ cd /my/project            # Change to Git working copy.
$ git-autocommit -i 60 -V   # Check every 60 seconds, verbose output.
```
... time passes ... then hit CTRL-C to abort git-autocommit
```
$ git-autocommit -i 60 -V   # Rerun
There are autocommits. Squash first? [y/N] y
Soft resetting to b98fdffcaca3571135a89466cdb1ad7f1c2798da Increased monitor sensor count.
Now commit your combined changes or execute 'git reset HEAD@{1}' to undo.

$ git commit -m "Extracted base class 'AbstractMonitor'" # Commit all autocommits as a single commit.
```

## History
```
Version  Change
-------------------------------------------------------------------------------
1.0      Intial version
1.1      Ensure that any uncommitted changes are auto-committed before asking
         user whether to squash first.
1.2      Warn if local modifications exist upon passing '-s'.
```

