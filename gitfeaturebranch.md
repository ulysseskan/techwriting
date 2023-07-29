# Using Git to Implement a New Feature/Change without Affecting the Main Branch

You just joined a team working on a project.  You want to create a new feature without affecting the main branch.  You can do this by creating a feature branch.  Feature branching has several advantages:

* It helps keep changes isolated
* It allows different team members to work on different features without affecting anyone else's work
* It allows you to do targeted testing on the feature branch without holding anyone else up in the event of a problem

## So how do you get started branching per feature?  
You can use [Git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git), as follows:

1. Clone the existing project's repository to a new directory:
   ```shell
   git clone /path/to/project mylocalproject
   ```
    (This command creates a new directory "mylocalproject" containing a clone of the main project.  This clone is on equal footing with the original project, and has its own copy of the original project's history.)

2. Create a new feature branch for the change and switch to that branch:
   ```shell
   git checkout -b feature
   ```
    (This both creates the new feature branch and switches to that branch in one command.  It's a bit faster than using two separate commands to do the same thing.)
   
3. Make your changes to any relevant files.  Then auto stage all modified files and commit with a message to the local repository:
   ```shell
   git commit -am "New feature implementing extra widget output"
   ```
    (**Note:** This is a bit faster than running `git add` beforehand, but take note: if you added any **new** files, you'll want to `git add` them prior to this step, as the auto stager won't stage brand new files automatically.  You can run `git status` to list newly added/changed/deleted files compared to the currently checked out commit.)
   
4. Double-check your remote repository URL:
   ```shell
   git remote -v
   ```
    (This shows you a list of existing remotes - both names and URLs.  Remotes are like nicknames for a repository, so you don't have to use the full URL every time you want to refer to a different repo.)
   
5. Send changes from your local feature branch to its remote counterpart:
   ```shell
   git push origin feature
   ```
    (This pushes the newly committed changes on your local computer to the remote branch named feature.)  (`origin` is the default name of the remote repository you cloned from)

Congratulations, you've just implemented the feature on your feature branch, and the main branch is not affected.  

## If you do want to merge your code into the main branch

After your code is reviewed, tested, and stable, if you do end up wanting to merge it into the main branch later on, you can create a merge request on the commits page.  Then request that your team lead (or an appropriate reviewer) review the code before they merge it into the main branch.

## References
* https://mirrors.edge.kernel.org/pub/software/scm/git/docs/gittutorial.html
* https://docs.gitlab.com/ee/gitlab-basics/feature_branch_workflow.html
