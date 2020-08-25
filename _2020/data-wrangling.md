---
layout: lecture
title: "Data Wrangling"
date: 2019-01-16
ready: true
video:
  aspect: 56.25
  id: sz_dsktIjt4
---

Have you ever wanted to take data in one format and turn it into a
different format? Of course you have! That, in very general terms, is
what this lecture is all about. Specifically, massaging data, whether in
text or binary format, until you end up with exactly what you wanted.

데이터를 하나의 형식으로 가져 와서 다른 형식으로 바꾸고 싶은 경험이 있으셨나요? 물론 있었을 거에요! 그리고 바로 이것이 이 강의의 내용입니다. 특히, 텍스트이든 이진 형식이든, 데이터를 원하는 형식으로 변화시키는 과정에 대해 보도록 하겠습니다. 

We've already seen some basic data wrangling in past lectures. Pretty
much any time you use the `|` operator, you are performing some kind of
data wrangling. Consider a command like `journalctl | grep -i intel`. It
finds all system log entries that mention Intel (case insensitive). You
may not think of it as wrangling data, but it is going from one format
(your entire system log) to a format that is more useful to you (just
the intel log entries). Most data wrangling is about knowing what tools
you have at your disposal, and how to combine them.

전 수업들에서 이미 우리는 기본적인 데이터 랭글링 또는 형태 전환 (data wrangling) 을 본적이 있습니다. 우리가 `|` 을 사용할때마다 우리는 일종의 데이터 랭글링을 하고 있는 것입니다. `journalctl | grep -i intel` 이라는 명령을 생각해 보세요. 이 명령은 시스템에 있는 모든 로그 중에서 Intel(대소문자 구분 안함) 가 들어간 로그를 찾습니다. 이것이 데이터를 랭글링 하는 것이라 생각하지 않을수도 있지만, 사실 이것은 한 형식 (시스템 로그 전체) 에서 다른, 당신에게 더 활용하기 쉬운 형식으로 (인텔 로그만) 바꾸는 것 입니다. 대부분의 데이터 랭글링은 사용할수 있는 도구를 알고 그것들을 결합하는 방법을 아는 것입니다. 

Let's start from the beginning. To wrangle data, we need two things:
data to wrangle, and something to do with it. Logs often make for a good
use-case, because you often want to investigate things about them, and
reading the whole thing isn't feasible. Let's figure out who's trying to
log into my server by looking at my server's log:

기본부터 시작해 봅시다. 데이터를 변환하기 위해서는 두 가지가 필요합니다: 변환할 데이터와, 그것을 가지고 무엇을 할지 입니다. 로그는 종종 유익한데, 그 이유는 우리가 보통 로그를 가지고 조사하기를 원하고, 전체를 다 읽는것은 실현가능하지 않기 때문입니다. 누가 로그인 하려고 하는지 내 서버의 로그를 보고 알아봅시다: 

```bash
ssh myserver journalctl
```

That's far too much stuff. Let's limit it to ssh stuff:

이것은 너무나 많은 로그를 불러옵니다. ssh 한 데이터로만 제한해 봅시다: 

```bash
ssh myserver journalctl | grep sshd
```

Notice that we're using a pipe to stream a _remote_ file through `grep`
on our local computer! `ssh` is magical, and we will talk more about it
in the next lecture on the command-line environment. This is still way
more stuff than we wanted though. And pretty hard to read. Let's do
better:

우리가 파이프을 사용해서 원격 파일을 `grep` 우리 로컬 컴퓨터로 스트리밍 하고 있습니다! `ssh` 는 마법같고, 이에 대해서는 명령어 환경에 대한 다음 수업에서 더 이야기하겠습니다. 아직 이 데이터에는 우리가 원하는 정보외에도 많은 것이 포함되어 있습니다. 그리고 읽기에도 쉽지 않습니다. 더 발전시켜볼게요: 

```bash
ssh myserver 'journalctl | grep sshd | grep "Disconnected from"' | less
```

Why the additional quoting? Well, our logs may be quite large, and it's
wasteful to stream it all to our computer and then do the filtering.
Instead, we can do the filtering on the remote server, and then massage
the data locally. `less` gives us a "pager" that allows us to scroll up
and down through the long output. To save some additional traffic while
we debug our command-line, we can even stick the current filtered logs
into a file so that we don't have to access the network while
developing:

왜 추가적인 따옴표를 넣냐구요? 그 이유는 우리의 로그는 상당히 클 수 있으며 모든 것을 우리 컴퓨터로 스트리밍 한 다음 필터링을 수행하는 것은 낭비이기 때문이에요. 대신, 우리는 원격 서버에서 필터링을 진행하고, 로컬로 데이터를 변환합니다. `less`는 긴 출력을 위아래로 스크롤 할 수 있는 "페이저"를 제공합니다. 추가 트래픽을 절약하려면 명령어(커맨드 라인)에서 디버깅을 할 때 프레픽을 좀 더 아끼기 위해서 우리는 현재 필터된 로그를 파일로 저장해 개발중 네트워크에 액세스 할 필요가 없도록 할 수도 있어요. 

```console
$ ssh myserver 'journalctl | grep sshd | grep "Disconnected from"' > ssh.log
$ less ssh.log
```

There's still a lot of noise here. There are _a lot_ of ways to get rid
of that, but let's look at one of the most powerful tools in your
toolkit: `sed`.

아직도 잡음이 많습니다. 제거하는 방법은 많지만, 그 중 가장 강력한 도구 중 하나를 살펴 보겠습니다: `sed`

`sed` is a "stream editor" that builds on top of the old `ed` editor. In
it, you basically give short commands for how to modify the file, rather
than manipulate its contents directly (although you can do that too).
There are tons of commands, but one of the most common ones is `s`:
substitution. For example, we can write:

`sed` 는 이전 `ed` 편집기 위에 빌드된 "스트림 편집기"입니다. 이는, 쉽게 말해서 내용을 직접 조작하는 대신 (그렇게 할 수 있지만) 파일을 수정하는 방법에 대한 짧은 명령을 제공합니다. 많은 명령어들이 있지만, 그중 가장 많이 사용되는 것은  `s`: 치환 입니다. 예를 들어서 다음과 같이 작성할 수 있습니다: 

```bash
ssh myserver journalctl
 | grep sshd
 | grep "Disconnected from"
 | sed 's/.*Disconnected from //'
```

What we just wrote was a simple _regular expression_; a powerful
construct that lets you match text against patterns. The `s` command is
written on the form: `s/REGEX/SUBSTITUTION/`, where `REGEX` is the
regular expression you want to search for, and `SUBSTITUTION` is the
text you want to substitute matching text with.

우리가 쓴것은 간단한 정규표현 (regular expression) 입니다. 패턴과 텍스트를 일치시킬 수 있는 아주 효과적인 구성 (construct) 입니다. 명령어 `s` 는 `s/REGEX/SUBSTITUTION/` 이런 형식으로 사용하는데, 이때 `REGEX` 는 검색하려는 정규표현이고, `SUBSTITUTION` 는 일치하는 텍스트를 대체 할 텍스트입니다.

## Regular expressions
## 정규 표현 (Regular expressions)

Regular expressions are common and useful enough that it's worthwhile to
take some time to understand how they work. Let's start by looking at
the one we used above: `/.*Disconnected from /`. Regular expressions are
usually (though not always) surrounded by `/`. Most ASCII characters
just carry their normal meaning, but some characters have "special"
matching behavior. Exactly which characters do what vary somewhat
between different implementations of regular expressions, which is a
source of great frustration. Very common patterns are:

정규표현은 일반적이고 유용해서, 시간을 갖고 이해할 필요가 있습니다. 바로 위 예시에서 봤던 표현을 보면: `/.*Disconnected from /` 
정규표현은 대체로 (늘 그렇지는 않지만) `/`로 둘려쌓여 있습니다. 대부분의 ASCII 문자는 정상적인 의미를 지니지만 일부 캐릭터는 "특별한" 일치 습성을 가지고 있습니다. 이러한 캐릭터들이 무엇을 하는지는 어떤 정규표현인지에 따라 다르기에 큰 좌절의 원천이 되기도 합니다. 매우 일반적인 패턴은 다음과 같습니다.

 - `.` means "any single character" except newline
 - `*` zero or more of the preceding match
 - `+` one or more of the preceding match
 - `[abc]` any one character of `a`, `b`, and `c`
 - `(RX1|RX2)` either something that matches `RX1` or `RX2`
 - `^` the start of the line
 - `$` the end of the line


 - `.` 는 줄 바꿈을 제외한 "모든 단일 문자"
 - `*` 0 개 이상의 선행 일치 
 - `+` 하나 이상의 이전 일치
 - `[abc]` `a`, `b`, `c` 중 한 문자와 일치 
 - `(RX1|RX2)` `RX1` 또는 `RX2` 에 일치하는 항목
 - `^` 줄의 시작
 - `$` 줄의 끝  


`sed`'s regular expressions are somewhat weird, and will require you to
put a `\` before most of these to give them their special meaning. Or
you can pass `-E`.

`sed`의 정규표현은 다소 이상한데, 특별한 의미를 부여하기 위해 대부분의 앞에 `\`을 붙여야 합니다. 또는 `-E`를 붙일수도 있습니다. 

So, looking back at `/.*Disconnected from /`, we see that it matches
any text that starts with any number of characters, followed by the
literal string "Disconnected from &rdquo;. Which is what we wanted. But
beware, regular expressions are trixy. What if someone tried to log in
with the username "Disconnected from"? We'd have:

따라서 `/.*Disconnected from /` 을 살펴보면, 이 표현은 임의의 수의 문자로 시작하고 그 뒤에 "Disconnected from &rdquo; 가 있다면 모두 일치하다고 판단합니다. 이것은 우리가 원하던 것입니다. 그렇지만, 조심해야 할것은 정규표현은 속입수가 있을수 있다는 것입니다. 누군가 "Disconnected from" 라는 아이디로 로그인을 한다면 어떻게 될까요? 

```
Jan 17 03:13:00 thesquareplanet.com sshd[2631]: Disconnected from invalid user Disconnected from 46.97.239.16 port 55920 [preauth]
```

What would we end up with? Well, `*` and `+` are, by default, "greedy".
They will match as much text as they can. So, in the above, we'd end up
with just

우리는 결국 어떤 결과를 받을까요? `*` 와 `+` 는 그리디 (greedy) 합니다. 가능한 한 많은 텍스트와 일치시킵니다. 그렇기 때문에, 위를 보면, 우리는 결국 이런 결과를 가지게 됩니다:

```
46.97.239.16 port 55920 [preauth]
```

Which may not be what we wanted. In some regular expression
implementations, you can just suffix `*` or `+` with a `?` to make them
non-greedy, but sadly `sed` doesn't support that. We _could_ switch to
perl's command-line mode though, which _does_ support that construct:

이 결과는 우리가 원하지 않는 것일수도 있습니다. 어떤 정규표현에서는, `*` 이나 `+` 앞에  `?`를 붙여서 그리디 하지 않게 만들수도 있지만, 슬프게도 `sed`는 그런 기능을 가지고 있지 않습니다. 우리는 펄(perl)의 명령어 모드로 바꿀수도 있습니다. 이는 다음과 같은 구성을 지원합니다:  

```bash
perl -pe 's/.*?Disconnected from //'
```

We'll stick to `sed` for the rest of this, because it's by far the more
common tool for these kinds of jobs. `sed` can also do other handy
things like print lines following a given match, do multiple
substitutions per invocation, search for things, etc. But we won't cover
that too much here. `sed` is basically an entire topic in and of itself,
but there are often better tools.

`sed`는 이러한 종류의 작업을 위한 일반적인 도구이기 때문에 이후 예제들에서 `sed` 를 사용하기로 하겠습니다. `sed` 는 다른 편리한 작업도 할 수 있는데, 예를 들어 일치하는 줄을 인쇄하거나, 여러 호출의 대체, 검색등을 할 수 있습니다. 그렇지만, 여기서는 그런것들을 모두 다루지는 않겠습니다. `sed` 는 그 자체로도 큰 주제이고, 다른 좋은 도구도 많기 때문입니다. 

Okay, so we also have a suffix we'd like to get rid of. How might we do
that? It's a little tricky to match just the text that follows the
username, especially if the username can have spaces and such! What we
need to do is match the _whole_ line:

좋아요, 우리는 또한 제거하고 싶은 접미사를 가지고 있습니다. 어떻게 이것을 성취할수 있을까요? 사용자 이름 다음에 오는 텍스트만 일치시키는 것은 약간 까다롭습니다. 특히 사용자 이름에 공백이 있거나 하면요. 이때 우리가 해야할 것은 라인 전체에 일치시키는 것입니다. 

```bash
 | sed -E 's/.*Disconnected from (invalid |authenticating )?user .* [^ ]+ port [0-9]+( \[preauth\])?$//'
```

Let's look at what's going on with a [regex
debugger](https://regex101.com/r/qqbZqh/2). Okay, so the start is still
as before. Then, we're matching any of the "user" variants (there are
two prefixes in the logs). Then we're matching on any string of
characters where the username is. Then we're matching on any single word
(`[^ ]+`; any non-empty sequence of non-space characters). Then the word
"port" followed by a sequence of digits. Then possibly the suffix
`[preauth]`, and then the end of the line.

[정규 표현 디버거 (regex
debugger)](https://regex101.com/r/qqbZqh/2) 를 통해 무슨 일이 일어나고 있는지 알아보도록 합시다. 시작은 전과 같습니다. 그리고 나서는, 우리는 유저 변수들을 매치합니다 (로그에는 두개의 접두사가 있습니다). 그런 다음 우리는 어떤 스트링이든 사용자 이름이 있는 곳에 매치합니다. 그리고 나서는 어떤 단어든 하나의 단어를 매치합니다. (`[^ ]+`; 비어있지 않은, 공백없이 연결된 시퀀스). 그리고 나서 "포트(port)" 와 그 뒤에 따라오는 일련의 숫자를 봅니다. 그리고 나서 있을수도 있는 접미사 `[preauth]` 를 확인하고, 줄의 끝을 확인합니다. 

Notice that with this technique, as username of "Disconnected from"
won't confuse us any more. Can you see why?

이 테크닉을 썼을때 사용자 이름이 "Disconnected from" 이더라도 우리에게 혼란을 주지 않습니다. 왜 그런지 보이시나요? 

There is one problem with this though, and that is that the entire log
becomes empty. We want to _keep_ the username after all. For this, we
can use "capture groups". Any text matched by a regex surrounded by
parentheses is stored in a numbered capture group. These are available
in the substitution (and in some engines, even in the pattern itself!)
as `\1`, `\2`, `\3`, etc. So:

하지만 한 가지 문제는 전체 로그가 비어 있다는 것입니다. 우리는 사용자 이름을 유지하고 싶기 때문입니다. 이를 위헤 우리는 캡쳐 그룹을 사용할수 있습니다. "" 로 둘러쌓여 있는 정규식과 일치하는 모든 텍스트 괄호는 번호가 매겨진 캡처 그룹에 저장됩니다. 이들은 대체해서 `\1`, `\2`, `\3`, 등등으로 사용할 수 있습니다 (그리고 일부 엔진에서는 패턴 자체에서도 가능합니다!) 

```bash
 | sed -E 's/.*Disconnected from (invalid |authenticating )?user (.*) [^ ]+ port [0-9]+( \[preauth\])?$/\2/'
```

As you can probably imagine, you can come up with _really_ complicated
regular expressions. For example, here's an article on how you might
match an [e-mail
address](https://www.regular-expressions.info/email.html). It's [not
easy](https://emailregex.com/). And there's [lots of
discussion](https://stackoverflow.com/questions/201323/how-to-validate-an-email-address-using-a-regular-expression/1917982).
And people have [written
tests](https://fightingforalostcause.net/content/misc/2006/compare-email-regex.php).
And [test matrices](https://mathiasbynens.be/demo/url-regex). You can
even write a regex for determining if a given number [is a prime
number](https://www.noulakaz.net/2007/03/18/a-regular-expression-to-check-for-prime-numbers/).

상상할수 있듯이, 우리는 정말 복잡한 정규식을 만들어낼수도 있습니다. 예를 들어, 다음은 [이메일 주소를](https://www.regular-expressions.info/email.html) 매치할수 있는 방법에 대한 글입니다. 이것은 [쉽지 않습니다](https://emailregex.com/). 그리고 이것에 대한 [다양한 토론이 존재합니다](https://stackoverflow.com/questions/201323/how-to-validate-an-email-address-using-a-regular-expression/1917982). 사람들은 [테스트를 쓰기](https://fightingforalostcause.net/content/misc/2006/compare-email-regex.php)도 했습니다.
그리고 [테스트 메트릭스](https://mathiasbynens.be/demo/url-regex)도 있습니다. 또, 주어진 숫자가 [소수인지 확인](https://www.noulakaz.net/2007/03/18/a-regular-expression-to-check-for-prime-numbers/) 하기 위해 정규식을 작성할 수도 있습니다.


Regular expressions are notoriously hard to get right, but they are also
very handy to have in your toolbox!

정규 표현식은 제대로 이해하기 어렵지만 도구 상자에 있으면 매우 편리합니다!

## Back to data wrangling
## 데이터 랭글링으로 돌아가기

Okay, so we now have

좋아요, 이제 우리는 다음을 가지고 있습니다: 

```bash
ssh myserver journalctl
 | grep sshd
 | grep "Disconnected from"
 | sed -E 's/.*Disconnected from (invalid |authenticating )?user (.*) [^ ]+ port [0-9]+( \[preauth\])?$/\2/'
```

`sed` can do all sorts of other interesting things, like injecting text
(with the `i` command), explicitly printing lines (with the `p`
command), selecting lines by index, and lots of other things. Check `man
sed`!

`sed`는 텍스트 삽입과 같은 모든 종류의 흥미로운 일을 할 수 있습니다(`i` 명령 사용). 명시 적으로 행 인쇄 (`p` 사용), 색인별로 줄 선택 및 기타 여러 가지를 할 수 있습니다. 더 알고 싶다면 `man sed`을 확인해 보세요!

Anyway. What we have now gives us a list of all the usernames that have
attempted to log in. But this is pretty unhelpful. Let's look for common
ones:

어쨌든. 이제 우리가 가진 모든 사용자 이름중 로그인을 시도한 사람들의 목록을 제공합니다. 그러나 이것은 매우 도움이 되지는 않습니다. 자주 보이는 이름들을 봅시다: 

```bash
ssh myserver journalctl
 | grep sshd
 | grep "Disconnected from"
 | sed -E 's/.*Disconnected from (invalid |authenticating )?user (.*) [^ ]+ port [0-9]+( \[preauth\])?$/\2/'
 | sort | uniq -c
```

`sort` will, well, sort its input. `uniq -c` will collapse consecutive
lines that are the same into a single line, prefixed with a count of the
number of occurrences. We probably want to sort that too and only keep
the most common logins:

`sort`는 입력을 정렬합니다. `uniq -c`는 연속적인 줄을 축소하고, 한 줄에 동일한 줄의 수를 발생 횟수를 접두사로 더합니다. 우리는 아마 이것도 정렬하고 가장 일반적인 로그인만 저장하고 싶을 것입니다: 

```bash
ssh myserver journalctl
 | grep sshd
 | grep "Disconnected from"
 | sed -E 's/.*Disconnected from (invalid |authenticating )?user (.*) [^ ]+ port [0-9]+( \[preauth\])?$/\2/'
 | sort | uniq -c
 | sort -nk1,1 | tail -n10
```

`sort -n` will sort in numeric (instead of lexicographic) order. `-k1,1`
means "sort by only the first whitespace-separated column". The `,n`
part says "sort until the `n`th field, where the default is the end of
the line. In this _particular_ example, sorting by the whole line
wouldn't matter, but we're here to learn!

`sort -n`은 사전식이 아닌 숫자식으로 정렬합니다. `-k1,1` 은 "공백으로 구분 된 첫 번째 열로만 정렬"을 의미합니다. `, n` 부분은 "n 번째 필드까지 정렬하는데, 여기서 기본값은 이 줄의 끝입니다" 를 의미합니다. 이 예제에서는 전체 행을 기준으로 정렬하는 것은 상관 없지만, 우리는 배우기 위해 왔습니다!

If we wanted the _least_ common ones, we could use `head` instead of
`tail`. There's also `sort -r`, which sorts in reverse order.

우리가 가장 일반적이지 않은 것을 원하면 `tail` 대신 `head`를 사용할 수 있습니다.역순으로 정렬하는`sort -r`도 있습니다.

Okay, so that's pretty cool, but we'd sort of like to only give the
usernames, and maybe not one per line?

좋습니다. 꽤 멋지지만 우리는 한줄에 하나씩만 주는것이 아니라 이용자 이름만 보고 싶을 수도 있습니다. 

```bash
ssh myserver journalctl
 | grep sshd
 | grep "Disconnected from"
 | sed -E 's/.*Disconnected from (invalid |authenticating )?user (.*) [^ ]+ port [0-9]+( \[preauth\])?$/\2/'
 | sort | uniq -c
 | sort -nk1,1 | tail -n10
 | awk '{print $2}' | paste -sd,
```

Let's start with `paste`: it lets you combine lines (`-s`) by a given
single-character delimiter (`-d`). But what's this `awk` business?

`paste` (붙여 넣기) 부터 시작해 보겠습니다: 이 커맨드는 주어진 줄 ( '-s')을 합치도록 단일 문자 구분 기호 (`-d`)를 사용합니다. 그렇다면 이 `awk` 은 무엇일까요? 

## awk -- another editor
## awk -- 또다른 문서 편집 프로그램 (에디터)


`awk` is a programming language that just happens to be really good at
processing text streams. There is _a lot_ to say about `awk` if you were
to learn it properly, but as with many other things here, we'll just go
through the basics.

`awk`는 텍스트 스트림 처리를 정말 잘하는 프로그래밍 언어입니다. 만약 당신이 'awk'에 대해 제대로 배우려면 말할 것들이 정말 많지만, 여기에있는 다른 많은 것들과 마찬가지로 우리는 기본만 보려고 합니다. 

First, what does `{print $2}` do? Well, `awk` programs take the form of
an optional pattern plus a block saying what to do if the pattern
matches a given line. The default pattern (which we used above) matches
all lines. Inside the block, `$0` is set to the entire line's contents,
and `$1` through `$n` are set to the `n`th _field_ of that line, when
separated by the `awk` field separator (whitespace by default, change
with `-F`). In this case, we're saying that, for every line, print the
contents of the second field, which happens to be the username!

**첫째, `{print $ 2}`는 무엇을 합니까? `awk` 프로그램은
선택적 패턴과 패턴이 주어진 줄과 일치합니다. 위에서 사용한 기본 패턴은
모든 라인. 블록 내에서`$0`은 전체 라인의 내용으로 설정됩니다.
그리고`$1`에서`$n`은 해당 줄의`n` 번째 _field_로 설정됩니다.
`awk` 필드 구분자로 구분 (기본적으로 공백, 변경
`-F`로). 이 경우, 우리는 모든 줄에 대해 두 번째 필드의 내용을 프린트 하라고 말하는 것이고, 그 필드는 사용자 이름입니다!

Let's see if we can do something fancier. Let's compute the number of
single-use usernames that start with `c` and end with `e`:

더 멋진 일을 할 수 있는지 봅시다. 딱 한번 사용된 사용자 이름중 `c`로 시작하고 `e`로 끝나는 이름의 수를 계산합시다:

```bash
 | awk '$1 == 1 && $2 ~ /^c[^ ]*e$/ { print $2 }' | wc -l
```

There's a lot to unpack here. First, notice that we now have a pattern
(the stuff that goes before `{...}`). The pattern says that the first
field of the line should be equal to 1 (that's the count from `uniq
-c`), and that the second field should match the given regular
expression. And the block just says to print the username. We then count
the number of lines in the output with `wc -l`.

**여기에 풀어야 할 것이 많습니다. 먼저, 이제 패턴이 있음을 확인하십시오. (`{...}`앞에 오는 것). 패턴은 첫 번째
줄의 필드는 1과 같아야 한다고 말하고 ( 'uniq
-c`), 두 번째 필드는 주어진 일반 필드와 일치해야 한다고 말합니다.
그리고 블록은 사용자 이름을 인쇄하라고 말합니다. 그런 다음 우리는
`wc -l`을 사용해서 출력의 행 수를 계산하고 싶습니다.

However, `awk` is a programming language, remember?

그러나`awk`는 프로그래밍 언어입니다. 기억하십니까? 

```awk
BEGIN { rows = 0 }
$1 == 1 && $2 ~ /^c[^ ]*e$/ { rows += $1 }
END { print rows }
```

`BEGIN` is a pattern that matches the start of the input (and `END`
matches the end). Now, the per-line block just adds the count from the
first field (although it'll always be 1 in this case), and then we print
it out at the end. In fact, we _could_ get rid of `grep` and `sed`
entirely, because `awk` [can do it
all](https://backreference.org/2010/02/10/idiomatic-awk/), but we'll
leave that as an exercise to the reader.

`BEGIN` 은 입력값의 시작과 일치하는 패턴입니다 (`END` 는 끝과 매치합니다). 이제, 행당 블록은 첫 번째 필드 (이 예제에서는 늘 1 이지만) 에서 나온 카운트를 더하고, 맨 끝에 인쇄합니다. 사실, 우리는 `grep` 와 `sed`를 완전히 제거할수도 있습니다. 그 이유는 `awk` [가 다 할수 있기 때문입니다](https://backreference.org/2010/02/10/idiomatic-awk/). 그렇지만 여기서는 독자에게 연습으로 남겨놓겠습니다. 

## Analyzing data
## 데이터 분석

You can do math! For example, add the numbers on each line together:
수학을 할 수 있습니다! 예를 들어 각 줄에 숫자를 함께 추가합니다.

```bash
 | paste -sd+ | bc -l
```

Or produce more elaborate expressions:

또는 보다 정교한 표현을 생성합니다: 

```bash
echo "2*($(data | paste -sd+))" | bc -l
```

You can get stats in a variety of ways.
[`st`](https://github.com/nferraz/st) is pretty neat, but if you already
have R:

다양한 방법으로 통계를 얻을 수 있습니다.
[`st`](https://github.com/nferraz/st)는 꽤 깔끔하지만 이미
R을 가지고 있다면:

```bash
ssh myserver journalctl
 | grep sshd
 | grep "Disconnected from"
 | sed -E 's/.*Disconnected from (invalid |authenticating )?user (.*) [^ ]+ port [0-9]+( \[preauth\])?$/\2/'
 | sort | uniq -c
 | awk '{print $1}' | R --slave -e 'x <- scan(file="stdin", quiet=TRUE); summary(x)'
```

R is another (weird) programming language that's great at data analysis
and [plotting](https://ggplot2.tidyverse.org/). We won't go into too
much detail, but suffice to say that `summary` prints summary statistics
about a matrix, and we computed a matrix from the input stream of
numbers, so R gives us the statistics we wanted!

R은 데이터 분석과 [플로팅](https://ggplot2.tidyverse.org/) 에 뛰어난 또 다른 (이상한) 프로그래밍 언어입니다. 세부 사항에 들어가지는 않겠지만,`summary`가 요약 통계를 출력한하고, 입력 스트림에서 행렬을 계산하기에, R은 우리가 원하는 통계를 제공한다는 것만 이야기해도 충분합니다.

If you just want some simple plotting, `gnuplot` is your friend:

간단한 플로팅을 원한다면 `gnuplot`이 당신의 친구입니다. 

```bash
ssh myserver journalctl
 | grep sshd
 | grep "Disconnected from"
 | sed -E 's/.*Disconnected from (invalid |authenticating )?user (.*) [^ ]+ port [0-9]+( \[preauth\])?$/\2/'
 | sort | uniq -c
 | sort -nk1,1 | tail -n10
 | gnuplot -p -e 'set boxwidth 0.5; plot "-" using 1:xtic(2) with boxes'
```

## Data wrangling to make arguments

Sometimes you want to do data wrangling to find things to install or
remove based on some longer list. The data wrangling we've talked about
so far + `xargs` can be a powerful combo:

때로는 설치할 항목을 찾기 위해 더 긴 목록을 기반으로 제거하기 위해 데이터 랭 글링을 수행합니다. 우리가 지금까지 이야기 한 데이터 랭 글링 + `xargs`는 강력한 콤보가 될 수 있습니다.

```bash
rustup toolchain list | grep nightly | grep -vE "nightly-x86" | sed 's/-x86.*//' | xargs rustup toolchain uninstall
```

## Wrangling binary data
## 바이너리 데이터 랭 글링 

So far, we have mostly talked about wrangling textual data, but pipes
are just as useful for binary data. For example, we can use ffmpeg to
capture an image from our camera, convert it to grayscale, compress it,
send it to a remote machine over SSH, decompress it there, make a copy,
and then display it.

지금까지 우리는 주로 텍스트 데이터 랭 글링에 대해 이야기했지만 파이프는 바이너리 데이터에도 유용합니다. 예를 들어 ffmpeg를 사용하여
카메라에서 이미지를 캡처하여 그레이 스케일로 변환하고 압축 한 다음
SSH를 통해 원격 컴퓨터로 전송하고 압축을 풀고 사본을 만들고 보여줍니다. 

```bash
ffmpeg -loglevel panic -i /dev/video0 -frames 1 -f image2 -
 | convert - -colorspace gray -
 | gzip
 | ssh mymachine 'gzip -d | tee copy.jpg | env DISPLAY=:0 feh -'
```

# Exercises
# 연습 

1. Take this [short interactive regex tutorial](https://regexone.com/).

1. 이 수업을 들으세요. [짧은, 직접해볼수 있는 정규식 수업](https://regexone.com/).

2. Find the number of words (in `/usr/share/dict/words`) that contain at
   least three `a`s and don't have a `'s` ending. What are the three
   most common last two letters of those words? `sed`'s `y` command, or
   the `tr` program, may help you with case insensitivity. How many
   of those two-letter combinations are there? And for a challenge:
   which combinations do not occur?

2. 적어도 세개의 `a`와 `'s`를 마지막으로 가지고 있지 않은 단어의 갯수를 `/usr/share/dict/words` 에서 찾아보세요. 세개의 가장 흔한 마지막 두 글자는 무엇인가요? `sed`의 `y` 명령이나 `tr` 프로그램이 도움이 될수 있습니다. 이런 두 글자 조합이 몇개나 있습니까? 그리고, 어떤 조합은 발생하지 않습니까? 

3. To do in-place substitution it is quite tempting to do something like
   `sed s/REGEX/SUBSTITUTION/ input.txt > input.txt`. However this is a
   bad idea, why? Is this particular to `sed`? Use `man sed` to find out
   how to accomplish this.

3. 내부 대체를 수행하려면 다음과 같이 하고 싶은 마음이 굴뚝같을수 있습니다: `sed s/REGEX/SUBSTITUTION/ input.txt > input.txt`. 그렇지만, 이것은 좋은 방법이 아닙니다. 왜 일까요? 이것이 `sed`에만 해당합니까? `man sed`을 사용해서 왜 그런지, 또 이를 수행하는 방법은 무엇일지 알아보세요. 

4. Find your average, median, and max system boot time over the last ten
   boots. Use `journalctl` on Linux and `log show` on macOS, and look
   for log timestamps near the beginning and end of each boot. On Linux,
   they may look something like:

4. 지난 10 번의 부팅시간 동안의 평균, 중앙값 및 최대 시스템 부팅 시간을 찾아보세요. 리눅스에서는 `journalctl`을 사용하고, macOS에서는 `log show`을 사용해서 각 부팅의 시작과 끝 부분에있는 로그 타임 스탬프를 확인하세요. 리눅스에서는 다음과 같은 것을 볼수 있을 것입니다: 

   ```
   Logs begin at ...
   ```
   ```
   systemd[577]: Startup finished in ...
   ```
   macOS에서는 , [다음과 같은 로그를 찾아보세요](https://eclecticlight.co/2018/03/21/macos-unified-log-3-finding-your-way/):
   ```
   === system boot:
   ```
   ```
   Previous shutdown cause: 5
   ```
5. **Look for boot messages that are _not_ shared between your past three
   reboots (see `journalctl`'s `-b` flag). Break this task down into
   multiple steps. First, find a way to get just the logs from the past
   three boots. There may be an applicable flag on the tool you use to
   extract the boot logs, or you can use `sed '0,/STRING/d'` to remove
   all lines previous to one that matches `STRING`. Next, remove any
   parts of the line that _always_ varies (like the timestamp). Then,
   de-duplicate the input lines and keep a count of each one (`uniq` is
   your friend). And finally, eliminate any line whose count is 3 (since
   it _was_ shared among all the boots).

5. 
6. **Find an online data set like [this
   one](https://stats.wikimedia.org/EN/TablesWikipediaZZ.htm), [this
   one](https://ucr.fbi.gov/crime-in-the-u.s/2016/crime-in-the-u.s.-2016/topic-pages/tables/table-1).
   or maybe one [from
   here](https://www.springboard.com/blog/free-public-data-sets-data-science-project/).
   Fetch it using `curl` and extract out just two columns of numerical
   data. If you're fetching HTML data,
   [`pup`](https://github.com/EricChiang/pup) might be helpful. For JSON
   data, try [`jq`](https://stedolan.github.io/jq/). Find the min and
   max of one column in a single command, and the sum of the difference
   between the two columns in another.




* Used Google Translation tool for translating and added personal edits for smooth translation and readability. 
* 번역을 위해 Google 번역 도구를 사용하고 원활한 번역과 가독성을 위해 개인 편집을 추가했습니다.