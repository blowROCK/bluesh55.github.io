---
layout: post
title: "Carrierwave 리사이징 옵션 정리"
date: 2017-11-07
tags: [dev]
---

[Carrierwave](https://github.com/carrierwaveuploader/carrierwave)는 루비 웹프레임워크에서 이미지 업로드를
손쉽게 할 수 있게 해주는 라이브러리다. **Carrierwave**로 이미지를 업로드할 때 사용되는 총 4가지의 리사이징 옵션을 정리해봤다.

### 1. resize_to_fit(width, height)

* 비율 유지
* 잘림 없음

이 옵션은 이미지의 비율을 유지하면서 사이즈를 변경한다. 가로 길이(width)와 세로 길이(height)를 입력 받는다.

원본 이미지의 가로, 세로 중 큰 것이 입력받은 값에 도달할 때 까지 비율을 유지하며 리사이징한다.

다음과 같이 오른쪽 위를 잡고 비율을 유지하며 사이즈를 조절한다고 생각하면 된다.

![](/public/img/blog/carrierwave_resizing/sc-1-1.png)

실제로 이미지 사이즈가 어떻게 변하는지 밑의 이미지를 통해 볼 수 있다.

![](/public/img/blog/carrierwave_resizing/sc-1-2.png)

![](/public/img/blog/carrierwave_resizing/sc-1-3.png)


값을 가로와 세로 중 하나만 지정하면 무조건 지정한 쪽에 맞추게 된다.

![](/public/img/blog/carrierwave_resizing/sc-1-4.png)


### 2. resize_to_limit(width, height)

* 비율 유지
* 잘림 없음

`resize_to_fit` 옵션과 동일하지만 원본 이미지 크기가 입력받은 width, height보다 작을 경우에는
리사이징 하지 않는다.


![](/public/img/blog/carrierwave_resizing/sc-1-5.png)

![](/public/img/blog/carrierwave_resizing/sc-1-6.png)

### 3. resize_to_fill(width, height, gravity = 'Center')

* 비율 유지 안함
* 잘림 있음

먼저 **resize_to_fit**과 같이 비율로 리사이징을 한다. 다른점은 **resize_to_fit**은 원본 이미지의 가로, 세로 중 큰 것을 기준으로 하지만,
이 옵션은 원본 이미지의 가로, 세로 중 작은 것을 기준으로 한다.

그리고 비율을 유지하는 리사이징이 끝나면 입력된 width, height 크기에 맞게 잘라낸다(Crop).
잘라내는 위치는 **gravity** 옵션으로 정할 수 있으며 다음과 같은 옵션들을 사용할 수 있다. 

```
NorthWest, North, NorthEast, West, Center, East, SouthWest, South, SouthEast
```

기본 값은 **Center**로 설정된다.

밑의 이미지는 `:resize_to_fill => [1024, 768, 'Center']`를 실행했을 때 리사이징이 적용되는 과정이다.

![](/public/img/blog/carrierwave_resizing/sc-1-7.png)

위의 과정이 적용된 실제 이미지 리사이징 결과는 다음과 같다.

![](/public/img/blog/carrierwave_resizing/sc-1-8.png)

![](/public/img/blog/carrierwave_resizing/sc-1-9.png)

### 4. resize_and_pad(width, height, background = :transparent, gravity = 'Center')

* 비율 유지 안함
* 잘림 없음

이 옵션은 조금 독특한 옵션인데, 이 옵션의 결과 이미지를 먼저 보면 다음과 같다.

![](/public/img/blog/carrierwave_resizing/sc-1-10.png)

![](/public/img/blog/carrierwave_resizing/sc-1-11.png)

![](/public/img/blog/carrierwave_resizing/sc-1-12.png)

이미지를 입력받은 width, height 사이즈로 만들지만, 원본 이미지를 변경하는  것이 아니라 여백을 채움으로써 사이즈를 맞춘다.

세 번째 옵션으로 여백의 색상을 지정할 수 있으며 기본 값은 **transparent(투명)**이다. 투명 옵션은 이미지 파일이
png 일 때 적용되고 그렇지 않다면 검은색으로 적용된다.

네 번째 옵션은 사이즈가 원본 이미지보다 커졌을 때 원본 이미지가 위치할 방향이다. 다음과 같은 옵션을 사용할 수 있다.

```
NorthWest, North, NorthEast, West, Center, East, SouthWest, South, SouthEast
```

기본 값은 **Center**로 설정된다.
