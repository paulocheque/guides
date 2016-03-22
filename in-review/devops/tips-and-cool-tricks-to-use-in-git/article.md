This entry is motivated by a tough week.

That's right, a week of challenges related to [scm][scmlink] (which is not the most relaxing thing in the world). A week full of failed attempts to [`cherry-pick`][cherrypick] and other Git things I probably would not use if the code was being managed "right".

Now, no longer weeping and trying to look on the positive side, I learned some cool tricks which can be useful to me in the future, and you :)

The idea here is not to teach Git nor present fundamentals. The idea is to show some tricks you'll want to have known before.

So here we go ...


### Alias (co > checkout)
You can create [alias] [alias] for any Git command.
For example, the following creates an alias **st** for the Git command **status**:

```shell
$ git config --global alias.st status 
```

Just replace the **alias** with the name that you want, followed by the original command.

It is better to type "git co" than "git checkout", right?
Just create the alias! I've seen people *(hint: myself)* using an alias of **g** for the command **git**. It is much laziness haha. With an alias **g** and **co**, the following command will achieve the same effect as `git checkout`:

```shell
$ g co
```

> It is a little different, but over time you earn enough in productivity.


### Autocomplete no shell

When using Git via command line, it is always useful to have autocompleted enabled. If you use the git shell by ruWindows, the autocomplete should already be set.

If the autocomplete is not enabled on your terminal, you can download the autocomplete script for Git from [Github][autocomplete].

After downloading, copy the file to the home directory, and add the line below to your _.bashrc_ file:

```shell
source ~/git-completion.bash
```

Ready. Now when you enter a git command and press *tab*, it should display all Git commands which start with _co_:

```shell
$ git co<tab>
```

```shell
commit config
```

### Checkout and reset on files

You may have used [checkout][checkout] and [reset][reset] to work with branches and commits.

What you may not know is that you can also use the **checkout** and **reset** commands on files.

When we use `git reset` on a file in git, it will **update** the staging area, causing a particular file to go **back** to the state it was in a previous commit.

This means we can remove the file *"jedi.js"* from the staging area, and cause it to be equal to its version in *HEAD* with the command:

```shell
$ git reset HEAD jedi.js
```

The result of the operation is: the file is no longer in staging and has the same content as the latest commit.

![Reset File](https://raw.githubusercontent.com/andreybleme/andreybleme.github.io/master/assets/img/resetfile.png "git reset file")

Pretty cool!
Now let's take a look at **`checkout`**.

Good old `git checkout` is typically used to switch between branches. When the command is used with a file path, the specified file is reset to the same state as the branch (__HEAD__ in this case).

```shell
$ git checkout HEAD jedi.js
```

The above command restores the file **"jedi.html"** to the state of the file pointed to by *HEAD*, and it is removed from staging.

![Checkout File](https://raw.githubusercontent.com/andreybleme/andreybleme.github.io/master/assets/img/checkoutfile.png "git checkout file")


> Save it somewhere, really. You will need one day.


Thus, you can only restore the file to a version pointed to by another  branch (or the same). You can make changes to it, and add the modified file to staging with `git add .`.

### Stash, to be happy

The command `git stash`, picks **all** the changes in staging and save it in a separate place. Thus, clearing your staging area.

That's really cool, because that way you can save a set of changes you have made but do not yet want to commit.

```shell
$ git stash
```

In particular, I use more [stash][stash] when I need to `git pull` and want to avoid conflicts between my local changes and upstream. 

To restore your local changes back to staging, you need to _apply_ your stash. The following command pops the latest changes that were **stashed away**:

```shell
$ git stash apply
```

> You can have more than one stash. By default, they will **always** be applied in FILO order.


### Amend, to redeem

If, after having committed some changes, you remembered that you forgot to add a modification to a file (it happens), you can use the [* --amend *][amend] option.

The following demonstrates a use case:

![Commit amend](https://raw.githubusercontent.com/andreybleme/andreybleme.github.io/master/assets/img/amend.png "git commit amend")

Here the parameter * - no-edit * makes the amend without changing the commit message.

### Diffs between commits

To see the changes of the last commit, you can use:
```shell
$ git log --stat
```

This command will show the files and the number of lines added and removed by file in each commit.

To see what exactly was changed in a commit, use [`git diff`] [diff].

To see the difference between two commits using [sha][sha]s of commits in hand (0da94be and 59ff30c), use:

```
$ git diff 0da94be 59ff30c
```

If you use GitHub, you may better see the differences there.
[+ see how][githubdiff]

### That's it folks !

There are some other tricks and cool stuff that Git offers but to avoid the post from becoming too big, I chose the ones I liked.

Add comments for any feedback :)

Until next **o/**

[scmlink]: (https://en.wikipedia.org/wiki/Version_control)
[cherrypick]:(http://imasters.com.br/artigo/24442/desenvolvimento/dica-git-da-semana-cherry-picking/)
[alias]: (https://git-scm.com/book/tr/v2/Git-Basics-Git-Aliases)
[checkout]: (https://www.atlassian.com/git/tutorials/undoing-changes/git-checkout)
[autocomplete]: (https://github.com/git/git/blob/master/contrib/completion/git-completion.bash)
[reset]: (https://www.atlassian.com/git/tutorials/undoing-changes/git-checkout)
[stash]: (https://git-scm.com/book/pt-br/v1/Ferramentas-do-Git-Fazendo-Stash)
[amend]: (https://git-scm.com/book/pt-br/v1/Git-Essencial-Desfazendo-Coisas)
[diff]: (https://git-scm.com/docs/git-diff)
[sha]: (https://git-scm.com/book/en/v2/Git-Internals-Git-Objects)
[githubdiff]: (https://help.github.com/articles/comparing-commits-across-time/)