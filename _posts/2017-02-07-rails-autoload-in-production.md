---
layout: post
title: "[Rails] 배포 환경에서 발생하는 Auto loading 문제"
date: 2017-01-29
tags: [dev]
---

레일즈는 소스 코드 파일을 로드하기 위해서
실제로 사용될 때 그제서야 파일을 로드하는 Auto loading[^1]과
서버가 실행될 때 한 번에 모든 파일을 로드하는 Eager loading 두 가지 방식을 사용한다.

며칠 전 까지만 해도 나는 Auto loading 방식만 대충 알고 있었고 Eager loading 방식에 대해서는 모르고 있었는데,
정말 우연한 계기로 두 방식을 상황에 맞게 적절히 사용하지 않으면 개발이나 테스트 환경이 아닌 **배포 환경**에서
문제가 될 수 있다는 것을 알게 되었다. 이 문제를 통해 레일즈의 Auto loading 방식과 Eager loading 방식에 대해서 좀 더 자세히 알아보자.


### 1. 어떤 문제가 있었는지?

lib 디렉토리에 존재하는 파일을 읽지 못하는 문제였다.
lib 디렉토리에 선언된 클래스를 불러오지 못해서 클래스를 사용하려고 하면
`NameError: uninitialized constant` 에러가 발생했다.

그런데 이상했던 점은 개발, 테스트 환경에서는 정상적으로 불러오다가 스테이징, 프로덕션 환경에
배포하기만 하면 불러오지 못한다는 것이었다. 테스트도 모두 성공하고 콘솔로 테스트해봐도
제대로 불러오는데 배포환경만 되면 에러를 뿜어대니 돌아버릴 지경이었다.


### 2. 문제의 원인

오랜 삽질과 검색 끝에 결국 원인을 찾아냈는데,
레일즈 4.2 버전에서 5.0으로 버전 업 될 때 production 환경에서 auto loading 기능을
비활성화 시키는 기능이 추가되었다.[^2]
이게 왜 문제가 되냐면 이전에는 개발 환경이든 배포 환경이든 auto loading을 사용했기 때문에
app 디렉토리 이외의 디렉토리를 로드할 땐 다음과 같이 설정하는게 일반적이었다.

```ruby
# config/application.rb

...
  config.autoload_paths << Rails.root.join('lib')
...
```

그런데 배포 환경에서 auto loading이 비활성화 되어버리니 위의 설정 코드가 무용지물이 되어버리고
lib 디렉토리를 참조할 수 없게 된 것이다.

### 3. 문제의 해결 방법은?

해결 방법은 간단하다. 배포 환경에서는 Eager loading 방식을 사용하므로
배포 환경일 땐 `config.eagerload_paths`에 경로를 추가하면 된다.

```ruby
# config/application

...
  if Rails.env.development? || Rails.env.test?
    config.autoload_paths << Rails.root.join('lib')
  else
    config.eager_load_paths << Rails.root.join('lib')
  end
...
```

그런데 실제로 오토 로딩되는 경로는 `autoload_paths + eager_load_paths`이기 때문에[^3]
설정 코드를 더 간단히 하기 위해 `config.eager_load_paths`만 작성해도 된다.

```ruby
# config/application

...
  config.eager_load_paths << Rails.root.join('lib')
...
```

### 참고

1. [https://collectiveidea.com/blog/archives/2016/07/22/solutions-to-potential-upgrade-problems-in-rails-5](https://collectiveidea.com/blog/archives/2016/07/22/solutions-to-potential-upgrade-problems-in-rails-5)
2. [http://blog.arkency.com/2014/11/dont-forget-about-eager-load-when-extending-autoload/](http://blog.arkency.com/2014/11/dont-forget-about-eager-load-when-extending-autoload/)

[^1]: Lazy loading이라는 용어가 있지만 레일즈에선 auto loading이라고 부른다. Ruby 언어에서 소스 파일을 lazy하게 불러올 때 사용되는 autoload 키워드에서 따온 것 같은데, Ruby의 autoload는 3.0 버전 이후 deprecated 될 수 있다고 한다.

[^2]: [업그레이드 가이드 문서 참조](http://guides.rubyonrails.org/upgrading_ruby_on_rails.html#autoloading-is-disabled-after-booting-in-the-production-environment)

[^3]: [Rails Engine 코드 참조](https://github.com/rails/rails/blob/v5.0.0/railties/lib/rails/engine.rb#L683)
