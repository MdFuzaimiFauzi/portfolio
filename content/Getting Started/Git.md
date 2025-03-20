---
tags:
  - github
  - terminal
  - cmd
  - powershell
---

![[others/public/drawing/git_drawing.png]]


Git is a **version control** system that is used to track changes in code and collaborate efficiently.

# Git Clone the Repository

From what do I experienced, git clone can be from two types of cloning :-
1) Clone with **SSH**
2) Clone with **HTTP**

Both are same as the files that you will cloned are similar from both types of cloning.


Now, let's say we want to clone a repository from our **GitLab**, and here is the link in **HTTP** mode :

http://gitlab.stratusrd01.stratus.local:9131/RnD/im5-doc.git

1. Then, copy above link first.

Then open a desired folder, and inside the folder, right-click and choose **Open with Terminal**.

Then insert the code below :

```
git clone http://gitlab.stratusrd01.stratus.local:9131/RnD/im5-doc.git
```

The terminal will be updated with several lines in which it's processing the git clone request.

Here is the sample of the screenshot :

![Git Clone](others/public/images/gitclone.png)


When the git cloning request is done, it will appeared at the end of the line of the process like above.


---

## Checking the current Branch

It is super, super important to check in which branch you currently are.

In this sample, we are already in the im5-doc folder, right click to **Open the Terminal**.

In the correct directory, insert this code :


```
git branch -vv
```

As the result, the terminal will show in which branch you are currently are.
Example is in below screenshot :

![[others/public/images/gitbranch.png]]


From the screenshot above, it says we are currently at **update** branch.


---

## Create new Branch

To create the new branch and check out into the newly created branch, we can insert below code :

```
git checkout -b NewBranch
```

As from the sample of code, our new branch will be named as **NewBranch**.

---

## Switching between Branch

When it comes to git, most of the cases, we will be provided with multiple branches to be chosen of. 

In this case, we need to know on how to switch from the current branch into another branch. 

```
git checkout Main
```

In this code, we are switching into another branch named as **"Main"**.


---

## Git Add .

**Git Add .** is to add all changes in the working directory to the **staging area**.

```
git add .
```


---

## Git Commit

**Git Commit** is to record the changes to the local repository.

```
git commit -m "Added new features"
```

In this sample, we are also including the commit message using **-m**, 
and the message of this commit is **"Added new features"**.

---

## Git Push

Finally, to push all the changes we have been made into the remote branch, we are going to use git push.

```
git push
```









