
https://www.datacamp.com/cheat-sheet/git-cheat-sheet
https://www.golinuxcloud.com/git-set-upstream/

###### Git and GitHub
Repository: Project, or the place or folder where the project is kept e.g git repository
Git != Github
Git: Tool that tracks the changes in the code over time such as local repository.
GitHub: Website for online hosting of repositories e.g remote repository.

In git, a snapshot is a commit. A commit object includes a pointer to the main tree (the root directory), as well as other meta-data such as the committer, a commit message and the commit time

A git repository has two main components:
A collection of objects — blobs, trees, and commits.
A system of naming those objects — called references.
One type of reference is branches. Internally, git calls branches by the name heads. So we will create a directory for them — .git\refs\heads.
So, we need to create the HEAD, which is just a file residing at .git\HEAD. We can apply the following:

On Windows: > echo ref: refs/heads/master > .git\HEAD

On UNIX: $ echo "ref: refs/heads/master" > .git/HEAD

##### Basic definitions
Local repo or repository: A local directory containing code and files for the project
Remote repository: An online version of the local repository hosted on services like GitHub, GitLab, and BitBucket
Cloning: The act of making a clone or copy of a repository in a new directory
Commit: A snapshot of the project you can come back to
Branch: A copy of the project used for working in an isolated environment without affecting the main project
Git merge: The process of combining two branches together.

##### More advanced definitions
.gitignore file: A file that lists other files you want git not to track (e.g. large data folders, private info, and any local files that shouldn’t be seen by the public.)
Staging area: a cache that holds changes you want to commit next.
Git stash: another type of cache that holds unwanted changes you may want to come back later
Commit ID or hash: a unique identifier for each commit, used for switching to different save points.
HEAD (always capitalized letters): a reference name for the latest commit, to save you having to type Commit IDs. HEAD~n syntax is used to refer to older commits (e.g. HEAD~2 refers to the second-to-last commit).

##### Git Init
This command is used to create a local repository. Initialize git tracking inside the current directory
``````sh
 git init Demo_Repo_Name

``````
##### Git config command
This command sets the author name and email address to be used with your commits.
``````sh
git config user.name [your-name]
git config user.email [your.email@domain.com]
 git config --global user.name "ImDwivedi1" 
 git config --global user.email "Himanshudubey481@gmail.com" 

``````

#### How to Pull and Fetch Remote Repositories to local Git repo
Git Fetch is the command that tells the local repository that there are changes available in the remote repository without bringing the changes into the local repository.
Git Pull on the other hand brings the copy of the remote directory changes into the local repository.
``````sh
git init
git add <Filename>
git commit -m <Commit Message>
git remote add origin <Link to your remote repository>

git fetch origin "remote repo branch"
git push origin <branch name>
git pull --all

``````
##### Git add and commit
Adds current file changes to staging area from your local working directory
Saves changes along with a custom message to your stating area
Instead, changes are first registered in something called the index, or the staging area.

``````sh
git add .
git commit -m "message"
git commit -am "Your commit message"
``````

##### Branch in Git
Branches are special “copies” of the code base which allow you to work on different parts of a project and new features in an isolated environment. Changes made to the files in a branch won’t affect the “main branch” which is the main project development channel.
In most repositories, the main line of development is done in a branch called master. This is just a name, and it’s created when we use git init, making it is widely used.
How does git know what branch we’re currently on? It keeps a special pointer called HEAD. Usually, HEAD points to a branch, which in turns points to a commit.

##### Local Branch - Git repository
A local branch exists only on your local machine. All the changes you introduce and commit to your local repository are stored only on your local system.

##### Remote Branch - GitHub repository
A remote branch exists in a remote repository (most commonly referred to as origin by convention) and is hosted on a platform such as GitHub.

###### Creating local repositories 
There are two primary methods of cloning a repository - HTTPS syntax and SSH syntax.
HTTPS cloning is much simpler and the recommended cloning option by GitHub.

Clone a repository from remote hosts (GitHub, GitLab, DagsHub, etc.)
Initialize git tracking inside the current directory
Clone only a specific branch
Cloning into a specified directory

``````sh
git clone <remote_repo_url>
git clone https://github.com/your_username/repo_name.git

git init
git init [dir_name]
git clone -branch <branch_name> <repo_url>
git clone <repo_url> <dir_name>

``````
##### Creating Branch and checkout
``````sh
git branch -a  # View all branch
git branch <new-branch>  # Local branch create
git branch test   # HEAD to point to test

git checkout BRANCH-NAME
git checkout test  # Navigate to a new branch
# equivalent of running git branch test to create the branch, and then git checkout test to move HEAD to point to the new branch
git checkout -b test # create a branch and checkout to the new branch

``````
###### Git Push
git push command uploads content from a local repository to a remote repository.

``````sh
git push -u origin test
git push -u origin master

``````
#### Create  new Branch
To create a new branch, run the command:

``````sh
# Two-step method
git branch NEW-BRANCH-NAME
git checkout NEW-BRANCH-NAME

# Shortcut to create a new branch and switch
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
##### Managing remote repositories and sync with local repo
Create a new connection called <remote> to a remote repository on servers like GitHub, GitLab, DagsHub, etc.
Remove a connection to a remote repo called <remote>
When you push to a remote and you use the --set-upstream flag git sets the branch you are pushing to as the remote tracking branch of the branch you are pushing.

###### Sync Git and Github
Connect the local and remote repo together. using : git remote add <remote> <url_to_remote>
``````sh

git remote  # list remote
git remote add <remote> <url_to_remote>
git remote add origin <url_to_remote>
git remote rm <remote>
git remote rename <old_name> <new_name>

----- # setting upstream
## Push an existing repository from command line
git init
git remote add <upstream name> <upstream URL>
git remote add origin https://DevOpsmyway@dev.azure.com/DevOpsmyway/WorkItemAgile/_git/WorkItemAgile
## set on forked repo
git push -u origin --all

--
git branch --set-upstream-to <remote-branch> 
git push -u origin local-branch

``````

##### Git Logs
This command is used to check the commit history.
``````sh
git log  
``````
##### Cherry-Pick command
Cherry-picking in Git means choosing a commit from one branch and applying it to another.
This contrasts with other ways such as merge and rebase which normally apply many commits to another branch.

``````sh
#Make sure you are on the branch you want to apply the commit to.
git checkout main

# Execute the following by taking the commit or hash id of feature branch FeatureA and put it commits to main branch
git cherry-pick <commit-hash_of_feature_branch_A_example>
git status
git log

# If you cherry-pick from a public branch, you should consider using
git cherry-pick -x <commit-hash>
``````
##### Git Squash
If you are working on a project and trying to implement a new feature, you might commit your code a few times to test things out. This lets you see how the code works or looks. you might want to combine all those commits into a single commit, This process is called commit sqaushing

``````sh

``````
##### Git Merge
Merging is the process of combining the recent changes from several branches into a single new commit that will be on all those branches.
merging is the complement of branching in version control: a Branch allows you to work simultaneously with others on a particular set of files, whereas a Merge allows you to later combine separate work on branches that diverged from a common ancestor commit.
Time to merge the new feature! That is, merge these two branches, main and new_feature. Or, in Git's lingo, merge new_feature into main

``````sh
# John created a branch:
git checkout -b john_branch
git add lucy_in_the_sky_with_diamonds.md
git commit -m "Commit 5"

# Paul had started from main and created his own branch:
git checkout main

git checkout -b paul_branch
git add penny_lane.md
git commit -m "Commit 6"

# where we have two different branches, branching out from main, with different histories.
#John is happy with his branch (that is, his song), so he decides to merge it into the main branch
git checkout main
git merge john_branch
git merge paul_branch

# generate the differences
git diff main paul_branch

``````
##### Git Lists in Staging Area and remove files
lists the (names of the) files that are in the index, with no regard for what files are in the HEAD commit or the work-tree
``````sh
git status
git ls-files
git rm
git rm --cached file.ext

``````
##### Git 

``````sh

``````