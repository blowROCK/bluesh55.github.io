---
layout: post
title: "logrotate로 리눅스 로그 관리"
date: 2016-12-10
tags: [dev]
---

웹 서비스를 운영할 때 쉽게 놓칠 수 있는 것 중 하나가 로그 관리다.
로그를 관리하지 않아도 운영하는데는 문제가 없으며 운영 초반에는
로그를 관리해야할 정도로 트래픽이 발생하는 경우가 드물기 때문이다.

하지만 시간이 점점 지나고 트래픽이 늘어나면 로그는 고스란히 디스크에 쌓이게 되고,
이를 그대로 방치해두면 엄청난 디스크 용량을 낭비하게 된다.

끝도 없이 늘어나는 로그 파일을 관리하기 위해 리눅스에서는
**logrotate**라는 프로그램을 사용하면 좋다.
logrotate는 정해진 시간마다 로그 파일을 백업시켜주는데,
로그 파일이 무작정 늘어나는 것을 방지하기 위해 로그 파일의 최대 개수를 정해놓으면
최대 개수를 초과했을 때 가장 오래된 로그 파일을 삭제하고 새로운 로그 파일을 생성하면서
rotating 해주는 툴이다.

시스템에 설치되어있지 않다면 설치해준다.

```bash
sudo yum install -y logrotate
```

`/etc/logrotate.conf` 파일에 어떤 로그를 로테이팅할지 작성할 수 있다.
파일을 열어보면 기본으로 작성된 코드가 있을 것이다.

```
# see "man logrotate" for details
# rotate log files weekly
weekly

# keep 4 weeks worth of backlogs
rotate 4

# create new (empty) log files after rotating old ones
create

# use date as a suffix of the rotated file
dateext

# uncomment this if you want your log files compressed
#compress

# RPM packages drop log rotation information into this directory
include /etc/logrotate.d

# no packages own wtmp and btmp -- we'll rotate them here
/var/log/wtmp {
    monthly
    create 0664 root utmp
        minsize 1M
    rotate 1
}

/var/log/btmp {
    missingok
    monthly
    create 0600 root utmp
    rotate 1
}

# system-specific logs may be also be configured here.
```

중간에 `include /etc/logrotate.d`를 보면 `/etc/logrotate.d` 디렉토리의 파일들을
전부 불러오는 것을 알 수 있다. 그럼 이제 `/etc/logrotate.d` 디렉토리에
새로운 설정 파일을 생성하고 레일즈 프로젝트의 로그를 관리하도록 작성해보자.

```
# /etc/logrotate.d/myproject

/home/ec2-user/myproject/log/production.log {
  weekly
  rotate 4
  missingok
  dateext
  postrotate
    touch /home/ec2-user/myproject/tmp/restart.txt
  endscript
}
```

* weekly : 로그 파일을 1주일에 한 번 백업한다. 다른 옵션으로는 daily, monthly, yearly가 있다.
* rotate 4 : 최대 로그 파일 개수를 4개로 제한한다.
  weekly에 rotate 4면 최대 4주치의 로그만 기록되고 5주째부터는
  1주차 로그가 삭제되고 2, 3, 4, 5주차 로그가 저장된다.
* missingok : 로그 파일이 존재하지 않아도 에러를 발생시키지 않는다.
* dateext : 로그 파일에 YYYYMMDD 형식의 확장자를 추가한다.
* postrotate / endscript : rotate 이후에 실행될 명령어를 작성한다.
  로그 파일이 백업된 후 서버를 재시작시켜서 새로운 로그 파일에 작성할 수 있도록 한다.


이렇게 설정한 뒤에 한 번 실행해보자.

```
sudo /usr/sbin/logrotate /etc/logrotate.d/myproject
```

처음 실행했을 때는 아무 일도 일어나지 않는다.
왜냐면 logrotate는 실행될 때 `/var/lib/logrotate.status` 파일을 통해
정해진 기간이 지났는지 확인하는데 방금은 아무 정보가 없었기 때문이다.
`/var/lib/logrotate.status` 파일에 현재 날짜만 기록하고 로테이팅이 실행되지는 않았다.

```
# /var/lib/logrotate.status

logrotate state -- version 2
"/var/log/yum.log" 2016-1-1
"/home/ec2-user/myproject/log/production.log" 2016-12-10
"/var/log/dracut.log" 2016-1-1
"/var/log/wtmp" 2015-12-1
"/var/log/spooler" 2016-12-4
"/var/log/btmp" 2016-12-1
"/var/log/maillog" 2016-12-4
"/var/log/secure" 2016-12-4
"/var/log/messages" 2016-12-4
"/var/account/pacct" 2015-12-1
"/var/log/cron" 2016-12-4
```

production.log 파일이 오늘 날짜(2016-12-10)로 실행되었다는 것이 저장되었다.
이제 12월 17일이 되어야 1주일이 지났다는 것을 인식하고 로테이팅이 실행될 것이다.

지금 당장 확인해보려면 2016-12-10을 일주일 전인 2016-12-03으로 바꾼 뒤
`sudo /usr/sbin/logrotate /etc/logrotate.d/myproject`를 한번 더 실행해보면 된다.

