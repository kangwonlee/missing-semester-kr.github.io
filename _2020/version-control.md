---
layout: lecture
title: "Version Control (Git)"
date: 2019-01-22
ready: true
video:
  aspect: 56.25
  id: 2sjqTHE0zok
---

버전 컨트롤 시스템(VCSs)은 소스코드(또는 기타 파일이나 폴더들의 모음)의 변화를 추적하기 위해 사용되는 도구입니다.
이름에서 알수 있듯이 이 도구는 변경내역을 유지시켜주며, 나아가 협업을 용이하게 합니다.
VCSs는 일련의 스냅샷에서 폴더와 폴더안의 내용물들의 변화를 추적합니다. 각각의 스냅샷은 최상위 폴더 내의 파일 / 폴더의 모든 상태를 캡슐화 합니다.
VCSs는 누군가가 만들었던 각가의 스냅샷과 스냅샷에 연결된 메시지 등의 메타데이터를 유지합니다.

왜 버전 관리는 유용할까요? 혼자 작업을 하더라도 프로젝트의 지난 스냡샷을 보며 왜 변경사항 생겼는지를 기록하고, 병렬 개발 분기에서 작업을 하는 등 많은 일들을 할수 있게 해줍니다. 
다른 사람들과 일을 할때 버전관리는 다른 사람들의 변화를 볼수있는 귀중한 도구입니다. 게다가 동시 개발에서의 갈등을 해결해 줍니다.

현대 VCSs 는 아래와 같은 질문에 대한 답변을 쉽게 할수 있게 해줍니다. 

- 누가 이 모듈을 작정하였는가
- 언제 특정파일의 특정행이 수정되었는가. 누구에 의한 것인가? 왜 수정 되었는가?
- 지난 1000번이 넘는 수정들 중 언제 / 왜 특정 단위 태스트가 중지 되었었는가.


다른 VCSs가 존재하지만, **Git**은 사실상 버전관리 도구의 표준 입니다.
이 [XKCD comic](https://xkcd.com/1597/) 은 깃의 명성을 보여줍니다.

![xkcd 1597](https://imgs.xkcd.com/comics/git.png)

깃 인터페이스의 허술한 추상화로 인하여 하향식으로 방식으로 Git을 배우는것은(Git의 인터페이스/ 명령어 인터페이스 부터 시작하는것) 많은 혼란을 줄수 있있습니다.
하향식 학습은 몇가지 명령어를 외우고, 그것들을 마법주문처럼 생각하며, 문제가가 생길때마다 위의 만화에서의 접근방법을 따르게 할수 있습니다.

깃은 분명 투박한 인터페이스를 가지고 있지만, 그 이면의 디자인과 아이디어들은 아름답습니다.
나쁜 인터페이스는 암기 되어져야 하지만, 아름다운 디가인은 이해되어 집니다.
이러한 이유로 우리는 깃에 대한 데이터 모델부터 이후 명령어 인터페이스를 포함하는 상향식 설명을 제공합니다.
한번 데이터 모델을 이해하면 아래에 있는 데이터 모델을 잘 다루는 방법의 측면에서 명령어들을 더 잘 이해할수 있습니다.


# 깃의 데이터 모델

버전 관리를 위한 많은 필수적 접근법이 있습니다. 
갓은 기록 유지, 분기를 지원, 협업 기능 과 같이 버전과관리에 대한 모든 우수한 기능을 사용할수 있는 매우 정교한 모델을 가지고있습니다. 

## 스냅샷

깃은 최상위 디렉토리 내의 폴더와 파일 목록에대한 기록을 일련의 스냅샷으로 모델링 합니다.
깃에서는 파일은 "blob" 이라고 하며 그것은 단지 바이트 묶음임입니다
디렉토리는 "tree" 라고 하며 이름을 blob 또는 tree 에 매핑합니다(디렉토리는 다른 디렉토리들을 포함할수 있습니다.).
스냅샷은 추적중인 최상위 트리입니다. 
예를 들면 아래와 같은 트리를 가질수 있습니다. :

```
<root> (tree)
|
+- foo (tree)
|  |
|  + bar.txt (blob, contents = "hello world")
|
+- baz.txt (blob, contents = "git is wonderful")
```

최상위 트리에는 "foo" 트리("bar.txt" blob을 하나의 요로소로 포함한 자신)와 "baz.txt" blob 두 요소가 있습니다. 

## Modeling history: relating snapshots
어떻게 버전 제어 시스템을 스냅샵과 연관시켜야할까요? 간단한 모델 하나는 선형 히스토리를 갖는 것입니다. 
히스토리는 시간순으로 정렬된 스냅 샷의 목록입니다. 
여러 이유로 Git은 이와 같은 간단한 모델을 사용하지 않습니다.

Git에서 히스토리는 스냅 샷의 유향 비순환 그래프 (DAG)입니다. 
멋진 수학 용어처럼 들릴지 모르지만 겁 먹지 마세요. 
이 모든 것은 선행하는 "부모" 집합을 참조하는 Git의 각각의 스냅 샷을 뜻합니다.
두 개의 병렬 개발 분기를 결합 (merge)하는것처럼 하나의 스냅샹은 여러 부모로부터 내려올것이기 때문에
히스토리는 단일 부모 보다 부모 집합을 가집니다(선형 기록의 경우).

Git은 이러한 스냅 샷을 "커밋"이라고 부릅니다. 
커밋 기록을 시각화 해본다면 다음과 같습니다. :

```
o <-- o <-- o <-- o
            ^  
             \
              --- o <-- o
```

위의 ASCII문자로 만든 그림에서 `o`는 개별 커밋 (스냅 샷)에 해당합니다. 
화살표는 각 커밋의 부모를 가리 킵니다 ("이후"가 아니라 "이전" 관계). 
세 번째 커밋 후 히스토리는 두 개의 개별 분기로 분기됩니다. 
예를 들어, 서로 독립되어 병렬로 개발고있는 두 개의 개별 기능에 해당 될 수 있습니다. 
앞으로 이러한 분기를 병합하여 두 기능을 모두 통합하는 새 스냅 샷을 생성 할 수 있으며, 
만든어진 새로은 히스토리는 아래와 같습니다. 
새로 생성 된 merge 커밋은 굵게 굵은 글자로 보여집니다.:

<pre>
o <-- o <-- o <-- o <---- <strong>o</strong>
            ^            /
             \          v
              --- o <-- o
</pre>

Git의 커밋은 변경할 수 없습니다. 그렇다고 실수를 고칠 수 없다는 의미는 아닙니다. 
커밋 히스토리에 대한 "편집"은 실제로 완전히 새로운 커밋을 생성하고, 참조들(아래쪽을 보세요)이 
새 커밋을 가리 키도록 업데이트됩니다.

## Data model, as pseudocode

의사 코드로 작성된 Git의 데이터 모델을 보는 것은 유용할수 있습니다.

```
// a file is a bunch of bytes
type blob = array<byte>

// a directory contains named files and directories
type tree = map<string, tree | blob>

// a commit has parents, metadata, and the top-level tree
type commit = struct {
    parent: array<commit>
    author: string
    message: string
    snapshot: tree
}
```

깔끔하고 간결한 히스토리 모델입니다.

## Objects and content-addressing

object는 blob, tree, commit 입니다.

```
type object = blob | tree | commit
```
Git 데이터 저장소에서 모든 객체는 [SHA-1 해시](https://en.wikipedia.org/wiki/SHA-1)로 콘텐츠 주소가 지정됩니다.


```
objects = map<string, object>

def store(object):
    id = sha1(object)
    objects[id] = object

def load(id):
    return objects[id]
```

Blob, 트리 및 커밋은 모두 객체이며 이와 같은 방식으로 통합됩니다. 
다른 객체를 참조 할 때 디스크상의 표현에는 _포함_ 되지 않지만 해시에 의해 참조됩니다.

예를 들어 [위의](#snapshots) 디렉토리 구조의 트리는 다음과 같습니다. (`git cat-file -p 698281bc680d1995c5f4caaf3359721a5a58d48d`를 사용하여 시각화)

```
100644 blob 4448adbf7ecd394f42ae135bbeed9676e894af85    baz.txt
040000 tree c68d233a33c5c06e0340e4c224f0afca87c8ce87    foo
```

트리 자체에는 baz.txt(blob) 과 foo (트리)에 대한 포인터가 포함되어 있습니다 . 
해시에 의해 `git cat-file -p 4448adbf7ecd394f42ae135bbeed9676e894af85` 주소가 지정된 baz.txt 를 보면 다음과 같은 결과가 나타납니다.

```
git is wonderful
```

## References

이제 모든 스냅 샷은 SHA-1 해시로 식별 할 수 있습니다. 인간에게는 40 개의 16 진수 문자열을 잘 기억하지 못하기떄문에 불편합니다.

Git은 이문제를 해결하기 휘하여 SHA-1 해시에 "references" 라는 인간이 읽을수있는 이름을 사용합니다. 
References는 커밋에 대한 포인터입니다. 변경이 불가능한 객체와는 다르게 references는 변경 가능합니다 (새 커밋을 가리 키도록 업데이트 할 수 있음). 
예를 들어 'master' references는 일반적으로 주요 개발 분기의 최신 커밋을 가리 킵니다.

```
references = map<string, string>

def update_reference(name, id):
    references[name] = id

def read_reference(name):
    return references[name]

def load_reference(name_or_id):
    if name_or_id in references:
        return load(references[name_or_id])
    else:
        return load(name_or_id)
```

이를 통해 Git은 긴 16 진수 문자열 대신 "master"와 같이 사람이 읽을 수있는 이름을 사용하여 히스토리의 특정 스냅 샷을 참조 할 수 있습니다.

추가로 종종 히스토리 에서 "현재 위치"라는 개념을 웝합니다. 그래서 무엇과 연관 되었는지(우리가 어떻게 `parents` 커밋 필드들 설정하였는지) 알기 위하여 
새 스냅 샷을 남깁니다. 이떄 Git에서는 "현재 위치"는 "HEAD" 라는 특수 references를 사용합니다.
## Repositories

마침내, 우리는 Git 저장소가 `객체` 와 `references` 테이터 라는것을 (대략)정의 할 수 있습니다.

디스크에서 모든 Git 저장소는 객체와 references이며 이것은 Git의 데이터 모델의 전부입니다. 
모든 git명령은 객체를 추가하고 참조를 추가 / 수정 하는 커밋 DAG의 조작에 의해 매핑됩니다.

명령을 입력 할 때마다 명령어가 만들어내는 기본그래프 자료구조의 조작에 대해 생각해보세요.
반대로 커밋 DAG에 특정 종류의 변경을 시도하는 경우, 
예를 들어 "커밋되지 않은 변경 사항을 무시하고 'master' reference가 커밋`d83f9e`을 가르키도록 만들어 보세요"
이를 수행하는 명령은 있을 수 있습니다 (예 : git checkout master; git reset --hard 5d83f9e).


# Staging area

This is another concept that's orthogonal to the data model, but it's a part of
the interface to create commits.

One way you might imagine implementing snapshotting as described above is to have
a "create snapshot" command that creates a new snapshot based on the _current
state_ of the working directory. Some version control tools work like this, but
not Git. We want clean snapshots, and it might not always be ideal to make a
snapshot from the current state. For example, imagine a scenario where you've
implemented two separate features, and you want to create two separate commits,
where the first introduces the first feature, and the next introduces the
second feature. Or imagine a scenario where you have debugging print statements
added all over your code, along with a bugfix; you want to commit the bugfix
while discarding all the print statements.

Git accommodates such scenarios by allowing you to specify which modifications
should be included in the next snapshot through a mechanism called the "staging
area".

# Git command-line interface

To avoid duplicating information, we're not going to explain the commands below
in detail. See the highly recommended [Pro Git](https://git-scm.com/book/en/v2)
for more information, or watch the lecture video.

## Basics

{% comment %}

The `git init` command initializes a new Git repository, with repository
metadata being stored in the `.git` directory:

```console
$ mkdir myproject
$ cd myproject
$ git init
Initialized empty Git repository in /home/missing-semester/myproject/.git/
$ git status
On branch master

No commits yet

nothing to commit (create/copy files and use "git add" to track)
```

How do we interpret this output? "No commits yet" basically means our version
history is empty. Let's fix that.

```console
$ echo "hello, git" > hello.txt
$ git add hello.txt
$ git status
On branch master

No commits yet

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)

        new file:   hello.txt

$ git commit -m 'Initial commit'
[master (root-commit) 4515d17] Initial commit
 1 file changed, 1 insertion(+)
 create mode 100644 hello.txt
```

With this, we've `git add`ed a file to the staging area, and then `git
commit`ed that change, adding a simple commit message "Initial commit". If we
didn't specify a `-m` option, Git would open our text editor to allow us type a
commit message.

Now that we have a non-empty version history, we can visualize the history.
Visualizing the history as a DAG can be especially helpful in understanding the
current status of the repo and connecting it with your understanding of the Git
data model.

The `git log` command visualizes history. By default, it shows a flattened
version, which hides the graph structure. If you use a command like `git log
--all --graph --decorate`, it will show you the full version history of the
repository, visualized in graph form.

```console
$ git log --all --graph --decorate
* commit 4515d17a167bdef0a91ee7d50d75b12c9c2652aa (HEAD -> master)
  Author: Missing Semester <missing-semester@mit.edu>
  Date:   Tue Jan 21 22:18:36 2020 -0500

      Initial commit
```

This doesn't look all that graph-like, because it only contains a single node.
Let's make some more changes, author a new commit, and visualize the history
once more.

```console
$ echo "another line" >> hello.txt
$ git status
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

        modified:   hello.txt

no changes added to commit (use "git add" and/or "git commit -a")
$ git add hello.txt
$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

        modified:   hello.txt

$ git commit -m 'Add a line'
[master 35f60a8] Add a line
 1 file changed, 1 insertion(+)
```

Now, if we visualize the history again, we'll see some of the graph structure:

```
* commit 35f60a825be0106036dd2fbc7657598eb7b04c67 (HEAD -> master)
| Author: Missing Semester <missing-semester@mit.edu>
| Date:   Tue Jan 21 22:26:20 2020 -0500
|
|     Add a line
|
* commit 4515d17a167bdef0a91ee7d50d75b12c9c2652aa
  Author: Anish Athalye <me@anishathalye.com>
  Date:   Tue Jan 21 22:18:36 2020 -0500

      Initial commit
```

Also, note that it shows the current HEAD, along with the current branch
(master).

We can look at old versions using the `git checkout` command.

```console
$ git checkout 4515d17  # previous commit hash; yours will be different
Note: checking out '4515d17'.

You are in 'detached HEAD' state. You can look around, make experimental
changes and commit them, and you can discard any commits you make in this
state without impacting any branches by performing another checkout.

If you want to create a new branch to retain commits you create, you may
do so (now or later) by using -b with the checkout command again. Example:

  git checkout -b <new-branch-name>

HEAD is now at 4515d17 Initial commit
$ cat hello.txt
hello, git
$ git checkout master
Previous HEAD position was 4515d17 Initial commit
Switched to branch 'master'
$ cat hello.txt
hello, git
another line
```

Git can show you how files have evolved (differences, or diffs) using the `git
diff` command:

```console
$ git diff 4515d17 hello.txt
diff --git c/hello.txt w/hello.txt
index 94bab17..f0013b2 100644
--- c/hello.txt
+++ w/hello.txt
@@ -1 +1,2 @@
 hello, git
 +another line
```

{% endcomment %}

- `git help <command>`: get help for a git command
- `git init`: creates a new git repo, with data stored in the `.git` directory
- `git status`: tells you what's going on
- `git add <filename>`: adds files to staging area
- `git commit`: creates a new commit
    - Write [good commit messages](https://tbaggery.com/2008/04/19/a-note-about-git-commit-messages.html)!
    - Even more reasons to write [good commit messages](https://chris.beams.io/posts/git-commit/)!
- `git log`: shows a flattened log of history
- `git log --all --graph --decorate`: visualizes history as a DAG
- `git diff <filename>`: show differences since the last commit
- `git diff <revision> <filename>`: shows differences in a file between snapshots
- `git checkout <revision>`: updates HEAD and current branch

## Branching and merging

{% comment %}

Branching allows you to "fork" version history. It can be helpful for working
on independent features or bug fixes in parallel. The `git branch` command can
be used to create new branches; `git checkout -b <branch name>` creates and
branch and checks it out.

Merging is the opposite of branching: it allows you to combine forked version
histories, e.g. merging a feature branch back into master. The `git merge`
command is used for merging.

{% endcomment %}

- `git branch`: shows branches
- `git branch <name>`: creates a branch
- `git checkout -b <name>`: creates a branch and switches to it
    - same as `git branch <name>; git checkout <name>`
- `git merge <revision>`: merges into current branch
- `git mergetool`: use a fancy tool to help resolve merge conflicts
- `git rebase`: rebase set of patches onto a new base

## Remotes

- `git remote`: list remotes
- `git remote add <name> <url>`: add a remote
- `git push <remote> <local branch>:<remote branch>`: send objects to remote, and update remote reference
- `git branch --set-upstream-to=<remote>/<remote branch>`: set up correspondence between local and remote branch
- `git fetch`: retrieve objects/references from a remote
- `git pull`: same as `git fetch; git merge`
- `git clone`: download repository from remote

## Undo

- `git commit --amend`: edit a commit's contents/message
- `git reset HEAD <file>`: unstage a file
- `git checkout -- <file>`: discard changes

# Advanced Git

- `git config`: Git is [highly customizable](https://git-scm.com/docs/git-config)
- `git clone --depth=1`: shallow clone, without entire version history
- `git add -p`: interactive staging
- `git rebase -i`: interactive rebasing
- `git blame`: show who last edited which line
- `git stash`: temporarily remove modifications to working directory
- `git bisect`: binary search history (e.g. for regressions)
- `.gitignore`: [specify](https://git-scm.com/docs/gitignore) intentionally untracked files to ignore

# Miscellaneous

- **GUIs**: there are many [GUI clients](https://git-scm.com/downloads/guis)
out there for Git. We personally don't use them and use the command-line
interface instead.
- **Shell integration**: it's super handy to have a Git status as part of your
shell prompt ([zsh](https://github.com/olivierverdier/zsh-git-prompt),
[bash](https://github.com/magicmonty/bash-git-prompt)). Often included in
frameworks like [Oh My Zsh](https://github.com/ohmyzsh/ohmyzsh).
- **Editor integration**: similarly to the above, handy integrations with many
features. [fugitive.vim](https://github.com/tpope/vim-fugitive) is the standard
one for Vim.
- **Workflows**: we taught you the data model, plus some basic commands; we
didn't tell you what practices to follow when working on big projects (and
there are [many](https://nvie.com/posts/a-successful-git-branching-model/)
[different](https://www.endoflineblog.com/gitflow-considered-harmful)
[approaches](https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow)).
- **GitHub**: Git is not GitHub. GitHub has a specific way of contributing code
to other projects, called [pull
requests](https://help.github.com/en/github/collaborating-with-issues-and-pull-requests/about-pull-requests).
- **Other Git providers**: GitHub is not special: there are many Git repository
hosts, like [GitLab](https://about.gitlab.com/) and
[BitBucket](https://bitbucket.org/).

# Resources

- [Pro Git](https://git-scm.com/book/en/v2) is **highly recommended reading**.
Going through Chapters 1--5 should teach you most of what you need to use Git
proficiently, now that you understand the data model. The later chapters have
some interesting, advanced material.
- [Oh Shit, Git!?!](https://ohshitgit.com/) is a short guide on how to recover
from some common Git mistakes.
- [Git for Computer
Scientists](https://eagain.net/articles/git-for-computer-scientists/) is a
short explanation of Git's data model, with less pseudocode and more fancy
diagrams than these lecture notes.
- [Git from the Bottom Up](https://jwiegley.github.io/git-from-the-bottom-up/)
is a detailed explanation of Git's implementation details beyond just the data
model, for the curious.
- [How to explain git in simple
words](https://smusamashah.github.io/blog/2017/10/14/explain-git-in-simple-words)
- [Learn Git Branching](https://learngitbranching.js.org/) is a browser-based
game that teaches you Git.

# Exercises

1. If you don't have any past experience with Git, either try reading the first
   couple chapters of [Pro Git](https://git-scm.com/book/en/v2) or go through a
   tutorial like [Learn Git Branching](https://learngitbranching.js.org/). As
   you're working through it, relate Git commands to the data model.
1. Clone the [repository for the
class website](https://github.com/missing-semester/missing-semester).
    1. Explore the version history by visualizing it as a graph.
    1. Who was the last person to modify `README.md`? (Hint: use `git log` with
       an argument)
    1. What was the commit message associated with the last modification to the
       `collections:` line of `_config.yml`? (Hint: use `git blame` and `git
       show`)
1. One common mistake when learning Git is to commit large files that should
   not be managed by Git or adding sensitive information. Try adding a file to
   a repository, making some commits and then deleting that file from history
   (you may want to look at
   [this](https://help.github.com/articles/removing-sensitive-data-from-a-repository/)).
1. Clone some repository from GitHub, and modify one of its existing files.
   What happens when you do `git stash`? What do you see when running `git log
   --all --oneline`? Run `git stash pop` to undo what you did with `git stash`.
   In what scenario might this be useful?
1. Like many command line tools, Git provides a configuration file (or dotfile)
   called `~/.gitconfig`. Create an alias in `~/.gitconfig` so that when you
   run `git graph`, you get the output of `git log --all --graph --decorate
   --oneline`.
1. You can define global ignore patterns in `~/.gitignore_global` after running
   `git config --global core.excludesfile ~/.gitignore_global`. Do this, and
   set up your global gitignore file to ignore OS-specific or editor-specific
   temporary files, like `.DS_Store`.
1. Clone the [repository for the class
   website](https://github.com/missing-semester/missing-semester), find a typo
   or some other improvement you can make, and submit a pull request on GitHub.
