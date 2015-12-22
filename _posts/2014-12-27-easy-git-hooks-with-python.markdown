---
layout: post
title: Git Hooks with Python
date: '2014-12-27 03:14:30'
comments: true
---

[Git hooks](http://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks) have long provided the ability for you to validate commits, perform continuous integration, continuous deployment and any number of other arbitrary actions. 

Git hooks are often run as a bash script. For the hobby programmer or this may be a bit annoying to have to learn how to use Bash just to make sure each commit will always include a meaningful commit message linked with a JIRA message, or to take a restart a server after files have been updated. 

Thankfully it's practical and simple to make the git hook gracefully hand off the grunt work to a python script (or any other language for that matter). 

###Getting Started
Today we're going to delve into the client side git hooks. They come in several flavours all meant for hooking into different parts of the process. 

* `pre-commit` Runs before you've even entered your commit message. Here you can take a look at the current snapshot. You're able to check for all sorts of code requirements here such as code style, tests passing, builds running, etc. A non-zero return from a script would abort the commit.
* `prepare-commit-message` This will run before the user is asked for their commit message. It allows a hook to edit the default message to provide to the user. This would enable you to create a commit that would set the default message as something like the classes the classes that have been added. A non-zero return from a script would abort the commit. This hook is provided the following parameters:
 * Reference to file that holds the commit message so far. This can be edited to be changed to whatever you're after.
 * Type of commit (merge, fast forward, etc)
 * SHA-1 of the commit 
* `commit-message` Runs prior to a commit being completely verified but after the commit message has been created. This will run with a single parameter which is a file reference to the commit message. A non-zer exit value will abort the commit.
* `post-commit` Runs once the commit has been completely finalized. A non-zero return from this hook will not cause the commit to fail. This is mainly used for notification purposes for that reason.

###Example - `commit-message` Hook

Unsurprisingly we have a small sidebar for the Windows users to get this working. As windows doesn't recognise the [shebang](http://en.wikipedia.org/wiki/Shebang_%28Unix%29) (#!) at the start of a file we will only have the option of writing the git hook in a way that `cmd.exe` will understand. So to get started we create this small file in your base directory of your poject.

**.git/hooks/commit-msg**

~~~
#!/bin/sh
python .git/hooks/commit-message.py
~~~
{: .language-bash}

This file can still be used on Linux/Mac (as to why we've put the shebang in the start still). This will then mean that before any commit is created and placed in the git log the `pre-commit-hook.py` file will run. The return code for the python script will then be used as the overall return code for the script.

In all further references to `.git/hooks/commit-msg` you can assume we're talking about `.git/hooks/commit-msg.py` if you're using Windows.

####Using Python
So lets open up our python file and create a small tool that will perform some basic action for us. 

**.git/hooks/commit-msg**

~~~
#!/usr/bin/env python

import sys

print "Starting commit-msg hook"

sys.exit(0)
~~~
{: .language-python}

This example will exit with a system code of 0. Zero indicates success, any non zero value will indicate that the commit is unsuccesful. When creating a commit now we will get the following output:

~~~
paul@laptop ~/gitProjects/easygitwithpython $ git add *
paul@laptop ~/gitProjects/easygitwithpython $ git commit -m "Hi there"
Starting pre-commit hook
[master b5254f4] Starting commit-msg hook
 1 file changed, 1 insertion(+), 1 deletion(-)
paul@laptop ~/gitProjects/easygitwithpython $ 
~~~
{: .language-shell}

###Getting more useful
We can extend our previous example to make use of the commit data. An example of this is making sure that a commit message is above a certain length and matches a regular expression for a JIRA issue.

**.git/hooks/commit-msg**

~~~
import sys, re

#Required parts 
requiredRegex = "[A-Z]{2,}-\\d+"
requiredLength = 15

#Get the commit file
commitMessageFile = open(sys.argv[1]) #The first argument is the file
commitMessage = commitMessageFile.read().strip()

if len(commitMessage) < requiredLength:
    print "Commit message is less than the required 15 characters."
    sys.exit(1)
    
if re.search(requiredRegex, commitMessage) is not None:
    print "A JIRA issue must be linked with a commit"
    sys.exit(1)

print "Commit message is validated"
sys.exit(0)
~~~
{: .language-python}

The prepare-commit-msg git hook will have a single argument, that is a file reference to the commit message that has been set. This hook runs before the commit has fully been validated and as such a non-zero return code will still cause the git commit to fail. 

You can see the following confirmation of this hook

~~~
paul@laptop ~/gitProjects/easygitwithpython $ git commit -m "short"
Commit message is less than the required 15 characters.
paul@laptop ~/gitProjects/easygitwithpython $ git commit -m "A long message, no JIRA issue"
A JIRA issue must be linked with a commit
paul@laptop ~/gitProjects/easygitwithpython $ git commit -m "A long message, with JIRA-123"
Commit message is validated
[master 0ecdd0d] A long message, with JIRA-123
 1 file changed, 1 insertion(+)
paul@laptop ~/gitProjects/easygitwithpython $ 
~~~ 
{: .language-shell}

Another windows caveat. When using commit hooks that require passing through the argument to the Python file you'll need to have your prepare-commit-msg to pass through those arguments manually.

**.git/hooks/commit-msg**

~~~
#!/bin/sh
python commit-msg.py $1
~~~
{: .language-bash}

This will pass through the first argument to Python and will get you to the same level that Unix systems are up to. 

### Using Server Side Hooks
One problem that comes with hooks is that you are not able to enforce them on any client who has cloned your repository. This may seem annoying but it definitely for good reason. If one was able to enforge git hooks on a repository then you'd be effectively forcing your users to run arbitrary code on their machines. 

For this reason it's often better to rely on push hooks. This will allow you to only run the code on a trusted place. So with that in mind, we can delve into our possible [server side git hooks](http://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks#Server-Side-Hooks). They are as follows.

 * **pre-receive** - Gets a list of references that are being changed. A non-zero return will cause all to be rejected.
 * **update** - Runs for each branch that is being pushed. A non-zero return will cause the branch individually to be rejected
 * **post-receive** - Runs once the commits have been succesfully pushed. Usually used for notifications or for deploying to a production site.
 
_We'll go further into server side hooks in a further post_
 
