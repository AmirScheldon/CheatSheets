cd # GIT COMMANDS:
1. **clone**
> bring a repository that is hosted sw like Github into a folder on your local machine.

` git clone respository address`

* ls = list everything in the directory(NOT HIDDEN FILES)
* ls -la = list everything in the directory(INCLUDING HIDDEN FILES)

2. **status**
> shows all of the files that were created, updated or deleted but not have not saved in commit yet. \
    1. modified files \
    2. untracked files

` git status `

3. **init**
> initialize files to push

` git init `

4. **add**
> tell git to which file/files to track

` git add `

=> dot(.) tells git to track all files(include utracked and modified files(could be updated)). \

` git add . `

=> ou can name the specific file that you want to add or update.

` git add FILENAME `

5. **commit**
> save files in git

` git commit -m 'Title message' -m 'Description message' `

* -am: add and message at the same time(only works on modified files)

` git commt -am 'commit meessage'

6. **push**
> upload git commits to a repo like github

* **HOW TO GET SSH-KEY?**

 ` ssh-keygen -t rsa -b 4096 -C "github e-mail address" `

 above command gives two keys: \
    1. key(private)\
    2. key.pub

* print key.pub:

` cat key.pub`
 
copy the SSH-KEY and paste it github/settings/SSH and CPC keys/SSH keys.

` git push origin master `

=> use '-u' to set a default

` git push -u origin branch-name `

7. **pull**
> Download changes from remote repo to your local machine, the opposite of push

` git pull `

8. **reset**
> undo modified stages

` git reset `

can use name of file you want to be unstaged

` git rest FILENAME `

use HEAD to to jumback into earlier commites.\
use HEAD~1 to jumpback into last commit(unstage).

` git reset HEAD~1 `

use HEAD commit-code to jumpback to specific commit.

` git reset HEAD commit-code `

to unstage and completely remove commites

` git reset --hard   commit-code `

9. **log**
> see all commites you have done till now

` git log `


10. **checkout**

 Undo all changes made to the file since the last commit

` git chechout -- FILENAME`

switch between branches

` git checkout  branch-name `

11. **rm**
> delete file from git and system.

` git rm FILENAME`
* **GIT WORKFLOW**
> code -> git add -> git commit -> git push

# GIT BRANCHING
master branch : main or default branch in our repository

show all branches

` git branch `

create new branch

` git checout -b branch-name `

delete a branch

` git branch -d branch-name `

switch between branches

` git checkout  branch-name `

* every commit you do in a branch just applies in that branch untill you merge branch with master branch

shows diffrent between two versions

` git diff branch-name `

* **TWO WAYS TO MERGE**
1. commandbase way:

` git merge branch-name `

2. github interface:

first, push changes into github

` git push  origin branch-name`

second, send a pull request(PR)
> pull request: to pull your code into another branch

# GIT MERGE CONFLICT
> write code in the same line the branch that you want to merge into

=> use vs code interface 

# FORKING
> create a complete copy of a repo

# .gitigonre
> it makes git to ignore some files/dirs in project.

create .gitignore file in your project dir
` touch .gitignore`
add file to ignore to .gitignore => filename
add dir to ignore to .gitignore => /dirname

# origin
> When you clone a project from the internet, the main workflow is named 'origin'. To push code to your GitHub correctly, use this code.

`git push origin master`

> If your code on GitHub has been changed, you can pull it to your 'master' branch using this code.

` git pull master origin`




