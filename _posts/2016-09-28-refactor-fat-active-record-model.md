---
layout: post
title: "[번역] 액티브레코드 모델을 리팩토링하는 7가지 방법"
date: 2016-09-28
tags: [dev]
---

2년 가까이 레일즈로 개발을 해오면서 Rails 프로젝트에 OOP를 적용하기 힘들다는 것을 많이 느꼈다.
Java로 안드로이드 앱을 개발할 땐 클래스나 인터페이스도 많이 만들고, 부모 클래스를 만들어서 상속받는 등등
OOP를 적용하며 재밌게 개발했었는데 Rails로 개발할 땐 컨트롤러나 모델 클래스를 제외하면
클래스를 거의 만들지 않고 개발하고 있었다.

그래서 Rails에도 OOP를 적용할 수 있는 방법을 찾아보다가 Code Climate에서 작성한 글을 발견했다.
2012년 글이지만 좋은 글인 것 같아서 메모해둘 겸 번역해봤다. 영어 실력이 부족해서 오역이 있을 수도 있다.

원문 출처 : [http://blog.codeclimate.com/blog/2012/10/17/7-ways-to-decompose-fat-activerecord-models/](http://blog.codeclimate.com/blog/2012/10/17/7-ways-to-decompose-fat-activerecord-models/)

---

여러 팀들이 레일즈 어플리케이션의 퀄리티를 향상시키기 위해 [Code Climate](https://codeclimate.com/)(오픈 소스 코드 검사기)를 사용 하고 나서
모델에 모든 코드를 쑤셔 넣는 습관을 없애게 되었다.
"Fat model"은 앱이 커질수록 유지 보수를 어렵게 만든다.
컨트롤러를 도메인 로직으로 어지럽히는 것보다는 낫지만,
대부분은 **단일 책임 원칙**을 지키지 못한다.
유저가 하는 행동과 연관되어 있다면 그것은 **단일 책임**이 아니기 때문이다.


초반에는 단일 책임 원칙을 적용하기 쉽다. ActiveRecord 모델 클래스는 persistence(영속성)와 associations(관계)만 관리하면 된다.
하지만 시간이 지날수록 클래스는 점점 커지게 된다.
본질적으로 persistence를 책임져야 하는 모델 클래스는 사실상 모든 비즈니스 로직도 같이 책임지게 되어버린다.
그리고 1년, 2년이 지나면 당신의 User 클래스는 500줄이 넘는 코드와 몇 백개의 퍼블릭 메서드를 갖게 될 것이다.
물론 Callback Hell은 덤이다.

어플리케이션에 본질적인 복잡성을 추가하려면, 케이크 반죽을 팬의 바닥에 바를 때 처럼
작고 캡슐화된 객체들에 퍼트려야 한다.
그리고 "Fat model"은 당신이 반죽을 처음 부었을 때 볼 수 있는 커다란 반죽 덩어리와 같다.
"Fat model"을 분해하고, 로직을 고르게 펴 바르려면 리팩토링을 해야한다.
이 프로세스를 반복하다 보면 인터페이스가 잘 정의된 간단한 객체들을 볼 수 있을 것이다.

레일즈에 OOP를 적용하기 어렵다고 생각할 수도 있다.
나도 처음엔 그렇게 생각했다. 하지만 계속 연구하고 연습해보니 레일즈 프레임워크에는 문제가 없었다.
문제는 레일즈의 쉽게 스케일링할 수 없는 "컨벤션"이다.
자세히 말하면 [Active Record 패턴](http://martinfowler.com/eaaCatalog/activeRecord.html)이 명쾌하게 처리할 수 있는 정도를
초과한 **복잡성을 관리할만한 컨벤션**이 없기 때문이다.
하지만 운좋게도 우리는 레일즈가 제공하지 않는 부분에 객체 지향을 기반으로 하는 원칙들과 best practices를 적용시킬 수 있다.

## 모델에서 믹스인을 추출하지 말 것

이것부터 먼저 얘기해보자.
나는 메서드 세트를 거대한 ActiveRecord 클래스에서 "concerns" 혹은 단 하나의 모델에만 사용되는 모듈로 옮기는 행위를 해서는 안된다고 생각한다.
한 번은 누군가 이런 얘기를 하는 것을 들었다.

> "Any application with an app/concerns directory is concerning (번역 불가)."

그리고 나는 이것에 동의한다. 상속(inheritance)보다 구성(composition)을 더 선호한다.
(역주: OOP 개념의 inheritance와 composition을 말한다.)
믹스인을 이렇게 사용하는 것은 마치 어질러진 방을 청소할 때 물건들을 아무 서랍에나 쑤셔박는 것과 같다.
겉은 깨끗해 보이지만 어떤 물건이 어디에 있는지 바로 확인하기 어렵고, 필요한 물건을 찾으려고할 때 매우 힘들어질 것이다.

자, 그럼 이제 리팩토링을 해보자!

## 1. Value Object

[Value Object](http://c2.com/cgi/wiki?ValueObject)는 비교 연산될 때 자신의 상태보다 **값**에 의존하는 간단한 객체이다.
이것은 대부분 변경할 수 없는(immutable) 값이다.
```Date```, ```URI```, 그리고 ```Pathname```은 루비 표준 라이브러리에 포함된 Value Object인데,
이것 뿐만 아니라 당신은 Value Object를 직접 정의할 수 있다(아니, 반드시 정의 해야한다).
ActiveRecord로부터 Value Object를 추출하는 것은 가장 쉬운 리팩토링 작업이다.

레일즈에서는 자신과 연관되어 있는 로직을 가지는 속성(들)이 있을 때 Value Object를 만들면 좋다.
단순 텍스트나 숫자 값 이상의 그 어떤 것이든 Value Object의 대상이 될 수 있다.

예를 들어, 텍스트 메시징 어플리케이션은 ```PhoneNumber``` Value Object를 가질 수 있다.
e-커머스 어플리케이션은 ```Money``` 클래스가 필요할 것이다.  
Code Climate에서는 간단히 A부터 F까지 각 클래스나 모듈이 받은 등급을 나타내는 ```Rating```이라는 Value Object를 만들었다.
Ruby의 ```String``` 클래스를 사용할 수 있었지만(실제로 사용해보기도 했다) ```Rating``` 클래스를 사용하니 데이터와 기능을 결합시킬 수 있었다.

```ruby
class Rating
  include Comparable

  def self.from_cost(cost)
    if cost <= 2
      new("A")
    elsif cost <= 4
      new("B")
    elsif cost <= 8
      new("C")
    elsif cost <= 16
      new("D")
    else
      new("F")
    end
  end

  def initialize(letter)
    @letter = letter
  end

  def better_than?(other)
    self > other
  end

  def <=>(other)
    other.to_s <=> to_s
  end

  def hash
    @letter.hash
  end

  def eql?(other)
    to_s == other.to_s
  end

  def to_s
    @letter.to_s
  end
end
```

```ConstantSnapshot``` 클래스는 이제 Rating 객체를 퍼블릭 메서드로 노출시킬 수 있다.

```ruby
class ConstantSnapshot < ActiveRecord::Base
  # …

  def rating
    @rating ||= Rating.from_cost(cost)
  end
end
```

이것은 ```ConstantSnapshot``` 클래스의 코드를 줄이는 것 뿐만 아니라 몇 가지의 장점이 있다.

* ```#worse_than?```과 ```#better_than?``` 메서드는 Ruby의 빌트인 오퍼레이터(e.g. >, <)보다 등급(rating)을 비교하는데 있어서 더 직관적이다.
* ```#hash```와 ```#eql?```을 정의하는 것은 ```Rating``` 클래스를 해시 키로 사용할 수 있게 만든다.
Code Climate에서는 이것을 ```Enumberable#group_by```를 사용해서 등급으로 그룹을 만들 때 사용한다.
* ```#to_s``` 메서드를 사용하면 추가적인 작업 없이 ```Rating``` 객체를 문자열이나 템플릿에 interpolating 할 수 있다. (역주: interpolating -> "Hello #{rating} World")
* 별도의 클래스로 정의했기 때문에 주어진 "remediation cost"로부터 올바른 Rating 객체를 리턴하는 팩토리 메서드를 만들 수 있다.
(역주: ```Rating.from_cost``` 메서드를 말하는 듯 하다)

## 2. Service Object

시스템상에서 일어나는 몇몇 액션들은 Service Object로 캡슐화할 수 있다. 액션이 다음의 기준을 하나 이상 만족하면 Service Object를 사용한다.

* 액션이 복잡하다 (e.g. 회계 기간이 끝날 때 결산하는 작업)
* 액션이 여러개의 모델을 사용한다 (e.g. ```Order```, ```CreditCard```, ```Customer```를 사용하는 e-커머스 결제 시스템)
* 액션이 외부의 서비스와 상호작용한다 (e.g. SNS에 포스팅)
* 액션이 모델의 근본적인 핵심 기능이 아니다 (e.g. 정해진 기간이 끝난 데이터들을 모두 삭제하는 작업)
* 액션을 처리할 수 있는 여러가지 방법이 존재한다 (e.g. 액세스 토큰이나 비밀번호로 유저 인증)

예를 들어, ```User#authenticate``` 메서드를 ```UserAuthenticator```로 빼낼 수 있다.

```ruby
class UserAuthenticator
  def initialize(user)
    @user = user
  end

  def authenticate(unencrypted_password)
    return false unless @user

    if BCrypt::Password.new(@user.password_digest) == unencrypted_password
      @user
    else
      false
    end
  end
end
```

그리고 ```SessionController```는 이렇게 될 것이다.

```ruby
class SessionsController < ApplicationController
  def create
    user = User.where(email: params[:email]).first

    if UserAuthenticator.new(user).authenticate(params[:password])
      self.current_user = user
      redirect_to dashboard_path
    else
      flash[:alert] = "Login failed."
      render "new"
    end
  end
end
```

## 3. Form Object

다수의 ActiveRecord 모델이 하나의 폼에 의해 업데이트 될 때 Form Object는 서로 다른 데이터들을 캡슐화 할 수 있다.
이것은 (개인적으로 deprecated 되어야 한다고 생각하는)[accepts_nested_attributes_for](http://api.rubyonrails.org/classes/ActiveRecord/NestedAttributes/ClassMethods.html)
를 사용하는 것보다 훨씬 괜찮은 방법이다.
가장 일반적인 예제는 ```Company```와 ```User```를 동시에 생성하는 회원가입 폼이다.

```ruby
class Signup
  include Virtus

  extend ActiveModel::Naming
  include ActiveModel::Conversion
  include ActiveModel::Validations

  attr_reader :user
  attr_reader :company

  attribute :name, String
  attribute :company_name, String
  attribute :email, String

  validates :email, presence: true
  # … more validations …

  # Forms are never themselves persisted
  def persisted?
    false
  end

  def save
    if valid?
      persist!
      true
    else
      false
    end
  end

private

  def persist!
    @company = Company.create!(name: company_name)
    @user = @company.users.create!(name: name, email: email)
  end
end
```

ActiveRecord와 같은 attribute 기능을 얻기 위해 [Virtus](https://github.com/solnic/virtus)를 사용했다.
Form Object는 ActiveRecord와 유사하게 동작할 것이고, 따라서 컨트롤러도 ActiveRecord를 사용하는 것 처럼 작성하면 된다.

```ruby
class SignupsController < ApplicationController
  def create
    @signup = Signup.new(params[:signup])

    if @signup.save
      redirect_to dashboard_path
    else
      render "new"
    end
  end
end

```

이것은 위와 같이 간단한 경우에 잘 동작한다.
하지만 폼의 persistence 로직이 너무 복잡하다면 하나의 Service Object로 합칠 수도 있다.  
As a bonus, since validation logic is often contextual, it can be defined in the place exactly where it matters instead of needing to guard validations in the ActiveRecord itself.  
(역주: 해석하지 못했으나 Signup 클래스에서 validation 하는 것을 말하는 듯 합니다.)

## 4. Query Object

ActiveRecord 서브클래스(=스코프 혹은 클래스 메서드) 정의를 복잡하게 만드는 SQL 쿼리가 있으면 Query Object를 고려해봐야 한다.
각각의 Query Object는 비즈니스 규칙에 따라 결과셋을 반환하는 책임을 가진다.
예를 들어 시험 버전이 만료된 계정을 찾는 Query Object는 이렇게 작성할 수 있다.

```ruby
class AbandonedTrialQuery
  def initialize(relation = Account.scoped)
    @relation = relation
  end

  def find_each(&block)
    @relation.
      where(plan: nil, invites_count: 0).
      find_each(&block)
  end
end
```

백그라운드 잡에서 이메일을 보내기 위해 이렇게 사용할 수 있다.

```ruby
AbandonedTrialQuery.new.find_each do |account|
  account.send_offer_for_support
end
```

```ActiveRecord::Relation``` 인스턴스는 Rails 3부터 [일급 객체](https://ko.wikipedia.org/wiki/%EC%9D%BC%EA%B8%89_%EA%B0%9D%EC%B2%B4)이기
때문에 Query Object의 입력값으로 사용하기 좋다.
이렇게 하면 구성(composition)을 사용해서 쿼리를 결합할 수 있다.

```ruby
old_accounts = Account.where("created_at < ?", 1.month.ago)
old_abandoned_trials = AbandonedTrialQuery.new(old_accounts)
```

이런 클래스는 격리 상태로 테스트하지 않는 것이 좋다.
올바른 로우를 알맞은 순서로 리턴하는지, 조인이나 eager loading이 제대로 동작하는지 확인해야 하므로
객체와 데이터베이스를 함께 테스트 해야 한다.
(e.g. [N + 1 쿼리](http://guides.rubyonrails.org/active_record_querying.html#eager-loading-associations) 방지)

## 5. View Object

만약 순전히 디스플레이 목적으로 필요한 로직이라면, 그것은 모델에 속하면 안된다.
자신에게 물어보라, **"내가 만약 이 어플리케이션에 음석 인식 UI를 구현하고 있다면, 이게 필요한가?"**
만약 아니라면 Helper나 View Object에 작성하는것을 고려해봐야 한다.  
(역주: 음성 인식 UI는 View가 없으니 "View가 없는 상태에서도 이 코드가 모델 클래스에 있어야 하는가?"를 묻는 것 같다.)

예를 들어, Code Climate에서 도넛 차트는 코드베이스(e.g. [Rails on Code Climate](https://codeclimate.com/github/rails/rails))
의 스냅샷을 기반으로 클래스 등급을 분해 하고, **View**로 캡슐화 된다.

```ruby
class DonutChart
  def initialize(snapshot)
    @snapshot = snapshot
  end

  def cache_key
    @snapshot.id.to_s
  end

  def data
    # pull data from @snapshot and turn it into a JSON structure
  end
end
```

나는 ```View```와 ```ERB (or Haml/Slim)```템플릿 사이의 one-to-one 관계를 자주 볼 수 있었다.
이것 때문에 레일즈에 사용 가능한 [Two Step View](http://martinfowler.com/eaaCatalog/twoStepView.html) 패턴을
구현하는 방법을 찾아 봤지만, 아직 명확한 방법을 찾지는 못했다.

Note: 레일즈 커뮤니티에서 "Presenter"라는 용어가 자주 보이는데, 나는 이 용어를 잘못 사용하지 않기 위해 사용을 피하는 편이다.
[Jay Fields](http://blog.jayfields.com/2007/03/rails-presenter-pattern.html)는 위에서 언급했던 "Form Object"를
설명하기 위해 "Presenter"라는 용어를 사용했다.
또한 레일즈에서는 "View"라는 용어를 "template"을 나타내기 위해 사용한다.
애매한 부분을 피하기 위해 가끔 View Object를 "View Model"로 표현하기도 한다.


## 6. Policy Object

때때로 복잡한 읽기 명령은 자신의 객체를 받을 수 있다.
이런 경우에는 Policy Object를 사용할 수 있다.
이렇게 하면, 예를 들어 **"유저가 분석 목적에 맞게 활성화 되어 있는지"**와 같은 모델과 관계 없는 로직이 모델 클래스에 들어가지 않게 할 수 있다.

예를 들어

```ruby
class ActiveUserPolicy
  def initialize(user)
    @user = user
  end

  def active?
    @user.email_confirmed? &&
    @user.last_login_at > 14.days.ago
  end
end
```

위의 Policy Object는 유저가 이메일 인증을 거쳤고, 최근 2주 안에 로그인 했을 때 active 상태로 간주한다는 비즈니스 규칙을 캡슐화한 것이다.
또한 유저가 액세스할 수 있는 데이터를 통제하는 Authorizer와 같은 그룹 규칙으로도 Policy Object를  사용할 수 있다.

Policy Object는 Service Object와 비슷하지만 나는 명령을 **작성 할 때** Service Object 용어를 사용하고,
Policy Object는 **읽을 때** 사용한다.  
또한 Query Object와도 비슷할 수 있는데, Query Object는 결과를 반환하기 위해 SQL을 실행하는 것에 포커스 되어 있는 반면에
Policy Object는 이미 메모리에 로드된 모델 객체에서 실행된다.

## 7. Decorator

Decorator는 기존의 여러 작업들을 겹겹이 쌓을 수 있게 하며, 콜백과 비슷한 목적을 제공한다.
일부 상황에서만 실행되는 콜백이거나, 모델에 포함되어있는 콜백이 너무 많은 책임을 모델에 부여할 경우 Decorator를 사용하면 좋다.

블로그 포스트에 댓글을 등록했을 때 등록한 댓글을 페이스북의 담벼락에도 등록되게 하고 싶지만
이 작업이 ```Comment``` 클래스와 엮여있을 필요는 없다.
콜백에 너무 많은 책임을 추가하면 느리고 깨지기 쉬운 테스트가 되거나
전혀 무관한 테스트 케이스에 대해 부작용을 없애고 싶은 충동을 느낄 수 있다.

페이스북 포스팅 로직을 Decorator로 만들면 이렇게 될 수 있다.

```ruby
class FacebookCommentNotifier
  def initialize(comment)
    @comment = comment
  end

  def save
    @comment.save && post_to_wall
  end

private

  def post_to_wall
    Facebook.post(title: @comment.title, user: @comment.author)
  end
end
```

컨트롤러에서는 다음과 같이 사용한다.

```ruby
class CommentsController < ApplicationController
  def create
    @comment = FacebookCommentNotifier.new(Comment.new(params[:comment]))

    if @comment.save
      redirect_to blog_path, notice: "Your comment was posted."
    else
      render "new"
    end
  end
end
```

각 레이어가 기존의 인터페이스에 대한 책임을 지기 때문에 Decorator는 Service Object와 다르다고 할 수 있다.
Decorator를 만들면 사용하는 입장에서는 ```FacebookCommentNotifier``` 인스턴스를 마치 ```Comment```를 사용하듯이 쓰면 된다.
루비는 표준 라이브러리로 [메타프로그래밍을 사용해 데코레이터를 쉽게 만드는](https://robots.thoughtbot.com/evaluating-alternative-decorator-implementations-in)
몇 가지의 기능들을 제공한다.

## 마치며

레일즈 어플리케이션에도 모델 레이어의 복잡성을 관리하는 많은 툴이 존재한다.
이것들은 당신이 레일즈를 버리는 것을 요구하지 않는다.
ActiveRecord는 환상적인 라이브러리지만, 어떤 패턴이든간에 그것에만 너무 의존하는 경우 문제가 생길 수 있다.
ActiveRecord가 persistence 기능만 수행하도록 제한하라.
모델에 있는 로직을 전체로 확산시키기 위해 이런 테크닉들을 적용하다 보면 좀 더 유지하기 쉬운 어플리케이션이 될 것이다.

또한 당신이 알아야 할 것은 여기에 작성된 많은 패턴들은 상당히 간단하다는 것이다.
객체는 단지 "Plain Old Ruby Objects" (PORO)를 다른 방식으로 사용했고, 이것이 바로 핵심이자 OOP의 장점이다.
모든 문제들은 프레임워크나 라이브러리에 의해 해결될 필요가 없으며, 이름을 짓는 것도 매우 중요하다.


