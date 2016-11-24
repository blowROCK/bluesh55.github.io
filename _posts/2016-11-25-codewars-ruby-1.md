---
layout: post
title: "[Ruby] Don't rely on luck"
date: 2016-11-25
tags: [codewars]
---

![](/public/img/blog/codewars/3/1.png)

**문제**: 랜덤 함수의 결과값을 맞춰라. 운에 맡기지 말 것.

이상한 문제가 나왔다. 랜덤한 값을 반환하는 Kernel 모듈의 rand() 함수의 반환 값을 맞추면
된다. 테스트 코드는 다음과 같다.

```ruby
# This is exactly what the real test fixture looks like.
lucky_number = (Kernel::rand() * 100 + 1).floor
Test.assert_equals(guess, lucky_number, "Sorry. Unlucky this time.")
```

우리가 작성할 코드에는 guess 변수를 선언하는 한 줄의 코드만 적혀있다.

```ruby
guess = 10
```

내가 푼 방법은 Kernel 모듈의 rand 함수를 재정의하는 것이었다.

```ruby
module Kernel
  def self.rand
    10
  end
end

guess = 1001
```

rand 함수의 반환 값을 10으로 고정시켜서 lucky_number가 1001이 되도록 만든다.
하지만 다른 모듈을 재정의하면 의도치 않은 오류가 발생할 수 있으므로 좋은 방법은 아닌 것 같다.

또 다른 방법은 srand를 사용하는 것이다.
srand는 랜덤 함수의 시드를 설정하는데, 설정한 시드가 같으면 랜덤함수에서 같은 값이 반환된다.
이것을 이용해서 rand 함수의 반환값을 예측할 수 있다.

```ruby
srand 1
guess = 42
```

시드를 1로 설정하면 rand 함수의 반환값이 처음엔 무조건 0.417022004...가 나오게된다.
이 값에 수식을 적용하면 lucky_number는 42가 된다는 것을 예측할 수 있다.


간단하지만 풀이 방법이 재밌었던 문제였다 ^^


* 문제 링크 : [https://www.codewars.com/kata/dont-rely-on-luck/ruby](https://www.codewars.com/kata/dont-rely-on-luck/ruby)
