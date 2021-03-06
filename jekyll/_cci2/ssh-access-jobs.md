---
layout: classic-docs
title: "Debugging Jobs Over SSH"
short-title: "Debugging Jobs Over SSH"
description: "How to access a build container using SSH on CircleCI 2.0"
categories: [troubleshooting]
order: 20
---

*[Basics]({{ site.baseurl }}/2.0/basics/) > Debugging Jobs Over SSH*

This document describes how to access a build container using SSH on CircleCI 2.0 in the following sections:

* TOC
{:toc}

## Overview
Often the best way to troubleshoot problems is to SSH into a build container and inspect 
things like log files, running processes, and directory paths.

CircleCI 2.0 gives you the option to access all jobs via SSH.

1. To start a build with SSH enabled, select the 'Rebuild with SSH' option from
the 'Rebuild' dropdown menu:
![Rebuild with SSH](  {{ site.baseurl }}/assets/img/docs/rebuild-ssh-dropdown.png)

2. To see the connection details, expand the 'Enable SSH' section in the build output where you will see the SSH command needed to connect:
![SSH connection details](https://circleci-discourse.s3.amazonaws.com/optimized/2X/5/57f50e26ec245d0373c4265ec4375641553bdbdb_1_690x295.png)	
![SSH connection details](https://circleci-discourse.s3.amazonaws.com/optimized/2X/5/514e8aec3e8017dac8e8d401d22432026b473161_1_690x281.png)

     The details are displayed again in the 'Wait for SSH' section at the end of the job.

3. SSH to the running build (using the same SSH key
that you use for GitHub or Bitbucket) to perform whatever troubleshooting
you need to.

The build VM will remain available for **30 minutes after the build finishes running**
and then automatically shut down. (Or you can cancel it.)

**Note**: If your build has parallel steps, CircleCI launches more than one VM
to perform them. Thus, you'll see more than one 'Enable SSH' and
'Wait for SSH' section in the build output.

## Debugging: "Permission denied (publickey)"

If you run into permission troubles trying to SSH to your build, try
these things:

### Ensure Authentication With GitHub/Bitbucket

A single command can be used to test that your keys are set up as expected. For 
GitHub run:

```
$ ssh git@github.com
```

or for Bitbucket run:

```
ssh -Tv git@bitbucket.org
```

and you should see:

```
Hi :username! You've successfully authenticated...
```

for GitHub or for Bitbucket:

```
logged in as :username.
```

If you _don't_ see output like that, you need to start by
[troubleshooting your SSH keys with GitHub](https://help.github.com/articles/error-permission-denied-publickey)/
[troubleshooting your SSH keys with Bitbucket](https://confluence.atlassian.com/bitbucket/troubleshoot-ssh-issues-271943403.html).

### Ensure Authenticating as the Correct User

If you have multiple accounts, double-check that you are
authenticated as the right one!

In order to SSH into a CircleCI build, the username must be one which has
access to the project being built!

If you're authenticating as the wrong user, you can probably resolve this
by offering a different SSH key with `ssh -i`. See the next section if
you need a hand figuring out which key is being offered.

### Ensure the Correct Key is Offered to CircleCI

If you've verified that you can authenticate as the correct
user, but you're still getting "Permission denied" from CircleCI, you
may be offering the wrong credentials to us. (This can happen for
several reasons, depending on your SSH configuration.)

Figure out which key is being offered to GitHub that authenticates you, by
running:

```
$ ssh -v git@github.com

# or

$ ssh -v git@bitbucket.com
```

In the output, look for a sequence like this:

```
debug1: Offering RSA public key: /Users/me/.ssh/id_rsa_github
<...>
debug1: Authentication succeeded (publickey).
```

This sequence indicates that the key /Users/me/.ssh/id_rsa_github is the one which
GitHub accepted.

Next, run the SSH command for your CircleCI build, but add the -v flag.
In the output, look for one or more lines like this:

```
debug1: Offering RSA public key: ...
```

Make sure that the key which GitHub accepted (in our
example, /Users/me/.ssh/id_rsa_github) was also offered to CircleCI.

If it was not offered, you can specify it via the `-i` command-line
argument to SSH. For example:

```
$ ssh -i /Users/me/.ssh/id_rsa_github -p 64784 ubuntu@54.224.97.243
```
