

#### How to Fetch Remote Branches in Git
When you clone a repository, you can access all its remote branches.

The git fetch command goes out to your remote project and pulls down all the data from that remote project that you don’t have yet.

If you want to fetch remote branches and merge them with your work or modify your current work, you can use the git pull command.
``````sh
git branch -r

git fetch
git fetch origin "remote repo branch"

git pull --all

``````
##### Git add and commit

``````sh
git add .
git commit -m "message"
git commit -am "Your commit message"

``````
##### Branch in Git
A branch in Git is a separate, safe, and isolated area of development that diverges from the main project.

Branches allow developers to work on and test new features, fix bugs, experiment with new ideas and reduce the risk of breaking the stable code in the codebase.

##### Local Branch
A local branch exists only on your local machine. All the changes you introduce and commit to your local repository are stored only on your local system.

##### Remote Branch
A remote branch exists in a remote repository (most commonly referred to as origin by convention) and is hosted on a platform such as GitHub.

``````sh
git branch -a  # View all branch
git branch <branch-name>  # Local branch create
git branch test

git checkout BRANCH-NAME
git checkout test  # Navigate to a new branch

``````
Once you have committed the changes to your local branch you can push the local branch to the remote repository (origin) using the git push <remote> <local> syntax.

``````sh
git push -u origin test

``````
#### Create  new Branch
To create a new branch, run the command:

``````sh
# Two-step method
git branch NEW-BRANCH-NAME
git checkout NEW-BRANCH-NAME

# Shortcut
git checkout -b NEW-BRANCH-NAME

``````
#### Rename a Branch
To rename a branch, run the command:

``````sh
git branch -m OLD-BRANCH-NAME NEW-BRANCH-NAME
# Alternative
git branch --move OLD-BRANCH-NAME NEW-BRANCH-NAME

``````
#### Delete a Branch
To delete a branch, run the command:

``````sh
git branch -d BRANCH-TO-DELETE

# Alternative:
git branch --delete BRANCH-TO-DELETE

``````
#### Compare branches
You can compare branches with the git diff command

``````sh
git diff FIRST-BRANCH..SECOND-BRANCH

``````

##### How to Check Out the Remote Branch
if you wanted a copy of the remote branch new-feature, you would do the following:

``````sh
git checkout -b <new-local-branch-name> origin/<remote-branch-name>

git checkout -b new-feature origin/new-feature
``````
##### Set Upstream for Remote Branch
When you push to a remote and you use the --set-upstream flag git sets the branch you are pushing to as the remote tracking branch of the branch you are pushing.

``````sh
git branch --set-upstream-to <remote-branch>
git push -u origin local-branch

``````

