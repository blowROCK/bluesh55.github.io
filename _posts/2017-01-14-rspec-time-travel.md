---
layout: post
title: "[Rails] 타임 헬퍼로 시간 테스트하기"
date: 2017-01-14
tags: [dev]
---

테스트 코드를 실행할 때 현재 시간을 변경하고 싶을 때가 있다.
예를 들어 모임 시간을 가지는 모임(Meeting) 클래스가 있고 모임 시간이 지났을 때
종료 상태를 반환하는 메서드를 테스트한다고 해보자. 먼저 모임 클래스는 다음과 같다.

```ruby
class Meeting
  attr_accessor :time

  def initialize(time)
    @time = time
  end

  def finished?
    time < Time.now
  end
end
```

모임의 시간은 임의로 정할 수 있지만 `Time.now`가 반환하는 시간은
변경하기 까다롭다.
이럴 때 레일즈에서 제공하는 타임 헬퍼 모듈
(`ActiveSupport::Testing::TimeHelpers`)을 사용하면 좋다.

```ruby
# spec/unit/meeting_spec.rb
require 'rails_helper'

describe Meeting do
  include ActiveSupport::Testing::TimeHelpers

  describe "#finished?" do
    let(:meeting_time) { Time.now }
    let(:meeting) { Meeting.new(meeting_time) }

    context "모임 시간이 지났을 때" do
      before do
        # 현재 시간을 하루 뒤로 설정한다.
        travel(1.day)
      end

      it { expect(meeting.finished?).to eq(true) }
    end

    context "모임 시간이 지나지 않았을 때" do
      before do
        # 현재 시간을 모임 시간 하루 전으로 설정한다.
        travel_to(meeting_time - 1.day)
      end

      it { expect(meeting.finished?).to eq(false) }
    end
  end
end
```

나는 타임 헬퍼를 시간으로 정렬되는 것을 테스트할 때 사용하고 있다.
데이터베이스에 2개의 레코드를 생성한 뒤 정렬 결과를 확인하는 방식인데,
테스트 코드가 너무 빨리 실행되서인지 레코드의 `created_at`이 같아지는 바람에
정렬 테스트의 결과가 실행할 때 마다 달라지는 문제가 있었다.


```ruby
it "최신순으로 정렬되어야 함" do
  old_meeting = create(:meeting)
  new_meeting = create(:meeting)

  # old_meeting과 new_meeting의 created_at이 같음!
  expect(sorting code).to eq([new_meeting, old_meeting])
end
```

이 때 현재 시간을 1초 정도 딜레이 시켜주면 정상적으로 테스트할 수 있다.
물론 `created_at` 값을 직접 설정해도 되지만 코드를 읽을 때 타임 헬퍼를 사용하는 것이
좀 더 직관적으로 보인다.

```ruby

# created_at 변경
it "최신순으로 정렬되어야 함" do
  old_meeting = create(:meeting, created_at: Time.now)
  new_meeting = create(:meeting, created_at: Time.now + 1.second)

  # old_meeting과 new_meeting의 created_at이 1초 차이남
  expect(sorting code).to eq([new_meeting, old_meeting])
end

# 타임 헬퍼 사용
it "최신순으로 정렬되어야 함" do
  old_meeting = create(:meeting)
  travel 1.second
  new_meeting = create(:meeting)

  # old_meeting과 new_meeting의 created_at이 1초 차이남
  expect(sorting code).to eq([new_meeting, old_meeting])
end
```

이 때 주의할 점은 테스트가 끝나도 현재 시간이 초기화되지 않는다는 것이다.
전체 테스트에서 `travel 1.second` 코드를 300번 사용하면 현재 시간이 5분이 밀리게 된다.
이것 때문에 뜻밖의 오류가 발생할 수 있으니
반드시 테스트 종료 후에 `travel` 메서드로 변경한 시간을 초기화 시키는
`travel_back` 메서드를 실행시켜 주어야 한다. 이것은 `rails_helper.rb` 설정 파일에 작성할 수 있다.
(RSpec 기준)

```ruby
# spec/rails_helper.rb

...

RSpec.configure do |config|
  ...
  config.include ActiveSupport::Testing::TimeHelpers
  ...

  config.after(:each) do
    travel_back
  end
end
```

`rails_helper.rb` 파일에서 타임 헬퍼를 include 하면 모든 테스트 코드에서 사용할 수 있기 때문에
이제 `meeting_spec.rb`에서 include 코드를 작성할 필요는 없다.

```ruby
# spec/unit/meeting_spec.rb
require 'rails_helper'

describe Meeting do
  # include ActiveSupport::Testing::TimeHelpers

  describe "#finished?" do
    ...
  end
end
```
