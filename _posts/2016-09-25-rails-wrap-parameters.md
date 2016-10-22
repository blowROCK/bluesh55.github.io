---
layout: post
title: "wrap_parameters"
date: 2016-09-25
tags: [dev]
---

레일즈로 개발하다 보면 클라이언트에서 파라미터를 특정 엘리먼트로 래핑해서 보내야하는 경우가 있다.
예를 들어 Devise는 요청의 모든 파라미터들을 user로 감싸서 보내야한다.
하지만 클라이언트 입장에서는 이 작업이 번거롭고 불편하다.

```json
// Parameters
{
  "email": "example@gmail.com",
  "name": "홍길동"
}

// Nested parameters
{
  "user": {
    "email": "example@gmail.com",
    "name": "홍길동"
  }
}
```

이런 경우에 ```wrap_parameters```를 사용할 수 있다.
```wrap_parameters```는 클라이언트의 요청이 서버에 전달되기전에 클라이언트와 서버 중간에서 파라미터를 특정 엘리먼트로 래핑해준다.
결과적으로 클라이언트와 서버 모두 각자 편한 방법으로 파라미터를 처리할 수 있다.

컨트롤러 상단에 작성할 수 있으며 어떤 엘리먼트로 감쌀지, 어떤 포맷의 요청에 처리할지 등을 지정할 수 있다.

```ruby
class RegistrationsController < Devise::RegistrationsController
  wrap_parameters :user, format: [:url_encoded_form, :multipart_form, :json]
  ...
end
```

위 코드는 Devise의 회원가입 컨트롤러에서 ```wrap_parameters```를 사용한 것이다.

이제 클라이언트에서 ```{"email": "example@gmail.com", "password": "12341234"}``` 이렇게 요청을 보내도,  
서버에서는 ```{"user": {"email": "example@gmail.com", "password": "12341234"}}``` 이렇게 받을 수 있다.

---

```wrap_parameter```는 몇 가지 옵션을 제공하는데, 특정 포맷의 요청에만 적용되도록 할 수 있다.

```
url_encoded_form : Content-Type: Application/x-www-form-urlencode
multipart_form : Content-Type: multipart/formed-data
json : Content-Type: Application/json
```

어떤 파라미터를 래핑할 것인지 혹은 어떤 파라미터는 래핑하지 않을 것인지에 대한 옵션도 제공한다.

```ruby
# whitelist
wrap_parameters :user, format: [:json], include: [:email, :password]

# blacklist
wrap_parameters :user, format: [:json], exclude: [:password_confirmation]
```

## 참조 링크
* [http://api.rubyonrails.org/classes/ActionController/ParamsWrapper.html](http://api.rubyonrails.org/classes/ActionController/ParamsWrapper.html)
