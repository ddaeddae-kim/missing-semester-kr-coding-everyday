---
layout: lecture
title: "Data Wrangling"
date: 2019-01-16
ready: true
video:
  aspect: 56.25
  id: sz_dsktIjt4
---

데이터를 하나의 형식으로 가져 와서 다른 형식으로 바꾸고 싶은 경험이 있으셨나요? 물론 있었을 거에요! 그리고 바로 이것이 이 강의의 내용입니다. 특히, 텍스트이든 이진 형식이든, 데이터를 원하는 형식으로 변화시키는 과정에 대해 보도록 하겠습니다. 

전 수업들에서 이미 우리는 기본적인 데이터 랭글링 또는 형태 전환 (data wrangling) 을 본적이 있습니다. 우리가 `|` 을 사용할때마다 우리는 일종의 데이터 랭글링을 하고 있는 것입니다. `journalctl | grep -i intel` 이라는 명령을 생각해 보세요. 이 명령은 시스템에 있는 모든 로그 중에서 Intel(대소문자 구분 안함) 가 들어간 로그를 찾습니다. 이것이 데이터를 랭글링 하는 것이라 생각하지 않을수도 있지만, 사실 이것은 한 형식 (시스템 로그 전체) 에서 다른, 당신에게 더 활용하기 쉬운 형식으로 (인텔 로그만) 바꾸는 것 입니다. 대부분의 데이터 랭글링은 사용할수 있는 도구를 알고 그것들을 결합하는 방법을 아는 것입니다. 

기본부터 시작해 봅시다. 데이터를 변환하기 위해서는 두 가지가 필요합니다: 변환할 데이터와, 그것을 가지고 무엇을 할지 입니다. 로그는 종종 유익한데, 그 이유는 우리가 보통 로그를 가지고 조사하기를 원하고, 전체를 다 읽는것은 실현 가능하지 않기 때문입니다. 누가 로그인 하려고 하는지 내 서버의 로그를 보고 알아봅시다: 

```bash
ssh myserver journalctl
```

이것은 너무나 많은 로그를 불러옵니다. ssh를 한 데이터로만 제한해 봅시다. 

```bash
ssh myserver journalctl | grep sshd
```

우리는 파이프을 사용해서 원격 파일을 `grep` 우리 로컬 컴퓨터로 스트리밍 하고 있습니다! `ssh` 는 마법같고, 이에 대해서는 명령어 환경에 대한 다음 수업에서 더 이야기하겠습니다. 아직 이 데이터에는 우리가 원하는 정보 외에도 많은 것이 포함되어 있습니다. 그리고 읽기에도 쉽지 않습니다. 더 발전시켜 봅니다. 

```bash
ssh myserver 'journalctl | grep sshd | grep "Disconnected from"' | less
```

왜 추가적인 따옴표를 넣냐구요? 그 이유는 우리의 로그는 상당히 클 수 있으며 전체를 우리 컴퓨터로 스트리밍 한 다음 필터링을 수행하는 것은 낭비이기 때문이에요. 대신, 우리는 원격 서버에서 필터링을 진행하고, 로컬에서 데이터를 변환합니다. `less`는 긴 출력을 위아래로 스크롤 할 수 있는 "페이저"를 제공합니다. 추가 트래픽을 절약하려면 명령어(command-line)에서 디버깅을 할 때 프레픽을 좀 더 아끼기 위해서 우리는 현재 필터된 로그를 파일로 저장해 개발 중 네트워크에 액세스 할 필요가 없도록 할 수도 있어요. 

```console
$ ssh myserver 'journalctl | grep sshd | grep "Disconnected from"' > ssh.log
$ less ssh.log
```

아직도 잡음이 많습니다. 제거하는 방법은 많지만, 그 중 가장 강력한 도구 중 하나인 `sed`을 살펴 보겠습니다. 

`sed` 는 이전 `ed` 편집기 위에 빌드된 "스트림 편집기"입니다. 이는, 쉽게 말해서 내용을 직접 조작하는 대신 (그렇게 할 수 있지만) 파일을 수정하는 방법에 대한 짧은 명령을 제공합니다. 많은 명령어들이 있지만, 그중 가장 많이 사용되는 것은  `s`: 치환 입니다. 예를 들어서 다음과 같이 작성할 수 있습니다. 

```bash
ssh myserver journalctl
 | grep sshd
 | grep "Disconnected from"
 | sed 's/.*Disconnected from //'
```

우리가 쓴것은 간단한 정규표현 (regular expression) 입니다. 패턴과 텍스트를 일치시킬 수 있는 아주 효과적인 구성 (construct) 입니다. 명령어 `s` 는 `s/REGEX/SUBSTITUTION/` 이런 형식으로 사용하는데, 이때 `REGEX` 는 검색하려는 정규표현이고, `SUBSTITUTION` 는 일치하는 텍스트를 대체 할 텍스트입니다.

## 정규 표현 (Regular expressions)

정규표현은 일반적이고 유용해서, 시간을 갖고 이해할 필요가 있습니다. 바로 위 예시에서 봤던 표현을 보면: `/.*Disconnected from /` 
정규표현은 대체로 (늘 그렇지는 않지만) `/`로 둘려쌓여 있습니다. 대부분의 ASCII 문자는 정상적인 의미를 지니지만 일부 캐릭터는 "특별한" 일치 습성을 가지고 있습니다. 이러한 캐릭터들이 무엇을 하는지는 어떤 정규표현인지에 따라 다르기에 큰 좌절의 원천이 되기도 합니다. 매우 일반적인 패턴은 다음과 같습니다.

 - `.` :줄 바꿈을 제외한 "모든 단일 문자"
 - `*` :0 개 이상의 선행 일치 
 - `+` :하나 이상의 이전 일치
 - `[abc]` :`a`, `b`, `c` 중 한 문자와 일치 
 - `(RX1|RX2)` :`RX1` 또는 `RX2` 에 일치하는 항목
 - `^` :줄의 시작
 - `$` :줄의 끝  

`sed`의 정규표현은 다소 이상한데, 특별한 의미를 부여하기 위해 대부분의 앞에 `\`을 붙여야 합니다. 또는 `-E`를 붙일수도 있습니다. 

따라서 `/.*Disconnected from /` 을 살펴보면, 이 표현은 임의의 수의 문자로 시작하고 그 뒤에 "Disconnected from &rdquo; 가 있다면 모두 일치하다고 판단합니다. 이것은 우리가 원하던 것입니다. 그렇지만, 조심해야 할 것이 있습니다. 누군가 "Disconnected from" 라는 아이디로 로그인을 한다면 어떻게 될까요? 

```
Jan 17 03:13:00 thesquareplanet.com sshd[2631]: Disconnected from invalid user Disconnected from 46.97.239.16 port 55920 [preauth]
```

우리는 결국 어떤 결과를 받을까요? `*` 와 `+` 는 그리디 (greedy) 합니다. 가능한 한 많은 텍스트와 일치시킵니다. 그렇기 때문에, 위를 보면, 우리는 결국 이런 결과를 가지게 됩니다.

```
46.97.239.16 port 55920 [preauth]
```

이 결과는 우리가 원하지 않는 것일수도 있습니다. 어떤 정규표현에서는, `*` 이나 `+` 앞에  `?`를 붙여서 그리디 하지 않게 만들수도 있지만, 안타깝게도 `sed`는 그런 기능을 가지고 있지 않습니다. 우리는 펄(perl)의 명령어 모드로 바꿀수도 있습니다. 이는 다음과 같은 구성을 지원합니다.

```bash
perl -pe 's/.*?Disconnected from //'
```

`sed`는 이러한 종류의 작업을 위한 일반적인 도구이기 때문에 이후 예제들에서 `sed` 를 사용하기로 하겠습니다. `sed` 는 다른 편리한 작업도 할 수 있는데, 예를 들어 일치하는 줄을 인쇄하거나, 여러 호출의 대체, 검색등을 할 수 있습니다. 그렇지만, 여기서는 그런것들을 모두 다루지는 않겠습니다. `sed` 는 그 자체로도 큰 주제이고, 다른 좋은 도구도 많기 때문입니다. 

좋아요, 우리는 또한 제거하고 싶은 접미사를 가지고 있습니다. 어떻게 이것을 성취할수 있을까요? 사용자 이름 다음에 오는 텍스트만 일치시키는 것은 약간 까다롭습니다. 특히 사용자 이름에 공백이 있거나 하면요. 이때 우리가 해야할 것은 라인 전체에 일치시키는 것입니다. 

```bash
 | sed -E 's/.*Disconnected from (invalid |authenticating )?user .* [^ ]+ port [0-9]+( \[preauth\])?$//'
```

[정규 표현 디버거 (regex
debugger)](https://regex101.com/r/qqbZqh/2) 를 통해 무슨 일이 일어나고 있는지 알아보도록 합시다. 시작은 전과 같습니다. 그리고 나서는, 우리는 유저 변수들을 매치합니다 (로그에는 두개의 접두사가 있습니다). 그런 다음 우리는 어떤 스트링이든 사용자 이름이 있는 곳에 매치합니다. 그리고 나서는 어떤 단어든 하나의 단어를 매치합니다. (`[^ ]+`; 비어있지 않은, 공백없이 연결된 시퀀스). 그리고 나서 "포트(port)" 와 그 뒤에 따라오는 일련의 숫자를 봅니다. 그리고 나서 있을수도 있는 접미사 `[preauth]` 를 확인하고, 줄의 끝을 확인합니다. 

이 테크닉을 썼을때 사용자 이름이 "Disconnected from" 이더라도 더이상 혼란을 주지 않습니다. 왜 그런지 보이시나요? 

하지만 한 가지 문제는 전체 로그가 비어 있다는 것입니다. 우리는 사용자 이름을 유지하고 싶기 때문입니다. 이를 위해 우리는 "캡쳐 그룹"을 사용할수 있습니다. 괄호로 둘러쌓여 있는 정규식과 일치하는 모든 텍스트는 번호가 매겨진 캡처 그룹에 저장됩니다. 이들은 대체해서 `\1`, `\2`, `\3`, 등등으로 사용할 수 있습니다 (그리고 일부 엔진에서는 패턴 자체에서도 가능합니다!) 

```bash
 | sed -E 's/.*Disconnected from (invalid |authenticating )?user (.*) [^ ]+ port [0-9]+( \[preauth\])?$/\2/'
```

상상할수 있듯이, 우리는 정말 복잡한 정규식을 만들어낼수도 있습니다. 예를 들어, 다음은 [이메일 주소를](https://www.regular-expressions.info/email.html) 매치할수 있는 방법에 대한 글입니다. 이것은 [쉽지 않습니다](https://emailregex.com/). 그리고 이것에 대한 [다양한 토론이 존재합니다](https://stackoverflow.com/questions/201323/how-to-validate-an-email-address-using-a-regular-expression/1917982). 사람들은 [테스트를 쓰기](https://fightingforalostcause.net/content/misc/2006/compare-email-regex.php)도 했습니다.
그리고 [테스트 메트릭스](https://mathiasbynens.be/demo/url-regex)도 있습니다. 또, 주어진 숫자가 [소수인지 확인](https://www.noulakaz.net/2007/03/18/a-regular-expression-to-check-for-prime-numbers/) 하기 위해 정규식을 작성할 수도 있습니다.

정규 표현식은 제대로 이해하기 어렵지만 도구 상자에 있으면 매우 편리합니다!

## 데이터 랭글링으로 돌아가기

좋아요, 이제 우리는 다음을 가지고 있습니다.

```bash
ssh myserver journalctl
 | grep sshd
 | grep "Disconnected from"
 | sed -E 's/.*Disconnected from (invalid |authenticating )?user (.*) [^ ]+ port [0-9]+( \[preauth\])?$/\2/'
```

`sed`는 텍스트 삽입과 같은 모든 종류의 흥미로운 일을 할 수 있습니다(`i` 명령 사용). 명시 적으로 행 인쇄 (`p` 사용), 인덱스로 라인 선택 및 기타 여러 가지를 할 수 있습니다. 더 알고 싶다면 `man sed`을 확인해 보세요!

어쨌든, 이제 우리가 가진 모든 사용자 이름 중 로그인을 시도한 사람들의 목록을 제공합니다. 그러나 이것은 매우 도움이 되지는 않습니다. 자주 보이는 이름들을 봅시다: 

```bash
ssh myserver journalctl
 | grep sshd
 | grep "Disconnected from"
 | sed -E 's/.*Disconnected from (invalid |authenticating )?user (.*) [^ ]+ port [0-9]+( \[preauth\])?$/\2/'
 | sort | uniq -c
```

`sort`는 입력을 정렬합니다. `uniq -c`는 연속적인 줄을 축소하고, 한 줄에 동일한 줄의 수를 발생 횟수를 접두사로 더합니다. 우리는 아마 이것도 정렬하고 가장 자주 등장하는 로그인만 저장하고 싶을 것입니다: 

```bash
ssh myserver journalctl
 | grep sshd
 | grep "Disconnected from"
 | sed -E 's/.*Disconnected from (invalid |authenticating )?user (.*) [^ ]+ port [0-9]+( \[preauth\])?$/\2/'
 | sort | uniq -c
 | sort -nk1,1 | tail -n10
```

`sort -n`은 사전식이 아닌 숫자식으로 정렬합니다. `-k1,1` 은 "공백으로 구분 된 첫 번째 열로만 정렬"을 의미합니다. `, n` 부분은 "n 번째 필드까지 정렬하는데, 여기서 기본값은 이 줄의 끝입니다" 를 의미합니다. 이 예제에서는 전체 행을 기준으로 정렬하는 것은 상관 없지만, 이것도 좋은 배움의 기회입니다!

우리가 가장 일반적이지 않은 것을 원하면 `tail` 대신 `head`를 사용할 수 있습니다.역순으로 정렬하는`sort -r`도 있습니다.

좋습니다. 꽤 멋지지만 우리는 한 줄에 하나씩만 보는것이 아니라 이용자 이름만 보고 싶을 수도 있습니다. 

```bash
ssh myserver journalctl
 | grep sshd
 | grep "Disconnected from"
 | sed -E 's/.*Disconnected from (invalid |authenticating )?user (.*) [^ ]+ port [0-9]+( \[preauth\])?$/\2/'
 | sort | uniq -c
 | sort -nk1,1 | tail -n10
 | awk '{print $2}' | paste -sd,
```

`paste` (붙여 넣기) 부터 시작해 보겠습니다. 이 커맨드는 주어진 줄 (`-s`)을 합치도록 단일 문자 구분 기호 (`-d`)를 사용합니다. 그렇다면 이 `awk` 은 무엇일까요? 

## awk -- 또다른 문서 편집 프로그램 (에디터)


`awk`는 텍스트 스트림 처리를 정말 잘하는 프로그래밍 언어입니다. 만약 당신이 `awk`에 대해 제대로 배우려면 말할 것들이 정말 많지만, 여기에있는 다른 많은 것들과 마찬가지로 우리는 기본만 보려고 합니다. 

첫째, `{print $ 2}`는 무엇을 합니까? `awk` 프로그램은 선택적 패턴과 패턴이 주어진 줄과 일치할시, 무엇을 하고 싶은지 설명하는 블록을 받습니다. 위에서 사용한 기본 패턴은
모든 라인을 일치시킵니다. 블록 내에서`$0`은 전체 라인의 내용으로 설정됩니다 그리고 `awk` 필드 구분자로 구분되었다면 (기본적으로 공백, 변경
`-F`로), `$1`에서`$n`까지는 해당 줄의`n` 번째 필드로 설정됩니다. 이 경우, 우리는 모든 줄에 대해 두 번째 필드의 내용을 프린트 하라고 말하는 것이고, 그 필드는 사용자 이름입니다!

더 멋진 일을 할 수 있는지 봅시다. 딱 한번 사용된 사용자 이름중 `c`로 시작하고 `e`로 끝나는 이름의 수를 계산합니다.

```bash
 | awk '$1 == 1 && $2 ~ /^c[^ ]*e$/ { print $2 }' | wc -l
```

여기에 풀어야 할 것이 많습니다. 먼저, 이제 패턴이 있음을 확인하십시오. (`{...}`앞에 오는 것 입니다). 패턴은 첫 번째 줄의 필드는 1과 같아야 한다고 말하고 ( 이것은 `uniq -c` 을 사용해 얻은 숫자입니다), 두 번째 필드는 주어진 주어진 정규표현과 일치해야 한다고 말합니다. 그리고 블록은 사용자 이름을 인쇄하라고 말합니다. 그런 다음 우리는 `wc -l`을 사용해서 출력값의 행 수를 계산하고 싶습니다.

그러나`awk`는 프로그래밍 언어입니다. 기억하십니까? 

```awk
BEGIN { rows = 0 }
$1 == 1 && $2 ~ /^c[^ ]*e$/ { rows += $1 }
END { print rows }
```

`BEGIN` 은 입력값의 시작과 일치하는 패턴입니다 (`END` 는 끝과 매치합니다). 이제, 행당 블록은 첫 번째 필드 (이 예제에서는 늘 1 이지만) 에서 나온 카운트를 더하고, 맨 끝에 인쇄합니다. 사실, 우리는 `grep` 와 `sed`를 완전히 제거할수도 있습니다. 그 이유는 `awk` [가 다 할수 있기 때문입니다](https://backreference.org/2010/02/10/idiomatic-awk/). 그렇지만 여기서는 독자에게 연습으로 남겨놓겠습니다. 

## 데이터 분석

수학을 할 수 있습니다! 예를 들어 각 줄에 숫자를 함께 추가합니다.

```bash
 | paste -sd+ | bc -l
```

또는 보다 정교한 표현을 생성합니다: 

```bash
echo "2*($(data | paste -sd+))" | bc -l
```

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

R은 데이터 분석과 [플로팅(plotting)](https://ggplot2.tidyverse.org/) 에 뛰어난 또 다른 (이상한) 프로그래밍 언어입니다. 세부 사항에 들어가지는 않겠지만,`summary`가 요약 통계를 출력한하고, 입력 스트림에서 행렬을 계산하기에, R은 우리가 원하는 통계를 제공한다는 것만 이야기해도 충분합니다.

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

## 인수를 만들기위한 데이터 랭글링

때로는 설치할 항목을 찾기 위해 또는 더 긴 목록을 기반으로 제거하기 위해 데이터 랭글링을 수행합니다. 우리가 지금까지 이야기 한 데이터 랭 글링 + `xargs`는 강력한 콤보가 될 수 있습니다.

```bash
rustup toolchain list | grep nightly | grep -vE "nightly-x86" | sed 's/-x86.*//' | xargs rustup toolchain uninstall
```

## 바이너리 데이터 랭글링 

지금까지 우리는 주로 텍스트 데이터 랭 글링에 대해 이야기했지만 파이프는 바이너리 데이터에도 유용합니다. 예를 들어 ffmpeg를 사용하여
카메라에서 이미지를 캡처하여 그레이 스케일로 변환하고 압축 한 다음
SSH를 통해 원격 컴퓨터로 전송하고 압축을 풀고 사본을 만들고 보여줍니다. 

```bash
ffmpeg -loglevel panic -i /dev/video0 -frames 1 -f image2 -
 | convert - -colorspace gray -
 | gzip
 | ssh mymachine 'gzip -d | tee copy.jpg | env DISPLAY=:0 feh -'
```

# 연습 

1. 이 수업을 들으세요. [짧은, 직접해볼수 있는 정규식 수업](https://regexone.com/).

2. 적어도 세개의 `a`와 `'s`를 마지막으로 가지고 있지 않은 단어의 갯수를 `/usr/share/dict/words` 에서 찾아보세요. 세개의 가장 흔한 마지막 두 글자는 무엇인가요? `sed`의 `y` 명령이나 `tr` 프로그램이 도움이 될수 있습니다. 이런 두 글자 조합이 몇개나 있습니까? 그리고, 어떤 조합은 발생하지 않습니까? 

3. 내부 대체를 수행하려면 다음과 같이 하고 싶은 마음이 굴뚝같을수 있습니다: `sed s/REGEX/SUBSTITUTION/ input.txt > input.txt`. 그렇지만, 이것은 좋은 방법이 아닙니다. 왜 일까요? 이것이 `sed`에만 해당합니까? `man sed`을 사용해서 왜 그런지, 또 이를 수행하는 방법은 무엇일지 알아보세요. 

4. 지난 10 번의 부팅시간 동안의 평균, 중앙값 및 최대 시스템 부팅 시간을 찾아보세요. 리눅스에서는 `journalctl`을 사용하고, macOS에서는 `log show`을 사용해서 각 부팅의 시작과 끝 부분에있는 로그 타임 스탬프를 확인하세요. 리눅스에서는 다음과 같은 것을 볼수 있을 것입니다. 

   ```
   Logs begin at ...
   ```
   ```
   systemd[577]: Startup finished in ...
   ```
   macOS에서는 , [다음과 같은 로그](https://eclecticlight.co/2018/03/21/macos-unified-log-3-finding-your-way/)를 찾아보세요:
   ```
   === system boot:
   ```
   ```
   Previous shutdown cause: 5
   ```

5. 지난 세번의 리부팅중에서 공유 되지 않은 부팅 메세지를 찾습니다  ( `journalctl`의 `-b` 플래그 참고). 이 작업을 여러단계로 나눠보세요. 먼저, 과거의 세번의 부팅 로그만 가져오는 방법을 찾습니다. 이것을 위해 사용하는 도구에 적용 가능한 플래그가 있을수도 있고, `sed '0,/STRING/d'`를 사용해서 `STRING`과 일치하는 행 이전의 모든 행을 제거할 수도 있습니다. 다음으로 언제나 변하는 줄의 일부 (예를 들면 타임스탬프) 를 제거합니다. 그리고, 입력 줄의 중복을 제거하고 각 줄의 개수를 셉니다 (`uniq`를 사용해 보세요). 마지막으로 개수가 3인 모든 줄을 제거합니다 (이들은 부트에서 공유되었기 때문입니다).



6. [데이터 1](https://stats.wikimedia.org/EN/TablesWikipediaZZ.htm), [데이터 2](https://ucr.fbi.gov/crime-in-the-u.s/2016/crime-in-the-u.s.-2016/topic-pages/tables/table-1) 또는 [데이터 3](https://www.springboard.com/blog/free-public-data-sets-data-science-project/) 같은 온라인 데이터를 찾으세요. `curl` 을 사용하여 데이터를 가져오고 숫자 데이터의 2개의 열만 추출합니다. HTML 데이터를 가져오는 경우 [`pup`](https://github.com/EricChiang/pup) 이 유용할수 있습니다. JSON 데이터의 경우, [`jq`](https://stedolan.github.io/jq/) 를 사용해보세요. 단일 명령으로 한 열의 최솟값과 최댓값을 찾고, 두 열 사이의 차잇값의 합을 찾아보세요. 


## 번역 
* Used Google Translation tool for translating and added personal edits for smooth translation and readability. 
* 번역을 위해 Google 번역 도구를 사용하고 원활한 번역과 가독성을 위해 개인 편집을 추가했습니다.