---
layout: post
title:  "git history log 수정"
date: 2015-12-08
categories: git
---

이 내용은 [이곳](https://git-scm.com/book/ko/v1/Git-%EB%8F%84%EA%B5%AC-%ED%9E%88%EC%8A%A4%ED%86%A0%EB%A6%AC-%EB%8B%A8%EC%9E%A5%ED%95%98%EA%B8%B0 "git history log 수정")을 참조하여 정리 하였다.  
filter-branch 내용은 나중에 정리 하자.  

# amend

```sh
$ git commit --amend
```

해당 옵션(amend)를 수행하면 텍스트 편집기가 실행 되며 마지막 커밋 메시지를 수정 할수 있다.  

이때 SHA-1 값이 변경 되기 때문에 과거의 커밋을 변경 할때는 주의해야 한다.   

## 예제

```sh
returns@returns gitTest$ touch a
returns@returns gitTest$ git status
On branch master

Initial commit

Untracked files:
  (use "git add <file>..." to include in what will be committed)

	a

nothing added to commit but untracked files present (use "git add" to track)
returns@returns gitTest$ git add a
returns@returns gitTest$ git commit -m 'test1'
[master (root-commit) 10d1ab3] test1
 1 file changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 a
returns@returns gitTest$ git commit --amend
[master 88ed773] test2
 1 file changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 a
returns@returns gitTest$ git log
commit 88ed773851051dba4662c6a363f0ad14020096ba
Author: DoO <returns@lycos.co.kr>
Date:   Tue Dec 8 09:56:55 2015 +0900

    test2
```

# rebase 이용

## 여러 커밋을 수정

amend 옵션으로는 여러 커밋 메시지를 수정못한다.  
그렇지만 rebase 명령을 이용하면 수정 할수 있다.  
현재 작업하는 브랜치에서 각 커밋을 수정하는것이 아니라 어느 시점 부터 HEAD 까지의 커밋을 한번에 Rebase 한다.  

git rebase 명령에 -i 옵션을 추가 하면 어느 시점 부터 HEAD 까지 reabse 할지를 결정 할수 있다.  
보통 HEAD^2(HEAD 로부터 2번째 커밋 까지) 나 HEAD~3(HEAD 로부터 3번째 커밋 까지) 으로 한다.  

```sh
$ git rebase -i HEAD~3
```

중앙 서버에 Push 한 커밋은 수정할수 있지만 절대 고치지 않아야 한다.   
여럿이서 협업 하였을때 push 한 커밋을 rebase 하면 sha-1 값과 여럿이서 보는 로그들이 변경 된다.   
이는 협업에 혼선을 가져올수 있다.   

git rebase 를 수행하면 아래와 같은 내용을 담고 편집기가 수행된다.  

```sh
pick b7d9797 posting in 2015-12-07
pick eb143e9 posting in 2015-12-07

# Rebase 0af50ca..eb143e9 onto 0af50ca
#
# Commands:
#  p, pick = use commit
#  r, reword = use commit, but edit the commit message
#  e, edit = use commit, but stop for amending
#  s, squash = use commit, but meld into previous commit
#  f, fixup = like "squash", but discard this commit's log message
#  x, exec = run command (the rest of the line) using shell
#
# These lines can be re-ordered; they are executed from top to bottom.
#
# If you remove a line here THAT COMMIT WILL BE LOST.
#
# However, if you remove everything, the rebase will be aborted.
#
# Note that empty commits are commented out
```

## 커밋 순서 변경

## 커밋 합치기

rebase 옵션으로 수행하고 pick 으로 지정되어 있는 커밋 로그를 squash 로 변경 하면 된다.   
squash 로 변경하면 이전 커밋과 합쳐진다.    
합쳐 질때는 커밋 로그 메시지도 수정할수 있다.    

```
$ git --rebase HEAD~n
```

### 예제

로그 메시지 확인후 rebase 를 수행한다.    

```
returns@returns gitTest$ git log
commit 64ea572d37b923e30be9b277ec022860a9dd074f
Author: DoO <returns@lycos.co.kr>
Date:   Tue Dec 8 10:32:14 2015 +0900

    add c

commit 018988ef6bc5b8d8c8e83b68512fb74a5eb80531
Author: DoO <returns@lycos.co.kr>
Date:   Tue Dec 8 10:32:02 2015 +0900

    add b

commit 88ed773851051dba4662c6a363f0ad14020096ba
Author: DoO <returns@lycos.co.kr>
Date:   Tue Dec 8 09:56:55 2015 +0900

    test2
returns@returns gitTest$ git rebase -i HEAD~2
```

합칠 커밋을 pick 에서 squash 로 변경 한다.    
squash 로 변경하면 위의 커밋과 합쳐 진다.    

```
pick 018988e add b
squash 64ea572 add c

# Rebase 88ed773..64ea572 onto 88ed773
#
# Commands:
#  p, pick = use commit
#  r, reword = use commit, but edit the commit message
#  e, edit = use commit, but stop for amending
#  s, squash = use commit, but meld into previous commit
#  f, fixup = like "squash", but discard this commit's log message
#  x, exec = run command (the rest of the line) using shell
#
# These lines can be re-ordered; they are executed from top to bottom.
#
# If you remove a line here THAT COMMIT WILL BE LOST.
#
# However, if you remove everything, the rebase will be aborted.
#
# Note that empty commits are commented out
```

커밋 메시지를 수정한다.    

```
# This is a combination of 2 commits.
# The first commit's message is:

add b

# This is the 2nd commit message:

add c

# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
# rebase in progress; onto 88ed773
# You are currently editing a commit while rebasing branch 'master' on '88ed773'.
#
# Changes to be committed:
#       new file:   b
#       new file:   c
#
```

결과를 확인한다.    

```
[detached HEAD d636d61] add b & c
 2 files changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 b
 create mode 100644 c
Successfully rebased and updated refs/heads/master.
returns@returns gitTest$ git log
commit d636d6115f3fc9c3b36e5b29d1d740a9ea27deb7
Author: DoO <returns@lycos.co.kr>
Date:   Tue Dec 8 10:32:02 2015 +0900

    add b & c

commit 88ed773851051dba4662c6a363f0ad14020096ba
Author: DoO <returns@lycos.co.kr>
Date:   Tue Dec 8 09:56:55 2015 +0900

    test2
```

## 커밋 분리하기

커밋을 분리하는것은 기존 커밋을 reset 하고 Stage 를 여러개로 나눠서 원하는 커밋을 수행한다.  

```
$ git reset HEAD^

```

### 예제

git reset 수행후 하나 파일 씩 나눠서 커밋을 수행한다.   

```
returns@returns gitTest$ git log
commit 392aa6964e0bb0486d2f9d1b4e2a49ed8e45ff15
Author: DoO <returns@lycos.co.kr>
Date:   Tue Dec 8 10:55:12 2015 +0900

    add d & e

commit d636d6115f3fc9c3b36e5b29d1d740a9ea27deb7
Author: DoO <returns@lycos.co.kr>
Date:   Tue Dec 8 10:32:02 2015 +0900

    add b & c

commit 88ed773851051dba4662c6a363f0ad14020096ba
Author: DoO <returns@lycos.co.kr>
Date:   Tue Dec 8 09:56:55 2015 +0900

    test2
returns@returns gitTest$ git reset HEAD~2
returns@returns gitTest$ git status
On branch master
Untracked files:
  (use "git add <file>..." to include in what will be committed)

	b
	c
	d
	e

nothing added to commit but untracked files present (use "git add" to track)

returns@returns gitTest$ git add b
returns@returns gitTest$ git commit -m 'add b'
[master 2bcd161] add b
 1 file changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 b
returns@returns gitTest$ git add c
returns@returns gitTest$ git commit -m 'add c'
[master 189d818] add c
 1 file changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 c
returns@returns gitTest$ git add d
returns@returns gitTest$ git commit -m 'add d'
[master ead86a4] add d
 1 file changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 d
returns@returns gitTest$ git add e
returns@returns gitTest$ git commit -m 'add e'
[master 0b58e8b] add e
 1 file changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 e
returns@returns gitTest$ git log
commit 0b58e8b6ed1a7769ae3300bf1bbea76a99a909c8
Author: DoO <returns@lycos.co.kr>
Date:   Tue Dec 8 10:57:33 2015 +0900

    add e

commit ead86a4ef2597439628f926c0679fe679745059c
Author: DoO <returns@lycos.co.kr>
Date:   Tue Dec 8 10:57:29 2015 +0900

    add d

commit 189d818ed4c12f0c4e5ed033317237bb64eedbd6
Author: DoO <returns@lycos.co.kr>
Date:   Tue Dec 8 10:57:26 2015 +0900

    add c

commit 2bcd1612374e0145351d8231556caaf274071d4b
Author: DoO <returns@lycos.co.kr>
Date:   Tue Dec 8 10:57:19 2015 +0900

    add b

commit 88ed773851051dba4662c6a363f0ad14020096ba
Author: DoO <returns@lycos.co.kr>
Date:   Tue Dec 8 09:56:55 2015 +0900

    test2
```
