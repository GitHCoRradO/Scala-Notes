## Pro Git 2nd Edition

### Chapter 3 Git Branching
#### How Git stores data: Git doesn’t store data as a series of changesets or differences, but instead as a series of snapshots.
1. Each snapshot contains 3 parts: **blobs** (each representing the contents of the files), one **tree** that lists the contents of the directory and specifies which file names are stored as which blobs, and one **commit** with the pointer to that root tree and all the commit metadata.
#### What is a branch?
1. A branch in Git is simply a lightweight movable pointer to one of these commits. The default branch name in Git is ```master```. Every time you commit, the ```master``` branch pointer moves forward automatically.

   ``` 
   
                                    master
                                      |
                                      v
        -----         -----         -----
       |123ef| <---  |124ge| <---  |125tg|
        -----         -----         -----
   ```
2. To create a new branch: ```git branch <new branch name>```. This creates a new pointer to the same commit you’re currently on. ```git branch testing```

   ```
                                     HEAD
                                      |
                                      v
                                    master
                                      |
                                      v
        -----         -----         -----
       |123ef| <---  |124ge| <---  |125tg|
        -----         -----         -----
                                      ^
                                      |
                                   testing
   
   ```
3. How does Git know what branch you’re currently on? It keeps a special pointer called ```HEAD```. The git branch command only created a new branch — it didn’t switch to that branch. You can easily see this by running a simple ```git log``` command that shows you where the branch pointers are pointing. This option is called ```--decorate```.
4. To switch to an exsiting branch: ```git checkout testing```. This moves ```HEAD``` to point to the ```testing``` branch.

   ```
                                    master
                                      |
                                      v
        -----         -----         -----
       |123ef| <---  |124ge| <---  |125tg|
        -----         -----         -----
                                      ^
                                      |
                                   testing
                                      ^
                                      |
                                    HEAD
   
   ```
5. when you commit, you move the current branch pointer and HEAD forward, but not the other branches.
   ```
                                                  HEAD
                                                   |
                                                   v
                                                 master
                                                   |
                                                   v
        -----         -----         -----        -----
       |123ef| <---  |124ge| <---  |125tg| <--- |126rg|
        -----         -----         -----        -----
                                      ^
                                      |
                                   testing
   ```
6. switch to testing branch and commit:
   ```
                                                 master
                                                   |
                                                   v
        -----         -----         -----        -----
       |123ef| <---  |124ge| <---  |125tg| <--- |126rg|
        -----         -----         -----        -----
                                      ^
                                      |
                                    -----
                                   |127th| <-- testing <-- HEAD
                                    -----
   ```
7. You can also see this easily with the ```git log``` command. If you run ```git log --oneline --decorate --graph --all``` it will print out the history of your commits, showing where your branch pointers are and how your history has diverged.
   ``` 
   
   * 127th (HEAD, testing) testing's commit.
   | * 126rg (master) master's commit.
   |/
   * 125tg 3rd commit.
   * 124ge 2nd commit.
   * 123ef 1st commit.
   ```
8. Creating a new branch and switching to it at the same time: ```git checkout -b <newbranchname>```
9. checkout to another branch changes the files to the state of that commit pointed to by that branch.
#### Basic branching and merging
1. we have 3 commits and master branch. now we create a new branch iss53, make a new commit. then we checkout to master again. create a new branch hotfix. make a new commit.
   ``` 
              master  hotfix <- HEAD
               |      |
               v      v
   C0 <- C1 <- C2 <- C4 
               ^
               |
               C3 <- iss53
   
   ```
2. we decide we want hotfix branch merged into master branch: ```git checkout master``` and ```git merge hotfix```. In this case, the master pointer simply moves forward; this is called a "fast-forward".
3. now hotfix is merged into master, hotfix is not needed. we delete it: ```git branch -d hotfix```
4. we switch to branch iss53 and make a new commit.
   ``` 
                    master
                      |
                      v
   C0 <- C1 <- C2 <- C4 
               ^
               |
               C3 <- C5 <- iss53 <- HEAD
   
   ```
5. we want the iss53 merged into master. we do this with:
   ``` 
   git checkout master
   git merge iss53
   ```
6. Git does a simple three-way merge, using the two snapshots(C4 and C5) pointed to by the branch tips and the common ancestor(C2)of the two. Instead of just moving the branch pointer forward, Git creates a new snapshot that results from this three-way merge and automatically creates a new commit(C6) that points to it. This is referred to as a **merge commit**, and is special in that it has more than one parent.
   ``` 
                           master <- HEAD
                            |
                            v
   C0 <- C1 <- C2 <- C4 <-- C6
               ^            |
               |            |
               C3 <- C5 <---
                     ^
                     |
                    iss53
   ```
7. sometimes branches going different ways may modify the same parts of the same files. In this case, git will not let you  merge such files. you have to solve the conflicts to enable the merge.
#### Branch management
1. ```git branch``` with no arguments gets you a simple listing of the current branches.
   ``` 
   $ git branch
   iss53
   * master
   testing
   ```
2. ```git branch -v``` to see the last commit on each branch.
   ``` 
   iss53 21343gre fix a bug.
   * master 5745gre1 init commit.
   testing 213rew43 testing somefeatures.
   ```
3. To see which branches are already merged into the branch you’re on, you can run ```git branch --merged```. Because you already merged in iss53 earlier, you see it in your list. Branches on this list without the * in front of them are generally fine to delete with git branch -d; you’ve already incorporated their work into another branch, so you’re not going to lose anything.
   ``` 
   iss53
   * master
   ```
4. To see all the branches that contain work you haven’t yet merged in, you can run ```git branch --no-merged```. This shows your other branch. Because it contains work that isn’t merged in yet, trying to delete it with git branch -d will fail.
   ``` 
   testing
   ```
#### Branching workflows
1. Git uses a simple three-way merge, merging from one branch into another multiple times over a long period is generally easy to do. Many Git developers have a workflow that embraces this approach, such as having only code that is entirely stable in their master branch — possibly only code that has been or will be released. They have another parallel branch named develop or next that they work from or use to test stability — it isn’t necessarily always stable, but whenever it gets to a stable state, it can be merged into master. It’s used to pull in topic branches (short-lived branches, like your earlier iss53 branch) when they’re ready, to make sure they pass all the tests and don’t introduce bugs.
#### Remote branches
1. Git’s clone command automatically names it origin for you, pulls down all its data, creates a pointer to where its master branch is, and names it origin/master locally.
2. Tracking branches: Checking out a local branch from a remote-tracking branch automatically creates what is called a “tracking branch” (and the branch it tracks is called an “upstream branch”). Tracking branches are local branches that have a direct relationship to a remote branch. If you’re on a tracking branch and type git pull, Git automatically knows which server to fetch from and which branch to merge in.
3. Git does not sync with remote branches automatically. we run ```git fetch <remote>```.
4. When you want to share a branch with the world, you need to push it up to a remote to which you have write access.
5. You can delete a remote branch using the --delete option to git push. If you want to delete your serverfix branch from the server, you run the following: ```git push origin --delete serverfix```
#### Rebasing
1. Instead of merging to integrate updates, we can use ```rebase``` to achieve the same result.
   ``` 
                master                                          master
                |                                                 |
                v                merge                            v 
   C0 <- C1 <- C2              ==========>   C0 <- C1 <- C2 <---- C4
          ^                                         ^              |
          |                                         |              |
          C3 <-experiment                           C3 <-----------
                                                    ^
                                                    |
                                                  experiment
   ```
      ``` 
      $ git checkout experiment
      $ git rebase master
      ```
   ``` 
                master                                   master       
                |                                         |        
                v                rebase                   v         
   C0 <- C1 <- C2              ==========>   C0 <- C1 <- C2 <- C4'
          ^                                                    ^   
          |                                                    |   
          C3 <-experiment                                  experiment
   
   ```
2. rebase makes your commit history clean, but it's not without its drawbacks.
3. Merge vs Rebase: In general the way to get the best of both worlds is to rebase local changes you’ve made but haven’t shared yet before you push them in order to clean up your story, but never rebase anything you’ve pushed somewhere.

### Chapter 7 Git Tools
#### Rewriting history
1. ```git commit --amend``` to edit the commit message as well as change files.
2. if your amendments are suitably trivial (fixing a silly typo or adding a file you forgot to stage) such that the earlier commit message is just fine, you can simply make the changes, stage them, and avoid the unnecessary editor session entirely with: ```git commit --amend --no-edit```
#### Changing muliple commits messages
1. ```git rebase -i HEAD~3```
2. you can reorder the commits and edit commit messages. when in git rebase interactive mode.
3. Squashing commits, to squash the last 3 commits into 1 commit, ```pick``` to last commit and ```squash``` the newer 2 commits.
   ``` 
   $ git rebase -i HEAD~3
   pick f7f3f6d changed my name a bit
   squash 310154e updated README formatting and added blame
   squash a5f4a0d added cat-file
   ```
#### Nuclear Option ```filter-branch```
1. you can remove a file from every commit, make a subdirectory the new root, change email addresses globally.
