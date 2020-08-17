---
layout: lecture
title: "버전 컨트롤(Git)"
date: 2019-01-22
ready: true
video:
  aspect: 56.25
  id: 2sjqTHE0zok
---
버전 컨트롤 시스템 (VCSs)는 소스 코드(혹은 다른 파일, 폴더들의 모음)의 변화를 추적하기 위해 사용되는 도구입니다. 이름이 암시하는 것과 같이, 이러한 도구들은 변화의 히스토리(history)를 보존하는 데 도움을 줍니다; 뿐만 아니라, 협업을 더욱 용이하게 만들기도 합니다. VCSs는 폴더와 그 내용의 변화를 일련의 스냅샷(Snapshot)으로 저장하며, 각 스냅샷은 최상위 디렉토리에 속하는 파일/폴더의 전체 상태를 캡슐화합니다. 또한 VCSs는 누가 각 스냅샷을 만들고, 각 스냅샷에 연관된 메세지 등의 메타테이터를 유지합니다.

왜 버전 컨트롤이 유용한 걸까요? 비록 당신이 혼자 일한다고 하더라도, 이러한 도구들은 프로젝트의 이전 스냅샷을 볼 수 있게 하고, 왜 이러한 변화가 이루어졌는지에 대한 로그를 기록하고, 개발할 때 별도의 브랜치에서 작업하는 것 외에도 훨씬 더 많은 일들을 할 수 있게 해줍니다. 다른 이들과 일할 때라면, 다른 사람들이 변경한 것을 보는 만큼 동시에 개발할 때의 충돌을 해결하는 귀중한 도구가 됩니다.

또한 현대의 VCSs는 쉽게(그리고 종종 자동으로) 이러한 질문에 대답할 수 있게 해줍니다 : 

- 누가 이 모듈을 작성했는가?
- 언제 특정 파일의 특정 라인이 수정되었는가? 누가 수정했고, 왜 이것이 수정되었는가?
- 지난 1000개의 수정에서, 언제/왜 특정한 단위 테스트가 동작을 멈추는가?

다른 VCSs들이 있음에도, **Git**은 버전 컨트롤의 사실상 표준입니다.
이 [XKCD comic](https://xkcd.com/1597/) 은 Git의 특징을 잘 짚고 있습니다:

![xkcd 1597](https://imgs.xkcd.com/comics/git.png)

Git의 인터페이스는 다소 추상적이기 때문에(leaky abstraction), Git을 하향식(top-down)으로 배우는 것(커맨드 라인 인터페이스(command-line interface)부터 시작하는 것)은 많은 혼란을 일으킬 수 있습니다. 약간의 명령어만을 기억하고 그것들을 마치 주문처럼 생각하여, 언제든 무엇인가 잘못된다면 위 만화의 내용을 따라하는 것이 더 좋습니다.

Git이 못생긴 인터페이스를 갖고 있는 것은 인정하지만, 내재된 디자인과 아이디어는 아름답습니다. 못생긴 인터페이스는 _기억될_ 필요가 있지만, 아름다운 디자인은 _이해될_ 수 있습니다. 이러한 이유로, 우리는 Git에 대해서, 먼저 데이터 모델로 시작하여 나중에 커맨드 라인 인터페이스를 다루는 상향식(bottom-up)으로 설명할 것입니다. 일단 데이터 모델을 이해하고 나면, 어떻게 명령어가 데이터 모델을 다루는지에 대해 더 잘 이해하게 될 것입니다.

# Git의 데이터 모델

버전 컨트롤에 바로 접근 가능한 수많은 접근법이 있습니다. Git은 히스토리를 유지하고, 브랜치를 지원하고, 협업을 가능하게 하는 등 버전 컨트롤의 모든 유용한 기능을 가능하게 하는, 효과적으로 계획된 모델입니다.

## 스냅샷

Git은 최상위 디렉토리에 속하는 파일과 폴더 모음의 히스토리를 일련의 스냅샷으로 형성합니다. Git의 용어에서, 파일은 바이트 묶음(a bunch of bytes)이라는 뜻의 "blob"이라고 불립니다.  디렉토리는 "Tree"라고 불리우며, 이는 blob이나 tree에 이름을 지정합니다 (그래서 디렉토리들은 다른 디렉토리를 포함할 수 있습니다). 스냅샷은 추적되는(tracked) 최상위 트리입니다. 예를 들어, 우리는 다음과 같이 트리를 갖게 됩니다:

```
<root> (tree)
|
+- foo (tree)
|  |
|  + bar.txt (blob, contents = "hello world")
|
+- baz.txt (blob, contents = "git is wonderful")
```

최상위 트리는 두 개의 원소를 갖습니다. "foo" 트리( 해당 트리는 "bar.txt"라는 blob 원소를 갖고 있습니다)와, "baz.txt"라는 blob입니다.

## 히스토리 생성: 스냅샷의 연결

어떻게 버전 컨트롤 시스템이 스냅샷들을 연결할 수 있을까요? 한가지 단순한 모델은 선형 히스토리가 될것입니다. 히스토리는 시간 순서의 스냅샷 리스트가 될 것입니다. 여러 가지 이유로, Git은 이런 단순한 모델을 사용하지 않습니다.

Git에서, 히스토리는 방향 비순환 그래프(directed acyclic graph, DAG)입니다. 매우 수학 용어처럼 들릴 지 모르겠지만, 너무 겁먹지는 마세요. Git의 각 스냅샷은 선행하는 "부모" 스냅샷들(a set of parents)을 참조한다는 의미입니다. (선형 히스토리와 같이) 단일 부모가 아닌 부모 스냅샷들인 이유는, 예를 들어 두 개의 병렬 개발 브랜치를 합치는(머지하는(merging)) 경우 때문에 스냅샷이 여러 개의 부모로부터 내려올 수 있기 때문입니다.

Git은 이러한 스냅샷들을 "커밋(Commits)"이라고 합니다. 커밋 히스토리를 시각화하면 다음과 같이 보이게 됩니다:

```
o <-- o <-- o <-- o
            ^  
             \
              --- o <-- o
```

위의 아스키 아트에서, `o`들은 각각의 커밋(스냅샷)입니다. 화살표들은 각 커밋의 부모를 가리킵니다(이는 "앞서는" 관계가 아닌, "뒤를 잇는" 관계입니다). 세 번째 커밋 이후에, 히스토리는 두 개의 분리된 브랜치로 갈라집니다. 이것이 의미할 수 있는 것은, 예를 들어서 두 분리된 기능이 동시에, 독립적으로 각각 개발될 수 있다는 점입니다. 이후에, 이러한 브랜치들은 아마 새로운 스냅샷을 만들기 위해 머지되어 각 기능을 모두 포함하고, 굵은 표시로 보이는 것과 같이 새롭게 생성된 머지 커밋과 함께 다음과 같은 새 히스토리를 만들 것입니다

<pre>
o <-- o <-- o <-- o <---- <strong>o</strong>
            ^            /
             \          v
              --- o <-- o
</pre>

깃의 커밋은 변경할 수 없습니다. 실수가 정정될 수 없다는 것을 의미하는 것은 아니며, 한편 그저 커밋 히스토리에 대한 "수정"이 완전히 새로운 커밋을 만들고, 새롭게 가리키도록 레퍼런스(아래를 보세요)가 업데이트되게 됩니다.

## 의사 모델로써의 데이터 모델

유사 코드로 작성된 Git의 데이터 모델을 보는 것이 유익할 수 있습니다:

```
// 파일은 바이트의 묶음입니다.
type blob = array<byte>

// 디렉토리는 이름 지어진 파일과 디렉토리를 포함합니다
type tree = map<string, tree | blob>

// 커밋은 부모와, 메타데이터, 그리고 최상위 트리를 갖고 있습니다.
type commit = struct {
    parent: array<commit>
    author: string
    message: string
    snapshot: tree
}
```

이는 깔끔하고, 단순한 히스토리 모델입니다.

## 객체와 content-addressing

"객체" 는 blob, 트리, 혹은 커밋입니다:

```
type object = blob | tree | commit
```

Git 데이터 저장소에서, 모든 객체들은 그들의 [SHA-1
hash](https://en.wikipedia.org/wiki/SHA-1)로 인해 컨텐츠 자체가 주소 역할을 합니다.

```
objects = map<string, object>

def store(object):
    id = sha1(object)
    objects[id] = object

def load(id):
    return objects[id]
```

Blob, 트리, 그리고 커밋들은 이렇게 통일되어, 모두 객체가 됩니다. 그들이 다른 객체를 참조할 때, 실제로 디스크 상의 존재를 _포함_ 하지 않고, 그들의 해시를 참조합니다.

예를 들어, 
(`git cat-file -p 698281bc680d1995c5f4caaf3359721a5a58d48d` 명령어로 시각화되는) [위의](#스냅샷) 예시 디렉토리 구조는, 다음과 같이 보입니다:

```
100644 blob 4448adbf7ecd394f42ae135bbeed9676e894af85    baz.txt
040000 tree c68d233a33c5c06e0340e4c224f0afca87c8ce87    foo
```

트리는 스스로 내용물인, `baz.txt` (blob) 와 `foo`(트리)로의 포인터를 포함합니다. 만약 우리가 `git cat-file -p 4448adbf7ecd394f42ae135bbeed9676e894af85`로 baz.txt와 일치하는 해시로 내용물을 본다면, 우리는 이러한 것을 얻을 수 있습니다:

```
git is wonderful
```

## 레퍼런스

이제, 모든 스냅샷은 그들의 SHA-1 해시에 의해 확인될 수 있습니다. 사람은 40자리의 16진수 글자를 기억하기 힘들기 때문에, 이는 불편하다고 할 수 있습니다.

이런 문제에 대한 Git의 해결책은 "레퍼런스" 라고 불리는 SHA-1 해시에 대한 사람이 읽을 수 있는 이름입니다. 레퍼런스는 커밋에 대한 포인터입니다. 수정이 불가능한 객체와 다르게, 레퍼런스는 수정 가능합니다(다른 커밋을 가리키도록 업데이트될 수 있습니다). 예를 들어, `master` 레퍼런스는 주로 개발 중인 메인 브랜치의 최신 커밋을 표시합니다.

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

이런 식으로, Git은 긴 16진수 문자열 대신, 히스토리 내의 특정 스냅샷을 참조하도록 하기 위해 "master"와 같이 사람이 읽을 수 있는 이름을 사용합니다.

우리는 종종 히스토리에서 "우리가 지금 어디 있는지"에 대해 알고 싶어 합니다. 그래서 우리가 새 스냅샷을 설정할 때, 우리는 (커밋의 `부모` 필드를 우리가 어떻게 설정하냐에 따라) 무엇에 연관되어 있는지 알게 됩니다. Git에서, 이 "우리가 지금 어디 있는지" 는 특별한 레퍼런스인 "헤드(HEAD)" 라고 불립니다.

## 레포지토리

마침내, 우리는 (대충은) Git _레포지토리_ 가 무엇인지를 정의할 수 있습니다: 바로 데이터 `객체`와 `레퍼런스` 입니다.

디스크 상에서, 모든 Git 저장소들은 객체와 레퍼런스입니다 : 그곳에 있는 모든 것들이 Git의 데이터 모델이라는 것입니다. 모든 `git` 명령어는 객체를 추가하고 레퍼런스를 추가/갱신하여 커밋 DAG의 일부 처리에 매핑됩니다.

당신이 어떤 커맨드를 입력할 때, 명령이 내재된 그래프 자료구조에 어떠한 처리를 하는지 생각해보세요. 반대로, 만약 당신이 커밋 DAG에 어떤 특정한 변화를 주고자 할 때-예를 들어 "커밋되지 않은 변화를 삭제하고 'master' 레퍼런스를 `5d83f9e` 커밋으로 만들고자 할 때"-아마도 그러한 일을 하는 명령어가 있을 것입니다 (예를 들어 이번 사례에서는, `git checkout master; git reset
--hard 5d83f9e`입니다.)

# 준비 영역(Staging area)

이것 데이터 모델과는 직교하는 또다른 개념 이지만, 커밋을 만들기 위한 인터페이스의 일부입니다.

여러분이 위 설명대로 스냅샷의 구현을 상상할 수 있는 방법 중 하나는 작업 디렉토리의 _현재 상태_ 에 기반한 새 스냅샷을 만드는 "create snapshot" 명령어를 사용하는 것입니다. 

몇몇 버전 컨트롤 도구는 이렇게 동작하겠지만, Git에서는 그렇지 않습니다. 우리는 무결한 스냅샷을 원하며, 현재 상태로부터 스냅샷을 만드는 것이 항상 이상적이지 않을 수 있습니다. 예를 들어, 여러분이 두 분리된 기능을 구현하는 경우를 상상해보면, 당신은 첫 번째는 첫 번째 기능을, 두 번째는 두 번째 기능을 도입하고자 두 개의 분리된 커밋을 만들고자 할 것입니다. 혹은 버그 수정처럼 코드 전반에 디버깅하는 프린트문을 넣었다고 상상해봅시다; 당신은 모든 프린트문을 삭제하는 동안 버그 삭제를 커밋하길 원할 것입니다.

Git은 "준비 영역"이라는 메카니즘을 통해 어떠한 수정사항이 다음 스냅샷에 들어가야 하는지 여러분이 명시할 수 있도록 함으로써 이러한 시나리오를 가능하게 합니다.

# Git 커맨드 라인 인터페이스

중복된 내용를 피하기 위해, 우리는 아래의 명령어를 자세히 설명하지는 않을 것입니다. 더 많은 정보를 위해서 [Pro Git](https://git-scm.com/book/en/v2)을 보는 것을 추천하며, 혹은 강의 비디오를 보시길 바랍니다.

## 기초

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

- `git help <command>`: git 명령어에 대한 도움말을 봅니다
- `git init`: 새로운 git 저장소를 만듭니다. 데이터는 `.git` 디렉토리에 저장됩니다
- `git status`: 현재 진행 상태를 알려줍니다
- `git add <filename>`: 파일을 준비 영역에 추가합니다
- `git commit`: 새 커밋을 만듭니다
    - [좋은 커밋 메세지](https://tbaggery.com/2008/04/19/a-note-about-git-commit-messages.html)를 작성하시길 바랍니다!
    - [좋은 커밋 메세지](https://chris.beams.io/posts/git-commit/)를 작성하여야 하는 이유는 다음을 참고하세요!
- `git log`: 단일화된 히스토리 로그를 보여줍니다
- `git log --all --graph --decorate`: 히스토리를 DAG로 시각화합니다
- `git diff <filename>`: 마지막 커밋과의 차이를 보여줍니다
- `git diff <revision> <filename>`: 스냅샷 사이에 파일의 차이를 보여줍니다
- `git checkout <revision>`: HEAD와 현재 브랜치를 갱신합니다

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

- `git branch`: 브랜치를 보여줍니다
- `git branch <name>`: 브랜치를 생성합니다
- `git checkout -b <name>`: 브랜치를 생성하여 이동합니다
    - `git branch <name>; git checkout <name>`와 동일합니다
- `git merge <revision>`: 현재 브랜치로 머지합니다
- `git mergetool`: 머지 충돌을 해결하기 위한 멋진 도구를 사용합니다
- `git rebase`: 패치들을 새로운 베이스로 rebase합니다

## Remotes

- `git remote`: remote 리스트를 보여줍니다
- `git remote add <name> <url>`: remote를 추가합니다
- `git push <remote> <local branch>:<remote branch>`: 객체를 remote로 보내고, remote 레퍼런스를 추가합니다
- `git branch --set-upstream-to=<remote>/<remote branch>`: 지역과 remote 브랜치 사이의 연관성을 추가합니다.
- `git fetch`: remote로부터 객체/레퍼런스를 가져옵니다
- `git pull`: `git fetch; git merge`와 동일합니다
- `git clone`: remote로부터 레포지토리를 다운로드합니다.

## Undo

- `git commit --amend`: 커밋의 내용/메세지를 수정합니다
- `git reset HEAD <file>`: 파일을 unstaged 상태로 만듭니다
- `git checkout -- <file>`: 변화를 삭제합니다

# Advanced Git

- `git config`: Git [커스터마이징](https://git-scm.com/docs/git-config)이 용이합니다
- `git clone --depth=1`: 전체 버전 히스토리를 제외한, 얕은 클론을 만듭니다
- `git add -p`: 상호 작용 가능한 staging입니다
- `git rebase -i`: 상호 작용 가능한 rebasing입니다
- `git blame`: 누가 어떠한 라인을 마지막으로 수정했는지를 보여줍니다
- `git stash`: 작업 디렉토리의 수정사항을 임시로 삭제합니다
- `git bisect`: 히스토리를 이진 탐색합니다 (예를 들면, 회귀 분석에 대해서)
- `.gitignore`: 의도적으로 추적되지 않는 파일을 무시하도록 [명시합니다](https://git-scm.com/docs/gitignore)

# Miscellaneous

- **GUIs**: Git을 위한 수많은 [GUI 클라이언트](https://git-scm.com/downloads/guis)가 존재합니다. 우리는 개인적으로 이런 것들을 사용하지 않고 대신 커맨드 라인 인터페이스를 사용합니다.
- **Shell integration**: 쉘 프롬프트의 부분으로써 Git 상태를 확인하는 것은 엄청나게 편리한 일입니다([zsh](https://github.com/olivierverdier/zsh-git-prompt),
[bash](https://github.com/magicmonty/bash-git-prompt)). 때때로 [Oh My Zsh](https://github.com/ohmyzsh/ohmyzsh)같은 프레임워크에도 포함되어 있습니다.
- **Editor integration**: 위와 비슷하게, 많은 기능을 포함하는 편리한 확장 기능입니다.[fugitive.vim](https://github.com/tpope/vim-fugitive)은 Vim을 위한 표준 중 하나입니다.
- **Workflows**: 우리는 여러분에게 데이터 모델과, 몇 가지 기본 명령어를 알려드렸습니다. 하지만 큰 프로젝트에서 작업할 때 어떤 방식을 따라야 할지는 알려드리지 않았습니다 ([수많은](https://nvie.com/posts/a-successful-git-branching-model/)
[다른](https://www.endoflineblog.com/gitflow-considered-harmful)
[접근법들이 있습니다](https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow) ) 
- **GitHub**: Git은 Github가 아닙니다. Github는 [pull requests](https://help.github.com/en/github/collaborating-with-issues-and-pull-requests/about-pull-requests)라는 방법으로, 다른 프로젝트에 코드로 기여하기 위한 특정한 방법을 갖고 있습니다. 
- **Other Git providers**: Github는 특별한 것이 아닙니다 : [GitLab](https://about.gitlab.com/)과 [BitBucket](https://bitbucket.org/)과 같은, 수많은 Git 저장소가 존재합니다. 

# 참고자료

- [Pro Git](https://git-scm.com/book/en/v2) 은 **매우 추천하는 읽을 거리**입니다. 챕터 1--5를 보면 Git을 능숙하게 사용하기 위해 필요한 대부분을 여러분에게 알려줄 것이며, 이제 여러분은 데이터 모델을 이해하게 될 것입니다. 나중의 챕터는 몇 가지 흥미롭고, 고급의 자료를 담고 있습니다.
- [Oh Shit, Git!?!](https://ohshitgit.com/) 은 몇 가지 Git에서의 실수를 해결하기 위한 짧은 가이드입니다.
- [Git for Computer Scientists](https://eagain.net/articles/git-for-computer-scientists/) 는 이 강의 노트보다도 적은 유사 코드와 많은 다이어그램을 통해, Git의 데이터 모델에 대한 짧은 설명을 담고 있습니다.
- [Git from the Bottom Up](https://jwiegley.github.io/git-from-the-bottom-up/)
은 호기심이 많은 이들을 위해, Git의 상세한 구현에 대해 데이터 모델 이상의 자세한 설명을 담고 있습니다.
- [How to explain git in simple words](https://smusamashah.github.io/blog/2017/10/14/explain-git-in-simple-words)
- [Learn Git Branching](https://learngitbranching.js.org/) 은 브라우저에 기반한 게임으로 여러분에게 Git에 대해 알려줍니다.

# 연습해보기

1. 만약 Git에 대한 어떠한 경험도 갖고 있지 않다면, [Pro Git](https://git-scm.com/book/en/v2)의 처음 두 책터를 읽어 보거나 [Learn Git Branching](https://learngitbranching.js.org/)의 튜토리얼을 해 보세요. 직접 해 보면서, GIt 명령어를 데이터 모델과 연관지어 보세요.
2. [수업 웹사이트의 레포지토리](https://github.com/missing-semester/missing-semester)를 클론해보세요.
    1. 그래프로 시각화하여 버전 히스토리를 탐색해보세요.
    2. `README.md`를 마지막으로 수정한 사람이 누구인가요? (힌트:`git log`를 인수와 함께 사용해 보세요.)
    3. `_config.yml`의  `collections:` 줄을 마지막으로 수정한 것과 관련된 커밋 메시지가 무엇인가요? (힌트 : `git blame`과 `git show` 명령어를 사용하세요)
3. Git을 배울 때 가장 흔한 실수는 Git으로 관리되어서는 안되는 큰 파일을 커밋하거나 민감한 정보를 추가하는 것입니다. 파일을 레포지토리에 추가하고, 커밋을 한 뒤 이러한 파일을 히스토리에서 삭제해보세요([이것](https://help.github.com/articles/removing-sensitive-data-from-a-repository/)을 참고해 보세요)=
4. Github에서 어떤 레포지토리를 클론해서, 존재하는 파일 중 하나를 수정해보세요. `git stash`를 했을 때 무슨 일이 일어나나요? `git log --all --oneline`을 실행할 때 무엇을 볼 수 있나요? `git stash`로 한 것을 되돌리기 위해 `git stash pop`을 실행시켜 보세요. 어떤 상황에서 이것이 쓸모가 있을까요?
5. 많은 커맨드 라인 도구와 같이, Git은 `~/.gitconfig`라는 설정 파일(혹은 닷 파일(dotfile))을 제공합니다. `git graph`를 실행할 때, `git log --all --graph --decorate --oneline`의 출력을 얻을 수 있도록 `~/.gitconfig`에 alias를 추가해 보세요.
6. `git config --global core.excludesfile ~/.gitignore_global`를 실행한 뒤 `~/.gitignore_global`에 전역 생략 패턴(global ignore patterns)을 정의할 수 있습니다. 이것을 실행하고, 여러분의 `.DS_Store`과 같은, OS 혹은 편집기에서 생성된 임시 파일을 무시하기 위해 전역 gitignore파일을 설정해보세요.
7. [수업 웹사이트의 레포지토리](https://github.com/missing-semester/missing-semester)를 클론해보세요. 오타를 찾거나 여러분이 할 수 있는 개선사항을 만들어보고, Github에 pull request를 해보세요.
