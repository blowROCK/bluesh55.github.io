---
layout: post
title: "안드로이드 스튜디오 키스토어 비밀번호를 찾는 방법"
date: 2016-09-24
tags: [dev]
---

안드로이드 앱을 구글 플레이에 올릴 때 키 스토어로 서명을 한다.
이 때 몇 가지 비밀번호를 하는데, 이 비밀번호를 잊어버렸을 땐 어떻게 해야할까?
키 스토어를 재발급 받아서 서명할 수는 있지만, 이미 구글플레이에 앱이 한 번 올라간 상태라면 동일한 앱으로 취급하지 않기 때문에
기존에 업로드된 앱이 아닌 새로운 앱으로 올려야 한다.

그렇다고 비밀번호 찾기 기능을 제공하는 것도 아니다.
사실 이 비밀번호를 잊어버렸다면 다시 되찾기는 거의 불가능에 가깝다.

그런데 당신이 정말 운이 좋다면 비밀번호를 찾을 수 있는 한 가지 방법이 있다.
바로 안드로이드 스튜디오의 로그 파일을 뒤져보는 것이다.

## 우리를 구원해 줄 로그 파일(맥 기준)

안드로이드 스튜디오는 항상 로그를 남긴다.
당신이 키 스토어 비밀번호를 생성했을 때도 로그가 생성되었을 것이다.
그리고 그 로그에는 우리의 비밀번호가 고스란히 남아있다. 평소엔 있는지도 몰랐던 이 파일을 찾아보자.

먼저 ~/Library/Logs/AndroidStudio2.0/ 폴더에 들어간다. 꼭 2.0일 필요는 없고 설치된 안드로이드 스튜디오 버전에 맞춰 들어가자.  
폴더에 들어가면 여러개의 idea.log 파일이 존재한다. 이제 이 파일들을 하나 하나 열어서 ```.password=```를 검색해본다.

운이 좋다면 다음과 같은 로그를 찾을 수 있다.

```
-Pandroid.injected.signing.store.password=mystorepassword, 
-Pandroid.injected.signing.key.alias=myandroidkey, 
-Pandroid.injected.signing.key.password=mykeypassword,
```
```.password=``` 뿐만 아니라 다양한 조합으로 검색해보길 바란다.

---

## Master Password를 찾는 방법은 없나요?

얘는 어딘가에 저장되는 값이 아니라서 위 방법처럼 쉽게 찾지는 못 할듯하다.

---

## 비밀번호를 맞게 입력한 것 같은데 자꾸 틀렸다고 나온다면?

이 비밀번호들은 전부 한/영을 구분하기 때문에 한/영전환을 한 뒤에 시도해보시길.

---

## TL;DR

1. 안드로이드 로그 파일을 뒤져서 비밀번호를 찾아낸다.
2. 기억나는 비밀번호와 한/영 조합을 모두 입력해본다.

---

## 참고 링크

* [http://stackoverflow.com/questions/28034899/how-to-retrieve-key-alias-and-key-password-for-signed-apk-in-android-studiomigr#answer-31232804](http://stackoverflow.com/questions/28034899/how-to-retrieve-key-alias-and-key-password-for-signed-apk-in-android-studiomigr#answer-31232804)

## 작성 날짜

2016-06-24


