---
layout: post
title: "[개발 로그] Mysql을 EC2 -> RDS로 변경했을 때 생긴 문제 해결하기"
date: 2023-07-14 15:07 +0900
---

## Mysql을 EC2 → RDS 정보로 변경했을 때 생긴 문제 해결하기

### 문제 의심 1

접속하려는 mysql 데이터베이스의 SSL설정의 기본값이 true인 경우, 접속하려는 클라이언트가 ssl로 접속하지 않고, ssl이 아닌 일반적인 연결로 접속하기 때문에 발생한다. mysql 서버쪽에서는 유효하지 않은 패킷이 넘어오는 것이기 때문에 오류를 낸다.

### 문제 해결 1

JDBC 연결하는 URL에 useSSL=false 를 추가한다.

```sql
show session status like "ssl_cipher";
```

![스크린샷 2023-07-11 오후 3.02.20.png](https://github.com/SWM-team-forever/dadamda-backend/assets/91049936/c2243ed4-b4f4-4e38-b46b-ff7c75aae72e)

TLS로 접속한 것을 알 수 있다.

SSL 접속이 아니면 value에는 빈 값이 나온다.

local mysql에 useSSL=false 옵션을 주고 연결했을 떄,

```sql
show session status like "ssl_cipher";
```

![스크린샷 2023-07-11 오후 3.06.41.png](https://github.com/SWM-team-forever/dadamda-backend/assets/91049936/29369302-36cb-40ef-bb25-f04e6c678be9)

빈 값이 나온다.

하지만 배포 설정 파일에는 해당 옵션이 추가되어 있다.

문제가 해결되지 않았다.

### 문제 의심 2

위의 에러 메세지는 상당히 많은 에러를 포괄하는 메세지이므로 맨 밑의 에러에 집중해야한다.

```bash
Caused by: org.hibernate.HibernateException: Access to DialectResolutionInfo cannot be null when 'hibernate.dialect' not set
```

### 문제 해결 2

```yaml
spring:
  profiles:
    active: dev, dev-secret
  jpa:
    database: mysql
    database-platform: org.hibernate.dialect.MySQL5InnoDBDialect
```

application.yml 파일에 해당 코드를 추가해주었다.

해당 하위 에러 메세지는 뜨지 않았지만 최상단 에러 메세지는 그대로 출력됐다.

문제가 해결되지 않았다.

### 문제 의심 3

혹시 EC2와 RDS 네트워크 문제 아닐까?

**RDS 인바운드 규칙**

![스크린샷 2023-07-11 오후 3.46.49.png](https://github.com/SWM-team-forever/dadamda-backend/assets/91049936/ef86d11a-ef8d-43ae-9e61-eb0cd7bdebf1)

**RDS 아웃바운드 규칙**

![스크린샷 2023-07-11 오후 3.49.53.png](https://github.com/SWM-team-forever/dadamda-backend/assets/91049936/1db16432-60b0-4fc2-96aa-9c8d2d412ba6)

### 문제 해결 3

**RDS 인바운드 규칙 변경**

![스크린샷 2023-07-11 오후 3.51.15.png](https://github.com/SWM-team-forever/dadamda-backend/assets/91049936/91557353-3bca-4b1b-aee5-89576844ba66)

**해결 완료**
