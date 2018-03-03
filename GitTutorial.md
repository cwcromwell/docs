
=Tutorial for Git First-Timers: Clone A Repo And Submit Edits to the WTD Guide

== 1. If you've never downloaded Git, get the latest version:
https://git-scm.com/downloads


== 2. Set up Git by telling it your name:
https://help.github.com/articles/setting-your-username-in-git/

== 3. And your email address:
https://help.github.com/articles/setting-your-commit-email-address-in-git/

== 4. Make a local folder and clone the repo

(HTTPS method is recommended for first-timers):
https://help.github.com/articles/which-remote-url-should-i-use/#cloning-with-https-urls-recommended

Be sure to clone from your GitHub, and not from the WTD repo -- you won't be able to push changes back upstream if you choose the wrong repo. Your username should be visible in the address, if you are using the HTTPS method.

(If you want to use the SSH method you have to create a key pair, which will take longer, but free you from having to type your password.)

== 5. Read the guide for contributors

== 6. Use the website to find a page you'd like to edit. (Let's start with small changes.)

== 7. Navigate through the folder structure in your local Git repo (the copy on your laptop) to find the file.

== 8. Open and edit the file, using a plain text editor.

If you don't have one that you like, you can download Atom, created by the folks at GitHub:
https://atom.io/

Pay special attention to the markup language. Git is written with Markdown:
https://guides.github.com/features/mastering-markdown/

== 9. Save the file.

== 10. Add the file:

````
git add <file name>
````

== 11. Check your status to make sure Git noticed:

````
git status
````

You should see the name of your edited file in red. That means it has been changed, but has not been committed. In this example, the file documentarians.rst has been edited and then added.

````
modified:   documentarians.rst
````

== 12. Go ahead and commit.

````
git commit -a
````

You have to enter comments for the commit, so your terminal will automatically open a text editing utility. You can probably save and close by typing CONTROL + O, followed by CONTROL + X.

== 13. Check your status again. It should have changed.

````
On branch master
Your branch is ahead of 'origin/master' by 1 commit.
  (use "git push" to publish your local commits)
````

Your local branch should be ahead by one commit, compared to the remote repo that you cloned.

== 14. You can push the changes up to your remote repo by doing a push:

````
git push origin master

````

If this succeeds, you should see some output that confirms success:


````
[master ceb7dbf] Changes made to documentarians.rst. This is a practice run of tutorial for WTD SF.
 1 file changed, 9 insertions(+), 9 deletions(-)
Clarences-MacBook-Pro:www Clarence$ git status
On branch master
Your branch is ahead of 'origin/master' by 1 commit.
  (use "git push" to publish your local commits)
nothing to commit, working tree clean
Clarences-MacBook-Pro:www Clarence$ git push origin master
Counting objects: 4, done.
Delta compression using up to 2 threads.
Compressing objects: 100% (4/4), done.
Writing objects: 100% (4/4), 625 bytes | 0 bytes/s, done.
Total 4 (delta 3), reused 0 (delta 0)
remote: Resolving deltas: 100% (3/3), completed with 3 local objects.
To https://github.com/cwcromwell/www.git
   5606fb6..ceb7dbf  master -> master

````

You're not done yet. The changes are in your GitHub repo, but you need to share them with the folks who own the original project.

== 15. Create a pull request.

a. Go to the original WTD repo (not yours) and push the button that says New Pull Request. (It's only on the main page.)

b. Click the blue link that says Compare Across Forks

c. Use the third pulldown window from the left to choose the name of your fork repo
You should see the comment you saved when you made the commit. And if you scroll down, you should be able to see a diff of the changes to the document. 

d. Remember to push the green button that says "Create Pull Request"

== 16. Your done. Time to celebrate. 

For more information on how to do this, see the GitHub documentation:
https://help.github.com/articles/creating-a-pull-request-from-a-fork/

== 17. Check out these resources to keep learning:
