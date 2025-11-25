# Docker
- Docker Compose 실행 명령어
```bash
$ docker compose up --build -d
```
- Docker Compose 종료
``` bash
$ docker compose down
# -v는 volumes까지 삭제
$ docker compose down -v
```

- Spring Boot Log 보기
```bash
$ docker compose logs -f backend #backend는 docker compose에 설정한 그룹(서비스) 이름, DNS 이름
```

- 사용 이유
 - Production과 동일한 환경으로 테스트를 진행하기 위함이고, 환경 일관성을 제공하여 내 컴퓨터에서는 잘 되는데 방지
 - Docker는 확장성(sacle out), 이식성, 격리성(각 앱의 라이브러리 버전 충돌), 마이크로 서비스에 각각의 서비스를 제공할 수 있는 점에서 향후 애플리케이션이 확장하는데 유연함을 제공하기 때문에 선택하였습니다.
 - Docker Compose 사용 이유는 DB 컨테이너를 build하고 Docker Network 연결..등 수동 작업을 docker compose로 자동화 해주고 service_healthy와 always를 설정하여 스프링부트의 DB 커넥션 문제로 인한 종료와  불필요한 재 반복을 막으면서 재 시작을 해줍니다. 이러한 편리한 기능 때문에 Docker Compose 기능을 사용하였습니다. 


# MariaDB
- Version: 10.11.14
- MariaDB 버전과 부가 정보 보기   
```sql
  SHOW VARIABLES LIKE '%VERSION%';
  ```

# Redis
- 향후 AWS 스케일아웃 환경에서 SSE는 정상 작동하지 않고 메일 서버 인증과 확장성을 위해 도입
- WSL Ubuntu 환경 설치 및 실행 가이드
```bash
# 설치
$ curl -fsSL https://packages.redis.io/gpg | sudo gpg --dearmor -o /usr/share/keyrings/redis-archive-keyring.gpg

$ echo "deb [signed-by=/usr/share/keyrings/redis-archive-keyring.gpg] https://packages.redis.io/deb $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/redis.list

$ sudo apt-get update
$ sudo apt-get install redis

# 실행
$ redis-server --daemonize yes

# Redis 연결
$ redis-cli

# 연결 테스트
127.0.0.1:6379> ping
PONG
```


# 트러블 슈팅
<details>
 <summary>리눅스 CSV Batch 작업</summary>
  - CSV 파일 인코딩 확인 후 변환
 
  ```bash
 #파일이 UTF-8로 인코딩 되어 있는지 확인
 file -i recipe_data_241226.csv

 #시스템 인코딩 변경
 LANG="ko_KR.UTF-8"

#윈도우에서는 CSV파일을 CRLF(\r\n) 줄 바꿈 사용, 리눅스는 LF(\n)만 사용하기에 줄바꿈 변환
 sudo apt install dos2unix
 dos2unix recipe_data_241226.csv

 #권한 변경
 chmod +r recipe_data_241226.csv
  ```


 - 컴포즈로 작성된 백엔드 로그 결과
 ```bash
 studio-recipe  | Caused by: java.sql.SQLSyntaxErrorException: (conn=4) Data too long for column 'ckg_mtrl_cn' at row 1
studio-recipe  |        at org.mariadb.jdbc.export.ExceptionFactory.createException(ExceptionFactory.java:289) ~[mariadb-java-client-3.5.5.jar!/:na]
studio-recipe  |        at org.mariadb.jdbc.export.ExceptionFactory.create(ExceptionFactory.java:378) ~[mariadb-java-client-3.5.5.jar!/:na]
studio-recipe  |        at org.mariadb.jdbc.message.ClientMessage.readPacket(ClientMessage.java:187) ~[mariadb-java-client-3.5.5.jar!/:na]
studio-recipe  |        at org.mariadb.jdbc.client.impl.StandardClient.readPacket(StandardClient.java:1380) ~[mariadb-java-client-3.5.5.jar!/:na]
studio-recipe  |        at org.mariadb.jdbc.client.impl.StandardClient.readResults(StandardClient.java:1319) ~[mariadb-java-client-3.5.5.jar!/:na]
studio-recipe  |        at org.mariadb.jdbc.client.impl.StandardClient.readResponse(StandardClient.java:1238) ~[mariadb-java-client-3.5.5.jar!/:na]
studio-recipe  |        at org.mariadb.jdbc.client.impl.StandardClient.execute(StandardClient.java:1162) ~[mariadb-java-client-3.5.5.jar!/:na]
studio-recipe  |        at org.mariadb.jdbc.ClientPreparedStatement.executeInternal(ClientPreparedStatement.java:87) ~[mariadb-java-client-3.5.5.jar!/:na]
studio-recipe  |        at org.mariadb.jdbc.ClientPreparedStatement.executeLargeUpdate(ClientPreparedStatement.java:307) ~[mariadb-java-client-3.5.5.jar!/:na]
studio-recipe  |        at org.mariadb.jdbc.ClientPreparedStatement.executeUpdate(ClientPreparedStatement.java:284) ~[mariadb-java-client-3.5.5.jar!/:na]
studio-recipe  |        at com.zaxxer.hikari.pool.ProxyPreparedStatement.executeUpdate(ProxyPreparedStatement.java:61) ~[HikariCP-6.3.2.jar!/:na]
studio-recipe  |        at com.zaxxer.hikari.pool.HikariProxyPreparedStatement.executeUpdate(HikariProxyPreparedStatement.java) ~[HikariCP-6.3.2.jar!/:na]
studio-recipe  |        at org.hibernate.engine.jdbc.internal.ResultSetReturnImpl.executeUpdate(ResultSetReturnImpl.java:194) ~[hibernate-core-6.6.26.Final.jar!/:6.6.26.Final]
studio-recipe  |        ... 161 common frames omitted
studio-recipe  |
studio-recipe  | 2025-10-11T08:46:48.384Z  INFO 1 --- [nio-8080-exec-1] o.s.batch.core.step.AbstractStep         : Step: [recipeDataMigrationStep] executed in 405ms
studio-recipe  | 2025-10-11T08:46:48.405Z  INFO 1 --- [nio-8080-exec-1] o.s.b.c.l.s.TaskExecutorJobLauncher      : Job: [SimpleJob: [name=recipeDataMigrationJob]] completed with the following parameters: [{'jobId':'{value=1760172407885, type=class java.lang.String, identifying=true}','run.id':'{value=1760172407886, type=class java.lang.Long, identifying=true}'}] and the following status: [FAILED] in 444ms
studio-recipe  | 2025-10-11T08:46:48.405Z  INFO 1 --- [nio-8080-exec-1] c.recipe.controller.BatchJobController   : Batch Job 'recipeDataMigrationJob' 실행 완료. Status: FAILED
 ```

- [Data too long for column] 최대 길이 문제
  ```java
  @Lob // Lob으로 되어 있었지만 해당 컬럼의 크기 문제 때문에 TEXT 명시적으로 적용
    @Column(name = "ckg_mtrl_cn", columnDefinition = "TEXT") 
  ```
  
</details>

<details>
 <summary>MailService & Redis 도입 후 테스트 서버 오류</summary>

 ## 주요 에러들
 ```bash
 studio-recipe   | org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'authControllerImpl' defined in URL [jar:nested:/app.jar/!BOOT-INF/classes/!/com/recipe/controller/AuthControllerImpl.class]: Unsatisfied dependency expressed through constructor parameter 1: Error creating bean with name 'mailService': Injection of autowired dependencies failed
 
studio-recipe   |  Caused by: org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'mailService': Injection of autowired dependencies failed
 ```

## 해결
- 초반에는 env 파일에서 환경 변수를 읽지 못하는 점이였다.
  초반에는 같은 방법으로 하였지만 해결이 되지 않아 리셋을 하였지만 구문 오류가 있었던 것 같다.
```bash
#environment에 환경 변수 추가
====================================
  SPRING_MAIL_HOST: ${MAIL_HOST}
  SPRING_MAIL_PORT: ${MAIL_PORT}
  MAIL_USERNAME: ${MAIL_USERNAME}
  MAIL_PASSWORD: ${MAIL_PASSWORD}
====================================
          or
  env_file:
  ./.env # .env 파일의 모든 변수가 컨테이너로 주입된다.
```

- 정상 동작
```bash
# ID 찾기
$ curl -X POST \
-H "Content-Type: application/json" \
-d '{
"email" : "dbsghks34@naver.com"
}' http://localhost:8080/studio-recipe/auth/send-verification
인증 번호 성공적으로 발송되었습니다.%

$ curl -X POST \
> -H "Content-Type: application/json" \
> -d '{
quote> "email" : "dbsghks34@naver.com",
quote> "verificationCode" : 243414,
quote> "purpose" : "FIND_ID"
quote> }' http://localhost:8080/studio-recipe/auth/verify-code
{"message":"이메일 인증이 성공했습니다.","token":"28046c58-1aa4-42e4-b86d-a8e12c663c07"}%

$ curl -X POST \
-H "Content-Type: application/json" \
-d '{
"token" : "28046c58-1aa4-42e4-b86d-a8e12c663c07"
}' http://localhost:8080/studio-recipe/auth/find-id
springboot!%

# 잘못된 토큰 신청 -> 토큰 만료 / 비밀번호 바꾸기 정상 동작                                                               
```

- 그 외
   - 우분투 상에서 설치된 Redis와 포트 번호 충돌 -> Docker Compose에 레디스 서비스 포트 번호 변경 6378:6379
 
</details>

# 원룸 레시피 추천 시스템 기술 문서

## 목차
1. [시스템 아키텍처](#1-시스템-아키텍처)
2. [시스템 워크플로우](#2-시스템-워크플로우)
3. [추천 알고리즘 상세](#3-추천-알고리즘-상세)
4. [보안 구현](#4-보안-구현)
5. [데이터베이스 설계](#5-데이터베이스-설계)
6. [성능 최적화](#6-성능-최적화)
7. [배포 및 운영](#7-배포-및-운영)
8. [확장 가능성](#8-확장-가능성)

---

## 1. 시스템 아키텍처

### 1.1 전체 구조
```
┌─────────────────────────────────────────────────────────────┐
│                        Frontend Layer                        │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │   React      │  │  React       │  │   Axios      │      │
│  │   Router     │  │  Components  │  │   API        │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
└─────────────────────────────────────────────────────────────┘
                              ↕ HTTP/REST
┌─────────────────────────────────────────────────────────────┐
│                      Backend Layer                           │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │  Spring      │  │  Spring      │  │   Spring     │      │
│  │  Security    │  │  MVC         │  │   Batch      │      │
│  │  + JWT       │  │  Controllers │  │              │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
│                                                               │
│  ┌──────────────────────────────────────────────────────┐  │
│  │          추천 알고리즘 레이어                          │  │
│  │  ┌──────────────────┐  ┌──────────────────┐         │  │
│  │  │ RecipeRecommend  │  │ IngredientRecommend│       │  │
│  │  │ Algorithm        │  │ Algorithm         │        │  │
│  │  └──────────────────┘  └──────────────────┘         │  │
│  └──────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
                              ↕
┌─────────────────────────────────────────────────────────────┐
│                    Persistence Layer                         │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │   MariaDB    │  │    Redis     │  │   JPA/       │      │
│  │   10.11.14   │  │    Cache     │  │   Hibernate  │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
└─────────────────────────────────────────────────────────────┘
                              ↕
┌─────────────────────────────────────────────────────────────┐
│                  Infrastructure Layer                        │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │   Docker     │  │   Docker     │  │    Mail      │      │
│  │   Compose    │  │   Network    │  │   Service    │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
└─────────────────────────────────────────────────────────────┘
```

### 1.2 기술 스택

#### Frontend
- **React 19.1.1**: UI 컴포넌트 기반 개발
- **React Router DOM 7.9.4**: SPA 라우팅
- **Axios 1.12.2**: HTTP 클라이언트
- **React Icons 5.5.0**: 아이콘 라이브러리
- **Vite 7.1.3**: 빌드 도구 및 개발 서버

#### Backend
- **Spring Boot 3.5.5**: 애플리케이션 프레임워크
- **Spring Security 6.x**: 인증/인가
- **Spring Batch**: 대용량 CSV 데이터 마이그레이션
- **Spring Data JPA**: ORM
- **JWT (JJWT 0.11.5)**: 토큰 기반 인증
- **Java 17**: OpenJDK 17

#### Database & Cache
- **MariaDB 10.11.14**: 메인 데이터베이스
- **Redis 6-alpine**: 세션 관리 및 캐싱
  - 이메일 인증 코드 저장 (TTL: 5분)
  - 비밀번호 재설정 토큰 (TTL: 10분)

#### Infrastructure
- **Docker Compose**: 컨테이너 오케스트레이션
- **Docker Network**: 컨테이너 간 통신
- **Gmail SMTP**: 이메일 발송

---

## 2. 시스템 워크플로우

### 2.1 사용자 인증 플로우
```
┌─────────┐                                    ┌──────────┐
│  Client │                                    │  Backend │
└────┬────┘                                    └────┬─────┘
     │                                              │
     │  1. POST /auth/register                     │
     │  { id, pwd, email, birth, gender }          │
     ├────────────────────────────────────────────>│
     │                                              │
     │                          2. Password Encoding│
     │                          (BCrypt)            │
     │                                              │
     │  3. 201 Created                              │
     │<────────────────────────────────────────────┤
     │                                              │
     │  4. POST /auth/login                         │
     │  { id, password }                            │
     ├────────────────────────────────────────────>│
     │                                              │
     │              5. UserDetails 조회 및 검증     │
     │              6. JWT Token 생성               │
     │                 - Access Token (TTL 설정)   │
     │                 - Refresh Token (TTL 설정)  │
     │                                              │
     │  7. 200 OK + Tokens                          │
     │  { accessToken, refreshToken, expiresIn }    │
     │<────────────────────────────────────────────┤
     │                                              │
     │  8. 이후 모든 요청에 Authorization Header   │
     │  Authorization: Bearer {accessToken}         │
     ├────────────────────────────────────────────>│
     │                                              │
     │         9. JwtAuthenticationFilter 검증      │
     │         10. SecurityContext 설정             │
```

### 2.2 이메일 인증 플로우 (아이디/비밀번호 찾기)
```
┌─────────┐      ┌──────────┐      ┌───────┐      ┌────────┐
│  Client │      │  Backend │      │ Redis │      │  SMTP  │
└────┬────┘      └────┬─────┘      └───┬───┘      └───┬────┘
     │                │                 │               │
     │  1. POST /auth/send-verification│               │
     │  { email }     │                 │               │
     ├───────────────>│                 │               │
     │                │                 │               │
     │         2. 6자리 숫자 코드 생성  │               │
     │         (SecureRandom)           │               │
     │                │                 │               │
     │                │  3. SET key     │               │
     │                │  TTL: 300초     │               │
     │                ├────────────────>│               │
     │                │                 │               │
     │                │  4. SMTP 발송   │               │
     │                │  (Thymeleaf 템플릿)            │
     │                ├───────────────────────────────>│
     │                │                 │               │
     │  5. 200 OK     │                 │               │
     │<───────────────┤                 │               │
     │                │                 │               │
     │  6. POST /auth/verify-code       │               │
     │  { email, code, purpose }        │               │
     ├───────────────>│                 │               │
     │                │                 │               │
     │                │  7. GET key     │               │
     │                ├────────────────>│               │
     │                │  8. value       │               │
     │                │<────────────────┤               │
     │                │                 │               │
     │         9. 코드 검증              │               │
     │         10. 일회용 토큰 생성 (UUID)│              │
     │         11. Redis 저장 (TTL: 600초)              │
     │                │                 │               │
     │  12. 200 OK    │                 │               │
     │  { token }     │                 │               │
     │<───────────────┤                 │               │
```

### 2.3 레시피 추천 워크플로우
```
┌─────────┐                    ┌──────────────┐
│  Client │                    │   Backend    │
└────┬────┘                    └──────┬───────┘
     │                                │
     │  1. GET /api/recommend/{userId}│
     ├───────────────────────────────>│
     │                                │
     │             2. UserReferences 조회
     │             (사용자 행동 이력)  │
     │                                │
     │             3. RecipeScoreCalculator 실행
     │             ┌──────────────────┐
     │             │ - VIEW: 0.5점    │
     │             │ - LIKE: 3.0점    │
     │             └──────────────────┘
     │                                │
     │             4. 레시피별 점수 집계│
     │             5. 점수 내림차순 정렬│
     │             6. 상위 10개 선택   │
     │                                │
     │  7. 200 OK                     │
     │  [ {recipe, score}, ... ]      │
     │<───────────────────────────────┤
```

### 2.4 재료 기반 추천 워크플로우
```
┌─────────┐                          ┌──────────────┐
│  Client │                          │   Backend    │
└────┬────┘                          └──────┬───────┘
     │                                      │
     │  1. POST /api/recommend/ingredient   │
     │  ["양파", "달걀", "소금"]            │
     ├─────────────────────────────────────>│
     │                                      │
     │               2. 전체 레시피 조회    │
     │                                      │
     │               3. 각 레시피 재료 파싱 │
     │               (split by [,\n])       │
     │                                      │
     │               4. 재료 매칭 계산      │
     │               ┌─────────────────────┐
     │               │ matchedCount /      │
     │               │ totalIngredients    │
     │               │ = ingredientMatchRate│
     │               └─────────────────────┘
     │                                      │
     │               5. 최종 점수 계산      │
     │               ┌─────────────────────┐
     │               │ 재료일치율 × 0.7 +  │
     │               │ 인기도 × 0.3        │
     │               └─────────────────────┘
     │                                      │
     │               6. 상위 10개 선택      │
     │                                      │
     │  7. 200 OK                           │
     │  [ {recipe, score}, ... ]            │
     │<─────────────────────────────────────┤
```

### 2.5 CSV 배치 마이그레이션 워크플로우
```
┌──────────┐                      ┌─────────────┐
│  Client  │                      │   Backend   │
└────┬─────┘                      └──────┬──────┘
     │                                   │
     │  1. GET /batch/run-recipe-csv     │
     ├──────────────────────────────────>│
     │                                   │
     │              2. Spring Batch Job 실행
     │              ┌──────────────────────┐
     │              │ RecipeDataMigrationJob│
     │              └──────────────────────┘
     │                                   │
     │              3. FlatFileItemReader
     │              - CSV 파일 읽기      │
     │              - Chunk Size: 1000   │
     │              - Encoding: UTF-8    │
     │                                   │
     │              4. ItemProcessor     │
     │              - 재료 데이터 파싱   │
     │              - 공백 제거          │
     │              - "|" → "/"          │
     │              - 날짜 변환          │
     │                                   │
     │              5. JpaItemWriter     │
     │              - Bulk Insert        │
     │              - Chunk 단위 커밋    │
```

---

## 3. 추천 알고리즘 상세

### 3.1 행동 기반 협업 필터링

#### 알고리즘 개요
사용자의 과거 행동(조회, 좋아요)을 기반으로 개인화된 레시피를 추천합니다.

#### 점수 계산 공식
```java
// 행동별 가중치
VIEW_WEIGHT = 0.5
LIKE_WEIGHT = 3.0

// 레시피별 최종 점수
recipeScore = Σ(actionWeight)

// 예시:
// - 레시피 A를 3번 조회: 0.5 × 3 = 1.5점
// - 레시피 A를 1번 좋아요: 3.0 × 1 = 3.0점
// - 최종 점수: 4.5점
```

#### 추천 프로세스

1. **데이터 수집**: UserReferences 테이블에서 사용자 행동 이력 조회
2. **점수 계산**: RecipeScoreCalculator로 레시피별 점수 집계
3. **정렬 및 선택**: 점수 내림차순 정렬 후 상위 10개 선택
4. **기본 추천**: 행동 이력이 없는 경우 인기 레시피(좋아요 수 기준) 반환

#### 기술적 특징

- **메모리 효율**: HashMap을 사용한 O(n) 복잡도
- **확장성**: 가중치 조정으로 행동 중요도 변경 가능
- **콜드 스타트 해결**: 신규 사용자에게 인기 레시피 제공

---

### 3.2 재료 기반 콘텐츠 필터링

#### 알고리즘 개요
사용자가 보유한 재료를 기반으로 만들 수 있는 레시피를 추천합니다.

#### 점수 계산 공식
```java
// 1. 재료 일치율 계산
ingredientMatchRate = matchedCount / totalIngredients

// 2. 인기도 정규화
popularityScore = rcmmCnt / 1000.0

// 3. 최종 점수 (가중 평균)
finalScore = ingredientMatchRate × 0.7 + popularityScore × 0.3

// 예시:
// - 레시피 재료: ["양파", "달걀", "소금", "후추"] (4개)
// - 사용자 재료: ["양파", "달걀", "소금"] (3개)
// - 일치 개수: 3개
// - 재료 일치율: 3/4 = 0.75
// - 좋아요 수: 500
// - 인기도: 500/1000 = 0.5
// - 최종 점수: 0.75 × 0.7 + 0.5 × 0.3 = 0.675
```

#### 재료 파싱 로직
```java
// CSV 데이터 정제
String ingredients = rawIngredients.trim()
    .replaceAll("^\\[.+?]\\s*", "")     // 대괄호 제거
    .replaceAll("\\s(\\d+)\\s", "_$1")  // 숫자 보호
    .replaceAll("\\s*\\|\\s", "/")      // 구분자 통일
    .replaceAll(" ", "");               // 공백 제거

// 재료 목록 생성
List<String> recipeIngredients = 
    Arrays.stream(ingredients.split("[,\\n]"))
        .map(String::trim)
        .filter(s -> !s.isEmpty())
        .collect(Collectors.toList());
```

#### 추천 프로세스

1. **재료 매칭**: 사용자 재료와 레시피 재료 비교 (contains 사용)
2. **일치율 계산**: 일치하는 재료 수 / 전체 레시피 재료 수
3. **하이브리드 스코어링**: 재료 일치율(70%) + 인기도(30%)
4. **정렬 및 선택**: 최종 점수 내림차순으로 상위 10개 반환

#### 기술적 특징

- **정밀 추천**: 재료 일치율에 높은 가중치(0.7) 부여
- **인기도 보정**: 품질 좋은 레시피 우선 추천(가중치 0.3)
- **유연한 매칭**: 부분 문자열 매칭으로 재료 변형 허용

---

## 4. 보안 구현

### 4.1 인증/인가 체계

#### JWT 기반 인증
```java
// Access Token: 짧은 유효기간 (환경변수 설정)
// Refresh Token: 긴 유효기간 (환경변수 설정)

// 토큰 구조
{
  "sub": "userId",           // DB PK
  "auth": "ROLE_GUEST",      // 권한
  "username": "loginId",     // 로그인 ID
  "iat": 1234567890,         // 발행 시간
  "exp": 1234571490          // 만료 시간
}
```

#### 보안 필터 체인
```
Request
   ↓
JwtAuthenticationFilter
   ├─ 토큰 추출 (Authorization Header)
   ├─ 토큰 검증 (서명, 만료시간)
   ├─ SecurityContext 설정
   └─ 다음 필터로 전달
   ↓
Controller
```

### 4.2 비밀번호 보안

- **암호화**: BCryptPasswordEncoder (강도: 기본값)
- **검증**: `passwordEncoder.matches(raw, encoded)`
- **정책**: 
  - 8~32자
  - 영문 대소문자, 숫자, 특수문자 필수 포함
  - 정규식 검증

### 4.3 이메일 인증 보안

- **인증 코드**: 6자리 숫자 (SecureRandom 사용)
- **유효 시간**: 5분 (Redis TTL)
- **일회용 토큰**: UUID 기반 (10분 유효)
- **용도 분리**: TokenPurpose enum (FIND_ID, RESET_PASSWORD)

---

## 5. 데이터베이스 설계

### 5.1 핵심 엔티티

#### User (사용자)
```sql
USER_ID (PK, AUTO_INCREMENT)
ID (UNIQUE, VARCHAR(50))
PWD (VARCHAR(255), BCrypt)
NAME (VARCHAR(100))
NICKNAME (UNIQUE, VARCHAR(50))
EMAIL (UNIQUE, VARCHAR(50))
BIRTH (DATE)
GENDER (ENUM: 'F', 'M')
ROLE (ENUM: 'ADMIN', 'GUEST')
CREATED_AT
MODIFIED_AT
```

#### Recipe (레시피)
```sql
RCP_SNO (PK, AUTO_INCREMENT)
RCP_TTL (VARCHAR)              -- 제목
CKG_NM (VARCHAR)               -- 요리명
INQ_CNT (INT)                  -- 조회수
RCMM_CNT (INT)                 -- 추천수
CKG_MTH_ACTO_NM (VARCHAR)      -- 조리방법
CKG_MTRL_ACTO_NM (VARCHAR)     -- 재료 대분류
CKG_KND_ACTO_NM (VARCHAR)      -- 음식 종류
CKG_MTRL_CN (TEXT)             -- 재료 내용
CKG_INBUN_NM (VARCHAR)         -- 인분
CKG_DODF_NM (VARCHAR)          -- 난이도
CKG_TIME_NM (VARCHAR)          -- 조리시간
FIRST_REG_DT (DATETIME)        -- 등록일
RCP_IMG_URL (VARCHAR)          -- 이미지 URL
```

#### UserReferences (사용자 행동)
```sql
PREFERENCE_ID (PK, AUTO_INCREMENT)
USER_ID (FK → USER)
RCP_SNO (FK → RECIPE)
PREFERENCE_TYPE (ENUM: 'VIEW', 'LIKE')
CREATED_AT
MODIFIED_AT
```

#### Like (좋아요)
```sql
LIKE_ID (PK, AUTO_INCREMENT)
USER_ID (FK → USER)
RCP_SNO (FK → RECIPE)
CREATED_AT
MODIFIED_AT
UNIQUE(USER_ID, RCP_SNO)       -- 중복 방지
```

### 5.2 인덱스 전략
```sql
-- 추천 알고리즘 성능 최적화
CREATE INDEX idx_user_references_user 
    ON USER_REFERENCE(USER_ID);
CREATE INDEX idx_user_references_recipe 
    ON USER_REFERENCE(RCP_SNO);

-- 정렬 성능 최적화
CREATE INDEX idx_recipe_rcmm_cnt 
    ON RECIPES(RCMM_CNT DESC);
CREATE INDEX idx_recipe_inq_cnt 
    ON RECIPES(INQ_CNT DESC);
```

---

## 6. 성능 최적화

### 6.1 캐싱 전략

**Redis 활용**:
- 이메일 인증 코드 (TTL: 5분)
- 비밀번호 재설정 토큰 (TTL: 10분)
- 향후 확장: 인기 레시피 캐싱, 세션 관리

### 6.2 배치 처리

**Spring Batch 활용**:
- Chunk 단위 처리: 1000건
- FlatFileItemReader: CSV 스트림 처리
- JpaItemWriter: Bulk Insert

### 6.3 쿼리 최적화

**JPA Fetch 전략**:
- `@ManyToOne(fetch = FetchType.LAZY)`: N+1 방지
- `@EntityGraph`: 필요 시 즉시 로딩

---

## 7. 배포 및 운영

### 7.1 Docker Compose 구성
```yaml
services:
  - mariadb: 데이터베이스 (포트 3306)
  - redis: 캐시 서버 (포트 6378 → 6379)
  - backend: Spring Boot (포트 8080)

volumes:
  - mariadb_data: 데이터 지속성
  - redis_data: AOF 지속성

healthcheck:
  - mariadb: mariadb-admin ping
  - 서비스 간 의존성 관리
```

### 7.2 환경 변수 관리
```properties
# Database
DB_ROOT_PASSWORD, DB_DATABASE_NAME
DB_USERNAME, DB_PASSWORD

# Mail (SMTP)
MAIL_HOST, MAIL_PORT
MAIL_USERNAME, MAIL_PASSWORD

# JWT
JWT_SECRET_KEY
JWT_ACCESS_TOKEN_VALIDITY_IN_SECONDS
JWT_REFRESH_TOKEN_VALIDITY_IN_SECONDS

# Frontend
FRONT_URL
```

---

## 8. 확장 가능성

### 8.1 알고리즘 확장

1. **협업 필터링 고도화**:
   - 사용자 간 유사도 계산 (Cosine Similarity)
   - Matrix Factorization 적용

2. **하이브리드 추천**:
   - 행동 기반 + 재료 기반 결합
   - 시간대, 계절 등 컨텍스트 반영

3. **딥러닝 도입**:
   - 레시피 이미지 기반 추천
   - 자연어 처리를 통한 재료 인식

### 8.2 시스템 확장

1. **마이크로서비스 분리**:
   - 추천 서비스 독립 배포
   - 배치 처리 서비스 분리

2. **캐싱 레이어 강화**:
   - Redis Cluster 구성
   - CDN 활용 (이미지, 정적 자원)

3. **검색 기능 고도화**:
   - Elasticsearch 도입
   - 전문 검색, 자동완성 개선

---

## 부록

### A. 프로젝트 구조
studio-recipe/

├── recipe/       # Backend (Spring Boot)

│   ├── src/main/

│   │   ├── java/com/recipe/

│   │   │   ├── algorithm/     # 추천 알고리즘

│   │   │   ├── config/        # 설정 (Security, JWT, Batch)

│   │   │   ├── controller/    # REST API

│   │   │   ├── domain/        # Entity, DTO

│   │   │   ├── repository/    # JPA Repository

│   │   │   └── service/       # 비즈니스 로직

│   │   └── resources/

│   │       ├── application-*.yml

│   │       └── data/          # CSV 데이터

│   ├── Dockerfile

│   └── docker-compose.yml

│
└── recommended-recipe/        # Frontend (React + Vite)

├── src/

│   ├── api/               # Axios 설정

│   ├── components/        # React 컴포넌트

│   ├── page/              # 페이지 컴포넌트

│   └── main.jsx

└── vite.config.js


### B. 주요 API 엔드포인트

#### 인증 관련
- `POST /auth/register` - 회원가입
- `POST /auth/login` - 로그인
- `POST /auth/send-verification` - 인증 코드 발송
- `POST /auth/verify-code` - 인증 코드 검증
- `POST /auth/find-id` - 아이디 찾기
- `POST /auth/reset-password` - 비밀번호 재설정

#### 레시피 관련
- `GET /recipes/{recipeId}` - 레시피 상세
- `GET /main-pages` - 메인 페이지 레시피 목록
- `POST /likes/{recipeId}` - 좋아요
- `DELETE /likes/{recipeId}` - 좋아요 취소
- `GET /likes` - 좋아요 히스토리

#### 추천 관련
- `GET /api/recommend/{userId}` - 행동 기반 추천
- `POST /api/recommend/ingredient` - 재료 기반 추천

####배치 관련
- GET /batch/run-recipe-csv - CSV 마이그레이션 실행
