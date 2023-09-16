Common Operations
=================

Cleaning a Repo
---------------
So, you've been working in a repo for a while and you want to completely blow
away its contents and revert to what is currently on the server. To remove all
of the untracked files and directories that become empty along the way, use `git
clean -df`. Note that there is a `--dry-run` option as well that is useful. This
does not delete ignored files, so it's often the desired operation. If you want
to get rid of everything, including files matching the patterns in `.gitignore`
there is the `-x` option that can be added.
```bash
$ git -df           # delete everything but ignored files
$ git -dfx          # whole smash, get rid of everything
# If you want to do a dry run first before murdering everything
$ git -dfx --dry-run
```

