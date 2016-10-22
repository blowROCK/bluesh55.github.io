---
layout: post
title: "MYSQL에 이모티콘 저장하기"
date: 2016-10-11
tags: [dev]
---

![](/public/img/blog/emoji/emojis.png)

MYSQL에 이모티콘을 저장하려 하면 `Incorrect string value` 에러가 발생하면서 저장되지 않는 문제가 발생했다.

```mysql
mysql> desc faqs;
+------------+--------------+------+-----+---------+----------------+
| Field      | Type         | Null | Key | Default | Extra          |
+------------+--------------+------+-----+---------+----------------+
| id         | int(11)      | NO   | PRI | NULL    | auto_increment |
| question   | varchar(255) | YES  |     | NULL    |                |
| answer     | text         | YES  |     | NULL    |                |
| created_at | datetime     | NO   |     | NULL    |                |
| updated_at | datetime     | NO   |     | NULL    |                |
+------------+--------------+------+-----+---------+----------------+
5 rows in set (0.00 sec)

mysql> insert into faqs(question, answer, created_at, updated_at)
          values('This is question', 'This is answer 😎 ', NOW(), NOW());
ERROR 1366 (HY000): Incorrect string value: '\xF0\x9F\x98\x8E ' for column 'answer' at row 1

mysql> select * from faqs;
Empty set (0.00 sec)
```

utf8 인코딩은 3바이트까지 지원하는데 이모티콘은 4바이트를 차지하기 때문에 문제가 발생한다고 한다.
이를 해결하기 위해 [MYSQL 5.5.3](https://dev.mysql.com/doc/relnotes/mysql/5.5/en/news-5-5-3.html) 부터
4바이트를 지원하는 **utf8mb4**라는 캐릭터 셋이 추가되었다.
그러므로 이모티콘을 지원하기 위해서는 테이블의 캐릭터 셋을 utf8에서 utf8mb4로 변경하면 된다.



## 레일즈 프로젝트에서 마이그레이션 하기

기존 테이블들의 캐릭터 셋을 변경하는 마이그레이션 파일을 생성한다.

```bash
rails g migration ConvertCharsetToUtf8mb4
```

Raw SQL을 작성해서 원하는 테이블의 캐릭터 셋 속성을 변경해준다.
그리고 인덱싱되는 VARCHAR 형식의 컬럼이 있다면 최대 길이를 191로 변경해야 한다.
InnoDB 엔진의 인덱스 최대 길이는 767 bytes 이기 때문에 최대 글자 수는
3 bytes의 utf8은 255자, 4 bytes의 utf8mb4에서는 191자가 된다.

```ruby
class ConvertCharsetToUtf8mb4 < ActiveRecord::Migration[5.0]
  def change
    execute("ALTER TABLE table_name CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci")
    execute("ALTER TABLE table_name CHANGE column_name VARCHAR(191) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci")
  end    
end  
```

`config/database.yml` 파일에 인코딩을 변경해준다.

```yml
production:
  ...
  encoding: utf8mb4
  collation: utf8mb4_unicode_ci
```

이제 마이그레이트를 실행하면 utf8 대신 utf8mb4가 적용되고 이모티콘이 잘 저장되는 것을 볼 수 있다.

```mysql
mysql> insert into tags(name, created_at, updated_at) values ("Emoji😎 ", NOW(), NOW());
Query OK, 1 row affected (0.00 sec)
mysql> select * from tags;
+----+------------+---------------------+---------------------+
| id | name       | created_at          | updated_at          |
+----+------------+---------------------+---------------------+
|  1 | Emoji😎     | 2016-10-11 20:19:02 | 2016-10-11 20:19:02 |
+----+------------+---------------------+---------------------+
1 row in set (0.00 sec)
```

마이그레이트를 진행하면서
`Mysql2::Error: Specified key was too long; max key length is 767 bytes: CREATE UNIQUE INDEX` 에러가 발생할 수도 있다.
이 문제는 이미 생성되어 있는 인덱스의 길이가 아직 255로 지정되어 있어서 발생하는 것 같다.
만약 이 문제가 발생한다면 마이그레이션 파일에 인덱스를 지우고 새로 만드는 코드를 작성해주자.

```ruby
class ConvertCharsetToUtf8mb4 < ActiveRecord::Migration[5.0]
  def change
    remove_index(table_name, index_name)
    add_index(table_name, index_name, unique: true, using: :btree, length: {column_name: 191}

    execute("ALTER TABLE table_name CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci")
    execute("ALTER TABLE table_name CHANGE column_name VARCHAR(191) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci")
  end    
end  
```

## 참고 링크

* [http://blog.arkency.com/2015/05/how-to-store-emoji-in-a-rails-app-with-a-mysql-database/](http://blog.arkency.com/2015/05/how-to-store-emoji-in-a-rails-app-with-a-mysql-database/)
