= git-sync

There's a bunch of web deployment systems lately that use git for uploading applications.

I hate this.

Deployments are not *just* a checkpoint in your application development, they are also something that itself needs to be developed and tested.  When I use one of these systems I find myself with lots of this:

    $ ...edit app.conf...
    $ git commit -a -m "conf" ; git push deploy

Pthfb.

That said, git is not a bad file synchronization tool.  It's actually pretty darn fast; faster than rsync because it keeps more state around (rsync has to recheck the world each time).  And it gives you a nice history of what you've done.

This command lets you use git like a sync tool, separate from using it as your source control.  It's written as a git extension because whatever.

== Using git-sync

Put `git-sync` somewhere on your `$PATH`.  You can call `git-sync` or `git sync`.

You must then configure your git repository with a few commands:

    # Where to upload to:
    $ git config --add sync.default.remote git@deploymenthost.com:repo.git
    # Kind of a scratch location to keep the files in:
    $ git config --add sync.default.repo .git/sync

Now you can just run `git sync` and it will upload everything, including uncommitted work.  (It also uploads untracked files, which... maybe it shouldn't.  But files in `.gitignore` should be ignored regardless.)

You can also run a build step everytime you do a sync:

    # Optional setting:
    $ git config --add sync.default.build tools/build

Now everytime you run `git sync` it will run the script in `tools/build` and upload the results.  You can use this to run a source compressor or whatever else you might want to do before an upload.  The *results* of your build will be committed and pushed.  The commad will be run with the current working directory of the files ready to be uploaded (not your normal source repository).

If you want to leave files out of what gets sync'd, but still keep those files in your source repository, create a file named `.syncignore` (similar to `.gitignore`).  E.g., put `tools/` into that file to keep your build tools from getting uploaded.

Note in all these that `default` is a string you can change, e.g., you can make configuration values for `production` (`sync.production.remote`) and do `git sync production`.

== History

You'll still have a nice set of history of what you've uploaded, though that history will be strictly per-remote.  It won't be directly attached to your main repository, but the commit messages will show the relation.

Each commit has a commit log message `deploying $version` where the version the output of `git describe --always --dirty`.  That command outputs the tag, branch, or if necessary just the latest revision of the repository.  The string `-dirty` is appended when there is uncommitted work.
