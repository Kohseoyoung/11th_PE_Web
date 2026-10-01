- 3계층 아키텍처 이외에 어떤 아키텍처들이 있을까?

    #### 1. 계층형 아키텍처 (Layered)
    
    - **구조**: Controller → Service → Repository (왼쪽에서 오른쪽으로만 의존)
    - **장점**: 단순하고 익숙하며 대부분의 프레임워크가 이 구조를 전제로 한다.
    - **단점**: Service가 DB 기술(JPA 등)에 직접 의존해서, 비즈니스 로직이 기술에 묶이기 쉽다. 규모가 커지면 Service가 비대해진다.
    - **적합**: 소~중규모 CRUD 프로젝트, 학습용
    
    #### 2. 헥사고날 아키텍처 (Ports & Adapters)
    
    - **구조**: 도메인(비즈니스 로직)을 중심에 두고, DB·웹·외부 API는 Port(인터페이스)와 Adapter(구현체)로 연결한다.
    - **핵심**: 의존성이 항상 바깥에서 안쪽(도메인)으로 향한다.
    - **장점**: DB나 프레임워크를 바꿔도 도메인 코드는 그대로고, 테스트가 쉽다.
    - **단점**: 인터페이스와 클래스가 늘어 초기 구조가 복잡하다.
    
    #### 3. 클린 아키텍처 (Clean Architecture)
    
    - **구조**: 동심원 형태로 Entity → Use Case → Interface Adapter → Frameworks & Drivers
    - **핵심**: 의존성 규칙. 안쪽 원은 바깥 원을 절대 모른다.
    - **관계**: 헥사고날과 목표가 같은 "도메인 보호" 계열이다.
    - **적합**: 비즈니스 로직이 복잡하고 오래 유지보수해야 하는 프로젝트
    
    #### 4. 어니언 아키텍처 (Onion)
    
    - 도메인 모델을 가장 안쪽에 두고 바깥으로 층을 쌓는 구조다. 클린, 헥사고날과 같은 계열이다.
    
    #### 5. MVC / MVP / MVVM (화면 패턴)
    
    | 패턴 | 구성 | 특징 |
    | --- | --- | --- |
    | MVC | Model - View - Controller | Spring MVC가 대표적이다. Controller가 요청을 받아 처리한다. |
    | MVP | Model - View - Presenter | Presenter가 View와 Model을 완전히 중개한다. |
    | MVVM | Model - View - ViewModel | 데이터 바인딩으로 View가 자동 갱신된다. (Android, Vue 등) |
    
    #### 6. 모놀리식 vs 마이크로서비스
    
    |  | 모놀리식 | 마이크로서비스 (MSA) |
    | --- | --- | --- |
    | 구조 | 하나의 애플리케이션 | 기능별 독립 서비스 여러 개 |
    | 배포 | 통째로 한 번에 | 서비스별 독립 배포 |
    | 장점 | 단순하고 개발/디버깅이 쉬움 | 확장성, 장애 격리, 기술 선택 자유 |
    | 단점 | 커질수록 수정/배포 부담 | 통신, 데이터 일관성, 운영 복잡도 |
    - **모듈러 모놀리스**: 배포는 하나지만 내부를 모듈별로 엄격히 분리한 절충안이다. MSA 전 단계로 많이 선택한다.
    
    #### 7. 이벤트 기반 아키텍처 (Event-Driven)
    
    - **구조**: 서비스끼리 직접 호출하지 않고 이벤트(메시지)를 발행하고, 관심 있는 쪽이 구독한다. (Kafka, RabbitMQ 등)
    - **장점**: 결합도가 낮고 비동기 처리가 가능하다.
    - **단점**: 흐름 추적과 디버깅이 어렵고, 최종 일관성(eventual consistency)을 감수해야 한다.
    
    #### 8. CQRS
    
    - **핵심**: 쓰기(Command)와 읽기(Query) 모델을 분리한다.
    - **장점**: 읽기와 쓰기를 각각 최적화할 수 있어 조회 성능에 유리하다.
    - **단점**: 구조가 복잡해지므로 필요할 때만 도입한다. 이벤트 기반과 자주 함께 쓴다.
    
    #### 9. 서버리스 (Serverless)
    
    - **구조**: 서버 관리 없이 함수 단위(AWS Lambda 등)로 실행하고, 쓴 만큼만 과금한다.
    - **장점**: 운영 부담이 적고 자동 확장된다.
    - **단점**: 콜드 스타트, 벤더 종속, 장시간 작업에 부적합
    
    ---
    
    #### 한눈에 비교
    
    | 아키텍처 | 복잡도 | 주 목적 | 언제 쓰나 |
    | --- | --- | --- | --- |
    | 계층형 | 낮음 | 단순한 관심사 분리 | 일반 CRUD, 학습용 |
    | 헥사고날/클린/어니언 | 중~높음 | 도메인 보호, 테스트 용이 | 복잡한 비즈니스 로직 |
    | MSA | 높음 | 독립 배포, 확장성 | 대규모 서비스, 여러 팀 |
    | 이벤트 기반 | 높음 | 느슨한 결합, 비동기 | 서비스 간 연동이 많을 때 |
    | CQRS | 높음 | 읽기/쓰기 분리 최적화 | 조회 부하가 큰 서비스 |
    | 서버리스 | 중간 | 운영 부담 최소화 | 간헐적·이벤트성 작업 |
- SQL Injection을 포함한 대표적인 웹 보안 공격 기법

  #### 1. SQL Injection

    - **원리**: 사용자 입력이 SQL 쿼리에 문자열로 이어붙여져, 데이터가 아니라 명령어로 실행된다.
    - **예시**: 책 제목에 `' ); DROP TABLE book; --`를 넣으면 테이블 삭제 쿼리가 함께 실행될 수 있다.
    - **피해**: 데이터 유출·변조·삭제, 로그인 우회
    - **방어**
        - 파라미터 바인딩 사용: JdbcTemplate의 `?`, JPA의 `:param`, MyBatis의 `#{}`
        - DB 계정에 최소 권한만 부여
        - 에러 메시지를 사용자에게 그대로 노출하지 않기

    ```java
    // 위험: 문자열 이어붙이기
    "INSERT INTO book (title) VALUES ('" + title + "')"
    
    // 안전: 바인딩
    jdbcTemplate.update("INSERT INTO book (title) VALUES (?)", title);
    ```

  #### 2. XSS (Cross-Site Scripting)

    - **원리**: 게시글이나 댓글에 악성 스크립트를 넣어두면, 그 페이지를 보는 다른 사용자의 브라우저에서 실행된다.
    - **유형**
        - Stored: 스크립트가 DB에 저장되어 계속 실행됨
        - Reflected: URL 파라미터가 응답에 그대로 반영됨
        - DOM-based: 클라이언트 JS가 입력을 안전하지 않게 처리함
    - **피해**: 쿠키·세션 탈취, 화면 조작, 피싱
    - **방어**
        - 출력 시 HTML 이스케이프 (템플릿 엔진의 자동 이스케이프 활용)
        - CSP(Content-Security-Policy) 헤더 적용
        - 쿠키에 `HttpOnly` 설정

  #### 3. CSRF (Cross-Site Request Forgery)

    - **원리**: 로그인된 사용자가 악성 페이지를 방문하면, 브라우저가 쿠키를 자동으로 붙여 사용자 모르게 요청(송금, 비밀번호 변경 등)을 보낸다.
    - **XSS와의 차이**: XSS는 사용자 브라우저에서 스크립트를 실행하는 것이고, CSRF는 사용자 권한으로 요청만 위조하는 것이다.
    - **방어**
        - CSRF 토큰 검증
        - 쿠키에 `SameSite=Lax/Strict` 설정
        - 중요 작업 시 재인증

  #### 4. 인증/세션 공격

    - **무차별 대입 / 크리덴셜 스터핑**: 비밀번호를 반복 시도하거나 유출된 계정 정보를 재사용한다.
    - **세션 하이재킹 / 세션 고정**: 세션 ID를 탈취하거나 미리 심어둔다.
    - **방어**
        - 비밀번호는 BCrypt 등으로 해싱해서 저장
        - 로그인 시도 횟수 제한, MFA 도입
        - 로그인 성공 시 세션 ID 재발급

  #### 5. 접근 제어 취약점 (Broken Access Control, IDOR)

    - **원리**: `/books/1`을 `/books/2`로 바꾸기만 해도 남의 데이터가 보이는 식으로, 서버가 "이 사용자가 이 데이터에 접근해도 되는지"를 확인하지 않는 경우다.
    - **방어**: 모든 요청마다 서버에서 소유권과 권한을 검사한다. 프론트 화면에서 버튼을 숨기는 것만으로는 부족하다.

  #### 6. SSRF (Server-Side Request Forgery)

    - **원리**: 서버가 사용자가 준 URL로 요청을 보내는 기능(이미지 가져오기, 웹훅 등)을 악용해, 외부에서 접근할 수 없는 내부 서비스나 클라우드 메타데이터에 접근한다.
    - **방어**: 허용 도메인 목록(allowlist), 내부 IP 대역 차단, 리다이렉트 제한

  #### 7. 파일 업로드 취약점

    - **원리**: 실행 가능한 파일(웹셸 등)을 이미지인 척 업로드하고 서버에서 실행되게 만든다.
    - **방어**: 확장자·MIME 타입 검증, 파일명 재생성, 업로드 경로의 실행 권한 제거, 웹 루트 밖에 저장

  #### 8. 커맨드 인젝션

    - **원리**: 입력값이 OS 명령어에 그대로 들어가 임의 명령이 실행된다. SQL 인젝션의 OS 버전이다.
    - **방어**: 셸 명령 호출 자체를 피하고, 불가피하면 인자를 분리해 전달하며 입력을 검증한다.

  #### 9. DoS / DDoS

    - **원리**: 대량의 요청이나 무거운 요청으로 서버 자원을 고갈시켜 정상 서비스를 막는다.
    - **방어**: Rate limiting, CDN, WAF, 오토스케일링, 무거운 쿼리에 타임아웃 설정

  #### 10. MITM (중간자 공격)

    - **원리**: 통신 경로에서 패킷을 가로채거나 변조한다. (공용 와이파이 등)
    - **방어**: 전 구간 HTTPS, HSTS 헤더, 인증서 검증

    ---

  #### 공통 방어 원칙

    1. **입력은 절대 신뢰하지 않기**: 검증은 서버에서 한다. 프론트 검증은 우회할 수 있다.
    2. **데이터와 명령을 분리하기**: SQL 바인딩, 출력 이스케이프가 여기에 해당한다.
    3. **최소 권한 원칙**: DB 계정, 서버, 사용자 권한 모두 필요한 만큼만 준다.
    4. **에러 메시지 최소화**: 스택 트레이스나 DB 구조가 노출되지 않게 한다.
    5. **의존성 최신화**: 알려진 취약점이 있는 라이브러리를 방치하지 않는다.
    6. **로깅과 모니터링**: 이상 징후를 빨리 발견할 수 있게 한다.
- 커넥션 풀(Connection Pool)이란?

  #### 1. 왜 필요한가

  DB에 연결하는 건 생각보다 비싼 작업이다.

  **커넥션 풀이 없을 때 (요청마다 새로 연결)**

    1. TCP 연결 수립 (3-way handshake)
    2. DB 인증 (아이디/비밀번호 확인)
    3. 세션 초기화
    4. 쿼리 실행
    5. 연결 종료

  쿼리 실행은 몇 ms인데 연결 생성·종료에 훨씬 더 오래 걸리는 경우가 많고, 요청이 몰리면 DB가 감당 못 할 만큼 연결이 생긴다.

  **커넥션 풀이 있을 때**

    1. 앱 시작 시 커넥션을 미리 N개 만들어 풀에 보관
    2. 요청이 오면 풀에서 하나 **빌려서(borrow)** 사용
    3. 쿼리가 끝나면 종료하지 않고 풀에 **반납(return)**
    4. 다음 요청이 그 커넥션을 **재사용**

    ---

  #### 2. 비유

  | 커넥션 풀 없음 | 커넥션 풀 있음 |
      | --- | --- |
  | 손님이 올 때마다 우산을 새로 사고 버림 | 우산꽂이에 우산 10개를 두고 빌렸다 반납 |
    
  ---

  #### 3. 동작 흐름

    ```
    [요청1] ─┐
    [요청2] ─┼→ [ 커넥션 풀 (C1, C2, C3 ... C10) ] ⇄ [ DB ]
    [요청3] ─┘      빌리고 → 쓰고 → 반납
    ```

    - 풀에 남은 커넥션이 있으면 → 즉시 빌려준다.
    - 전부 사용 중이면 → 반납될 때까지 **대기**한다. (대기 시간 초과 시 예외 발생)

    ---

  #### 4. 장점과 단점

  | 장점 | 단점 / 주의점 |
      | --- | --- |
  | 연결 생성 비용 절감으로 응답 속도 향상 | 풀 크기 설정이 부적절하면 오히려 성능 저하 |
  | DB 동시 연결 수를 제한해서 DB 보호 | 커넥션을 반납하지 않으면 풀이 고갈됨 (커넥션 누수) |
  | 자원 재사용으로 안정적인 운영 | 풀이 너무 크면 DB 메모리·연결 한도 초과 |
    
  ---

  #### 5. Spring Boot에서는: HikariCP

  Spring Boot 2.0부터 **HikariCP가 기본 커넥션 풀**이다. 별도 설정 없이 `spring-boot-starter-jdbc`(또는 JPA)를 쓰면 자동으로 적용된다.

  실행 로그의 이 부분이 바로 그것이다.

    ```
    com.zaxxer.hikari.HikariDataSource : HikariPool-1 - Starting...
    ```

  #### 주요 설정 (application.yml)

    ```yaml
    spring:
      datasource:
        hikari:
          maximum-pool-size: 10        # 풀의 최대 커넥션 수 (기본 10)
          minimum-idle: 10             # 유휴 상태로 유지할 최소 커넥션 수
          connection-timeout: 30000    # 커넥션을 얻기 위해 기다리는 최대 시간(ms), 기본 30초
          idle-timeout: 600000         # 유휴 커넥션이 풀에 남아있는 최대 시간(ms)
          max-lifetime: 1800000        # 커넥션의 최대 수명(ms), 기본 30분
    ```

  | 설정 | 의미 |
      | --- | --- |
  | `maximum-pool-size` | 동시에 사용할 수 있는 최대 커넥션 수 |
  | `minimum-idle` | 놀고 있어도 유지할 최소 커넥션 수 |
  | `connection-timeout` | 풀이 꽉 찼을 때 대기하는 최대 시간. 넘으면 예외 |
  | `idle-timeout` | 안 쓰는 커넥션을 정리하는 기준 시간 |
  | `max-lifetime` | 커넥션 최대 수명. DB의 `wait_timeout`보다 짧게 설정 |
    
  ---

  #### 6. 풀 크기는 어떻게 정할까

    - **크다고 무조건 좋지 않다.** 커넥션이 많으면 DB의 컨텍스트 스위칭 비용이 늘고 오히려 느려질 수 있다.
    - 처음에는 기본값(10)으로 시작하고, 부하 테스트와 모니터링으로 조정하는 게 일반적이다.
    - 참고 공식(HikariCP 문서): `풀 크기 = (CPU 코어 수 × 2) + 유효 디스크 수` 정도를 출발점으로 삼는다.
    - 앱 인스턴스가 여러 대라면 **(인스턴스 수 × 풀 크기)**가 DB의 `max_connections`를 넘지 않게 해야 한다.

    ---

  #### 7. 자주 만나는 문제

  | 증상 | 원인 | 해결 |
      | --- | --- | --- |
  | `Connection is not available, request timed out after 30000ms` | 풀의 커넥션이 전부 사용 중 | 풀 크기 조정, 느린 쿼리 개선, 커넥션 누수 확인 |
  | 커넥션 누수 | 커넥션을 빌리고 반납하지 않음 | try-with-resources 사용, 트랜잭션 범위 점검 |
  | `Communications link failure` (오래 후) | DB가 유휴 커넥션을 먼저 끊음 | `max-lifetime`을 DB `wait_timeout`보다 짧게 |
  | 앱 시작 시 `Access denied` | 계정/DB명 설정 오류 | datasource url, username, password 확인 (DB 이름 오타로 겪은 그 오류) |
    
  ---

  #### 8. 핵심 요약

    1. 커넥션 풀은 **DB 연결을 재사용**해서 성능과 안정성을 높이는 기법이다.
    2. Spring Boot의 기본 구현체는 **HikariCP**다.
    3. 풀 크기는 **작게 시작해서 측정하며 조정**한다.
    4. 커넥션은 **반드시 반납**되어야 한다. (`JdbcTemplate`, JPA는 자동으로 반납해 준다.)
- Raw SQL vs ORM

  #### 1. 같은 기능, 코드로 비교

  **Raw SQL (JdbcTemplate)**

    ```java
    String sql = "INSERT INTO book (category_id, title, description, is_available) VALUES (?, ?, ?, true)";
    jdbcTemplate.update(sql, categoryId, title, description);
    
    String sql2 = "SELECT * FROM book WHERE title = ?";
    List<Map<String, Object>> books = jdbcTemplate.queryForList(sql2, title);
    ```

  **ORM (JPA)**

    ```java
    @Entity
    public class Book {
        @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
        private Long id;
        private String title;
        private String description;
        private boolean isAvailable = true;
    }
    
    public interface BookRepository extends JpaRepository<Book, Long> {
        List<Book> findByTitle(String title);
    }
    
    bookRepository.save(book);              // INSERT
    bookRepository.findByTitle(title);      // SELECT ... WHERE title = ?
    ```

  SQL을 한 줄도 안 썼지만, 실행될 때는 JPA가 SQL을 대신 만들어서 날린다.
    
  ---

  #### 2. 핵심 비교

  | 항목 | Raw SQL | ORM (JPA) |
      | --- | --- | --- |
  | SQL 작성 | 직접 작성 | 자동 생성 (복잡한 건 JPQL, QueryDSL) |
  | 생산성 | 단순 CRUD도 SQL을 매번 작성 | CRUD 코드가 거의 필요 없음 |
  | 성능 제어 | 쿼리를 세밀하게 튜닝 가능 | 자동 생성 쿼리는 예측이 어려울 수 있음 |
  | 학습 곡선 | SQL만 알면 시작 가능 | 영속성 컨텍스트, 연관관계 등 개념이 많음 |
  | DB 종속성 | SQL 문법이 DB에 묶임 | 방언(Dialect)으로 DB 교체가 비교적 쉬움 |
  | 유지보수 | 테이블 변경 시 SQL 문자열을 일일이 수정 | 엔티티만 수정 |
  | 컴파일 타임 검증 | SQL은 문자열이라 런타임에야 오류 발견 | 메서드명/엔티티 기반이라 일부 검증 가능 |
  | 복잡한 쿼리 | 자유롭게 작성 | 통계·다중 조인은 오히려 까다로움 |
    
  ---

  #### 3. 각각의 장단점

  #### Raw SQL

    - **장점**
        - 실행되는 SQL이 눈에 보여서 동작이 명확하다.
        - 복잡한 조인, 집계, DB 특화 기능을 자유롭게 쓸 수 있다.
        - 성능 튜닝이 쉽다.
        - 배우기 직관적이다.
    - **단점**
        - 반복적인 보일러플레이트가 많다. (컬럼 하나 추가하면 여러 SQL 수정)
        - 결과를 객체로 매핑하는 코드를 직접 짜야 한다.
        - 오타가 런타임에 드러난다.

  #### ORM

    - **장점**
        - 생산성이 높고 코드가 간결하다.
        - 객체지향적으로 도메인을 설계할 수 있다.
        - 변경 감지(Dirty Checking), 지연 로딩, 캐시 등 편의 기능이 많다.
    - **단점**
        - **N+1 문제** 등 자동 생성 쿼리로 인한 성능 함정이 있다.
        - 내부 동작을 모르면 디버깅이 어렵다.
        - 복잡한 통계 쿼리는 표현이 어려워 결국 네이티브 쿼리를 써야 한다.
        - 학습 비용이 크다.

    ---

  #### 4. 알아둘 개념: N+1 문제

    - 책 목록 1번 조회(1번) → 각 책의 카테고리를 조회하려고 책마다 쿼리가 추가로 나간다(N번).
    - 책이 100권이면 쿼리가 101번 실행된다.
    - 해결: `fetch join`, `@EntityGraph`, batch size 설정

    ---

  #### 5. 그 사이: SQL Mapper (MyBatis)

  | 구분 | 특징 | 대표 |
      | --- | --- | --- |
  | JDBC / JdbcTemplate | SQL 직접, 매핑 직접 | Spring JDBC |
  | SQL Mapper | SQL은 직접 쓰되 결과 매핑은 자동 | MyBatis |
  | ORM | SQL도 자동 생성, 객체 중심 | JPA/Hibernate |

  왼쪽일수록 제어권이 크고 편의성이 낮으며, 오른쪽일수록 편의성이 높고 추상화 수준이 높다.
    
  ---

  #### 6. 보안 관점 (SQL Injection)

    - Raw SQL: **문자열 이어붙이기 금지**, 반드시 `?` 바인딩을 써야 한다.
    - ORM / MyBatis `#{}`: 내부적으로 바인딩을 쓰므로 기본적으로 안전하다.
    - 단, 네이티브 쿼리를 문자열로 조립하거나 MyBatis `${}`를 쓰면 ORM에서도 위험하다.

    ---

  #### 7. 언제 무엇을 쓸까

  | 상황 | 추천 |
      | --- | --- |
  | CRUD 중심, 빠른 개발, 도메인 모델이 중요 | ORM (JPA) |
  | 복잡한 통계/리포트 쿼리, 레거시 DB, 쿼리 튜닝이 중요 | Raw SQL / MyBatis |
  | 학습 초기, SQL 동작 원리를 익히고 싶을 때 | Raw SQL |
  | 실무에서 흔한 조합 | JPA + 복잡한 조회는 QueryDSL 또는 네이티브 쿼리 |
    
  ---

  #### 8. 핵심 요약

    1. **Raw SQL은 제어권, ORM은 생산성**을 준다.
    2. 정답은 없고, 프로젝트 성격에 따라 **혼합해서** 쓰는 경우가 많다.
    3. ORM을 쓰더라도 **SQL을 이해해야** N+1 같은 문제를 잡을 수 있다.
    4. JdbcTemplate로 SQL을 직접 다루는 경험은, 나중에 JPA를 쓸 때 내부에서 어떤 쿼리가 나가는지 이해하는 기반이 된다.
- INSERT 외에 데이터를 다루는 핵심 SQL 문법

  #### 1. SELECT (조회)

  #### 기본형

    ```sql
    SELECT 컬럼1, 컬럼2 FROM 테이블;
    SELECT * FROM book;                          -- 전체 컬럼
    SELECT title, description FROM book;         -- 일부 컬럼만
    ```

  #### 조건 검색: WHERE

    ```sql
    SELECT * FROM book WHERE category_id = 1;
    SELECT * FROM book WHERE is_available = true AND category_id = 2;
    ```

  | 연산자 | 의미 | 예시 |
      | --- | --- | --- |
  | `=`, `<>` (또는 `!=`) | 같다, 다르다 | `category_id = 1` |
  | `>`, `<`, `>=`, `<=` | 대소 비교 | `book_id >= 10` |
  | `AND`, `OR`, `NOT` | 논리 연산 | `A AND B` |
  | `IN` | 목록 중 하나 | `category_id IN (1, 2, 3)` |
  | `BETWEEN` | 범위 | `book_id BETWEEN 1 AND 10` |
  | `LIKE` | 패턴 검색 | `title LIKE '%해리%'` |
  | `IS NULL` / `IS NOT NULL` | NULL 여부 | `description IS NULL` |

  > **주의**: NULL은 `= NULL`로 비교할 수 없고 반드시 `IS NULL`을 써야 한다.
  >

  #### LIKE 패턴

  | 패턴 | 의미 |
      | --- | --- |
  | `'해리%'` | "해리"로 시작 |
  | `'%포터'` | "포터"로 끝남 |
  | `'%리포%'` | "리포"를 포함 |
  | `'해_'` | "해" + 아무 한 글자 |

  #### 정렬: ORDER BY

    ```sql
    SELECT * FROM book ORDER BY title ASC;        -- 오름차순 (기본값)
    SELECT * FROM book ORDER BY book_id DESC;     -- 내림차순
    ```

  #### 개수 제한: LIMIT / OFFSET

    ```sql
    SELECT * FROM book ORDER BY book_id DESC LIMIT 10;            -- 상위 10개
    SELECT * FROM book ORDER BY book_id DESC LIMIT 10 OFFSET 20;  -- 21번째부터 10개 (페이징)
    ```

  #### 중복 제거: DISTINCT

    ```sql
    SELECT DISTINCT category_id FROM book;
    ```
    
  ---

  #### 2. UPDATE (수정)

    ```sql
    UPDATE book
    SET title = '새 제목', description = '새 설명'
    WHERE book_id = 1;
    ```

    - `SET`에 바꿀 컬럼을 쉼표로 나열한다.
    - **`WHERE`를 빼먹으면 테이블의 모든 행이 수정된다.** 가장 흔한 사고다.

  #### 예시: 대여 처리

    ```sql
    UPDATE book SET is_available = false WHERE book_id = 1;
    ```
    
  ---

  #### 3. DELETE (삭제)

    ```sql
    DELETE FROM book WHERE book_id = 1;
    ```

    - **`WHERE`를 빼먹으면 모든 행이 삭제된다.**
    - 조건 없이 전부 지우는 다른 방법들

  | 명령 | 특징 |
      | --- | --- |
  | `DELETE FROM book;` | 행 단위 삭제, 롤백 가능, 느림 |
  | `TRUNCATE TABLE book;` | 전체 비우기, 빠름, AUTO_INCREMENT 초기화, 롤백 어려움 |
  | `DROP TABLE book;` | 테이블 자체를 삭제 (구조까지 사라짐) |

  > 실무에서는 데이터를 진짜 지우지 않고 `is_deleted = true`로 표시하는 소프트 삭제(Soft Delete)도 많이 쓴다.
  >
    
  ---

  #### 4. 집계 함수와 GROUP BY

  #### 집계 함수

  | 함수 | 의미 |
      | --- | --- |
  | `COUNT()` | 개수 |
  | `SUM()` | 합계 |
  | `AVG()` | 평균 |
  | `MAX()` / `MIN()` | 최댓값 / 최솟값 |

    ```sql
    SELECT COUNT(*) FROM book;                                  -- 전체 책 수
    SELECT COUNT(*) FROM book WHERE is_available = true;        -- 대여 가능한 책 수
    ```

  #### GROUP BY (그룹별 집계)

    ```sql
    SELECT category_id, COUNT(*) AS book_count
    FROM book
    GROUP BY category_id;
    ```

  카테고리별 책 권수를 구하는 쿼리다.

  #### HAVING (그룹에 대한 조건)

    ```sql
    SELECT category_id, COUNT(*) AS book_count
    FROM book
    GROUP BY category_id
    HAVING COUNT(*) >= 5;
    ```

  | 구분 | WHERE | HAVING |
      | --- | --- | --- |
  | 적용 시점 | 그룹핑 **전** 행 필터 | 그룹핑 **후** 그룹 필터 |
  | 집계 함수 사용 | 불가 | 가능 |
    
  ---

  #### 5. JOIN (테이블 결합)

  `book`의 `category_id`가 `category` 테이블을 참조한다고 가정한다.

    ```sql
    SELECT b.title, c.name
    FROM book b
    INNER JOIN category c ON b.category_id = c.category_id;
    ```

  | JOIN 종류 | 결과 |
      | --- | --- |
  | `INNER JOIN` | 양쪽에 모두 매칭되는 행만 |
  | `LEFT JOIN` | 왼쪽 테이블 전체 + 매칭되는 오른쪽 (없으면 NULL) |
  | `RIGHT JOIN` | 오른쪽 테이블 전체 + 매칭되는 왼쪽 |
    - `b`, `c`는 테이블 별칭(alias)이다.
    - 카테고리가 없는 책까지 보고 싶으면 `LEFT JOIN`을 쓴다.

    ---

  #### 6. 서브쿼리

  쿼리 안에 쿼리를 넣는 방식이다.

    ```sql
    SELECT * FROM bookWHERE category_id IN (SELECT category_id FROM category WHERE name = '소설');
    ```
    
  ---

  #### 7. SQL 실행 순서 (중요)

  작성 순서와 실제 실행 순서가 다르다.

  | 작성 순서 | 실행 순서 |
      | --- | --- |
  | SELECT | 5. SELECT |
  | FROM | 1. FROM (JOIN 포함) |
  | WHERE | 2. WHERE |
  | GROUP BY | 3. GROUP BY |
  | HAVING | 4. HAVING |
  | ORDER BY | 6. ORDER BY |
  | LIMIT | 7. LIMIT |

  그래서 `WHERE`에서는 `SELECT`의 별칭(AS)을 쓸 수 없다.
    
  ---

  #### 8. Spring JdbcTemplate에서 쓰는 형태

    ```java
    // SELECT (여러 건)
    jdbcTemplate.queryForList("SELECT * FROM book WHERE category_id = ?", categoryId);
    
    // UPDATE
    jdbcTemplate.update("UPDATE book SET title = ? WHERE book_id = ?", title, bookId);
    
    // DELETE
    jdbcTemplate.update("DELETE FROM book WHERE book_id = ?", bookId);
    ```

    - INSERT, UPDATE, DELETE는 모두 `update()` 메서드를 쓴다.
    - SELECT는 `queryForList()` 등을 쓴다.
    - 값은 항상 `?` 바인딩으로 넘겨야 SQL Injection을 막을 수 있다.