If like Stef you're using a local branch per GitHub issue, you might want the following features: 

- Make all commits in that branch include the GitHub issue name and description. 
- The ability to start a branch for a GitHub issue which would mark the issue as In Progress. 
- The ability to merge a branch for a GitHub issue which would mark the issue as Fixed. 

This can all be automated and we describe how to set this up. 

# Get a GitHub API token 

You need to run a little shell script to get an API token:

```shell
curl -u "YOUR_GITHUB_USERNAME:YOUR_GITHUB_PASSWORD" \
 --data-binary '{"scopes":["public_repo"]}' https://api.github.com/authorizations \
 | grep -E '^  "token": "'|sed 's/  "token": "\(.*\)",/\1/'
```

*First replace `YOUR_GITHUB_USERNAME` and `YOUR_GITHUB_PASSWORD` with your proper credentials*.

Now remember this token and your GitHub user in your general git settings:

```shell
git config --global github.token TOKEN
git config --global github.user YOUR_GITHUB_USERNAME
```

# Install the GitHub commands

Simply make symlinks to your checkout of the commands to a folder in your `$PATH`:

```shell
cp -s git-ghbranch git-ghfix ~/bin/
``` 

h1. Install the JIRA prepare-commit-msg hook 

You need to make a a symbol of the custom hook in each project you want to use it in:

```shell
cp -s prepare-commit-msg .git/hooks/
``` 

# Start having fun 

## Create an issue branch 

`$ git ghbranch 23` 

This will do the following: 

1. Create a branch called 23 
1. Switch to the new branch 
1. Mark the 23 issue as in progress
1. Make sure the 23 issue is open 
1. Assign yourself to the issue

## Work 

```shell
$ echo "new" > new-file 
$ git commit new-file 
```

Your message will start with those lines: 

```
# Fix for #23: Do tons of fixes 

# [github] 
# View this issue at https://github.com/FroMage/git-github/issues/23 
# ...The usual git commit message follows... 
```

Note: While it is slightly inconvenient to start the GitHyb line with a comment, it is necessary because it allows you to
cancel the commit by exiting without saving. If we had started the commit file with a non-commented line, git would still
commit the file if you left the commit message unedited because it would contain a non-commented line. This is not what 
users expect, so you have to manually uncomment the GitHub line on every commit if you want to keep it in the commit message, 
but this is quite acceptable.

## Customising the pre-filled commit message

You can override the start line of the commit messages with the @github.commit.template@ config:

```
# Run this INSIDE your project (or make it --global if you prefer) 
$ git config github.commit.template 'Fix for #%i: %t'
```

The default template is `Fix for #%i: %t` but you can set it to anything, and `%i` will be replaced with the issue key, 
and `%t` with the issue title.

## What happens if? 

If the hook fails to connect to GitHub, or the issue does not exist or if you forgot to configure the hook with `git config`, 
you will get a comment in the commit file warning you about it. 

If the branch name does not match the GitHub key grammar, we will not even try to look it up in GitHub, saving time 
for `master` commits. 

If the GitHub issue can be resolved from GitHub, it will be cached in `.git/gh.cache` so that future commits on 
this branch are faster. 

# Merge the branch back to master when you have fixed the issue 

Note: the branch name argument is optional, it will default to the current branch if missing.

`$ git ghfix 23` 

This will do the following: 

1. Switch to the branch called 23 
1. Rebase the branch on master 
1. Switch to the master branch 
1. Merge the 23 branch on master 
1. Close the 23 branch 
1. Mark the 23 issue as Fixed 
1. Remove the "in progress" tag on #23
