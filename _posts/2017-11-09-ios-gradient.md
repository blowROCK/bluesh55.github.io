---
layout: post
title: "[iOS] 뷰에 그라데이션 적용하기"
date: 2017-11-09
tags: [dev]
---

뷰에 그라데이션 효과를 주고 싶을 땐 `CAGradientLayer` 객체를 만들어서 뷰의 레이어 위에 올리면 된다.

```swift
let containerView = UIImageView(frame: CGRect(x: 0, y: 0, width: 1024, height: 1024))

var gradient = CAGradientLayer()

// gradient layer 설정...

containerView.layer.addSublayer(gradient)
```

그렇다면 원하는대로 그라데이션을 만들기 위해서는 `CAGradientLayer`를 어떻게 설정해야하는지 알아보자.
이번 포스팅에서는 이런 그라데이션을 만들어 볼 것이다.

![](/public/img/blog/gradient/1.png)

## 1. 방향

먼저 그라데이션의 진행 방향을 정해보자. **시작점**과 **끝점**을 정해야하는데, 여기에 사용되는 좌표 체계는 다음과 같다.

```
(x,y)

0, 0-------------0.5, 0-----------------1, 0
|                   |                      |
|                   |                      |
|                   |                      |
|                   |                      |
|                   |                      |
|                   |                      |
0, 0.5-----------0.5, 0.5---------------1, 0.5
|                   |                      |
|                   |                      |
|                   |                      |
|                   |                      |
|                   |                      |
|                   |                      |
0, 1-------------0.5, 1-------------------1, 1

```

위에서 아래로 진행되는 그라데이션을 만들기 위해 `startPoint`, `endPoint`속성을 다음과 같이 설정한다.

```swift
// Swift 3
gradient.startPoint = CGPoint(x: 0.5, y: 0)
gradient.endPoint = CGPoint(x: 0.5, y: 1)
```

참고로 좌표는 꼭 0.5 단위로 지정하지 않아도 된다.

## 2. 색상

그라데이션은 반드시 2개 이상의 색상으로 이루어진다. `colors` 속성에 2개 이상의 `UIColor`가 담긴 배열을 넘기면 된다.

```swift
// Swift 3
gradient.colors = [
  UIColor(red:1, green:0, blue:0, alpha:1).cgColor, // Top
  UIColor(red:0, green:1, blue:0, alpha:1).cgColor, // Middle
  UIColor(red:0, green:0, blue:1, alpha:1).cgColor // Bottom
]
```

진행 방향에 맞게 배열 순서를 정해야하는 것에 주의하자.

## 3. 변화 위치

마지막으로, 색상의 변화를 줄 지점을 정할 수 있다. 
설정하려면 `locations` 속성에 위치 값을 배열로 넘기면 되고,
넘겨야하는 위치의 개수는 색상 개수와 같아야 한다.
필수 값은 아니라서 설정하지 않으면 다음과 같이 색상 개수대로 등분된다.

![](/public/img/blog/gradient/2.png)

눈을 크게 뜨고 보면 세로를 2등분하는 선이 보일 것이다. 이것은 다음과 같이 설정한 것과 같다.

```swift
// Swift 3

gradient.locations = [0, 0.5, 1]
// 0 ~ 0.5 : Red ~ Green
// 0.5 ~ 1 : Green ~ Blue
```

위에서 정한 목표대로 만들기 위해서는 0.3 정도 주면 된다.

```swift
// Swift 3

gradient.locations = [0, 0.3, 1]
// 0 ~ 0.3 : Red ~ Green
// 0.3 ~ 1 : Green ~ Blue
```

![](/public/img/blog/gradient/1.png)
