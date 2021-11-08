Was doing a test with `git rebase --interactive` since meet a problem in other project

The setup is following:
1. We have branch master
2. And also branch dev
3. Code being pushed to dev
4. Then it's merged to master, no fast forward (merge commits are in place)
5. master have initial commit m1
6. dev have commits d1,d2,d3,d4. Merged after d1, then after d3, then after d4.

Now the problem:
1. We realise there're multiple commit messages mistakes (on both, dev and master since merged)
2. Have to edit all d* messages to d*r (d1 - d1r, d2-d2r, etc.)
3. We want to fix commit messages and don't lose history (dependencies) to be able to look at graph
4. Maintaining profile activity is also important :p 

What we gonna do:
1. Rebase master branch peristing merge commits and rewording mistakes
2. Force push master
3. Fill dev branch with master branch content, then rebase to clear from merge commits
4. Force push dev

Turns out, to persist merge messages you should run
```git rebase -i --root --rebase-merges```
*root is just to start from initial commit

So, running rebase on master with following content
```
# rebasing master

# give name "onto" to current HEAD, nothing in rebased branch yet 
label onto 

# prepare branch to receive first initial commit
reset [new root]

# Branch MASTER
pick 7b47d8a m1
break
# give name "master" to current HEAD position | later we will merge to (after) THIS position
label master

# Branch DEV
# pick commit
reword 3bfef09 d1
break
# give name "dev" to current HEAD position | HERE we set entry into dev branch, splitting from master
label dev

# move HEAD pointer to "master" branch
reset master
# merge dev branch content
merge -C 5827909 dev # Merge branch 'dev'
break
# move pointer of "master" branch to created MERGE commit | later we will merge to (after) THIS position
label master

# Branch DEV
# move HEAD pointer back to "dev" branch since it was on "master"
reset dev
# pick next 2 commits
reword 0154055 d2
break
reword 7b0926a d3
break
# and move HEAD of "dev" branch to new position
label dev

# move HEAD pointer to "master" position
reset master
# merge dev branch content
merge -C a84881f dev # Merge branch 'dev'
break
# move pointer of "master" branch to created MERGE commit | later we will merge to (after) THIS position
label master

# Branch DEV
# move HEAD pointer back to "dev" branch since it was on "master"
reset dev
# pick next commit
reword c2e8858 d4
break
# and move HEAD of "dev" branch to new position
label dev

# move HEAD pointer to "master" position
reset master
# merge dev branch content
merge -C bc5faa8 dev # Merge branch 'dev'
break
```

Notice those break words. After each commit (pick,merge,reword...) it will bring us to console.

Here we need to run following command (I'm doing it on 8 november, and this is test project)
```GIT_COMMITTER_DATE="Mon Nov 8 16:54:40 2021 +0300" git commit --amend --date="Mon Nov 8 16:54:40 2021 +0300" --no-edit```

What we should do is to replace date and time according to historical order we want to maintain. In this case we can commit everything in 1 day, so we will just increase time by 1 second each time it asks for an action.

After running the command, use `git rebase --continue` to continue

---

When done here, force push to `master`

After that, remove `dev` branch and create new one with `master`'s content

Now rebase `dev` branch, but without merges
```git rebase -i --root```
And force push it too

---

In the end we have all needed commit messages edited along with merge messages, so graph shows right hierarchy and history is in the order!

## Before:

![graph1](https://github.com/SanariSan/rebase-mistakes/blob/master/assets/g1.png?raw=true)
![master-1](https://github.com/SanariSan/rebase-mistakes/blob/master/assets/cm1.png?raw=true)
![dev-1](https://github.com/SanariSan/rebase-mistakes/blob/master/assets/cd1.png?raw=true)


## After:

![graph2](https://github.com/SanariSan/rebase-mistakes/blob/master/assets/g2.png?raw=true)
![master-2](https://github.com/SanariSan/rebase-mistakes/blob/master/assets/cm2.png?raw=true)
![dev-2](https://github.com/SanariSan/rebase-mistakes/blob/master/assets/cd2.png?raw=true)

---

**Method has its' disadvantage - need to type percise date for every action**

**After all, if you decided to maintain both graph and history along with keeping profile Activity, I guess you really need it, just like me, so good luck!**

---