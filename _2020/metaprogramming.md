---
layout: lecture
title: "Metaprogramming"
details: build systems, dependency management, testing, CI
date: 2019-01-27
ready: true
video:
    aspect: 56.25
    id: _Ms1Z4xfqv4
---

우리가 메타프로그래밍(metaprogramming)이라고 할 때, 이 용어는 무얼 뜻하는 것일까요?  
이는 코드를 작성하는 것이나 더 효율적으로 작업하는 것보다는, _"프로세스(process)"_ 에 더 관련된 것들을 두루 일컫는 총칭이라고 할 수 있습니다.  
이 강의에서, 우리는 코드를 빌드하고 테스트하며, 의존성([dependency](https://en.wikipedia.org/wiki/Dependency_injection#Constructor_injection))들을 관리하기 위한 체계들을 살펴볼 것입니다.  
이러한 것들은 학생으로서의 여러분의 일상에서 마치 별로 중요하지 않은 것처럼 보일 지도 모르지만, 여러분이 인턴십을 통해서 더 큰 코드베이스([code base](https://en.wikipedia.org/wiki/Codebase))와 상호작용하거나 일단 "실제 세계"에 발을 들이게 되면, 이를 어디서든 보게 될 것입니다.  
이 강의의 목적을 위해 우리가 사용하고 있는 정의는 아니지만, 우리는 "메타프로그래밍"이 "[프로그램들을 대상으로 작동하는 프로그램들](https://en.wikipedia.org/wiki/Metaprogramming)" 또한 의미할 수 있음에 유의해야 합니다.

# [빌드 시스템(Build systems)](http://developinghappiness.com/?p=26)

문서를 [Latex(레이텍 또는 라텍)](https://namu.wiki/w/LaTeX)으로 작성할 때, 문서를 작성하기 위해 실행해야 하는 명령어는 무엇입니까?  
[벤치마크](https://namu.wiki/w/%EB%B2%A4%EC%B9%98%EB%A7%88%ED%81%AC)를 실행하고, 그 결과를 그래프로 그리고, 그 그래프를 문서에 삽입할 때는요?  
또는 사용하고 있는 클래스 안에 제공되어 있는 코드를 컴파일하고 테스트를 실행할 때는 어떤 명령어를 사용합니까?

대부분의 프로젝트들에는, 그것들이 코드를 포함하든 포함하지 않든, "**빌드 프로세스**"라고 하는 것이 있습니다. 이는 인풋(Input)에서 아웃풋(Output)으로 가기 위해서 수행할 필요가 있는 몇몇 순차적인 작업들을 뜻합니다.  
종종, 그 프로세스에는 많은 단계들과 많은 [브랜치(branch)](https://backlog.com/git-tutorial/kr/stepup/stepup1_1.html)들이 있을 수도 있습니다.  
이 도표를 생성하기 위해서 이걸 하고, 저 결과들을 생성하기 위해 저걸 하고, 최종 문서를 만들어 내기 위해서 또 뭔가 다른 걸 해야하는 것처럼 말이죠.  
우리가 이 강의에서 보아온 아주 많은 것들과 마찬가지로, 여러분은 제일 처음으로 이러한 골칫거리에 마주한 사람들이 아니고, 운 좋게도 여러분들을 도와줄 많은 도구들이 존재합니다!

이것들은 보통 "**빌드 시스템(build systems)**"이라고 불리고, _많은_ 것들이 있습니다.  
여러분이 무엇을 사용하는지는 당면한 과제, 선호하는 프로그래밍 언어, 그리고 프로젝트의 규모에 달려 있습니다.  
그렇지만 핵심적인 면에서는 모두 아주 비슷합니다.
여러분은 어떤 것으로부터 다른 것을 만들어 내기 위한 많은 _의존성(dependencies)_, _타겟_, 그리고 _규칙_ 들을 정의하게 됩니다.  
여러분이 빌드 시스템에게 여러분이 특정 타겟을 원한다는 것을 전하면, 빌드 시스템이 하는 일은 그 타겟의 [이행성을 띠는(transitive)](https://terms.naver.com/entry.nhn?docId=856345&cid=50376&categoryId=50376) 모든 의존성들을 찾아내고서는, 최종 타겟이 만들어지는 때까지 계속해서 중간의 타겟들을 만들어내는 규칙들을 적용하는 것입니다.
이상적으로는, 빌드 시스템은 의존성이 바뀌지 않은 타겟들과, 이전의 빌드에 의한 결과가 이미 사용가능한 경우에 대해서는, 불필요하게 규칙들을 실행하지 않는 방식으로 이를 수행합니다.

`make`는 가장 흔한 빌드 시스템 중 하나이고, 거의 모든 [UNIX](https://namu.wiki/w/%EC%9C%A0%EB%8B%89%EC%8A%A4)-기반 컴퓨터에 설치되어 있습니다. `make`는 결점이 있기는 하지만, 간단한 ~ 중간 정도의 복잡성을 지닌 프로젝트들에는 꽤 잘 작동합니다.  
여러분이 `make`를 실행하면 이것은 현재 디렉토리에 있는 `Makefile`이라고 하는 파일을 참조합니다.  
모든 타겟들과 의존성들, 그리고 규칙들은 그 파일 안에 정의되어 있습니다. 함께 하나 살펴보죠:

```make
paper.pdf: paper.tex plot-data.png
	pdflatex paper.tex

plot-%.png: %.dat plot.py
	./plot.py -i $*.dat -o $@
```

이 파일 안의 각각의 명령(directive)은 우변을 이용하여 좌변을 만들어 내는 방법에 대한 규칙입니다.
다르게 말하면, 우변에 이름지어진 것들은 의존성들이고, 좌변은 타겟입니다.  
들여쓰기된 블럭은 의존성들로부터 타겟을 만들어 내기 위한 프로그램들의 순서입니다.  
`make`에서는, 첫번째 명령(directive)은 최종 타겟의 초기값(default) 또한 정의합니다. 만약 여러분이 어떠한 인자도 없이 `make`를 실행한다면, 이 초기값이 바로 `make`가 만들어낼 최종 타겟이 되는 것입니다.  
대신에 여러분이 `make plot-data.png`와 같은 무언가를 실행시킨다면 `make`는 그 타겟을 빌드할 것입니다.

우변의 규칙(rule)에 있는 `%`는 하나의 "패턴"이고, 이것은 좌변과 우변에 있는 같은 문자열을 매치시킬 것입니다.  
예를 들면, 만약 타겟 `plot-foo.png`가 요청되면 `make`는 `foo.dat`과 `plot.py` 의존성들이 있는지 찾을 것입니다.  
자 이제 만약 우리가 `make`를 빈 소스 디렉토리에서 실행하면 무슨 일이 일어나는지 살펴봅시다.

```console
$ make
make: *** No rule to make target 'paper.tex', needed by 'paper.pdf'.  Stop.
```

`make`는 유용하게도, `paper.pdf`를 빌드하기 위해서는 `paper.tex`가 필요하며, 그 파일을 만들기 위한 방법을 알려주는 규칙을 가지고 있지 않다는 사실을 말해주고 있습니다.  
한 번 만들어 봅시다!

```console
$ touch paper.tex
$ make
make: *** No rule to make target 'plot-data.png', needed by 'paper.pdf'.  Stop.
```

흠, 흥미롭네요, `plot-data.png`를 만드는 규칙은 *존재*합니다. 하지만 그 규칙은 패턴 규칙(pattern rule)입니다.(구체적으로 어떻게 타겟 파일을 만들어내야 하는지 명시되어 있지 않음)  
소스 파일([foo](https://en.dict.naver.com/#/entry/enko/63f44f50ac9e4936a815f4be75f93d60).dat)이 존재하지 않기 때문에, `make`는 단지 파일을 만들 수 없다고만 이야기하고 있습니다.  
한 번 모든 파일을 다 만들어 보죠:

```console
$ cat paper.tex
\documentclass{article}
\usepackage{graphicx}
\begin{document}
\includegraphics[scale=0.65]{plot-data.png}
\end{document}
$ cat plot.py
#!/usr/bin/env python
import matplotlib
import matplotlib.pyplot as plt
import numpy as np
import argparse

parser = argparse.ArgumentParser()
parser.add_argument('-i', type=argparse.FileType('r'))
parser.add_argument('-o')
args = parser.parse_args()

data = np.loadtxt(args.i)
plt.plot(data[:, 0], data[:, 1])
plt.savefig(args.o)
$ cat data.dat
1 1
2 2
3 3
4 4
5 8
```

이제 `make`를 실행해보면 어떻게 될까요?

```console
$ make
./plot.py -i data.dat -o plot-data.png
pdflatex paper.tex
... lots of output ...
```

이제 보면, `make`가 PDF 파일을 하나 만들어 주었습니다. `make`를 다시 한 번 더 실행시키면 어떻게 될까요?

```console
$ make
make: 'paper.pdf' is up to date.
```

`make`가 아무 일도 하지 않았습니다. 왜냐고요? 왜냐면 그럴 필요가 없었기 때문이죠.
`make`는 모든 이전에 빌드된 타겟들이 나열된 의존성들에 대해서 최신의 상태라는 것을 체크했습니다.  
우리는 `paper.tex`를 수정하고 `make`를 재실행함으로써 이를 테스트해볼 수 있습니다:

```console
$ vim paper.tex
$ make
pdflatex paper.tex
...
```

`make`가 `plot.py`는 다시 실행하지 _않았다는_ 사실에 주목하세요, 그럴 필요가 없었으니까요 ( `plot-data.png`의 의존성들 중 어느 것도 바뀌지 않았기 때문이죠.)

# 의존성 관리(Dependency management)

더 거시적인 수준에서는, 여러분의 소프트웨어 프로젝트들은 또 다른 프로젝트들에 의존하고 있을 수 있습니다.  
여러분은 설치된 프로그램(예: `파이썬(python)`), 시스템 패키지(예: `openssl`), 또는 여러분이 사용하는 프로그래밍 언어 안에 있는 라이브러리(예: `matplotlib`)에 의존하고 있을지도 모릅니다.  
요즘에는, 대부분의 의존성들이 _레포지토리(repository)_ 를 통해서 가능할 텐데, 레포지토리는 단일한 공간에서 수많은 의존성들을 관리하고, 그것들을 설치할 수 있는 편리한 방법을 제공합니다.  
레포지토리 예시:

-   우분투 시스템 패키지(Ubuntu system packages)를 위한 [우분투 패키지 레포지토리(the Ubuntu package repository)](https://packages.ubuntu.com) - `apt` [툴(tool)](http://taewan.kim/tip/apt-apt-get/)을 통해 접근할 수 있음
-   프로그래밍 언어 루비(Ruby) 라이브러리들을 위한 [루비젬(RubyGem)](https://rubygems.org)
-   파이썬(Python) 라이브러리들을 위한 [파이파이(Pypi)](https://pypi.org)
-   [아치 리눅스](https://www.archlinux.org) 사용자-기여 패키지들을(Arch Linux user-contributed packges) 위한 [아치 사용자 레포지토리(Arch User Repository)](https://aur.archlinux.org)

이러한 레포지토리들과 상호작용하는 정확한 방법은 레포지토리마다 다르고 툴마다 다르기 때문에, 우리는 이 강의에서 특정 레포지토리의 너무 세부적인 사항까지는 들어가지 않을 것입니다. 우리가 _다룰_ 것은 레포지토리들이 모두 사용하는 몇몇 공통적인 용어입니다.  
그 중 첫 번째는 _버저닝(versioning)_ 입니다.  
다른 프로젝트들이 의존하는 대부분의 프로젝트들은 모든 배포(release)와 함께 _버젼 넘버(version number)_ 를 발행합니다. 보통 8.1.3 또는 64.1.20192004 와 같은 것입니다. 항상 그런 것은 아니지만, 보통 그것들은 숫자입니다.  
버전 넘버는 많은 목적에 기여하는데, 그 중 가장 중요한 것 중 하나는 소프트웨어가 계속 작동할 것이라는 것을 보장하는 것입니다.  
예를들면 제가 특정한 함수의 이름을 바꾼 라이브러리의 새 버전을 배포한다고 한 번 상상해 보세요. 제가 그 업데이트를 배포한 이후에 누군가가 제 라이브러리에 의존하는 소프트웨어를 빌드하려고 시도한다면, 그 빌드는 실패할 지도 모릅니다, 왜냐하면 그 빌드는 더 이상 존재하지 않는 함수를 호출할 수도 있기 때문이죠!  
버저닝(versioning)은 프로젝트가 다른 프로젝트의 어떤 특정한 버전, 또는 버전들의 범위에 의존하는지를 나타내도록 함으로써 이러한 문제를 해결하려 시도합니다.  
그런 식으로, 비록 라이브러리에 변화가 생길 지라도, 그 라이브러리에 의존하는 소프트웨어는 그 라이브러리의 더 예전 버전을 사용함으로써 계속 빌드할 수 있게 됩니다.

그렇긴 하지만 완벽하진 않죠!  
만약 제가 보안 업데이트를 하는데, 그 새로운 버전이 제 라이브러리의 퍼블릭 인터페이스(public interface, "[API](https://ko.wikipedia.org/wiki/API)")를 바꾸지 _않고_, 오래된 버전에 의존했던 어떤 프로젝트도 즉시 사용할 수 있는 것이라면 어떨까요?
여기서 서로 다른 숫자들을 가진 버전이 나오는 것입니다.
각각의 정확한 의미는 프로젝트들마다 다르지만, 상대적으로 공통적인 기준은 [시멘틱 버저닝(Semantic Versioning)](https://velog.io/@slaslaya/Semantic-Versioning-2.0.0-MAJOR-MINOR-PATCH와-명세에-관하여)입니다.  
시맨틱 버저닝에서 모든 버전 넘버는 다음의 형식을 따릅니다: **major.minor.patch**  
규칙은 다음과 같습니다:

-   만약 새로운 버전 배포가 API를 바꾸지 않는다면, **patch** 버전을 증가시킨다.
-   만약 이전의 버전들과 호환성을 유지하면서 API에 _추가_ 되는 것이 있다면, **minor** 버전을 증가시킨다.
-   만약 이전의 버전들과 호환이 되지 않는 방식으로 API를 바꾸게 된다면, **major** 버전을 증가시킨다.

이 방식은 이미 몇몇 이점을 제공합니다.
자, 만약 저의 프로젝트가 여러분의 프로젝트에 의존하고 있다면, 제가 그 프로젝트를 빌드할 때 이용했던 것과 같은 **major** 버전의 최신 배포 버전을 사용하는 것이 _안전해야_ 합니다, 제 프로젝트에서 사용하고 있는 **minor** 버전이 적어도 빌드할 그 당시의 것 이상인 한 말이죠.  
달리 말하면, 만약 제가 여러분의 라이브러리 버전 `1.3.7`에 의존하고 있다면, `1.3.8`, `1.6.1`, 심지어는 `1.3.0`을 이용해서 빌드하는 것 또한 _안전해야_ 합니다. 버전 `2.2.4`는 아마 괜찮지 않을 겁니다, 왜냐면 **major** 버전이 증가되었기 때문이죠.  
우리는 파이썬의 버전 넘버에서 시맨틱 버저닝의 예시를 확인할 수 있습니다. 여러분 중 다수는 아마 파이썬 2와 파이썬 3의 코드가 함께 잘 어울릴 수 없다는 것을 알고 있을 겁니다, **major** 버전 증가의 이유죠. 마찬가지로, 파이썬 3.5를 위해 쓰인 코드는 파이썬 3.7 버전에서는 실행되지만 아마 3.4에서는 실행되지 않을지도 모릅니다.

When working with dependency management systems, you may also come
across the notion of _lock files_. A lock file is simply a file that
lists the exact version you are _currently_ depending on of each
dependency. Usually, you need to explicitly run an update program to
upgrade to newer versions of your dependencies. There are many reasons
for this, such as avoiding unnecessary recompiles, having reproducible
builds, or not automatically updating to the latest version (which may
be broken). An extreme version of this kind of dependency locking is
_vendoring_, which is where you copy all the code of your dependencies
into your own project. That gives you total control over any changes to
it, and lets you introduce your own changes to it, but also means you
have to explicitly pull in any updates from the upstream maintainers
over time.

# Continuous integration systems

<!-- 28분20초 -->

As you work on larger and larger projects, you'll find that there are
often additional tasks you have to do whenever you make a change to it.
You might have to upload a new version of the documentation, upload a
compiled version somewhere, release the code to pypi, run your test
suite, and all sort of other things. Maybe every time someone sends you
a pull request on GitHub, you want their code to be style checked and
you want some benchmarks to run? When these kinds of needs arise, it's
time to take a look at continuous integration.

Continuous integration, or CI, is an umbrella term for "stuff that runs
whenever your code changes", and there are many companies out there that
provide various types of CI, often for free for open-source projects.
Some of the big ones are Travis CI, Azure Pipelines, and GitHub Actions.
They all work in roughly the same way: you add a file to your repository
that describes what should happen when various things happen to that
repository. By far the most common one is a rule like "when someone
pushes code, run the test suite". When the event triggers, the CI
provider spins up a virtual machines (or more), runs the commands in
your "recipe", and then usually notes down the results somewhere. You
might set it up so that you are notified if the test suite stops
passing, or so that a little badge appears on your repository as long as
the tests pass.

As an example of a CI system, the class website is set up using GitHub
Pages. Pages is a CI action that runs the Jekyll blog software on every
push to `master` and makes the built site available on a particular
GitHub domain. This makes it trivial for us to update the website! We
just make our changes locally, commit them with git, and then push. CI
takes care of the rest.

## A brief aside on testing

Most large software projects come with a "test suite". You may already
be familiar with the general concept of testing, but we thought we'd
quickly mention some approaches to testing and testing terminology that
you may encounter in the wild:

-   Test suite: a collective term for all the tests
-   Unit test: a "micro-test" that tests a specific feature in isolation
-   Integration test: a "macro-test" that runs a larger part of the
    system to check that different feature or components work _together_.
-   Regression test: a test that implements a particular pattern that
    _previously_ caused a bug to ensure that the bug does not resurface.
-   Mocking: the replace a function, module, or type with a fake
    implementation to avoid testing unrelated functionality. For example,
    you might "mock the network" or "mock the disk".

# Exercises

1.  Most makefiles provide a target called `clean`. This isn't intended
    to produce a file called `clean`, but instead to clean up any files
    that can be re-built by make. Think of it as a way to "undo" all of
    the build steps. Implement a `clean` target for the `paper.pdf`
    `Makefile` above. You will have to make the target
    [phony](https://www.gnu.org/software/make/manual/html_node/Phony-Targets.html).
    You may find the [`git ls-files`](https://git-scm.com/docs/git-ls-files) subcommand useful.
    A number of other very common make targets are listed
    [here](https://www.gnu.org/software/make/manual/html_node/Standard-Targets.html#Standard-Targets).
2.  Take a look at the various ways to specify version requirements for
    dependencies in [Rust's build
    system](https://doc.rust-lang.org/cargo/reference/specifying-dependencies.html).
    Most package repositories support similar syntax. For each one
    (caret, tilde, wildcard, comparison, and multiple), try to come up
    with a use-case in which that particular kind of requirement makes
    sense.
3.  Git can act as a simple CI system all by itself. In `.git/hooks`
    inside any git repository, you will find (currently inactive) files
    that are run as scripts when a particular action happens. Write a
    [`pre-commit`](https://git-scm.com/docs/githooks#_pre_commit) hook
    that runs `make paper.pdf` and refuses the commit if the `make`
    command fails. This should prevent any commit from having an
    unbuildable version of the paper.
4.  Set up a simple auto-published page using [GitHub
    Pages](https://help.github.com/en/actions/automating-your-workflow-with-github-actions).
    Add a [GitHub Action](https://github.com/features/actions) to the
    repository to run `shellcheck` on any shell files in that
    repository (here is [one way to do
    it](https://github.com/marketplace/actions/shellcheck)). Check that
    it works!
5.  [Build your
    own](https://help.github.com/en/actions/automating-your-workflow-with-github-actions/building-actions)
    GitHub action to run [`proselint`](http://proselint.com/) or
    [`write-good`](https://github.com/btford/write-good) on all the
    `.md` files in the repository. Enable it in your repository, and
    check that it works by filing a pull request with a typo in it.
