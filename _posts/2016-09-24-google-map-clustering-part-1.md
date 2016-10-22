---
layout: post
title: "구글맵 마커 클러스터링 해보기 -1-"
date: 2016-09-24
tags: [dev]
---

이번에 안드로이드 앱을 개발하면서 지도에 클러스터링을 해봤는데,
자료가 너무 없어서 삽질을 많이 했기 때문에 나중을 위해 블로그에 포스팅하기로 했다.

마커 클러스터링이란 지도에 표시되는 마커가 너무 많을 때 특정한 기준으로 마커들을
하나의 무리(Cluster)로 묶어주는 것이다.

![](/public/img/blog/clustering/0.png)

![](/public/img/blog/clustering/1.png)

안드로이드에서 지도를 사용할 때 흔히 구글맵, 다음지도를 많이 사용하는데
마커 클러스터링 기능은 현재 구글맵에서만 가능하다. (다음지도는 웹만 지원한다)


먼저 새로운 안드로이드 프로젝트를 생성하고, 기본 액티비티로 MapActivity를 선택한다.
주제가 클러스터링이니 구글맵에 관련된 코드나 Key 등록 방법 등은 건너 뛴다.
예제 코드는 [Github](https://github.com/bluesh55/google-map-clustering-example)에서 볼 수 있다.


## Google Maps Android API 유틸리티 라이브러리 추가하기

클러스터링 기능은 구글맵 기본 내장 기능이 아니기 때문에
구글에서 제공하는 [유틸리티 라이브러리](https://developers.google.com/maps/documentation/android-api/utility/?hl=ko)를 추가해서 사용해야 한다.

먼저 ```app gradle```에 ```come.google.maps.android:android-maps-utils:0.4```를 추가한다.

```gradle
dependencies {
  compile fileTree(dir: 'libs', include: ['*.jar'])
  testCompile 'junit:junit:4.12'
  compile 'com.android.support:appcompat-v7:23.1.1'
  compile 'com.google.android.gms:play-services-base:8.4.0'
  compile 'com.google.android.gms:play-services-maps:8.4.0'
  compile 'com.google.maps.android:android-maps-utils:0.4' // 추가
}
```

```Gradle Sync```를 하면 라이브러리가 설치된다.

## 지도에 띄울 마커 클래스 설계

클러스터링을 구현하기 전에 마커가 뿌려줄 데이터를 담고 있는 모델 클래스가 필요하다.
이번 예제에서는 간단하게 집(House)을 모델로 한다.

```java
public class House {
  private LatLng location;
  private String address;

  public House(LatLng location, String address) {
    this.location = location;
    this.address = address;
  }

  public LatLng getLocation() {
    return location;
  }

  public void setLocation(LatLng location) {
    this.location = location;
  }

  public String getAddress() {
    return address;
  }

  public void setAddress(String address) {
    this.address = address;
  }
}
```

위치좌표(location)와 주소(address) 두 가지의 속성을 가지고있는 간단한 클래스를 구현했다.

하나의 House 객체가 지도 위의 하나의 마커가 될텐데, 그렇게 되려면 ```ClusterItem``` 인터페이스를 구현해야 한다.


```java
public class House implements ClusterItem {
  private LatLng location;
  private String address;

  public House(LatLng location, String address) {
    this.location = location;
    this.address = address;
  }

  public LatLng getLocation() {
    return location;
  }

  public void setLocation(LatLng location) {
    this.location = location;
  }

  public String getAddress() {
    return address;
  }

  public void setAddress(String address) {
    this.address = address;
  }

  @Override
  public LatLng getPosition() {
    return location;
  }
}
```

```getPosition``` 메서드를 오버라이딩해서 House가 지도 위에 위치해야 할 곳의 좌표값을 리턴해준다.
자신의 위치 좌표를 ```location``` 속성으로 갖고있었으니 그대로 리턴하면 된다.

## ClusterManager

클러스터링을 할 때는 구글맵에 직접 마커를 찍지 않고,
```ClusterManager```에 House와 같은 Cluster Item Object를 추가하면
```ClusterManager```가 알아서 마커를 찍어주고 클러스터링까지 해준다.
그리고 반드시 GoogleMap의 **OnCameraChangeListener**로 ```ClusterManager```를 지정해주어야 클러스터링이 실행된다.

GoogleMap이 사용될 준비가 되었을 때 호출되는 ```onMapReady``` 메서드에서
임의의 마커 10개를 ```ClusterManager```에 추가해보자.

```java
@Override
public void onMapReady(GoogleMap googleMap) {
  mMap = googleMap;

  mMap.addMarker(new MarkerOptions().position(Seoul));
  mMap.moveCamera(CameraUpdateFactory.newLatLngZoom(Seoul, 14.0f));

  // 클러스터 매니저 생성
  ClusterManager<House> mClusterManager = new ClusterManager<>(this, mMap);
  mMap.setOnCameraChangeListener(mClusterManager);

  for(int i=0; i<10; i++) {
    double lat = SEOUL_LAT + (i / 200d);
    double lng = SEOUL_LNG + (i / 200d);
    mClusterManager.addItem(new House(new LatLng(lat, lng), "House"+i));
  }
}
```

실행 화면은 다음과 같다.

![](/public/img/blog/clustering/2.png)

## Algorithm과 Renderer

클러스터링은 크게 **알고리즘**과 **렌더러** 두 가지로 나눠져있다.
알고리즘은 클러스터를 어떤 위치에 생성할지, 어떤 마커를 어떤 클러스터에 넣을지를 연산한다.
렌더러는 클러스터를 이쁘게 출력해주는 역할을 한다.


알고리즘과 렌더러는 ```ClusterManager```의 ```setAlgorithm```과 ```setRenderer``` 메서드로 커스텀 클래스를 등록할 수 있으며
등록하지 않은 경우에는 **유틸리티 라이브러리**에 기본으로 탑재되어 있는
```DefaultClusterRenderer```와 ```NonHierarchicalDistanceBasedAlgorithm``` 클래스를 사용한다.

```DefaultClusterRenderer```는 위 실행화면에서 보았듯이 파란색 원에 마커 개수를 표시하는 형태로 출력해준다.
[코드보기](https://github.com/googlemaps/android-maps-utils/blob/master/library/src/com/google/maps/android/clustering/view/DefaultClusterRenderer.java)  
```NonHierarchicalDistanceBasedAlgorithm```은 거리 기반으로 가운데 위치에 클러스터를 만들어 주지만 정교하진 않다.
[코드보기](https://github.com/googlemaps/android-maps-utils/blob/master/library/src/com/google/maps/android/clustering/algo/NonHierarchicalDistanceBasedAlgorithm.java)

기본 클래스들을 그냥 사용해도 상관없지만 웬만하면 커스텀해서 사용하는 것을 추천한다. 

## 참고 링크

* [https://developers.google.com/maps/documentation/android-api/utility/marker-clustering#introduction](https://developers.google.com/maps/documentation/android-api/utility/marker-clustering#introduction)

# 작성 날짜

2016-04-27
