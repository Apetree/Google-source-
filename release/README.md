> **Warning: The "master" branch is no longer used.
>   Use "main" instead.**<br>
> https://gerrit.googlesource.com/git-repo/+/HEAD/release/README.md
> repo release process
>This is the process for creating
> a new release of repo, as well as all the
>  related topics and flows.
> Contents
> Schedule
> Release Freezes
> Launcher script
> Key management
> Registering a new key
> Self update algorithm
> Force an update
> Branch management
> Creating a new release >> Creating a signed tag
> > Push the new release
> > Announce the release
> > Project References
> > Schedule
There is no
> >  specific schedule for when
> >  releases are made. Usually it‘s more
> >  along the lines of “enough minor changes have been merged
> > ”, or "there’s a known issue the
> > maintainers know should get fixed". If you find a fix
> >  has been merged for an issue
> >  important to you, but hasn't been released after
> >  a week or so, feel free to contact us
> > to request a new release.
Release Freezes
We try to observe a regular schedule for when not to release. If something goes wrong, staff need to be active in order to respond quickly & effectively. We also don't want to disrupt
> > non-Google organizations if possible.
We generally follow the rules:
Release during
> >  Mon - Thu, 9:00 - 14:00 US PT
Avoid holidays
All regular
> >  US holidays
Large
> >  international
> >  ones if possible
All the various
> >  New Years
Jan 1 in
> > Gregorian calendar is the most obvious
Check for large Lunar New Years too
Follow the
> > normal Google production freeze schedule
Launcher
> >  script
The main repo
> > script serves as a standalone program and is often referred to as the “launcher script”. This makes it easy to copy around and install as you
> >  don't have to install any other files from the git repo.
Whenever major changes
> > are made to the launche
> > r script, you should increment the VERSION variable in the launcher itself. At runtime, repo will check this to see if it needs to be updated
> > and notify the user automatically
> > Key management
Every
> >  release has a git tag
> >  that is signed with a key that repo recognizes. Those keys are hardcoded inside of the repo launcher itself -- look for the KEYRING_VERSION and MAINTAINER_KEYS settings
> > Adding new keys to the repo
> > launcher will allow
> >  tags to be recognized by
> >  new keys, but only people using that updated version will be able to. Since the majority of users will be using a
n official launcher
> >  version, their version
> > will simply ignore any new signed tags.
If you want to add new keys, it's best to register
> >  them long ahead of time, and then wait for that updated launcher to make its way out to everyone. Even then, there will be a long tail of users
> > with outdated launchers, so be prepared for people asking questions.

Registering a new key
The process of actually adding a new key is quite simple.

Add the public half of the key to MAINTAINER_KEYS.
Increment KEYRING_VERSION so repo knows it needs to update.
Wait a long time after that version is in a release (~months) before trying to create a new release using those new keys.
Self update algorithm
When creating a new repo checkout with repo init, there are a few options that control how repo finds updates:

--repo-url: This tells repo where to clone the full repo project itself. It defaults to the official project (REPO_URL in the launcher script).
--repo-rev: This tells repo which branch to use for the full project. It defaults to the stable branch (REPO_REV in the launcher script).
Whenever repo sync is run, repo will, once every 24 hours, see if an update is available. It fetches the latest repo-rev from the repo-url. Then it verifies that the latest commit in the branch has a valid signed tag using git tag -v (which uses gpg). If the tag is valid, then repo will update its internal checkout to it.

If the latest commit doesn't have a signed tag, repo will fall back to the most recent tag it can find (via git describe). If that tag is valid, then repo will warn and use that commit instead.

If that tag cannot be verified, it gives up and forces the user to resolve.

Force an update
The repo selfupdate command can be used to force an immediate update. It is not subject to the 24 hour limitation.

Branch management
All development happens on the main branch and should generally be stable.

Since the repo launcher defaults to tracking the stable branch, it is not normally updated until a new release is available. If something goes wrong with a new release, an older release can be force pushed and clients will automatically downgrade.

The maint branch is used to track the previous major release of repo. It is not normally meant to be used by people as stable should be good enough. Once a new major release is pushed to the stable branch, then the previous major release can be pushed to maint. For example, when stable moves from v1.10.x to v1.11.x, then the maint branch will be updated from v1.9.x to v1.10.x.

We don‘t have parallel release branches/series. Typically all tags are made against the main branch and then pushed to the stable branch to make it available to the rest of the world. Since repo doesn’t typically see a lot of changes, this tends to be OK.

Creating a new release
When you want to create a new release, you‘ll need to select a good version and create a signed tag using a key registered in repo itself. Typically we just tag the latest version of the main branch. The tag could be pushed now, but it won’t be used by clients normally (since the default repo-rev setting is stable). This would allow some early testing on systems who explicitly select main.

Creating a signed tag
Lets assume your keys live in a dedicated directory, e.g. ~/.gnupg/repo/.

If you need access to the official keys, check out the internal documentation at go/repo-release. Note that only official maintainers of repo will have access as it describes internal processes for accessing the restricted keys.
# Pick the new version.
$ t=v2.30

# Create a new signed tag with the current HEAD.
$ ./release/sign-tag.py $t

# Verify the signed tag.
$ git show $t
Push the new release
Once you're ready to make the release available to everyone, push it to the stable branch.

Make sure you never push the tag itself to the stable branch! Only push the commit -- note the use of ^0 below.

$ git push https://gerrit-review.googlesource.com/git-repo $t
$ git push https://gerrit-review.googlesource.com/git-repo $t^0:stable
If something goes horribly wrong, you can force push the previous version to the stable branch and people should automatically recover. Again, make sure you never push the tag itself!

$ oldrev="whatever-old-commit"
$ git push https://gerrit-review.googlesource.com/git-repo $oldrev:stable --force
Announce the release
Once you do push a new release to stable, make sure to announce it on the repo-discuss@googlegroups.com group. Here is an example announcement.

You can create a short changelog using the command:

# If you haven't pushed to the stable branch yet, you can use origin/stable.
# If you have pushed, change origin/stable to the previous release tag.
# This assumes "main" is the current tagged release.  If it's newer, change it
# to the current release tag too.
$ git log --format="%h (%aN) %s" --no-merges origin/stable..main
Project References
Here's a table showing the relationship of major tools, their EOL dates, and their status in Ubuntu & Debian. Those distros tend to be good indicators of how long we need to support things.

Things in bold indicate stuff to take note of, but does not guarantee that we still support them. Things in italics are things we used to care about but probably don't anymore.

These are helper 
tools for 
managing official releases.
See the above more( located on ^)
[release process](/docs/release-process.md)
document for more details.
