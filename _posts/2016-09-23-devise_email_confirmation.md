---
layout: post
title:  "Devise 이메일 인증 구현하기"
date: 2016-09-23
tags: [dev]
---

Rails로 유저 기능을 구현할 때 [Devise](https://github.com/plataformatec/devise)를 많이 사용한다.
Devise는 유저와 관련된 많은 기능을 제공하는데, 몇 가지 읊어보자면 회원가입, 로그인, 로그아웃, 비밀번호 찾기, 이메일 인증, 소셜 로그인 등이 있다.
이번 글에서는 Devise의 많은 기능 중에서 이메일 인증 기능을 구현해보려고 한다. 코드만 보고싶다면 [여기](https://github.com/bluesh55/devise-email-confirmation-exmaple)서 보면 된다.

다음은 일반적인 회원가입이 이루어지는 순서다.

1. 유저가 회원가입 폼을 통해 가입 신청
2. 필요한 값을 올바르게 채워서 보냈다면 새로운 유저를 생성
3. 가입 완료

다음은 이메일 인증이 추가되었을 때이다.

1. 유저가 회원가입 폼을 통해 가입 신청
2. 필요한 값을 올바르게 채워서 보냈다면 인증되지 않은 새로운 유저를 생성
3. 바로 로그인되지 않고 이메일 인증을 완료해달라는 문구 띄움
4. 해당 이메일로 인증메일을 전송
5. 유저가 이메일 확인 후 이메일에 첨부되어있는 링크 클릭
6. 해당 링크에 접속하면 인증이 완료되며 2번에서 생성한 유저는 인증된 유저로 변경됨
7. 가입 완료되어서 이 때 부터 로그인 할 수 있음

유저가 가입을 완료하기 위한 단계가 4단계나 늘어났지만 우리가 해야 할 일은 많지 않다.

# 1. 유저 테이블에 인증 관련 컬럼 추가하기

Devise가 기본적으로 만들어주는 유저 모델에는 이메일 인증에 관련된 컬럼이 없다.
컬럼을 추가하기 위해서 마이그레이션 파일을 작성해야 하는데, Devise는 이미 주석으로 코드를 작성해놓았다.

```ruby
# db/migrate/~~~~_devise_create_users.rb

class DeviseCreateUsers < ActiveRecord::Migration
  def change
    create_table(:users) do |t|
      ## Database authenticatable
      t.string :email,              null: false, default: ""
      t.string :encrypted_password, null: false, default: ""

      ...

      ## Confirmable
      # t.string :confirmation_token
      # t.datetime :confirmed_at
      # t.datetime :confirmation_sent_at
      # t.string :unconfirmed_email # Only if using reconfirmable
      
      ...

    end
  end
 end
```

Confirmable 코드 4줄을 주석해제한 뒤에 `rake db:migrate`를 해주면 유저 테이블에 이메일 인증에 필요한 컬럼들이 생성된다.

컬럼을 추가하기 전에 이미 `rake db:migrate`를 실행해서 유저 테이블이 생성된 상태라면
새로운 마이그레이션 파일을 생성하고 위의 4가지 컬럼을 직접 추가해주면 된다.

```bash
$ rails g migration AddConfirmableToUser
```
```ruby
# db/migrate/~~~~~~_add_confirmable_to_user.rb
class AddConfirmableToUser < ActiveRecord::Migration
  def change
    add_column :users, :confirmation_token, :string
    add_column :users, :confirmed_at, :datetime
    add_column :users, :confirmation_sent_at, :datetime
    add_column :users, :unconfirmed_email, :string
  end
end
```

Devise는 ```confirmed_at``` 컬럼의 값이 nil인 경우 인증되지 않은 유저로 판단한다.  
새로운 유저가 생성되었을 때 ```confirmed_at```의 값이 nil이기 때문에 인증되지 않은 유저가 된다.
그리고 이메일 인증을 완료했을 때 ```confirmed_at```에 인증이 완료된 시간값이 채워지게 됨으로써 인증된 유저로 변경된다.

# 2. 인증 메일 전송하기

Rails는 메일을 보내기 위해 ```Mailer```를 사용한다.
Mailer는 어떤 SMTP 서버를 사용할 것인지, 누구에게 보낼 것인지, 어떤 내용을 보낼 것인지 등 메일에 관련된 모든 것을 담당한다.
Devise는 역시 Mailer도 제공하고 있다. 우리가 할 일은 단지 몇 가지의 설정 뿐이다.

## 2-1. 유저 모델 설정

Devise는 여러가지 기능들을 모델에 명시함으로써 on/off할 수 있다.
기본적으로 `registerable, rememberable, validatable` 등이 명시되어 있다.
이메일 인증을 위한 ```confirmable```은 기본옵션이 아니기 때문에 직접 명시해주자.

```ruby
# app/models/user.rb
class User < ActiveRecord::Base
  devise :database_authenticatable, :registerable,
         :recoverable, :rememberable, :trackable, :validatable, :confirmable
end
```

이 부분을 건너뛰면 밑의 설정을 정상적으로 하더라도 이메일이 발송되지 않으므로 주의하도록 하자.

## 2-2. SMTP 설정

메일을 보내기 위해서는 반드시 메일을 보내주는 SMTP 서버가 필요하다.
SMTP 서비스는 여러가지 있는데 여기서는 [Mailgun](https://mailgun.com)을 사용할 것이다.
Mailgun에 가입하면 1개의 샌드박스 도메인을 주는데 샌드박스 도메인으로 테스트 하다가 자신만의 도메인을 등록해서 사용하면 된다.
샌드박스 도메인은 하루 300건의 제한이 있다.

```ruby
# config/environments/development.rb

Rails.application.configure do
  ...
  config.action_mailer.delivery_method = :smtp
  config.action_mailer.smtp_settings = { 
    :authentication => :plain,
    :address => "smtp.mailgun.org",
    :port => 587,
    :domain => "sandbox93ac32d21a89c992f...d103.mailgun.org
",
    :user_name => "postmaster@sandbox93ac32d21a89c992f...d103.mailgun.org",
    :password => "4321098765431abcdef12345.."
  }
end
```
설정 파일에 자신의 샌드박스 도메인 정보를 입력하면 된다.
Git으로 코드를 관리할 때 위와 같이 민감한 계정 정보를 하드코딩하는 일은 없도록 하는 것이 좋지만,
Github 같은 remote repository에 업로드할 것이 아니라면 일단은 하드코딩해도 좋다.
[예제](https://github.com/bluesh55/devise-email-confirmation-exmaple)를 보면 민감한 정보를 [dotenv](https://github.com/bkeepers/dotenv)젬을 사용해서 환경변수로 관리하고 있다.

# 3. Host 설정

유저가 이메일에 첨부된 링크를 눌렀을 때 어떤 주소로 접속이 되어야 할까?
개발모드일 땐 ```localhost:3000```으로 접속이 되어야 할테고, 운영모드일 땐 서비스의 도메인으로 접속이 되어야 할 것이다.
이렇게 이메일에 첨부된 링크를 눌렀을 때 어떤 주소(host)로 이동해야 할지 설정해야 한다.

```ruby
# config/environments/development.rb
Rails.application.configure do
  ...

  config.action_mailer.default_url_options = {host:'localhost:3000'}
end
```

개발모드에서 확인하기 위해 ```localhost:3000```으로 설정했다.
여기까지 잘 따라왔다면 회원가입했을 때 해당 이메일로 `confirmation instructions`라는 제목의 이메일이 발송되고,
이메일에 첨부된 링크를 클릭하면 유저가 정상적으로 인증된 유저로 바뀌는 것을 볼 수 있을 것이다.

유저가 인증되었는지 확인하는 코드는 다음과 같다.
```ruby
user.confirmed?
```

# 4. Version

글 작성일 : 2015-12-22  
Devise 버전 :  3.5.3  
Rails 버전 : 4.2.1

