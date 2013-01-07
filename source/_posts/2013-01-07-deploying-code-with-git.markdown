---
layout: post
title: "Deploying code with Git"
date: 2013-01-07 11:35
comments: true
categories: [Git, Sysadmin]
keywords: [git, deploy, version control, git hooks, post-receive]
description: A couple of ways to deploy code via Git to a remote server.
---

This past year I finally committed to using Git. One of the benefits I was looking forward to was using it's distributed architecture for easy-peasy remote code deployment to multiple remote environments. It has indeed been a wonderful change which has made my life much easier, so I thought I would write up a quick blog post about it.

First let me just say that deploying with any kind of version control system (VCS) is a nice step up from deploying with an old school file transfer (usually over FTP). Here are couple of reasons why deploying with Git is nice:

 1. Fast, differential file transfer (only updates modified files)
 2. Easy to rollback to previous code
 3. Git "hook" scripts can run additional deployment tasks (see last Pro-Tip below)

There are a couple drawbacks to using pure Git to deploy however:

 1. Git does _not_ track file permissions, which could cause problems if some files or directories need special permissions
 2. Git does _not_ track empty directories. If you want it to, you have to use the hacky trick of placing an empty `.gitignore` in any empty directories you want to deploy
 3. No built in database deployment and migration tools

For many purposes however it is a nice, simple solution that falls between basic file transfers and more advanced deployment options (mentioned at the end).
<!-- more -->

##Strategy #1: SSH in to server, then `git pull`

This was the first and simplest thing I tried. If SSH is set up with a key so there's no need to enter a password, it's actually pretty quick. `cd` in to the repo and `git pull` to update your code - just like you would locally. You are already logged on to your server in case you need to do anything else too. Rolling back is as easy as `git checkout`.

But this is not as automagical as we can get with Git. You can do all this with SVN! Moving on...

##Strategy #2: Push to bare remote repo with post-receive hook

Now we are getting somewhere - pushing code without SSHing on to the server! This is one of the beautiful things about Git: it's distributed (decentralized), so you can set up a repository on another server and push changes to it, just like you push changes to your central code repository. Even better, you can set up multiple remote repos, so you can easily push code to testing and development environments as well.

We are going to do three things:

1.  __Set up a bare repo on the remote server in a secure directory (not the webroot)__ <br />A "bare" repo means that this repo does not have a checked out "head", and will only be used to push to (not be committed to).
2.  __Set up a post-receive hook on the remote server which will checkout the latest code__<br />The [Git hook](http://www.kernel.org/pub/software/scm/git/docs/githooks.html) will checkout the latest pushed code to the work tree which we define (the webroot). The reason we keep the git repo and the work tree (website files) separate is so our `.git` files are not accidentally made public on the server. The separation also makes it easier to do things to your code without messing with your repository (like make backups or symlinks, or delete it).
3.  __Set up a remote endpoint on the local machine so we can push to the new remote repo__ <br />Finally we are going to create a Git remote endpoint locally, so we can push code to the remote server.

Note: The following tutorial assumes you have your code already in a Git repo on your local machine, and you can SSH to your remote server.

#### On remote machine:

To get started, SSH on to the remote server you want to deploy to. Then set up a bare Git repo on the remote machine, like so:

    $ mkdir website.git
    $ cd website.git
    $ git init --bare

This creates a new empty repository in the `website.git` directory. Then create a post-receive hook by copying the sample one, and open it with your favorite text editor:

    $ cp hooks/post-receive.sample hooks/post-receive
    $ vim hooks/post-receive

Add the following line to the `post-receive` file (it should have `#!/bin/sh` at the top):

    GIT_WORK_TREE=/path/to/webroot/of/website git checkout -f

This file will run after the repo has received (_post_-receive, see?) new code from a `git push`. It will checkout the latest code that was pushed (`-f` forces it to, even if there are local changes) into the `GIT_WORK_TREE` we are declaring (which is our website webroot).

<aside>
### Strategy #2.b (slight tweak):

I found [another](http://someguyjeremy.com/blog/quick-and-dirty-git-deployment) [version](http://caiustheory.com/automatically-deploying-website-from-remote-git-repository) of this technique, which does things a little differently. It sets the work tree (your webroot) in the config file, instead of setting it in the hook script. And instead of using a bare repo, it sets the `denycurrentbranch = ignore` option, which basically overrides errors normally caused by pushing to a non-bare repo. I'm not positive why this is better, I'd love to hear if anyone knows!

So for this technique, after setting up your bare repo you just adjust the Git config like so:

    $ git config core.worktree /path/to/webroot/of/website
    $ git config core.bare false
    $ git config receive.denycurrentbranch ignore

Then your post-receive hook looks just like this (since the worktree is already set):

    git checkout -f
</aside>

Regardless which of the two techniques you use above, make sure your post-receive hook is executable so it will run properly:

    $ chmod +x hooks/post-receive

#### On local machine:

Now add a new remote to your local repo:

    $ git remote add deploy git@myserver.com:mywebsite.git

Now deploy your code with a push command! (This assumes you want to push the `master` branch to this remote).

    $ git push deploy +master:refs/heads/master

Now that the default head has been set you can push with the following, shorter command:

    $ git push deploy

<aside class="protip">
### Pro-Tip: `git-receive-pack: command not found` error

I got this error when I `git push`ed to my remote host? On my cheap shared host the $PATH for my user doesn't load for some reason, so it cannot find `git-receive-pack`. An easy way to fix this is to set the `uploadpack` and `receivepack` variables in the local Git config for your remote. Run these commands locally to set them (and change "deploy" to the name of your remote):

    $ git config remote.deploy.uploadpack /path/to/git-upload-pack
    $ git config remote.deploy.receivepack /path/to/git-receive-pack

You may need to to a `which git-receivepack` on your remote host to find the path you need to set. More info on [StackOverflow](http://stackoverflow.com/questions/225291/git-upload-pack-command-not-found-how-to-fix-this-correctly) and [this blog](http://www.twohard.com/blog/remedy-git-upload-pack-or-git-receive-pack-command-not-found-errors-when-you-have-limited-acces).
</aside>


## Strategy #3: Use other deployment software

Deploying with Git hooks is nice, and it's met my needs so far. But there are more powerful tools availible, like [Capistrano](https://github.com/capistrano/capistrano) and [Fabric](http://docs.fabfile.org). I have not used any of these tools, but it looks like they allow you do the same one-line deployment commands (which also pull from your git repo), as well as providing extra tasks to adjust file permissions, clear cache directories, make database changes, etc. If you have a complex deployment process you want to automate I would suggest looking at one of these.

<aside class="protip">
### Pro-Tip: Use your Git `post-receive` hook to run extra deployment tasks

What if you just need to run a few extra commands on your deploy? Or what if you already have a deployment script of some sort, but want to run it with an easy `push` command?

On one of my projects I have a large [Phing](http://www.phing.info/) script which runs a number of deployment tasks. I call it after I checkout the new code from Git in the `post-receive` hook, like so:

    GIT_WORK_TREE=/path/to/webroot/of/website git checkout -f
    phing -f /path/to/webroot/of/website/build.xml deploy-upgrade

You could simply add some additional shell commands, or call any other sort of additional deployment scripts the same way.
</aside>