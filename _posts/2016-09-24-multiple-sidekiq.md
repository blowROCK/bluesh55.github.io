---
layout: post
title: "Sidekiq 프로세스 2개 이상 실행하기"
date: 2016-09-24
tags: [dev]
---

레일즈에서 백그라운드 잡을 처리할 때 [Sidekiq](https://github.com/mperham/sidekiq)을 사용할 수 있는데, 
한 서버에 사이드킥을 사용하는 프로젝트를 2개 이상 돌리고 싶은 경우가 있다.
무작정 사이드킥 프로세스를 두 개 띄우면 서로 충돌이 난다.
이 문제를 해결하는 방법을 알아보자.

## Redis

먼저 사이드킥이 정상적으로 실행되기 위해서는 Redis가 필요하다.
Redis는 메모리에 key/value 형태로 값을 저장하는 Store 이다.

사이드킥 프로세스를 두 개 띄웠을 때 충돌이 나는 이유는 바로 이 Redis 때문이다.
Redis는 여러 개의 데이터베이스로 나눠서 관리하는데,
사이드킥에 따로 Redis 설정을 하지 않으면 디폴트로 0번 데이터베이스를 사용하게 되어있다.
2개 이상의 프로세스에서 0번 데이터베이스를 같이 사용하려다 보니 충돌이 일어나는 것이다.

이 문제를 해결하려면 각각 사용하는 데이터베이스를 다르게 해주면 된다.
Redis의 데이터베이스는 주소 뒤의 숫자로 구분하는데, 기본이 0번이니 나머지 하나는 1번을 사용하도록 한다.

```
redis://localhost:6379/0
redis://localhost:6379/1
```

## Sidekiq 설정

그래서 사이드킥은 접속할 Redis 주소를 지정할 수 있는 설정 옵션을 제공한다.
1번 데이터베이스를 사용할 프로젝트에 ```config/initializers/sidekiq.rb``` 설정파일을 생성하고
다음과 같이 작성한다.

```ruby
# config/initializers/sidekiq.rb

# 1번 데이터베이스로 접속 설정
Sidekiq.configure_server do |config|
  config.redis = { url: 'redis://localhost:6379/1' }
end

# 1번 데이터베이스로 접속 설정
Sidekiq.configure_client do |config|
  config.redis = { url: 'redis://localhost:6379/1' }
end
```

반드시 ```configure_server```와 ```configure_client```를 둘 다 작성해야 한다.
이렇게 설정한 뒤에 사이드킥 프로세스를 띄워보면 충돌없이 실행된다.

## 참고 링크

* [https://github.com/mperham/sidekiq/wiki/Using-Redis](https://github.com/mperham/sidekiq/wiki/Using-Redis)

## 작성 날짜

2016-06-10
