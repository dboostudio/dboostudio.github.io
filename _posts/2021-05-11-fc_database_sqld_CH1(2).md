---
title: <패스트캠퍼스> 데이터베이스와 SQLD CH1. 데이터베이스(2)
tags: LectureNote Fastcampus Database SQL
---

## Oracle 설치

나는 Window환경이 아니라서, 설치를 통해 진행할 수 없고 도커나 가상머신을 통한 DB설치를 진행해야만 했
다. 내가 선택한 설치 방법은 MacOS에 virtualBox + vagrant로 OracleDB 설정하기이다. 본 블로그
포스트로 올려뒀으니 참고하시길 바란다.

## SQLPLUS 활용하기

Oracle 서버에 터미널로 접속한 후, `sqlplus`를 입력하여 SQLPLUS로 진입한다.

- 실습용 계정 생성 및 권한 주기
~~~sql
ALTER SESSION SET  "_ORACLE_SCRIPT"=TRUE; --현재 세션 설정
~~~
위 설정은 12c로 들어오면서의 변경점을 적용하지 않겠다는 것이다.

1)사용자 계정 생성
~~~sql
CREATE USER OT IDENTIFIED BY 1234;
~~~

2) OT 계정에 권한을 주기
~~~sql
GRANT CONNECT, RESOURCE, DBA TO OT;
~~~

3) 테이블 스페이스 생성

- OT_DATA 테이블스페이스 생성
~~~sql
-- 테이블 스페이스 생성
CREATE TABLESPACE OT_DATA
datafile '/opt/oracle/oradata/OT_DATA.dbf' SIZE 1G
AUTOEXTEND ON NEXT 512M MAXSIZE UNLIMITED
LOGGING
ONLINE
PERMANENT
EXTENT MANAGEMENT LOCAL AUTOALLOCATE
BLOCKSIZE 8K
SEGMENT SPACE MANAGEMENT AUTO
FLASHBACK ON;
~~~

- OT_DATA 테이블스페이스 생성
~~~sql
-- SIZE 100M
CREATE TEMPORARY TABLESPACE OT_TMP TEMPFILE '/opt/oracle/oradata/OT_TMP.dbf' SIZE 100M AUTOEXTEND ON NEXT 100M MAXSIZE UNLIMITED;
~~~

~~~sql
-- DEFAULT 테이블 스페이스 지정
ALTER USER OT DEFAULT TABLESPACE OT_DATA;
ALTER USER OT TEMPRARY TABLESPACE OT_TMP;
~~~
