---
layout: post
title: "[WEB] 암호화 - 대칭키와 공개키, 그리고 해싱"
subtitle: "Symmetric, Asymmetric key & Hashing"
background: '/img/bg_technology.jpg'
categories: technology-web
---

비전공자다 보니 **보안 및 암호화** 지식이 거의 전무했다.

그러나 웹 프로젝트를 하면서, 

​	*"로그인 정보를 이런식으로 서버에 보내도 괜찮나? "*

​	*"세션ID가 이렇게 쿠키에 노출이 되어도 되나?"*

등등 자연스럽게 여러가지 의문이 들었다. 

그리고 보안 및 암호화에 대한 개념정립과 기초 지식이 필요하다는 것을 느꼈다.

해서 지금부터 차근차근 기본적인 웹 보안 지식에 대해 학습해보고자 한다.

---

#### 암호화

암호화란, 어떤 **평문(Plaintext)**을 특정 **암호화 알고리즘**을 통해 **암호문(Ciphertext)**로 만들어, 

제3자(해커)가 알아볼 수 없도록 감추는 것을 의미한다.

암호화 알고리즘에는 그 알고리즘을 푸는 열쇠, 즉 **키(key)**가 필요하다.

키가 없으면 암호문을 받아보는 사람이 암호문을 해석할 수 없을 것이다.

즉 **암호문을 보내는쪽과 받는 쪽 둘 다 해석하는 키가 필요하며,**

**이 키의 종류에 따라 암호화를 분류할 수 있다.**

<br/>

#### 1. 대칭키 방식 (Symmetric-Key)

![image](https://www.101computing.net/wp/wp-content/uploads/symmetric-encryption.png)

(출처 : https://www.101computing.net/symmetric-vs-asymmetric-encryption/)

암호화 키와 복호화 키가 **같음**을 의미하며, 즉 **하나의 키**를 이용한다.

(양쪽의 키가 같으므로 **Symmetric**)

양쪽이 동일한 키를 갖고있기 때문에 암호화-복호화의 속도가 빠르다는 장점이 있다.

그러나 이 방식에는 치명적인 문제가 있는데...

그것은 바로, **어떻게든 대칭키를 상대방에게 '배달'해야 한다는 점이다.**

네트워크 상에서 키를 상대방에게 넘긴다는 것은 통신이 발생한다는 의미이며,

중간에 해커의 공격으로 대칭키를 탈취당할 수 있다는 위험이 항상 도사리고 있다.

이를 해결하기 위해 골머리를 앓다가 등장한 방식이 바로..

<br/>

#### 2. 비대칭키 - 공개키 방식 (Asymmetric-Key / Public Key)

![image](https://www.cheapsslshop.com/blog/wp-content/uploads/2019/09/asymmetric-encryption.png)

(출처 : https://www.cheapsslshop.com/blog/symmetric-vs-asymmetric-encryption-whats-the-difference/ )

대칭키 방식이 하나의 키를 사용하는 것과 달리, 

비대칭키는 **두 개의 키를 사용**한다. 

다만 두 키의 속성과 역할이 다른데, **하나는 공개적으로 공유될 수 있으나(공개 키)**, 

**다른 하나는 반드시 자기자신만이 갖고있어야만 한다(비공개 키)**.

자, 이 비대칭키 메커니즘은 단순하다. 바로,

**암호화는 공개키로** / **복호화는 개인키로** 하는 것이 비대칭키 메커니즘이다.

<br/>

클라이언트와 서버를 예로 들어보자.

서버는 개인키와 공개키 2개를 갖고 있으며, 공개키만 클라이언트로 보낸다.

클라이언트는 서버에 보낼 개인정보를 **공개키로 암호화**하여 전송한다.

서버는 암호문을 받고, **서버가 가진 개인키로 복호화**하여 해독한다.

이 과정에서 암호문이 중간에 해커에게 탈취되더라도, **해커는 개인키가 없으므로 복호화 할 수 없다**.

<br/>

이렇게 완벽해 보이는 비대칭키 방식에는 **느리다**는 단점이 존재한다

(일반적으로 대칭키보다 1000배 느리다고 한다..).

느린 이유는 비대칭키가 사용하는 알고리즘의 복잡도가 워낙 강력해서..라고 한다.

그렇다면 아무리 보안이 뛰어난 비대칭키 방식이라도, 

비대칭키 방식만 고수하면 그로 인한 오버헤드가 더 많이 발생하지 않을까?

대칭키와 비대칭키 방식을 혼합하여 사용하면 더 나은 결과를 만들어낼 수 있지 않을까?

<br/>

#### 3. 대칭키 방식과 비대칭키 방식의 mixture

대칭키 방식의 치명적인 단점은 **키의 배달 문제**이다.

그런데, **공개키 방식을 활용하여 키를 안전하게 전달할 수 있다면?**

방법은 아래와 같다.

 1) 서버는 개인키를 갖고, 공개키를 클라이언트에게 전송한다.

 2) 클라이언트는 해당 공개키로 **대칭키를 암호화**한다.

 3) 클라이언트는 암호화된 대칭키를 서버에 전송한다.

 4) 서버는 **개인키로 암호화된 대칭키를 복호화한다**(대칭키를 얻는다).

 5) 그 이후 통신에서는 **서버와 클라이언트가 모두 대칭키로 통신한다**.

전송하려는 대칭키는 이미 공개키로 암호화되어있고, 

개인키를 가진 서버가 아니면 복호화할 수 없으므로, **대칭키를 안전하게 배달할 수 있다!**

이러한 방식은 비대칭키의 속도 문제와 대칭키의 보안 문제를 적절히 해결할 수 있는 방법으로,

다음 포스팅에서 살펴볼 **SSL 통신**의 기본 전략이다.

<br/>

#### 4. Hashing 암호화 - 단방향

위에서 살펴본 대칭키, 비대칭키는 복호화가 가능한 **양방향**암호화다.

그러나 암호는 **반드시 복호화될 필요는 없다.**

웹사이트 로그인 비밀번호의 경우를 예로 들어보자.

사용자의 비밀번호를 암호화하여 그 암호화된 결과를 DB에 저장하고,

사용자가 로그인 시도할 때 입력한 비밀번호의 암호화 결과와 DB에 저장된 암호화 결과를 비교한다면?

우리는 사용자의 원본 암호가 무엇인지 알 필요없이, 

시도된 로그인이 유효한지에 대해 검증이 가능하다.

이 방법대로라면 만약 DB 보안이 뚫리더라도 결국 원본 비밀번호는 알 수 없다는 장점이 있다.

이처럼 복호화가 불가능한 암호화 방식을 **단방향 암호화**라고 하며, 주로 Hash 알고리즘이 사용된다.

이러한 알고리즘에는 SHA-256, SHA-512 등이 존재하며,

MD5 등의 알고리즘은 현재 보안이 뚫려 사용하면 안되는 알고리즘이라고 한다.



