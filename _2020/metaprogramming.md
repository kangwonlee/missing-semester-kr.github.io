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

# Dependency management

<!-- 14분부터 -->

At a more macro level, your software projects are likely to have
dependencies that are themselves projects. You might depend on installed
programs (like `python`), system packages (like `openssl`), or libraries
within your programming language (like `matplotlib`). These days, most
dependencies will be available through a _repository_ that hosts a
large number of such dependencies in a single place, and provides a
convenient mechanism for installing them. Some examples include the
Ubuntu package repositories for Ubuntu system packages, which you access
through the `apt` tool, RubyGems for Ruby libraries, PyPi for Python
libraries, or the Arch User Repository for Arch Linux user-contributed
packages.

Since the exact mechanisms for interacting with these repositories vary
a lot from repository to repository and from tool to tool, we won't go
too much into the details of any specific one in this lecture. What we
_will_ cover is some of the common terminology they all use. The first
among these is _versioning_. Most projects that other projects depend on
issue a _version number_ with every release. Usually something like
8.1.3 or 64.1.20192004. They are often, but not always, numerical.
Version numbers serve many purposes, and one of the most important of
them is to ensure that software keeps working. Imagine, for example,
that I release a new version of my library where I have renamed a
particular function. If someone tried to build some software that
depends on my library after I release that update, the build might fail
because it calls a function that no longer exists! Versioning attempts
to solve this problem by letting a project say that it depends on a
particular version, or range of versions, of some other project. That
way, even if the underlying library changes, dependent software
continues building by using an older version of my library.

That also isn't ideal though! What if I issue a security update which
does _not_ change the public interface of my library (its "API"), and
which any project that depended on the old version should immediately
start using? This is where the different groups of numbers in a version
come in. The exact meaning of each one varies between projects, but one
relatively common standard is [_semantic
versioning_](https://semver.org/). With semantic versioning, every
version number is of the form: major.minor.patch. The rules are:

-   If a new release does not change the API, increase the patch version.
-   If you _add_ to your API in a backwards-compatible way, increase the
    minor version.
-   If you change the API in a non-backwards-compatible way, increase the
    major version.

This already provides some major advantages. Now, if my project depends
on your project, it _should_ be safe to use the latest release with the
same major version as the one I built against when I developed it, as
long as its minor version is at least what it was back then. In other
words, if I depend on your library at version `1.3.7`, then it _should_
be fine to build it with `1.3.8`, `1.6.1`, or even `1.3.0`. Version
`2.2.4` would probably not be okay, because the major version was
increased. We can see an example of semantic versioning in Python's
version numbers. Many of you are probably aware that Python 2 and Python
3 code do not mix very well, which is why that was a _major_ version
bump. Similarly, code written for Python 3.5 might run fine on Python
3.7, but possibly not on 3.4.

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
