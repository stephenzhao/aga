In the following answer, you will know how to extract a folder from a repository and make a git repository from it and then including it as a submodule instead of a folder.

Inspired from Gerg Bayer's article Moving Files from one Git Repository to Another, Preserving History

At the beginning, we have something like this:

<git repository A>
    someFolders
    someFiles
    someLib <-- we want this to be a new repo and a git submodule!
        some files
At the end, we will have something like this:

<git repository A>
    someFolders
    someFiles
    @submodule --> <git repository B>

<git repository B>
    someFolders
    someFiles
Create a new git repository from a folder in an other repository

Step 1

Get a fresh copy of the repository to split.

git clone <git repository A url>
cd <git repository A directory>
Step 2

The current folder will be the new repository so remove the current remote.

git remote rm origin
Step 3

Extract history of the desired folder and commit it

git filter-branch --subdirectory-filter <directory 1> -- --all
You should now have a git repository with the files from directory 1 in your repo's root with all related commit history.

Step 4

Create your online repository and push your new repository!

git remote add origin <git repository B url>
git push
You may need to set the upstream branch for your first push

git push --set-upstream origin master
Clean <git repository A> (optional, see comments)
We want to delete traces (files and commit history) of from so history for this folder is only there once.

This is based on Removing sensitive data from github.

Go to a new folder and

git clone <git repository A url>
cd <git repository A directory>
git filter-branch --force --index-filter 'git rm --cached --ignore-unmatch <directory 1> -r' --prune-empty --tag-name-filter cat -- --all
Replace by the folder you want to remove. -r will do it recursively inside the specified directory :). Now push to origin/master with --force

git push origin master --force
Boss Stage (See Note below)

Create a submodule from into

git submodule add <git repository B url>
git submodule update
git commit
Verify if everything worked as expected and push

git push origin master