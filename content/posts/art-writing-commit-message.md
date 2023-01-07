---
title: "The Art of Writing Good Commit Messages"
author: "Guillaume Legoy"
date: "2020-05-14"
---

{{< toc >}}

## Good Commit Messages Makes Us More Efficient Communicators


As data professionnals we spend a good chunck of our time communicating with others. We share analysis results and insights, we give presentations about our latest models performance, we build stunning visualizations. In order to do so we carefully craft automated emails and pixel-perfect powerpoint presentation, and spend hours picking the right color palettes for our graphs. However when it comes to commit messages in version control systems like Github, all hell breaks loose and our commit history looks like this (in descending order):

```
6. I give up.
5. Arrrrrrrrrrrgh
4. pls work.
3. fix stuff?
2. Remove Another Typo
1. Remove typo in login function causing tests to crash.
```

I'm barely caricaturing here, and I'm the first one to disregard commit messages as unimportant at times. But this is wrong. Commit messages are another form of communication between memebers of the same team. Carefully written commit messages are a gift we send to our future selves and other developers of our project. And while this subject has been covered many times in various blog posts all over the internet (links at the end), I only recently discovered it. So even though I'm late to the party, I feel it's a good habit to take and share with you. Before we discover together how to write proper commit messages, let's highlight their benefits:

1. They communicate context. Commit messages should hightlight in a very concise way **why** we coded a change.
2. They bring git commands to life. [`git log`](https://git-scm.com/docs/git-log) commands suddenly becomes more useful and our past commits starts telling a story. 
3. They make us more disciplined and systematic data scientists / analysts. When we get into the habit of writing proper commit messages, I believe we become more careful with our changes, like for instance appropriately separating commits that don't belong with each others.


Now let's see how we can craft clean commit messages.


## Make Your Commit Messages Syntax and Content Consistent

Many great articles from renowned software engineers highlights sets of rules to follow for proper commit messages etiquette (see links at the end of this post). While these rules may sometimes vary and everyone is free to create their own commit experience, some seem to be accepted as best practices:

1. Subject line should be within 50 characters (IDE like VS Code send a warning when the limit is crossed).
2. Insert a blank line between the commit subject and body (if there is a body).
3. Capitalize the first word of the subject line and don't add a period at the end.
4. Use the imperative mode (`Fix a typo` rather than `Fixing a typo`)
5. Put references to issue trackers like Jira at the end (I've also seen it as part of the commit subject, like this: `TASK-123 - Add awesome new feature that will make us rich`).


Using these guidelines we can now craft consistent commit messages to harvest the benefits we highlighted earlier. The last step is to create a template that we can then use for every commit messages.


## Create a git Commit Template


Git offers the option to create a custom git commit message that will appear every time we call `git commit` and help us remain consistent. Without further ado the following are the steps to create our template:

1. Create and open a template config file called .git-commit-template in our home folder

```bash
touch ~/.git-commit-template
vim ~/.git-commit-template
```

2. In the file we just created insert our template. Here is the one I use but feel free to adapt it to your taste:

```
# The goal of this template and the following guidelines
# is to make our commit history as clean, simple, readable,
# and easy to access as possible. In order to do so
# we decided to use the following structure:
#
# Subject line, summarize changes in around 50 characters or less
#
# (optional) A detailled explanation here. Let's keep it 
# concise at around 100 characters.
#
# (optional) This section explains the reason (the why) of our
# commit or what problem we solved. Detail any side effect
# or unintuitive consequence.
#
# Jira ticket(s) related to this issue are mentionned at the end:
#
# Resolves: TASK-123
# Relates to: TASK-456 
```

3. Tell git to use our commit message template

```bash
git config --global commit.template ~/.gitmessage
```

4. That's it we are done! If you enter `git commit` in the terminal in any project, the above message will appear and you can customize your commit message accordingly. However note it won't work if you use `git commit -m "Message here"`. Use this command for simpler one liner, but still remember to keep it around 50 characters, capital letter at the beginning, and no period.

Happy coding!


## Additional Resources

* https://chris.beams.io/posts/git-commit/#limit-50
* https://backlog.com/blog/git-commit-messages-bold-daring/
* https://git-scm.com/book/en/v2/Customizing-Git-Git-Configuration
* https://medium.com/@alex.wasik/create-a-custom-git-commit-template-84468232a459
