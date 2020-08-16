---
layout: lecture
title: "Security and Cryptography"
date: 2019-01-28
ready: true
video:
  aspect: 56.25
  id: tjwobAmnKTo
---

Last year's [security and privacy lecture](/2019/security/) focused on how you
can be more secure as a computer _user_. This year, we will focus on security
and cryptography concepts that are relevant in understanding tools covered
earlier in this class, such as the use of hash functions in Git or key
derivation functions and symmetric/asymmetric cryptosystems in SSH.

지난 해의 [security and privacy lecture](/2019/security/)는 컴퓨터 _사용자_ 측면에서 어떻게 더 안전해 질 수 있는가에 초점을 맞췄습니다. 이번에는, Git에서의 Hash 함수 혹은 키 유도 함수의 사용과 SSH 에서의 대칭/비대칭 암호 시스템과 같이 지난 수업에서 다룬 도구를 이해하는데 관련이 있는 보안과 암호학 개념에 초점을 맞출 것입니다.

This lecture is not a substitute for a more rigorous and complete course on
computer systems security ([6.858](https://css.csail.mit.edu/6.858/)) or
cryptography ([6.857](https://courses.csail.mit.edu/6.857/) and 6.875). Don't
do security work without formal training in security. Unless you're an expert,
don't [roll your own
crypto](https://www.schneier.com/blog/archives/2015/05/amateurs_produc.html).
The same principle applies to systems security.

이 강의는 컴퓨터 시스템 보안 ([6.858](https://css.csail.mit.edu/6.858/)) 혹은 암호학([6.857](https://courses.csail.mit.edu/6.857/) and 6.875)에 대한 완벽한 대체 강의가 아닙니다. 보안 업무를 하기 위해서는 보안에 관한 정규 교육을 받아야 합니다. 전문가가 아니라면, [본인 만든 암호를 사용하지 마세요](https://www.schneier.com/blog/archives/2015/05/amateurs_produc.html). 이 원칙은 시스템 보안에도 적용됩니다.


This lecture has a very informal (but we think practical) treatment of basic
cryptography concepts. This lecture won't be enough to teach you how to
_design_ secure systems or cryptographic protocols, but we hope it will be
enough to give you a general understanding of the programs and protocols you
already use.

이 강의는 기본적인 암호 개념을 비공식적(이지만 효율적)으로 다루고 있습니다. 이 강의는 독자가 어떻게 안전한 시스템이나 암호학 프로토콜을 _설계_ 할 수 있는지 가르치기에는 부족하지만, 독자가 이미 사용하는 프로그램과 프로토콜에 대해 일반적인 이해를 할 수 있도록 도움을 주고자 합니다.

# 엔트로피(Entropy)

[Entropy](https://en.wikipedia.org/wiki/Entropy_(information_theory)) is a
measure of randomness. This is useful, for example, when determining the
strength of a password.

[엔트로피](https://en.wikipedia.org/wiki/Entropy_(information_theory))는 임의성의 척도입니다. 이는 예를 들면 패스워드 강도를 결정할 때 유용합니다.

![XKCD 936: Password Strength](https://imgs.xkcd.com/comics/password_strength.png)

As the above [XKCD comic](https://xkcd.com/936/) illustrates, a password like
"correcthorsebatterystaple" is more secure than one like "Tr0ub4dor&3". But how
do you quantify something like this?

위의 [XKCD comic](https://xkcd.com/936/) 그림에서, "correcthorsebatterystaple"라는 패스워드는 "Tr0ub4dor&3"라는 패스워드보다 안전합니다. 하지만, 어떻게 이를 측정 할 수 있을까요?

Entropy is measured in _bits_, and when selecting uniformly at random from a
set of possible outcomes, the entropy is equal to `log_2(# of possibilities)`.
A fair coin flip gives 1 bit of entropy. A dice roll (of a 6-sided die) has
\~2.58 bits of entropy.

엔트로피는  _bits_ 단위로 측정되며, 가능한 결과 집합에서 무작위로 균일하게 선택 할 때, 엔트로피는 `log_2(# of possibilities)`와 동일합니다. 동전 던지기는 1 bit의 엔트로피를 제공합니다. 6면을 가진 주사위 굴리기는 ~2.58 bits의 엔트로피를 가집니다.

You should consider that the attacker knows the _model_ of the password, but
not the randomness (e.g. from [dice
rolls](https://en.wikipedia.org/wiki/Diceware)) used to select a particular
password.

공격자는 암호 _모델_ 을 알고 있지만, 특정 암호를 선택하는 데 사용되는 임의성(예 - [주사위 던지기](https://en.wikipedia.org/wiki/Diceware))은 알 수 없다는 점을 고려해야 합니다.

How many bits of entropy is enough? It depends on your threat model. For online
guessing, as the XKCD comic points out, \~40 bits of entropy is pretty good. To
be resistant to offline guessing, a stronger password would be necessary (e.g.
80 bits, or more).

몇 bits의 엔트로피가 충분할까요? 이는 당신의 위협 모델에 달렸습니다. 온라인 추측의 경우 위의 XKCD 만화에서 확인했듯이, ~40 bits의 엔트로피로 충분합니다. 오프라인 추측에 대응 할 경우 더 강력한 암호가 필요합니다. 

# Hash 함수

A [cryptographic hash
function](https://en.wikipedia.org/wiki/Cryptographic_hash_function) maps data
of arbitrary size to a fixed size, and has some special properties. A rough
specification of a hash function is as follows:

암호화를 위한 Hash 함수는 임의 크기의 데이터를 고정된 크기로 매핑하고, 몇 가지 특징을 가지고 있습니다. Hash 함수의 대략적인 명세는 아래와 같습니다.

```
hash(value: array<byte>) -> vector<byte, N>  (for some fixed N)
```

An example of a hash function is [SHA1](https://en.wikipedia.org/wiki/SHA-1),
which is used in Git. It maps arbitrary-sized inputs to 160-bit outputs (which
can be represented as 40 hexadecimal characters). We can try out the SHA1 hash
on an input using the `sha1sum` command:


Hash 함수의 한 예는 Git에서 사용하고 있는 [SHA1](https://en.wikipedia.org/wiki/SHA-1) 입니다. 이는 임의 크기의 입력값을 160 bit의 결과(40개의 16진수 문자로 표현할 수 있음)로 매핑합니다. SHA1 Hash는 `sha1sum` 명령어를 통해 사용 해 볼 수 있습니다.

```console
$ printf 'hello' | sha1sum
aaf4c61ddcc5e8a2dabede0f3b482cd9aea9434d
$ printf 'hello' | sha1sum
aaf4c61ddcc5e8a2dabede0f3b482cd9aea9434d
$ printf 'Hello' | sha1sum 
f7ff9e8b7bb2e09b70935a5d785e0cc5d9d0abf0
```

At a high level, a hash function can be thought of as a hard-to-invert
random-looking (but deterministic) function (and this is the [ideal model of a
hash function](https://en.wikipedia.org/wiki/Random_oracle)). A hash function
has the following properties:

높은 수준에서, Hash 함수는 도치하기 어렵고 무작위로 보이(지만 결정적인)는 함수로 보일 수 있으며, 이는 [Hash 함수의 이상적 모델](https://en.wikipedia.org/wiki/Random_oracle)입니다. Hash 함수는 아래의 특징을 가지고 있습니다.

- Deterministic: the same input always generates the same output.
- Non-invertible: it is hard to find an input `m` such that `hash(m) = h` for some desired output `h`.
- Target collision resistant: given an input `m_1`, it's hard to find a
different input `m_2` such that `hash(m_1) = hash(m_2)`.
- Collision resistant: it's hard to find two inputs `m_1` and `m_2` such that `hash(m_1) = hash(m_2)` (note that this is a strictly stronger property than target collision resistance).

- 결정적(Deterministic) : 같은 입력 값은 항상 같은 결과가 도출
- 복구 불가능(Non-invertible) : 원하는 출력값 `h`에 대하여 `hash(m) = h`를 만족하는 `m`을 찾기 어려움
- 약한 충돌 저항성(Target collision resistant) : `m_1`이 주어졌을 때, `hash(m_1) = hash(m_2)`를 만족하는 `m_2`를 찾기 어려움
- 강한 충돌 저항성(Collision resistant) : `hash(m_1) = hash(m_2)`를 만족하는 `m_1`과 `m_2`를 찾기 어려움(약한 충돌 저항성보다 강한 속성)

Note: while it may work for certain purposes, SHA-1 is [no
longer](https://shattered.io/) considered a strong cryptographic hash function.
You might find this table of [lifetimes of cryptographic hash
functions](https://valerieaurora.org/hash.html) interesting. However, note that recommending specific hash functions is beyond the scope of this lecture. If you are doing work where this matters, you need formal training in
security/cryptography.

참고 : 특정 목적으로 사용할 수도 있지만, SHA-1은 [더 이상](https://shattered.io/) 안전한 암호화를 위한 Hash 함수로 취급하지 않습니다. [lifetimes of cryptographic hash functions](https://valerieaurora.org/hash.html)가 아마 흥미로울 것 입니다. 그러나, 특정 Hash 함수를 추천하는 것은 이 강의의 범위를 벗어납니다. 만약 이 글을 읽는 독자가 이러한 문제를 가지고 있다면, 보안/암호학에 대한 정규 교육이 필요합니다.

## Applications

## 어플리케이션(Applications)

- Git, for content-addressed storage. The idea of a [hash
function](https://en.wikipedia.org/wiki/Hash_function) is a more general
concept (there are non-cryptographic hash functions). Why does Git use a
cryptographic hash function?
- A short summary of the contents of a file. Software can often be downloaded from (potentially less trustworthy) mirrors, e.g. Linux ISOs, and it would be nice to not have to trust them. The official sites usually post hashes alongside the download links (that point to third-party mirrors), so that the
hash can be checked after downloading a file.
- [Commitment schemes](https://en.wikipedia.org/wiki/Commitment_scheme).
Suppose you want to commit to a particular value, but reveal the value itself later. For example, I want to do a fair coin toss "in my head", without a trusted shared coin that two parties can see. I could choose a value `r = random()`, and then share `h = sha256(r)`. Then, you could call heads or tails (we'll agree that even `r` means heads, and odd `r` means tails). After you call, I can reveal my value `r`, and you can confirm that I haven't cheated by checking `sha256(r)` matches the hash I shared earlier.

- Git, 컨텐츠 주소 지정 스토리지용. [hash
함수](https://en.wikipedia.org/wiki/Hash_function)의 개념은 더 일반적입니다.(암호화의 목적이 아닌 Hash 함수가 있음). 왜 Git이 암호화 Hash 함수를 사용할까요?
- 파일 내용을 짧게 요약. 사용자들은 소프트웨어를 (덜 신뢰할 수 있는) 미러 사이트에서 다운받곤 하는데(예 - Linux ISO), 이는 신뢰하지 않는 것이 좋습니다. 공식 사이트에서는 보통 타사의 미러 페이지로 이동하는 다운로드 링크와 함께 파일의 Hash 값을 게시 합니다. 이를 통해 사용자는 파일을 다운로드 받은 후 다운로드 받은 파일의 Hash를 확인할 수 있습니다.
- [Commitment schemes](https://en.wikipedia.org/wiki/Commitment_scheme). 특정 값을 커밋하고 싶지만, 나중에 공개하고 싶다고 가정합니다. 예를 들어, 나는 두 당사자가 볼 수 있고 신뢰할 수 있는 공유 동전 없이 "내 머리속에" 공정한 동전 던지기를 하고 싶습니다. 나는 `r = random()`인 값을 선택한 다음 `h = sha256(r)`을 공유할 수 있습니다. 그 후, 앞면 혹은 뒷면을 외칠 수 있습니다(`r`이 짝수 일 경우는 앞면이며, 홀수일 경우에는 뒷면을 의미한다고 약속합니다.). 당신이 외친 후, 내 값인 `r`을 공개할 수 있고, `sha256(r)`이 이전에 공유 한 Hash와 일치하는지 확인하여 내가 속이지 않았음을 확인할 수 있습니다.


# Key derivation functions
# 키 유도 함수(Key derivation functions)

A related concept to cryptographic hashes, [key derivation
functions](https://en.wikipedia.org/wiki/Key_derivation_function) (KDFs) are
used for a number of applications, including producing fixed-length output for use as keys in other cryptographic algorithms. Usually, KDFs are deliberately slow, in order to slow down offline brute-force attacks.

암호화 Hash와 관련된 개념인 [key derivation
functions](https://en.wikipedia.org/wiki/Key_derivation_function) (KDFs)는 다른 암호 알고리즘에서 키로 사용하기 위한 고정 길이 출력값을 생성하는 것도 포함하여, 여러 응용 프로그램에서 사용됩니다. 일반적으로, KDF는 오프라인 무차별 대입 공격의 속도를 늦추기 위해 의도적으로 느립니다.

## Applications
## 어플리케이션(Applications)

- Producing keys from passphrases for use in other cryptographic algorithms
(e.g. symmetric cryptography, see below).
- Storing login credentials. Storing plaintext passwords is bad; the right
approach is to generate and store a random
[salt](https://en.wikipedia.org/wiki/Salt_(cryptography)) `salt = random()` for
each user, store `KDF(password + salt)`, and verify login attempts by
re-computing the KDF given the entered password and the stored salt.

- 다른 암호화 알고리즘에서 사용하기 위한 암호구문 키 생성 (예 - 대칭키 암호, 아래 참조)
- 로그인 자격증명 저장. 패스워드 평문 저장은 적절하지 않습니다. 올바른 접근 법은 각 사용자 별로 `salt = random()`인 랜덤한 [salt](https://en.wikipedia.org/wiki/Salt_(cryptography))를 생성하고, `KDF(password + salt)`를 저장합니다. 그 후 입력 된 패스워드와 저장된 salt로 제공된 KDF를 다시 계산하는 방법으로 로그인 시도를 검증합니다.

# Symmetric cryptography
# 대칭키 암호(Symmetric cryptography)

Hiding message contents is probably the first concept you think about when you
think about cryptography. Symmetric cryptography accomplishes this with the following set of functionality:

메세지를 숨긴다는 개념은 아마 독자가 암호학에 대해 생각했을 때 가장 먼저 떠오르는 개념일 것입니다. 대칭키 암호는 다음 기능들을 사용하여 이를 수행합니다.

```
keygen() -> key  (this function is randomized)

encrypt(plaintext: array<byte>, key) -> array<byte>  (the ciphertext)
decrypt(ciphertext: array<byte>, key) -> array<byte>  (the plaintext)
```

The encrypt function has the property that given the output (ciphertext), it's hard to determine the input (plaintext) without the key. The decrypt function has the obvious correctness property, that `decrypt(encrypt(m, k), k) = m`.

암호화 함수는 출력(암호문)을 제공하는 기능을 가지며, 키 없이 입력값(평문)을 알아내기 어렵습니다. 복호화 함수는 `decrypt(encrypt(m, k), k) = m`라는 명확한 정확성 속성을 가지고 있습니다.

An example of a symmetric cryptosystem in wide use today is
[AES](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard).

오늘날 널리 쓰이는 대칭키 암호의 한 예는 [AES](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard) 입니다.

## Applications
## 어플리케이션(Applications)

- Encrypting files for storage in an untrusted cloud service. This can be
combined with KDFs, so you can encrypt a file with a passphrase. Generate `key = KDF(passphrase)`, and then store `encrypt(file, key)`.

- 신뢰할 수 없는 클라우드 서비스에 파일을 저장할 때 파일을 암호화. 이는 KDF와 결합되어 암호로 파일을 암호화 할 수 있습니다. `key = KDF(passphrase)`로 키를 생성하고, `encrypt(file, key)`로 암호화 한 파일을 저장합니다.

# Asymmetric cryptography
# 비대칭 암호화

The term "asymmetric" refers to there being two keys, with two different roles.
"비대칭"이라는 용어는 두개의 키와 두개의 다른 역할이 있는 키를 의미합니다.

A private key, as its name implies, is meant to be kept private, while the
public key can be publicly shared and it won't affect security (unlike sharing
the key in a symmetric cryptosystem).
이름에서 알 수 있듯이 개인 키는 비공개로 유지되는 반면 공개 키는 공개적으로 공유될 수 있으며 보안에 영향을 미치지 않습니다(대칭 암호 시스템에서 키를 공유하는 것과 다르다).
 Asymmetric cryptosystems provide the
following set of functionality, to encrypt/decrypt and to sign/verify:
 비대칭 암호화 시스템은 암호화/암호화 및 서명/확인을 위한 다음과 같은 기능 집합을 제공합니다.

```
keygen() -> (public key, private key)  (this function is randomized)
keygen() -> (공개 키, 개인 키) (이 기능은 임의로 지정됨)

encrypt(plaintext: array<byte>, public key) -> array<byte>  (the ciphertext)
encrypt(일반텍스트:배열<byte>,공개 키) -> 배열<byte> (암호문)
decrypt(ciphertext: array<byte>, private key) -> array<byte>  (the plaintext)
decrypt(암호문:배열<byte>, 개인 키) -> 배열<byte> (일반텍스트)


sign(message: array<byte>, private key) -> array<byte>  (the signature)
서명(message: 배열<byte>, 개인 키) -> 배열<byte> (서명)
verify(message: array<byte>, signature: array<byte>, public key) -> bool  (whether or not the signature is valid)
검증(message: 배열<byte>, 서명: 배열<byte>, 공개 키) -> bool (서명이 유효한지 여부)
```

The encrypt/decrypt functions have properties similar to their analogs from
symmetric cryptosystems.
encrypt/decrypt 함수에는 대칭형 암호 시스템의 분석과 유사한 속성이 있습니다.
 A message can be encrypted using the _public_ key.
 메시지는 _공용_ 키를 사용하여 암호화할 수 있습니다.
Given the output (ciphertext), it's hard to determine the input (plaintext)
without the _private_ key.
(출력)을 고려하면, _개인_ 키 없이 입력(일반 텍스트)을 결정하기는 어렵습니다.
 The decrypt function has the obvious correctness
property, that `decrypt(encrypt(m, public key), private key) = m`.
해독 기능은 명백한 정확성 속성을 갖고 있습니다. 'decrypt(encrypt(m, 공개 키), 개인 키) = m',

Symmetric and asymmetric encryption can be compared to physical locks.
대칭과 비대칭 암호화를 물리적 잠금과 비교할 수 있습니다.
 A symmetric cryptosystem is like a door lock: anyone with the key can lock and
unlock it. 
 대칭 암호 시스템은 도어 잠금 장치와 같다: 키가 있는 사람은 누구나 잠그고 해제할 수 있습니다.
Asymmetric encryption is like a padlock with a key. 
 비대칭 암호화는 열쇠가 달린 자물쇠와 같다.
You could give
the unlocked lock to someone (the public key), they could put a message in a
box and then put the lock on, and after that, only you could open the lock
because you kept the key (the private key).
 잠금 해제된 잠금을 누군가(공용 키)에게 줄 수 있고, 메시지를 상자에 넣어 다음 잠금을 설정할 수 있다. 그리고 나서, 잠금 키(개인 키)를 보관하였기 때문에 잠금을 열 수 있습니다.

The sign/verify functions have the same properties that you would hope physical
signatures would have, in that it's hard to forge a signature. 
서명/확인 기능은 서명을 위조하기 어렵다는 것에서 물리적 서명이 갖고 있기를 바라는 것과 같은 속성을 가지고 있습니다.
No matter the
message, without the _private_ key, it's hard to produce a signature such that
`verify(message, signature, public key)` returns true. And of course, the
verify function has the obvious correctness property that `verify(message,
sign(message, private key), public key) = true`.
메시지든, _개인_ 열쇠 없이, 그런 서명을 하기는 어렵습니다. 'verify(message, signature, public key)사실로 돌아오다. 그리고 물론, 그 검증 기능은 다음과 같은 명백한 정확성을 갖고있습니다. 'verify(message, sign(message, 개인 키), 공용 키) = 사실'.

## Applications
## 응용프로그램

- [PGP email encryption](https://en.wikipedia.org/wiki/Pretty_Good_Privacy).
People can have their public keys posted online (e.g. in a PGP keyserver, or on
[Keybase](https://keybase.io/)). Anyone can send them encrypted email.
- [PGP email 암호화](https://en.wikipedia.org/wiki/Pretty_Good_Privacy). 사람들은 자신의 공개키를 온라인상에 올릴 수 있습니다. (PGP Keyserver나 [Keybase](https://keybase.io/) 등에). 그러면 누구나 그 사람에게 암호화된 메일을 보낼 수 있습니다.
- Private messaging. Apps like [Signal](https://signal.org/) and
[Keybase](https://keybase.io/) use asymmetric keys to establish private
communication channels.
- 개인 메시지. [Signal](https://signal.org/)그리고[Keybase](https://keybase.io/)같은 앱은 비대칭 키를 사용해 개인통신 채널을 설정합니다.
- Signing software. Git can have GPG-signed commits and tags. With a posted
public key, anyone can verify the authenticity of downloaded software.
- Signing software. Git에는 GPG-signed의 commits과 tags가 있을 수 있습니다. 게시된 공용 키를 사용하면, 누구나 다운로드된 소프트웨어의 신뢰성을 확인할 수 있습니다. 

## Key distribution
## 키 분포

Asymmetric-key cryptography is wonderful, but it has a big challenge of
distributing public keys / mapping public keys to real-world identities. There
are many solutions to this problem.
비대칭 키 암호화는 훌륭하지만, 공용 키를 배포하고 실제 ID에 공개키를 mapping하는 데는 큰 어려움이 있습니다. 이 문제에 대한 많은 해결책이 있습니다.
 Signal has one simple solution: trust on
first use, and support out-of-band public key exchange (you verify your
friends' "safety numbers" in person). 
 신호에는 한가지 간단한 해결책이 있습니다: 처음 사용할 때 신뢰할 수 있고, out-of-band공용 키 교환을 지지합니다.(당신의 당신의 "안전 번호"를 직접 확인)
PGP has a different solution, which is
[web of trust](https://en.wikipedia.org/wiki/Web_of_trust). 
PGP에는 신뢰할 수 있는 다른 해결방법이 있습니다. [web of trust](https://en.wikipedia.org/wiki/Web_of_trust)
Keybase has yet
another solution of [social
proof](https://keybase.io/blog/chat-apps-softer-than-tofu) (along with other
neat ideas). Each model has its merits; we (the instructors) like Keybase's
model.
Keybase는 다른 해결책을 갖고 있습니다.[사회적증거](https://keybase.io/blog/chat-apps-softer-than-tofu) (다른 생각들과 함께). 각 모델에는 장점이 있다; 우리(강사들)은 Keybase 모델들을 좋아합니다.

# Case studies
# 사례 연구

## Password managers
## 암호 관리자

This is an essential tool that everyone should try to use (e.g.
[KeePassXC](https://keepassxc.org/)).
이 도구는 모든 사용자가 사용해야하는 필수적인 도구입니다.(예:[KeePassXC](https://keepassxc.org/)).
 Password managers let you use unique,
randomly generated high-entropy passwords for all your websites, and they save
all your passwords in one place, encrypted with a symmetric cipher with a key
produced from a passphrase using a KDF.
암호 관리자를 사용하게 되면 임의로 만든 고유한 high-entropy 암호를 모든 웹 사이트에 사용할 수 있고, KDF를 사용해 생성된 키인 대칭 암호로 암호화된 모든 암호들을 한 곳에 저장할 수 있습니다.

Using a password manager lets you avoid password reuse (so you're less impacted
when websites get compromised), use high-entropy passwords (so you're less likely to
get compromised), and only need to remember a single high-entropy password.
암호 관리자를 사용하면 암호 재사용을 피할 수 있어 (그래서 당신은 웹사이트가 손상될 때 덜 영향을 받을 수 있으므로), 높은 엔트로피 암호를 사용할 수 있고 (따라서 손상될 가능성이 낮다), 높은 수준의 암호 하나만 기억하면 됩니다. 

## Two-factor authentication
## 2단계 인증

[Two-factor
authentication](https://en.wikipedia.org/wiki/Multi-factor_authentication)
(2FA) requires you to use a passphrase ("something you know") along with a 2FA
authenticator (like a [YubiKey](https://www.yubico.com/), "something you have")
in order to protect against stolen passwords and
[phishing](https://en.wikipedia.org/wiki/Phishing) attacks.
[2단계 인증](https://en.wikipedia.org/wiki/Multi-factor_authentication)(2FA)에서는 도난된 암호와 [phishing](https://en.wikipedia.org/wiki/Phishing)공격으로부터 보호하기위해 2FA 인증자([YuviKey](https://www.yubico.com/), "보유한 것"등)와 함께 암호("알고있는 것")를 사용해야 합니다.

## Full disk encryption
## 전체 디스크 암호화

Keeping your laptop's entire disk encrypted is an easy way to protect your data
in the case that your laptop is stolen. 
랩톱의 전체 디스크를 암호화된 상태로 유지하면 랩톱을 도난당한 경우 데이터를 쉽게 보관할 수 있습니다. 
You can use [cryptsetup +
LUKS](https://wiki.archlinux.org/index.php/Dm-crypt/Encrypting_a_non-root_file_system)
on Linux,
[BitLocker](https://fossbytes.com/enable-full-disk-encryption-windows-10/) on
Windows, or [FileVault](https://support.apple.com/en-us/HT204837) on macOS.
Linux에서 [암호설정 + LUKS](https://wiki.archlinux.org/index.php/Dm-crypt/Encrypting_a_non-root_file_system)을 사용하거나 Windows에서 [BitLocker](https://fossbytes.com/enable-full-disk-encryption-windows-10/)을 사용하거나 masOS에서 [FileVault](https://support.apple.com/en-us/HT204837)를 사용할 수 있습니다.
This encrypts the entire disk with a symmetric cipher, with a key protected by
a passphrase.
암호로 보호되는 키를 사용하여, 전체 디스크가 대칭 암호화가 됩니다.

## Private messaging
## 개인 메시지

Use [Signal](https://signal.org/) or [Keybase](https://keybase.io/). 
[신호](https://signal.org/) 또는 [Keybase](https://keybase.io/)를 사용합니다.
End-to-end
security is bootstrapped from asymmetric-key encryption. 
End-to-end의 보안은 비대칭 키 암호화에서 차단됩니다.
Obtaining your
contacts' public keys is the critical step here.
연락처의 공용 키를 얻는 것이 중요한 단계입니다.
 If you want good security, you
need to authenticate public keys out-of-band (with Signal or Keybase), or trust
social proofs (with Keybase).
안전한 보안을 원한다면 Out-of-band(신호 또는 keybase사용)로 공개 키를 인증하거나, 사회적 증명(keybase 사용)을 신뢰해야 합니다. 

## SSH
## 시큐어 셸

We've covered the use of SSH and SSH keys in an [earlier
lecture](/2020/command-line/#remote-machines). 
전 강의에서 SSH와 SSH키 사용에 대해 다루었습니다. [earlier lecture](/2020/command-line/#remote-machines)
Let's look at the cryptography
aspects of this.
이것의 암호적 측면을 살펴봅시다.

When you run `ssh-keygen`, it generates an asymmetric keypair, `public_key,
private_key`. This is generated randomly, using entropy provided by the
operating system (collected from hardware events, etc.).
당신이 달릴떄 'shh-keygen'비대칭 키패드를 생성하고, 'public_key,private_key'. 이것은 운영체제에 의해 제공된 엔트로피를 이용해 무작위로 생성됩니다(하드웨어 이벤트 등에서 수집).
 The public key is
stored as-is (it's public, so keeping it a secret is not important), but at
rest, the private key should be encrypted on disk. 
공용 키는 그대로 저장되며(공용 키 이므로 비밀 유지하는 것은 중요하지 않다), 개인 키는 디스크에 암호화되어야 합니다.
The `ssh-keygen` program
prompts the user for a passphrase, and this is fed through a key derivation
function to produce a key, which is then used to encrypt the private key with a
symmetric cipher.
'ssh-keygen'프로그램은 사용자에게 암호를 묻는 표시를 하고, 키 파생 기능으로 키를 생성하며, 이 기능은 개인키를 대칭 암호로 암호화하는 데 사용됩니다. 

In use, once the server knows the client's public key (stored in the
`.ssh/authorized_keys` file), a connecting client can prove its identity using
asymmetric signatures. 
사용중인 경우, 서버가 클라이언트의 공용 키(`.ssh/authorized_keys`에 저장된 파일)를 알게 되면, 연결 클라이언트는 비대칭 서명을 사용해 신원을 증명할 수 있습니다.
This is done through
[challenge-response](https://en.wikipedia.org/wiki/Challenge%E2%80%93response_authentication).
At a high level, the server picks a random number and sends it to the client.
이것은 [도전-반응](https://en.wikipedia.org/wiki/Challenge%E2%80%93response_authentication)을 통해 이루어집니다.
The client then signs this message and sends the signature back to the server,
which checks the signature against the public key on record. 
그런 다음 고객이 이 메시지에 서명하고 서명을 서버로 다시 보내면, 서명을 공용 키와 비교해 기록합니다.
This effectively
proves that the client is in possession of the private key corresponding to the
public key that's in the server's `.ssh/authorized_keys` file, so the server
can allow the client to log in.
이는 고객이 서버의 공용 키에 해당하는 개인 키를 소유하고 있음을 효과적으로 입증하며 `.ssh/authorized_keys`파일이 고객의 로그인을 허용할 수 있도록 합니다. 

{% comment %}
extra topics, if there's time
{% comment %}
시간이 있다면 추가의 주제
;
security concepts, tips
- biometrics
- HTTPS
{% endcomment %}
보안 개념, tips
-생체 인식 정보
-HTTPS
{% endcomment %}

# Resources
# 자원

- [Last year's notes](/2019/security/): from when this lecture was more focused on security and privacy as a computer user
- [작년참고사항](/2019/security/): 이 강의가 컴퓨터 사용자로써 보안과 사생활에 더 초점이 맞춰졌을 때부터
- [Cryptographic Right Answers](https://latacora.micro.blog/2018/04/03/cryptographic-right-answers.html): answers "what crypto should I use for X?" for many common X.
- [암호화권한답변](https://latacora.micro.blog/2018/04/03/cryptographic-right-answers.html): "x에 사용해야 하는 암호?"에 대한 답은 많은 일반적인 X이다.

# Exercises
# 운동들

1. **Entropy.**
1. **엔트로피.**
    1. Suppose a password is chosen as a concatenation of five lower-case
       dictionary words, where each word is selected uniformly at random from a
       dictionary of size 100,000.
       암호가 5개의 소문자 사전 단어를 합한 것으로 선택되었다고 가정해보고, 각 단어는 100,000개의 크기 사전에서 무작위로 고른다.
        An example of such a password is
       `correcthorsebatterystaple`. How many bits of entropy does this have?
       이러한 암호 예는 다음과 같다.`correcthorsebatterystaple`이 엔트로피는 몇 비트인가요?
    1. Consider an alternative scheme where a password is chosen as a sequence
       of 8 random alphanumeric characters (including both lower-case and
       upper-case letters). An example is `rg8Ql34g`. How many bits of entropy
       does this have?
       암호가 8개의 임의 영숫자(소문자, 대문자 모두 포함)로 선택되는 대체 체계를 고려해보자. 예를들면 `rg8Ql34g`엔트로피는 몇 비트인가요?
    1. Which is the stronger password?
    어느쪽이 더 강한 암호인가요?
    1. Suppose an attacker can try guessing 10,000 passwords per second. On
       average, how long will it take to break each of the passwords?
       공격하는 자가 초당 10,000개의 암호를 추측할 수 있다고 가정합니다. 각 암호들을 해독하는데 평균적으로 얼마나 걸립니까?
1. **Cryptographic hash functions.** Download a Debian image from a
   [mirror](https://www.debian.org/CD/http-ftp/) (e.g. [from this Argentinean
   mirror](http://debian.xfree.com.ar/debian-cd/current/amd64/iso-cd/).
   **암호화 해시 기능.** Debian이미지를 다운로드합니다. [거울](https://www.debian.org/CD/http-ftp/) (예: [이 아르헨티나의 거울에서](http://debian.xfree.com.ar/debian-cd/current/amd64/iso-cd/). 
   Cross-check the hash (e.g. using the `sha256sum` command) with the hash
   retrieved from the official Debian site (e.g. [this
   file](https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/SHA256SUMS)
   hosted at `debian.org`, if you've downloaded the linked file from the
   Argentinean mirror).
   해시를 교차 확인합니다. (예:`sha256sum`명령)사용하여 공식 Debian사이트(예[이 파일](https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/SHA256SUMS)은 `debian.org` 아르헨티나 미러에서 링크된 파일을 다운로드 한 경우 ) 클릭하세요.
1. **Symmetric cryptography.** 
1. **대칭 암호** 
Encrypt a file with AES encryption, using
   [OpenSSL](https://www.openssl.org/): `openssl aes-256-cbc -salt -in {input
   filename} -out {output filename}`.
   [OpenSSL](https://www.openssl.org/)를 사용하여 AES암호화로 파일 암호화: openssl aes-256-cbc -salt - in {입력 파일명}-out {출력 파일명}
    Look at the contents using `cat` or
   `hexdump`. Decrypt it with `openssl aes-256-cbc -d -in {input filename} -out
   {output filename}` and confirm that the contents match the original using
   `cmp`.
   `cat` 또는 `hexdump`를 사용해 내용을 보십시오. 암호를 해독{입력 파일명} -out{출력 파일명} 내용이 원본과 일치하는지 확인 `cmp`
1. **Asymmetric cryptography.**
1. **비대칭 암호**
    1. Set up [SSH
       keys](https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys--2)
       on a computer you have access to (not Athena, because Kerberos interacts
       weirdly with SSH keys).
       [SSH키](https://www.digitalocean.com/community/tutorials how-to-set-up-ssh-keys--2) (Athena가 아닌, Kerberos가 SSH키와 이상한 상호작용을 하기 때문에) 컴퓨터에 엑세스 할 수 있는 시스템에 설정합니다.
        Rather than using RSA keys as in the linked
       tutorial, use more secure [ED25519
       keys](https://wiki.archlinux.org/index.php/SSH_keys#Ed25519).
        링크된 튜토디얼처럼 RSA키를 사용하는 것이 아닌 더 안전한 [ED25519키](https://wiki.archlinux.org/index.php/SSH_keys#Ed25519)를 사용.
        Make sure
       your private key is encrypted with a passphrase, so it is protected at
       rest.
       개인 키가 비밀번호로 암호화 되었는지 확인하여 저장시 보호되도록 합니다.
    1. [Set up GPG](https://www.digitalocean.com/community/tutorials/how-to-use-gpg-to-encrypt-and-sign-messages)
    [GPG설정](https://www.digitalocean.com/community/tutorials/how-to-use-gpg-to-encrypt-and-sign-messages)
    1. Send Anish an encrypted email ([public key](https://keybase.io/anish)).
    Anish에게 암호화된 전자 메일([공개 키](https://keybase.io/anish))을 보냅니다.
    1. Sign a Git commit with `git commit -S` or create a signed Git tag with
       `git tag -s`. Verify the signature on the commit with `git show
       --show-signature` or on the tag with `git tag -v`.
       Git과 서명. `git commit -S`또는 서명된 Git 태그를 만들기 위해 `git tag -S`. 다음을 사용하여 커밋의 서명을 확인합니다. `git show--show-signature`또는  `git tag-V`와 함꼐 태그에 표시할 수 있습니다. 
