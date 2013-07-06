---
layout: post
title: "Improve your worflow with Git Flow"
date: 2013-07-06 14:58
comments: true
author: cyrilf
categories: 
- git
---

You're using Git on a day-to-day basis, but you often feel lost or your branching model looks like a spaghetti bowl?
Let me introduce you a perfect tool to end your headaches. It's Git Flow, it will help you by keeping a perfect branching model and make your team happy.

<!-- more -->

{% img center /images/gitflow/spaghetti-head.jpg 350 350 'spaghetti bowl' 'spaghetti bowl' %}
<p class="text-center small-text">Don't cry little boy, Git Flow to the rescue, keep reading.</p>

### Git

Ok, so first I assume that if you still reading this, you know what is `git`. In a nutshell, `git` is a version controlled system which you can use to maintain code for your project. Through it the collaboration with your team and the versioning is made easier.

`git` contains an interesting feature called `branching`. It allows you to create different lines of development with independant history and evolution. You can see this as a copy of your working directory and you continue your work in this new copy.

> Got it, I use Git and I already know that, where is the problem?

At this point, there is not.
The problem occurs when you start having a lot (plenty?) of branches. You could easily get lost and don't know what is up to date, what should be merged, is the master branch synchronized with the release branch and many many more questions.

For instance, you find a bug on your production site. You want to fix this as quickly as possible. To do so, you modify the code on master branch and push it back onto production. This is great, but this fix is now lost only on the master branch and won't be replicated on the other branchs.

That's why [@nvie](http://nvie.com) published an article of a ["Successful Git branching model"](http://nvie.com/posts/a-successful-git-branching-model) which explain, for me, the best branching model we can use for both private and work projects.

### Git branching model

I truly suggest you to read his article. It worth it.

But the main idea to keep in mind, is that there is branches with specific roles and all of these stay consistent and perfectly synchronized.

In a few words, you have a master branch containing the production code, then you have a develop branch that allows you to create different features from it. When a feature is done, you merge it back onto the develop branch. When a set of features are closed, you can then create a release branch to be sure that everything is ok and finally, merge it into master.

If you really understand this model then everything will be easier and maintenable over the time on your projects.
The only problem remaining is that you can easily made mistakes by using `git` and these mistakes can break the worflow.

### Git Flow to the rescue!

[Git Flow](https://github.com/nvie/gitflow) is a tool created by [@nvie](http://nvie.com) (again!). It's a collection of Git extensions that help us to follow his model really easily.

#### Installation

You can find some instructions [here](https://github.com/nvie/gitflow/wiki/Installation) about how to install it on your plateform.

#### Init

Once the instalation is done you can apply the branching model to a new directory or convert an existing one by entering the following in your console:

    git flow init

Then, it will ask you some questions about your naming convention. I suggest you to keep the default one, so just press `Enter` on each questions. (To avoid this step, enter `git flow init -d` on the previous command, it will accept all defaults).

    No branches exist yet. Base branches must be created now.
    Branch name for production releases: [master] 
    Branch name for "next release" development: [develop] 

    How to name your supporting branch prefixes?
    Feature branches? [feature/] 
    Release branches? [release/] 
    Hotfix branches? [hotfix/] 
    Support branches? [support/] 
    Version tag prefix? [] 

Cool, Git Flow is now ready to use. As you can notice you're on the develop branch.
As it specified on the worflow, all your development should now start from this branch.
Let's start using Git Flow

#### Use cases

Let's say that you want to start a new feature, for instance a contact form:

    git flow feature start contactForm

Behind the scenes, Git Flow will create a new branch called `feature/contactForm` and set it as the current working branch. From there you can continue to work as usual. You code, commit, code, commit, take a tea, code, commit..
Once your job on this feature is done, just finish it.

    git flow feature finish contactForm

In the background, Git Flow just merged your changes back to develop and removed your feature branch.

You can do the same process with other kind of branches like: release, support or hotfix. The behavior won't be the same depending on which one you choose.

For instance, let's talk about hotfix. You can start it as you've done for a feature:

    git flow hotfix start 1.0.3

This time, it will create a branch based on master. Then you fix what’s wrong on your code. Once you've dealt with it:

    git flow hotfix finish 1.0.3

It will merge this hotfix into master AND into develop too. It also add a tag to your branch for an easier versioning of your project and it erase this hotfix branch.
As the merge is done on both master and develop you don't have to worry about your master being ahead of you develop branch.

{% img center /images/gitflow/aligned_pasta.jpg 350 350 'aligned pasta' 'aligned pasta' %}

To conclude, Git Flow is a really great tool to apply a consistent and robust branching model. And another good point, it's open source! You can find it [here](https://github.com/nvie/gitflow).

I hope you're going to add this tool to your worflow and appreciate it.