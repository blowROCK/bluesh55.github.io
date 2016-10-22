---
layout: post
title: "당겨서 새로고침 간단하게 구현하기"
date: 2016-09-24
tags: [dev]
---

안드로이드에서 “당겨서 새로고침(Pull to Refresh)" UI를 간단히 구현해보자.

당겨서 새로고침 기능을 구현해주는 ```SwipeRefreshLayout```은 Android support library v4에 포함되어 있다.
먼저 gradle에 support v4 라이브러리를 추가한다.

```gradle
// build.gradle

dependencies {
  ...
  compile 'com.android.support:support-v4:23.4.0'
}
```

그리고 리프레시를 적용할 뷰를 ```SwipeRefreshLayout```으로 감싼다. 대부분 ```ListView``` 또는 ```RecyclerView```가 들어갈 것이다.

```xml
<android.support.v4.widget.SwipeRefreshLayout
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:id="@+id/swipe_layout">

  <ListView
    ...
  />
</android.support.v4.widget.SwipeRefreshLayout>
```

이제 ```SwipeRefreshLayout```을 객체로 만들고 ```OnRefreshListener``` 인터페이스를 등록한다.


```java
SwipeRefreshLayout mSwipeRefreshLayout = (SwipeRefreshLayout) findViewById(R.id.swipe_layout);
mSwipeRefreshLayout.setOnRefreshListener(this);
```

```OnRefreshListener``` 인터페이스는 ```onRefresh``` 메서드를 가지고 있다.
이 메서드는 사용자가 리스트를 끝까지 당겼다가 놓았을 때 호출된다. 여기에 리프레시 코드를 넣으면 된다.

```java
@Override
public void onRefresh() {
  // 코드
}
```

그리고 리프레시가 완료되었다면 반드시 ```setRefresing(false)```를 호출해야 한다.
그렇지 않으면 리프레시가 완료되어도 아이콘이 사라지지 않는다.

```java
@Override
public void onRefresh() {
  // 새로고침 코드
  ...
  // 새로고침 완료  
  mSwipeRefreshLayout.setRefreshing(false);
}
```

아이콘의 색상을 바꾸려면 ```setColorSchemaResources``` 메서드를 사용한다.

```java
mSwipeRefreshLayout.setColorSchemeResources(
  android.R.color.holo_blue_bright,
  android.R.color.holo_green_light,
  android.R.color.holo_orange_light,
  android.R.color.holo_red_light
);
```

화살표가 한바퀴 돌 때마다 각각의 색상으로 나타난다.

## 참고 링크

* [https://guides.codepath.com/android/Implementing-Pull-to-Refresh-Guide](https://guides.codepath.com/android/Implementing-Pull-to-Refresh-Guide)

## 작성 날짜

2016-08-15
