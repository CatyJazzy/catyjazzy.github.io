---
title: Git의 내부동작
date: 2024-07-15 10:00:00 +0900
categories: [Computer Science]
tags: [Git, Github]
---

# Git에 대하여

## Git이란 무엇인가?

git은 **`분산 버전관리 시스템`**이다. 2005년에 **리누스 토발즈에 의해 개발**되었다. 리눅스의 창시자로서 알게 되었는데, git의 창시자이기도 하다. 한 명의 개발자가 사용한다면 변경사항을 추적하고 기록하는 목적으로 사용할 수 있으며, 여러 명의 개발자가 사용한다면 각자의 작업과 공동의 작업물을 관리하는 데 사용할 수 있다. 

### 버전관리의 필요성

git이 왜 필요할까? 필요성은 너무나 명확해 보인다. 한 번이라도 발표를 포함한 팀 프로젝트의 경험이 있다면, 파워포인트를 공동 작업해본 경험이 있을 것이다. 이때 “피피티_초안” “피피티_최종” “피피티_최최종” “피피티_찐최종” 등 어떻게든 이름을 달리하여 최종본임을 명시하는 경우가 많다. 

일명 “찐최종”이 되기 전 중간에 각 버전을 저장하는 이유를 생각해보자. 

- **현재 진행상태를 기록하여 공유**하기 위해
- 이후 진행 과정에서 예기치 못한 오류로 **작업물이 훼손되는 상황에 대비**하여
- 이후 다양한 시도를 해보고 필요할 때 **특정 타이밍에 완성된 버전으로 돌아오기 위하여**

개발 과정도 마찬가지다. 그 많은 소스코드, 다양한 패키지, 그 안에서도 사용되는 여러 가지 버전과 세팅 등을 체계적으로 관리하지 않으면 위의 3가지를 해낼 수 없다. 버전관리시스템은 이렇게 직관적으로도 알 수 있는 필요성에 의해서 등장했다. 

### git의 위대함

git이 필요한 기능을 알맞게 제공해준다는 것은 알았으나 git만의 장점을 알지는 못했다. [SVN이라는 유사한 서비스](https://jindevelopetravel0919.tistory.com/208)를 설명하는 글을 통해 이를 더 명확하게 알 수 있었다. SVN은 SubVersion의 약자로 중앙집중관리식 소스 관리 툴이다. 각각의 개발자가 하나의 중앙 저장소에 commit하는 방식이다. 

![[https://jindevelopetravel0919.tistory.com/208](https://jindevelopetravel0919.tistory.com/208)](/assets/img/posts/git-cdn.png)

[https://jindevelopetravel0919.tistory.com/208](https://jindevelopetravel0919.tistory.com/208)

git과의 가장 큰 차이점은 바로 **`작업 내용을 중앙 서버에 바로 반영하는지의 여부`**다. 작업 내용을 바로 중앙 서버에 업로드하지 않음으로써 얻는 이점은 무엇일까?

- 업로드 때만 네트워크를 사용하고, 이전의 작업들과 이에 대한 기록을 로컬에서 네트워크 없이 남길 수 있다.
- 동시 업로드 시의 충돌이 더 적다. 브랜치와 merge의 별도 작업을 통해 충돌을 방지한다.

이외에도 git은 히스토리 관리 기능이 잘 제공되어, 필요한 버전에 대한 정보를 얻고 관리할 때 용이하다.

> 하지만, 위의 git의 강점은 **내부의 동작 원리**를 알아야 이해할 수 있다. git의 동작 원리를 알아보자.
> 

# Git 동작 원리

## 전체적인 영역

git은 크게 **working directory (작업하고 있는 폴더)와 remote repository (원격 저장소)로 나뉜다.** 이때 git은 SVN과 같이 working directory → remote repository로 바로 업데이트 되지 않고 **working directory 내부에서 더 세부적인 단계를 거친다.** 

아래의 도식을 통해 전체적인 구조를 참고해보자. 

![(출처: [https://inpa.tistory.com/entry/GIT-⚡️-개념-원리-쉽게이해](https://inpa.tistory.com/entry/GIT-%E2%9A%A1%EF%B8%8F-%EA%B0%9C%EB%85%90-%EC%9B%90%EB%A6%AC-%EC%89%BD%EA%B2%8C%EC%9D%B4%ED%95%B4))](/assets/img/posts/git-phase.png)

(출처: [https://inpa.tistory.com/entry/GIT-⚡️-개념-원리-쉽게이해](https://inpa.tistory.com/entry/GIT-%E2%9A%A1%EF%B8%8F-%EA%B0%9C%EB%85%90-%EC%9B%90%EB%A6%AC-%EC%89%BD%EA%B2%8C%EC%9D%B4%ED%95%B4))

## 핵심 명령어를 통해 동작 과정 살펴보기

핵심 명령어를 다음의 테이블로 간략하게 정의하고, 각 명령어 실행 시의 동작 과정을 통해 전체 흐름을 살펴보고자 한다.

| **git init** | 깃 저장소를 초기화한다.  |
| --- | --- |
| **git add** | staging area에 변경내용을 추가한다. |
| **git commit** | staging area의 변경 내용을 묶어서 정의한다. |
| **git push** | 로컬의 변경 사항을 서버로 업로드한다.  |

### git init

git init은 ***깃 저장소를 초기화***한다. ‘초기화’의 의미는 일반적인 디렉토리의 상태와 달리 이제 변경 사항들을 추적하고 인지한다는 뜻이다. 가장 큰 물리적 변화는 **`.git** 파일이 생성`된다는 점이다. 

.git 폴더의 구성을 알면, git이 동작하는 세부적인 방식을 알기 편하다. 필자의 경우, 1일차 미션을 수행했던 디렉토리에서 탐구해보겠다. (이 파트는 [블로그](https://tecoble.techcourse.co.kr/post/2021-07-08-dot-git/)를 이해하며 인용했다. 모든 것을 살펴보지는 않고, 동작 원리 이해에 도움이 되는 요소 위주로만 구성한다.)

![Untitled](/assets/img/posts/git-terminal-1.png)

---

먼저 **`objects 폴더`**다. 

![Untitled](/assets/img/posts/git-terminal-2.png)

알 수 없는 2글자의 폴더들과, 각 폴더가 가지고 있는 이상한 38글자를 확인할 수 있다. objects는 **실제 파일에 있는 값들을 해싱**하여 2자의 폴더명과 36자의 파일명으로 저장한다. 실제 파일에 있는 코드가 부분적으로 수정되면 새로운 해시값이 생기므로 파일의 차이를 인지하기에 용이하다. 

이때 objects에 저장되는 “파일”은 크게 3가지 카테고리로 나눌 수 있다. 

1) **Blob**: 파일의 메타 데이터를 제외하고 데이터 자체만을 저장한다. 같은 코드를 가진 파일이 여러 개 있어도 하나의 blob파일로 구성된다. 저장하는 파일은 소스코드에 국한되지 않고 이미지와 같은 다양한 형태를 아우른다.

2) **Tree**: 폴더 구조를 관리한다. tree는 파일 식별자, 파일 데이터 해시값, 파일의 이름 등이 저장된다. 폴더에 파일과 또다른 폴더로 이루어지는 것처럼, tree도 blob과 또다른 blob으로 이루어진다. 파일 식별자는 100644(읽기 파일blob), 100755(실행 파일 blob), 040000(디렉토리(tree))로만 구성된다.

* 참고로 blob에서 알아보았듯이 저장된 파일 데이터 해시값이 같다면 파일 내용이 동일함을 의미한다.

3) **commit**: 커밋을 할 때마다 커밋 파일을 저장한다. git으로 관리하고 있는 루트 tree의 해시값, author, commiter, 커밋 메시지의 정보를 담으며 parent에 직전 커밋의 해시값을 함께 저장한다. * 이는 이후 git commit에서 다 자세하게 알아볼 수 있다.

---

다음은 **`refs 폴더`**다.

이는 내 디렉토리 상의 캡처보다 gistory를 사용한 참고 블로그에서의 캡처본을 따왔다. gistory는 ~다.

![(출처: [https://tecoble.techcourse.co.kr/post/2021-07-08-dot-git/](https://tecoble.techcourse.co.kr/post/2021-07-08-dot-git/))](/assets/img/posts/git-terminal-3.png)

(출처: [https://tecoble.techcourse.co.kr/post/2021-07-08-dot-git/](https://tecoble.techcourse.co.kr/post/2021-07-08-dot-git/))

git에서 관리하는 브랜치 정보가 들어있으며 로컬과 원격 저장소를 담당하는 폴더로 구분되어있다. 구체적으로는 **브랜치별 마지막 커밋의 해시값을 저장**한다. main (구 master)브랜치는 git init시 자동으로 만들어진다. 

- 내 디렉토리에서의 캡처
    
    ![Untitled](/assets/img/posts/git-index.png)
    

---

다음은 **`index`** 파일이다.

index 파일은 이후 살펴볼 git add 명령어를 통해 staging area에 올라온 파일들을 저장한다. 자세한 구성은 아래를 참고하자. index 파일을 통해 수정된 파일이 있는지 확인하고, 커밋할 파일을 판단하기도 한다.

![(출처: [https://tecoble.techcourse.co.kr/post/2021-07-08-dot-git/](https://tecoble.techcourse.co.kr/post/2021-07-08-dot-git/))](/assets/img/posts/git-internals/Untitled%206.png)

(출처: [https://tecoble.techcourse.co.kr/post/2021-07-08-dot-git/](https://tecoble.techcourse.co.kr/post/2021-07-08-dot-git/))

---

이 정도면 최소한의 준비는 끝났다.

### git add

개념적으로는 변경된 사항들 중 일부 또는 전체를 **staging area에 올린다**. 이는 각종 add 명령어 옵션을 통해 세부적으로 조작할 수 있다. 실제 동작은 **git 인덱스의 내용과 비교하여 로컬에서 변경된 것들을 인덱스에 반영한다**. 

### git commit

개념적으로는 **staging area의 내용을 바탕으로 파일의 변화를  영구적으로 레포지토리에 기록**한다. 이때 [스냅샷📸](https://thebook.io/080212/0131/) 방식을 사용하는데, 커밋 시점의 모든 파일을 복사하는 것이 아니라 변경된 부분만 찾아 해당 내용을 저장하는 것이다. (= 변화된 부분만 찰칵! 사진 찍는 것이다.)

* *스냅샷을 사용하는 이유는 매번 파일을 복사하면 변경되지 않은 같은 내용도 반복해서 저장하기 때문에 용량의 부담이 생기기 때문이다. 또한, 수정된 사항을 확인하기 위해 매번 직접 찾아야 하므로 불편하다.* 

그럼 스냅샷을 통해 변경된 사항을 어떻게 찍는가? ‘변경’이라 하면 기본적으로 **비교 기준과 대상**이 있어야 하는데 git은 이전 커밋과 staging area를 비교한다. 이때, HEAD라는 포인터를 통해 가장 최신의 커밋을 저장하여 활용한다. 커밋이 이루어졌을 때는, 가장 최신 커밋을 부모로 하여 Linked List 형식으로 연결한다. 

![[https://thebook.io/080212/0131/](https://thebook.io/080212/0131/)](/assets/img/posts/git-list.png)

[https://thebook.io/080212/0131/](https://thebook.io/080212/0131/)

**그런데 무엇이 Linkded List 형식으로 연결되는 것일가?** git commit을 통해 커밋하면 아래와 같은 개체를 만든다. 이는 파일이 3개 있는 디렉토리 하나를 관리한다고 가정했을 때의 도식이다.

![(출처: [https://git-scm.com/book/ko/v2/Git-브랜치-브랜치란-무엇인가](https://git-scm.com/book/ko/v2/Git-%EB%B8%8C%EB%9E%9C%EC%B9%98-%EB%B8%8C%EB%9E%9C%EC%B9%98%EB%9E%80-%EB%AC%B4%EC%97%87%EC%9D%B8%EA%B0%80))](/assets/img/posts/git-tree.png)

(출처: [https://git-scm.com/book/ko/v2/Git-브랜치-브랜치란-무엇인가](https://git-scm.com/book/ko/v2/Git-%EB%B8%8C%EB%9E%9C%EC%B9%98-%EB%B8%8C%EB%9E%9C%EC%B9%98%EB%9E%80-%EB%AC%B4%EC%97%87%EC%9D%B8%EA%B0%80))

우리가 앞에서 살펴보았던, blob/tree개체들 → 루트 tree (깃에서 관리하고 있는 최상위 폴더) 개체 → 커밋 개체의 구조를 확인할 수 있다. 이때 Linked List로 연결되는 것은 바로 commit 개체다. 아래와 같이 parent로 직전 최신 커밋을 가리키며 연결되는 것이다.

![Untitled](/assets/img/posts/git-prev-tree.png)

* 이후에 살펴보겠지만 branch는 이 커밋 사이들을 이동할 수 있는 포인터와 같은 역할을 한다.

### git push

**지정한 원격 저장소에 로컬의 내용을 업데이트**한다. 세부적으로는 `git push 원격저장소별칭 브랜치명` 형태로 명령어를 사용한다. ~~(나의 경우 의식 없이 git push origin main을 사용했는데 . . . 이번 학습 정리를 통해 제대로 알게 되었다.)~~

특히`—-set-upstream`또는 `-u` 옵션을 통해 로컬 브랜치와 원격 브랜치를 연결하여 (=특정 원격 브랜치를 해당 로컬 브랜치의 업스트림으로 지정하여)  git push하면, 저장소별칭을 지정하지 않고 업스트림으로 지정되어 있는 원격 브랜치로 변경사항이 업로드된다. 

# Branch에 대해

### 브랜치의 필요성

git commit에서도 살펴봤듯이 **브랜치는 커밋 사이를 쉽게 이동할 수 있는 포인터와 같은 개념**이다. 추상적으로는 이렇고, 현실에서는 여러 개발자들과 협업할 때 독립된 작업을 수행하는 공간으로 이해하는 경우가 많다. 

![(출처: [https://6mini.github.io/git/2022/06/12/branch/](https://6mini.github.io/git/2022/06/12/branch/))](/assets/img/posts/git-branch.png)

(출처: [https://6mini.github.io/git/2022/06/12/branch/](https://6mini.github.io/git/2022/06/12/branch/))

브랜치를 사용하는 이유는 위의 그림과 같이 독립적인 작업을 수행한 후, 서로 합쳐서 하나의 브랜치에 여러 개의 작업을 반영할 수 있기 때문이다. 

### 기본 흐름

git 교과서를 통해 우리가 앞서 살펴본 HEAD포인터와 커밋 개체 간의 이동을 위주로 branch개념을 살펴보자. 일단 git init을 이후 브랜치를 추가하지 않으면 **아래와 같이 main(master)브랜치만 존재**한다. **HEAD는 최신 커밋을 가리킨다.** 

![Untitled](/assets/img/posts/git-branch-2.png)

이때 `git branch 브랜치명` 명령어를 통해 브랜치를 추가해보자. 

```bash
git branch testing
```

**새롭게 생성된 브랜치**도 HEAD가 가리키고 있던 **최신 커밋을 가리킨다.** 

![Untitled](/assets/img/posts/git-branch-3.png)

![Untitled](/assets/img/posts/git-branch-4.png)

```bash
git checkout testing
```

위의 명령어를 통해 브랜치를 testing으로 이동해보자. 어떤 일이 벌어질까? 

1) HEAD가 testing을 기준으로 가리키게 되며, 

2) 이후 커밋의 업데이트는 testing기준으로 반영된다.

1) HEAD가 testing을 가리킨다. 

![Untitled](/assets/img/posts/git-branch-5.png)

2) testing 브랜치 이동 후 새로운 커밋을 한 모습이다. main은 f30ab커밋에 그대로 머물러있다.

![Untitled](/assets/img/posts/git-branch-6.png)

이때 main(matser)브랜치로 다시 이동하면 HEAD가 main이 가리키고 있던 이전 커밋 f30ab을 가리키게 되고, working directory의 파일도 그 시점으로 돌려둔다. 

main 브랜치로 이동한 이후에도 계속 커밋하면 결국 특정 시점에서 갈라지는 브랜치가 생기기 마련이다. 이 때는 두 브랜치를 merge한다. 

![Untitled](/assets/img/posts/git-branch-7.png)

갈라지는 브랜치는 아래의 명령어로 할 수 있다. 나의 경우, 1차 hello출력 이후 학년을 소개하는 버전과 이름을 소개하는 버전을 각각 다른 브랜치에서 커밋하였는데 아래와 같이 확인할 수 있었다.

```bash
git log --oneline --decorate --graph --all
```

![Untitled](/assets/img/posts/git-log-terminal.png)

### 갈라진 브랜치와 merge

이렇게 갈라진 브랜치는 merge 명령어를 통해 합쳐줄 수 있다. 기본적으로 merge는 **합칠 브랜치에서 ← 합쳐질 브랜치를 통합시키는 방식**으로 실행하면 된다. merge의 방식은 케이스에 따라 나뉠 수 있다. 아래의 2가지 경우를 살펴보자.

**1) hotfix와 main(master)를 merge할 때**

![Untitled](/assets/img/posts/hotfix-branch.png)

hotfix를 통해 갑자기 등장한 긴급한 문제를 해결했다고 해보자. iss53은 53번째 이슈를 해결하기 위해 만들었던 브랜치다. hotfix는 main브랜치로 돌아가 긴급한 문제를 해결하기 위해 만든 브랜치다. C4버전에서 긴급한 문제를 해결했다. 

이제 main과 hotfix를 merge하고 싶어 아래와 같은 명령어를 실행한다

```bash
git merge hotfix
```

이 케이스의 경우 **`fast-forward`**방식으로 합쳐진다. hotfix가 가리키는 C4의 부모가 main의 C2이기 때문에 단순히 합칠 브랜치(main)의 포인터가 최신 커밋으로 이동하게 된다. 도식으로 확인하면 아래와 같다. 이후 `git branch -d hotfix` 를 통해 삭제하면 깔끔해진다. 

![Untitled](/assets/img/posts/hotfix-branch2.png)

**2) iss53과 main(master)를 merge할 때**

hotfix와 다르게 main(mater)와는 별개의 경로로 커밋을 하고 있던 iss53브랜치가 있었다. iss53 브랜치로 이동한 이후에도 계속 커밋이 있었다고 가정해보자.

![Untitled](/assets/img/posts/merge1.png)

이때 합칠 main(mater)브랜치로 이동하여 merge하면 어떻게 될까?

```bash
git merge iss53
```

이 케이스의 경우 **`3-way-Merge`**방식으로 합쳐진다. 최신 커밋으로 브랜치 포인터를 옮겨서 합치는 것이 아니라, 합쳐진 결과를 새로운 커밋으로 만든 후에 합칠 브랜치가 해당 커밋을 가리키도록 이동한다. (이 새로운 커밋은 여러 개의 부모를 갖게 된다.)

![Untitled](/assets/img/posts/merge2.png)

* ***보통 우리가 고통 받는 충돌은 이때 발생한다.*** 3-way Merge에 실패하는 것이다. Merge하는 두 브랜치에서 같은 파일의 일부를 동시에 수정하고 합쳤다면 git은 이를 처리하지 못하고 개발자가 변경사항 충돌을 직접 처리할 수 있도록 알린다. 아래의 형식으로 표시해준다. HEAD 버전은 merge 명령을 실행할 때의 브랜치 내용, 아래쪽은 합쳐질 대상의  브랜치 내용이다. 

```bash
<<<<<<< HEAD:index.html
<div id="footer">contact : email.support@github.com</div>
=======
<div id="footer">
 please contact us at support@github.com
</div>
>>>>>>> iss53:index.html
```

위의 형식에서 변경사항을 직접 합쳐주고 <<<<<<<, ======= 등의 표식도 지워준다.

```html
<div id="footer">
please contact us at email.support@github.com
</div>
```

**** merge외에도 rebase를 통해 브랜치를 합칠 수 있다.***

[Git - Rebase 하기](https://git-scm.com/book/ko/v2/Git-브랜치-Rebase-하기)

### 분산형 환경에서의 branch 관리

분산형 환경, **여러 개발자가 협업할 때 git을 활용하는 워크플로우는 다양하다.** 개발자의 수, 프로젝트의 특성, 협업 방식 등에 따라 여러 방식을 선택할 수 있다. 이 내용들은 아래의 링크에 예시를 통해 매우 상세하게 설명되어있으니 꼭 확인해보자.

[Git - 프로젝트에 기여하기](https://git-scm.com/book/ko/v2/분산-환경에서의-Git-프로젝트에-기여하기)

**`pull request`란 무엇인가?** 위에서 언급했듯 워크플로우 선택지는 다양하다. 모든 개발자가 SVN방식처럼 중앙집중형 방식으로 저장소에 쓰기 권한이 있을 수도 있고, 별도의 프로젝트 관리자만이 쓰기 권한을 가지게도 할 수 있다. 이때 ***pull request는 모든 변경 사항들이 병합되는 상위 흐름에서 합칠 때 각자의 작업(수정) 내용을 알리고, 수정한 내용을 적용해달라는 요청을 보내는 것***이다.

![(출처: [https://velog.io/@minrami1115/PRPull-Request란](https://velog.io/@minrami1115/PRPull-Request%EB%9E%80))](/assets/img/posts/git-pr.png)

(출처: [https://velog.io/@minrami1115/PRPull-Request란](https://velog.io/@minrami1115/PRPull-Request%EB%9E%80))

**** git fetch와 git pull의 차이***

git 문서를 읽다보면 두 명령이 혼동될 수 있다. 두 명령 모두 원격 저장소의 변경 사항을 로컬에 가져온다. 그러나 pull은 현재 작업 중이었던 로컬 브랜치와 merge까지 실행하고, fetch는 원격 저장소의 내용을 가져오기만 할 뿐 merge하진 않는다. 따라서 fetch의 경우 별도로 병합 과정을 거쳐야 한다.

# Git과 Github은?

**`github`**은 **git 원격 저장소를 유지하고 관리해주는 서비스**다. 유사한 서비스로 gitlab, gitea 등이 있다. 즉 원격 저장소를 내가 늘 해왔던 것처럼 꼭 github에서 만들지 않아도 된다는 것인데, 아래와 같이 로컬 서버에서 원격저장소를 생성한 예시를 볼 수 있었다.

[나만의 git 서버 만들기 - (3) 원격저장소 생성하기](https://velog.io/@kimjjs100/나만의-git-서버-만들기-3-원격저장소-생성하기)

day2미션 때 설치했던 가상 머신을 활용해서 직접 git 서버를 만들어보는 것도 좋을 것 같다 🤔

# Reference

[https://yanacoding.tistory.com/4](https://yanacoding.tistory.com/4)

[https://jindevelopetravel0919.tistory.com/208](https://jindevelopetravel0919.tistory.com/208)

[https://inpa.tistory.com/entry/GIT-⚡️-개념-원리-쉽게이해](https://inpa.tistory.com/entry/GIT-%E2%9A%A1%EF%B8%8F-%EA%B0%9C%EB%85%90-%EC%9B%90%EB%A6%AC-%EC%89%BD%EA%B2%8C%EC%9D%B4%ED%95%B4)

[https://it-eldorado.tistory.com/4](https://it-eldorado.tistory.com/4)

[https://git-scm.com/book/ko/v2/Git의-내부-Git-개체](https://git-scm.com/book/ko/v2/Git%EC%9D%98-%EB%82%B4%EB%B6%80-Git-%EA%B0%9C%EC%B2%B4)

[https://velog.io/@goodenough/Git-Git-이란](https://velog.io/@goodenough/Git-Git-%EC%9D%B4%EB%9E%80)

[https://tecoble.techcourse.co.kr/post/2021-07-08-dot-git/](https://tecoble.techcourse.co.kr/post/2021-07-08-dot-git/)

[https://thebook.io/080212/0131/](https://thebook.io/080212/0131/)

[https://6mini.github.io/git/2022/06/12/branch/](https://6mini.github.io/git/2022/06/12/branch/)

[https://git-scm.com/book/ko/v2/Git-브랜치-브랜치란-무엇인가](https://git-scm.com/book/ko/v2/Git-%EB%B8%8C%EB%9E%9C%EC%B9%98-%EB%B8%8C%EB%9E%9C%EC%B9%98%EB%9E%80-%EB%AC%B4%EC%97%87%EC%9D%B8%EA%B0%80)

[https://blog.naver.com/seek316/222344170901](https://blog.naver.com/seek316/222344170901)

[https://6mini.github.io/git/2022/06/12/branch/](https://6mini.github.io/git/2022/06/12/branch/)

[https://velog.io/@minrami1115/PRPull-Request란](https://velog.io/@minrami1115/PRPull-Request%EB%9E%80)

[https://wayhome25.github.io/git/2017/07/08/git-first-pull-request-story/](https://wayhome25.github.io/git/2017/07/08/git-first-pull-request-story/)

[https://docs.github.com/ko/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/about-pull-requests](https://docs.github.com/ko/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/about-pull-requests)