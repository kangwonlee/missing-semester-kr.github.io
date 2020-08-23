---
layout: lecture
title: "셸 툴과 스크립팅"
date: 2019-01-14
ready: true
video:
  aspect: 56.25
  id: kgII-YWo3Zw
---

이번 강의에선, 배시(bash)를 스크립팅 언어로 사용하는 기본 사항들과 여러분들이 앞으로 명령줄에서 가장 보편적인 작업을 할 때 사용되는 몇개의 셸 도구들에 대해서 소개할 것입니다.

# 셸(Shell) 스크립팅

지난 시간에 셸에서 명령들을 추출하고 파이프로 연결시키는 방법에 대해서 배웠습니다. 그러나 많은 상황에선 일련의 명령을 수행하고 조건문이나 반복문과 같은 제어 흐름 표현식을 사용해야 하는 경우가 있습니다.

쉘 스크립트은 살짝 복잡합니다. 대부분의 셸에는 변수, 제어 흐름(control flow) 및 자체 구문(syntax)을 가진 자체 스크립팅 언어가 있습니다. 쉘 스크립트와 다른 프로그래밍 언어와의 차이점은 쉘 관련 작업을 수행하기 위해 최적화되어 있다는 것입니다. 따라서 명령 파이프 라인을 만들고 파일을 저장하고 표준 입력을 받는 것은 쉘 스크립팅이 원시적이며 범용 스크립팅 언어보다 사용하기 쉽습니다. 이 섹션에서는 그 중 가장 일반적인 배시(bash) 스크립팅에 중점을 둘 것입니다.

배시에서 변수에 값을 할당할 때는 `foo=bar`와 같이  입력합니다. 변수의 값에 접근할 때는 `$foo`로 액세스합니다. `foo = bar`와 같이 띄어쓰기를 사용하면 `=` 와 `bar`를 사용하여 `foo` 프로그램을 호출하는 것으로 해석되므로 작동하지 않습니다. 일반적으로 셸 스크립트에서 공간 문자는 인수 분할을 수행합니다. 이런 규칙은 일반적인 프로그래밍 언어와 다르기 때문에 헷갈릴 수가 있으니 항상 유의하세요.

배시에서 문자열은 `'`나 `"` 기호로 선언 될 수 있지만 다른 의미를 가지게 됩니다. `'`로 둘러싸인 문자열은 문자열 자체를 뜻하지만  `"`로 둘러싸인 문자열은 변수값을 반환합니다.
```bash
foo=bar
echo "$foo"
# prints bar
echo '$foo'
# prints $foo
```

대부분의 프로그래밍 언어와 마찬가지로, 배시는 `if`, `case`, `while` 및 `for`를 포함한 제어 흐름 기술을 지원합니다. 마찬가지로, 배시는 인자를 만들고 사용하고 작동하는 기능들이 있습니다. 다음은 디렉토리를 만들고 `cd`를 수행하는 함수의 예시입니다.
```bash
mcd () {
    mkdir -p "$1"
    cd "$1"
}
```

`$1`은 script/function의 첫 번째 인자입니다. 다른 스크립팅 언어와 달리, bash는 인수, 오류 코드 및 기타 관련 변수를 참조하기 위해 다양한 특수 변수를 사용합니다. 아래에 그 중 일부를 적어놨습니다. 더 포괄적인 목록은 [여기](https://www.tldp.org/LDP/abs/html/special-chars.html)서 찾을 수 있습니다.

- `$0` - 스크립트 이름
- `$1` ~ `$9` - 스크립트의 인자들. `$1`부터 첫번쨰 인자입니다.
- `$@` - 모든 인자들
- `$#` - 인자의 수
- `$?` - 이전 명령을 반환하는 코드
- `$$` - 현재 스크립트에 대한 프로세스 식별 번호 (PID)
- `!!` - 인수를 포함하여 마지막 명령 전체를 포함합니다. 일반적으로는 사용 권한이 누락되어 실패할 때 사용합니다. `sudo`를 함께 써서 실패한 명령을 신속하게 다시 실행할 수 있습니다.
- `$_` - 마지막 명령에서 나온 마지막 인수입니다. 대화형 셸에 있는 경우 `Esc`를 입력한 후 `.`을 입력해 이 값을 신속하게 얻을 수 있습니다.

명령어들은 종종 'STDOUT`을 사용해서 출력을 하는데, 오류를 보고할 때는 좀 더 스크립트 친화적인 방식으로 `STDERR`를 사용해서 코드를 반환합니다. 리턴 코드 또는 종료 상태는 스크립트 또는 명령어들이 어떤 방식으로 실행 방법을 전달하는 지를 보여주는 것입니다. 값 0은 일반적으로 모든 것이 정상임을 의미합니다. 0이 아닌 다른 것은 오류가 발생했음을 의미합니다.

종료 코드는 `&&`(and 연산자) 와 `||`(or 연산자) 즉, [short-circuiting](https://en.wikipedia.org/wiki/Short-circuit_evaluation) 연산자들을 사용하여 조건부로 명령을 실행하는 데 사용할 수 있습니다. 그리고 세미콜론 ; 을 사용하여 동일한 행 내에서 명령을 분리 할 수도 있습니다. 
참(true)인 프로그램은 항상 0 리턴 코드를 가지며 거짓(false) 명령은 항상 1 리턴 코드를 갖습니다.몇 가지 예를 통해 한 번 봅시다. 

```bash
false || echo "Oops, fail"
# Oops, fail

true || echo "Will not be printed"
#

true && echo "Things went well"
# Things went well

false && echo "Will not be printed"
#

true ; echo "This will always run"
# This will always run

false ; echo "This will always run"
# This will always run
```

또 다른 일반적인 패턴은 명령의 출력을 변수로 가져 오는 것입니다. 이것은 _명령어대체_ 로 수행 할 수 있습니다. `$ (CMD)`를 기입 할 때마다 `CMD`가 실행되고 명령의 출력을 가져와 제자리에 대체합니다. 예를들어, 'for file in $ (ls)` 라는 코드를 수행하면 쉘은 먼저 `ls` 를 호출 한 다음 해당 값을 반복합니다. 위와 비슷하지만 상대적으로 덜 알려진 유사한 기능은 _절차대체(process replacement)_ 입니다. `<(CMD)` 는 `CMD` 를 실행하고 출력을 임시 파일에 배치하고 `<()` 를 해당 파일의 이름으로 대체합니다. 이 명령어는 보통 STDIN이 아닌 파일 형태로 값이 전달 될 때 유용합니다. 예를 들어, `diff <(ls foo) <(ls bar)` 는 디렉토리 `foo` 와 `bar` 에있는 파일 간의 차이점을 표시합니다.

위에 너무 많은 정보만을 제공했기 때문에, 이러한 기능 중 일부를 보여주는 예를 살펴 보겠습니다. 아래는 우리가 제공하는 인수,`foobar` 문자열에 대해 `grep` 을 반복하고, 찾을 수없는 경우 주석으로 파일에 추가합니다.

```bash
#!/bin/bash

echo "Starting program at $(date)" # Date will be substituted

echo "Running program $0 with $# arguments with pid $$"

for file in $@; do
    grep foobar $file > /dev/null 2> /dev/null
    # When pattern is not found, grep has exit status 1
    # We redirect STDOUT and STDERR to a null register since we do not care about them
    if [[ $? -ne 0 ]]; then
        echo "File $file does not have any foobar, adding one"
        echo "# foobar" >> "$file"
    fi
done
```

비교에서 우리는`$?`가 0과 같지 않은지 테스트했습니다. Bash는 이런 종류의 많은 비교를 구현합니다. manpage서 [`test`](https://www.man7.org/linux/man-pages/man1/test.1.html) 에 대한 자세한 목록을 찾을 수 있습니다. bash에서 비교를 수행 할 때는 간단한 대괄호`[]`대신 이중 대괄호`[[]]`를 사용하십시오. 'sh'로 이식 할 수는 없지만 실수 할 가능성은 낮습니다. 자세한 설명은 [여기] (http://mywiki.wooledge.org/BashFAQ/031) 서 확인할 수 있습니다. 

스크립트를 시작할 때 유사한 인수를 제공해야하는 경우가 종종 있을겁니다. Bash에는 이런 상황에서, 파일 이름 확장을 수행하여, 식을 보다 쉽게 확장 할 수있는 방법을 가지고 있습니다. 이러한 기술 중 하나가 쉘 _globbing_ 입니다.
- Wildcards - 일종의 와일드 카드 일치를 수행 할 때마다 `?` 및 `*` 를 사용하여 각각 하나 또는 원하는 양의 문자를 일치시킬 수 있습니다. 예를 들어 `foo`,`foo1`,`foo2`,`foo10` 및 `bar` 파일이 있을 때, `rm foo?` 명령어를 치면, `foo1` 과 `foo2` 를 삭제하는 반면에 `rm foo *` 는 `bar`를 뺀 모든 파일을 삭제합니다. 
- 중괄호(Curly braces) `{}` - 일련의 명령에 공통 부분 문자열이있을 때마다 bash에 중괄호를 사용하여이를 자동으로 확장 할 수 있습니다. 이것은 파일을 이동하거나 변환 할 때 매우 편리합니다.

```bash
convert image.{png,jpg}
# Will expand to
convert image.png image.jpg

cp /path/to/project/{foo,bar,baz}.sh /newpath
# Will expand to
cp /path/to/project/foo.sh /path/to/project/bar.sh /path/to/project/baz.sh /newpath

# Globbing techniques can also be combined
mv *{.py,.sh} folder
# Will move all *.py and *.sh files


mkdir foo bar
# This creates files foo/a, foo/b, ... foo/h, bar/a, bar/b, ... bar/h
touch {foo,bar}/{a..h}
touch foo/x bar/y
# Show differences between files in foo and bar
diff <(ls foo) <(ls bar)
# Outputs
# < x
# ---
# > y
```

<!-- Lastly, pipes `|` are a core feature of scripting. Pipes connect one program's output to the next program's input. We will cover them more in detail in the data wrangling lecture. -->

`bash` 스크립트를 작성하는 것은 까다롭고 직관적이지 않을 수 있습니다. sh / bash 스크립트에서 오류를 찾는 데 도움이되는 [shellcheck] (https://github.com/koalaman/shellcheck)와 같은 도구가 있으니 참고해보세요.

참고로, 터미널에서 호출하기 위해 반드시 bash로 스크립트를 작성할 필요는 없습니다. 예를 들어 다음은 인수를 역순으로 출력하는 간단한 Python 스크립트입니다.

```python
#!/usr/local/bin/python
import sys
for arg in reversed(sys.argv[1:]):
    print(arg)
```


커널은 스크립트 상단에 [shebang] (https://en.wikipedia.org/wiki/Shebang_ (Unix)) 줄을 포함했기 때문에 이 스크립트를 쉘 명령 대신 파이썬 인터프리터로 실행하는 것을 알고 있습니다. [`env`] (https://www.man7.org/linux/man-pages/man1/env.1.html) 명령을 사용하여 명령이 시스템 어디에 있든 확인하는 것을 shebang 줄을 작성하여 연습하는 것이 좋습니다. 이것이 스크립트의 이식성을 증가시킵니다. 위치를 확인하기 위해`env`는 첫 번째 강의에서 소개 한`PATH` 환경 변수를 사용합니다. 이 예에서 shebang 줄은`#! / usr / bin / env python`과 같습니다.

기억해야 할 셸 함수와 스크립트 간의 몇 가지 차이점은 다음과 같습니다.
-함수는 쉘과 동일한 언어로 작성되어야하며 스크립트는 모든 언어로 작성 될 수 있습니다. 이것이 대본에 shebang을 포함하는 것이 중요한 이유입니다.
-함수는 정의를 읽을 때 한 번로드됩니다. 스크립트는 실행될 때마다 로드됩니다. 이렇게하면 함수를 로드하는 속도가 약간 빨라지지만 변경할 때마다 정의를 다시로드해야합니다.
-함수는 현재 쉘 환경에서 실행되는 반면 스크립트는 자체 프로세스에서 실행됩니다. 따라서 함수는 환경 변수를 수정할 수 있습니다. 예를들어, 현재 디렉토리를 변경하십시오 를 스크립트는 할 수 없지만습니다. 스크립트는 [`export`] (https://www.man7.org/linux/man-pages/man1/export.1p.html)를 사용하여 내 보낸 값 환경 변수로 전달됩니다.
-모든 프로그래밍 언어와 마찬가지로 함수는 모듈성, 코드 재사용 및 쉘 코드의 명확성을 달성하기위한 강력한 구성입니다. 종종 쉘 스크립트는 자체 함수 정의를 포함합니다.

# Shell Tools

## 명령 사용 방법 찾기

이 시점에서`ls -l`,`mv -i` 및`mkdir -p`와 같은 별칭 섹션에서 명령에 대한 플래그를 찾는 방법이 궁금 할 것입니다. 좀 더 일반적으로 명령이 주어지면, 당신은 어떻게 그 명령어의 기능과 다른 옵션을 찾을 수 있습니까? 당신은 언제든지 구글링을 통해서 검색을 시작할 수 있지만, UNIX가 StackOverflow 보다 이전이므로, 이 정보를 가져 오는 기본 제공 방법이 따로 있습니다.

쉘 강의에서 보았 듯이 1 차 접근 방식은`-h` 또는`--help` 플래그를 사용하여 해당 명령들을 찾아볼 수 있습니다. 아니면, 더 자세한 접근 방식은`man` 명령을 사용하는 것입니다. manual의 줄임말 인 [`man`] (https://www.man7.org/linux/man-pages/man1/man.1.html)은 사용자가 지정한 명령에 대한 매뉴얼 페이지 (manpage라고 함)를 제공합니다. 예를 들어,`man rm`은 앞서 보여 준 `-i` 플래그를 포함하여 사용하는 플래그와 함께`rm` 명령의 동작을 출력합니다. 사실 지금까지 모든 명령에 대해 링크 한 것은, 명령에 대한 Linux 매뉴얼 페이지의 온라인 버전입니다. 기본이 아닌 명령도 개발자가 작성하여 설치 프로세스의 일부로 포함하면 매뉴얼 페이지 항목이 있습니다. ncurses 기반 도구와 같은 대화 형 도구의 경우, 프로그램 내에서 `: help` 명령을 사용하거나 `?` 를 입력하여 도움말에 액세스 할 수 있습니다.

Sometimes manpages can provide overly detailed descriptions of the commands, making it hard to decipher what flags/syntax to use for common use cases.
[TLDR pages](https://tldr.sh/) are a nifty complementary solution that focuses on giving example use cases of a command so you can quickly figure out which options to use.
For instance, I find myself referring back to the tldr pages for [`tar`](https://tldr.ostera.io/tar) and [`ffmpeg`](https://tldr.ostera.io/ffmpeg) way more often than the manpages.


## Finding files

One of the most common repetitive tasks that every programmer faces is finding files or directories.
All UNIX-like systems come packaged with [`find`](https://www.man7.org/linux/man-pages/man1/find.1.html), a great shell tool to find files. `find` will recursively search for files matching some criteria. Some examples:

```bash
# Find all directories named src
find . -name src -type d
# Find all python files that have a folder named test in their path
find . -path '**/test/**/*.py' -type f
# Find all files modified in the last day
find . -mtime -1
# Find all zip files with size in range 500k to 10M
find . -size +500k -size -10M -name '*.tar.gz'
```
Beyond listing files, find can also perform actions over files that match your query.
This property can be incredibly helpful to simplify what could be fairly monotonous tasks.
```bash
# Delete all files with .tmp extension
find . -name '*.tmp' -exec rm {} \;
# Find all PNG files and convert them to JPG
find . -name '*.png' -exec convert {} {.}.jpg \;
```

Despite `find`'s ubiquitousness, its syntax can sometimes be tricky to remember.
For instance, to simply find files that match some pattern `PATTERN` you have to execute `find -name '*PATTERN*'` (or `-iname` if you want the pattern matching to be case insensitive).
You could start building aliases for those scenarios, but part of the shell philosophy is that it is good to explore alternatives.
Remember, one of the best properties of the shell is that you are just calling programs, so you can find (or even write yourself) replacements for some.
For instance, [`fd`](https://github.com/sharkdp/fd) is a simple, fast, and user-friendly alternative to `find`.
It offers some nice defaults like colorized output, default regex matching, and Unicode support. It also has, in my opinion, a more intuitive syntax.
For example, the syntax to find a pattern `PATTERN` is `fd PATTERN`.

Most would agree that `find` and `fd` are good, but some of you might be wondering about the efficiency of looking for files every time versus compiling some sort of index or database for quickly searching.
That is what [`locate`](https://www.man7.org/linux/man-pages/man1/locate.1.html) is for.
`locate` uses a database that is updated using [`updatedb`](https://www.man7.org/linux/man-pages/man1/updatedb.1.html).
In most systems, `updatedb` is updated daily via [`cron`](https://www.man7.org/linux/man-pages/man8/cron.8.html).
Therefore one trade-off between the two is speed vs freshness.
Moreover `find` and similar tools can also find files using attributes such as file size, modification time, or file permissions, while `locate` just uses the file name.
A more in-depth comparison can be found [here](https://unix.stackexchange.com/questions/60205/locate-vs-find-usage-pros-and-cons-of-each-other).

## Finding code

Finding files by name is useful, but quite often you want to search based on file *content*. 
A common scenario is wanting to search for all files that contain some pattern, along with where in those files said pattern occurs.
To achieve this, most UNIX-like systems provide [`grep`](https://www.man7.org/linux/man-pages/man1/grep.1.html), a generic tool for matching patterns from the input text.
`grep` is an incredibly valuable shell tool that we will cover in greater detail during the data wrangling lecture.

For now, know that `grep` has many flags that make it a very versatile tool.
Some I frequently use are `-C` for getting **C**ontext around the matching line and `-v` for in**v**erting the match, i.e. print all lines that do **not** match the pattern. For example, `grep -C 5` will print 5 lines before and after the match.
When it comes to quickly searching through many files, you want to use `-R` since it will **R**ecursively go into directories and look for files for the matching string.

But `grep -R` can be improved in many ways, such as ignoring `.git` folders, using multi CPU support, &c.
Many `grep` alternatives have been developed, including [ack](https://beyondgrep.com/), [ag](https://github.com/ggreer/the_silver_searcher) and [rg](https://github.com/BurntSushi/ripgrep).
All of them are fantastic and pretty much provide the same functionality.
For now I am sticking with ripgrep (`rg`), given how fast and intuitive it is. Some examples:
```bash
# Find all python files where I used the requests library
rg -t py 'import requests'
# Find all files (including hidden files) without a shebang line
rg -u --files-without-match "^#!"
# Find all matches of foo and print the following 5 lines
rg foo -A 5
# Print statistics of matches (# of matched lines and files )
rg --stats PATTERN
```

Note that as with `find`/`fd`, it is important that you know that these problems can be quickly solved using one of these tools, while the specific tools you use are not as important.

## Finding shell commands

So far we have seen how to find files and code, but as you start spending more time in the shell, you may want to find specific commands you typed at some point.
The first thing to know is that the typing up arrow will give you back your last command, and if you keep pressing it you will slowly go through your shell history.

The `history` command will let you access your shell history programmatically.
It will print your shell history to the standard output.
If we want to search there we can pipe that output to `grep` and search for patterns.
`history | grep find` will print commands that contain the substring "find".

In most shells, you can make use of `Ctrl+R` to perform backwards search through your history.
After pressing `Ctrl+R`, you can type a substring you want to match for commands in your history.
As you keep pressing it, you will cycle through the matches in your history.
This can also be enabled with the UP/DOWN arrows in [zsh](https://github.com/zsh-users/zsh-history-substring-search).
A nice addition on top of `Ctrl+R` comes with using [fzf](https://github.com/junegunn/fzf/wiki/Configuring-shell-key-bindings#ctrl-r) bindings.
`fzf` is a general-purpose fuzzy finder that can be used with many commands.
Here is used to fuzzily match through your history and present results in a convenient and visually pleasing manner.

Another cool history-related trick I really enjoy is **history-based autosuggestions**.
First introduced by the [fish](https://fishshell.com/) shell, this feature dynamically autocompletes your current shell command with the most recent command that you typed that shares a common prefix with it.
It can be enabled in [zsh](https://github.com/zsh-users/zsh-autosuggestions) and it is a great quality of life trick for your shell.

Lastly, a thing to have in mind is that if you start a command with a leading space it won't be added to your shell history.
This comes in handy when you are typing commands with passwords or other bits of sensitive information.
If you make the mistake of not adding the leading space, you can always manually remove the entry by editing your `.bash_history` or `.zhistory`.

## Directory Navigation

So far, we have assumed that you are already where you need to be to perform these actions. But how do you go about quickly navigating directories?
There are many simple ways that you could do this, such as writing shell aliases or creating symlinks with [ln -s](https://www.man7.org/linux/man-pages/man1/ln.1.html), but the truth is that developers have figured out quite clever and sophisticated solutions by now.

As with the theme of this course, you often want to optimize for the common case.
Finding frequent and/or recent files and directories can be done through tools like [`fasd`](https://github.com/clvv/fasd) and [`autojump`](https://github.com/wting/autojump).
Fasd ranks files and directories by [_frecency_](https://developer.mozilla.org/en/The_Places_frecency_algorithm), that is, by both _frequency_ and _recency_.
By default, `fasd` adds a `z` command that you can use to quickly `cd` using a substring of a _frecent_ directory. For example, if you often go to `/home/user/files/cool_project` you can simply use `z cool` to jump there. Using autojump, this same change of directory could be accomplished using `j cool`.

More complex tools exist to quickly get an overview of a directory structure [`tree`](https://linux.die.net/man/1/tree), [`broot`](https://github.com/Canop/broot) or even full fledged file managers like [`nnn`](https://github.com/jarun/nnn) or [`ranger`](https://github.com/ranger/ranger)

# Exercises

1. Read [`man ls`](https://www.man7.org/linux/man-pages/man1/ls.1.html) and write an `ls` command that lists files in the following manner

    - Includes all files, including hidden files
    - Sizes are listed in human readable format (e.g. 454M instead of 454279954)
    - Files are ordered by recency
    - Output is colorized

    A sample output would look like this

    ```
    -rw-r--r--   1 user group 1.1M Jan 14 09:53 baz
    drwxr-xr-x   5 user group  160 Jan 14 09:53 .
    -rw-r--r--   1 user group  514 Jan 14 06:42 bar
    -rw-r--r--   1 user group 106M Jan 13 12:12 foo
    drwx------+ 47 user group 1.5K Jan 12 18:08 ..
    ```

{% comment %}
ls -lath --color=auto
{% endcomment %}

1. Write bash functions  `marco` and `polo` that do the following.
Whenever you execute `marco` the current working directory should be saved in some manner, then when you execute `polo`, no matter what directory you are in, `polo` should `cd` you back to the directory where you executed `marco`.
For ease of debugging you can write the code in a file `marco.sh` and (re)load the definitions to your shell by executing `source marco.sh`.

{% comment %}
marco() {
    export MARCO=$(pwd)
}

polo() {
    cd "$MARCO"
}
{% endcomment %}

1. Say you have a command that fails rarely. In order to debug it you need to capture its output but it can be time consuming to get a failure run.
Write a bash script that runs the following script until it fails and captures its standard output and error streams to files and prints everything at the end.
Bonus points if you can also report how many runs it took for the script to fail.

    ```bash
    #!/usr/bin/env bash

    n=$(( RANDOM % 100 ))

    if [[ n -eq 42 ]]; then
       echo "Something went wrong"
       >&2 echo "The error was using magic numbers"
       exit 1
    fi

    echo "Everything went according to plan"
    ```

{% comment %}
#!/usr/bin/env bash

count=0
until [[ "$?" -ne 0 ]];
do
  count=$((count+1))
  ./random.sh &> out.txt
done

echo "found error after $count runs"
cat out.txt
{% endcomment %}

1. As we covered in the lecture `find`'s `-exec` can be very powerful for performing operations over the files we are searching for.
However, what if we want to do something with **all** the files, like creating a zip file?
As you have seen so far commands will take input from both arguments and STDIN.
When piping commands, we are connecting STDOUT to STDIN, but some commands like `tar` take inputs from arguments.
To bridge this disconnect there's the [`xargs`](https://www.man7.org/linux/man-pages/man1/xargs.1.html) command which will execute a command using STDIN as arguments.
For example `ls | xargs rm` will delete the files in the current directory.

    Your task is to write a command that recursively finds all HTML files in the folder and makes a zip with them. Note that your command should work even if the files have spaces (hint: check `-d` flag for `xargs`)
    {% comment %}
    find . -type f -name "*.html" | xargs -d '\n'  tar -cvzf archive.tar.gz
    {% endcomment %}

1. (Advanced) Write a command or script to recursively find the most recently modified file in a directory. More generally, can you list all files by recency?
