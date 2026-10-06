- **JPA / Hibernate와 TypeORM** — ORM과 구현체·라이브러리의 역할은 어떻게 다른가?

  #### 1. ORM (Object-Relational Mapping)의 기본 개념

    - ORM은 객체 지향 프로그래밍의 객체(Object)와 관계형 DB의 테이블(Table)을 매핑하여, SQL을 직접 쓰는 대신 객체의 메서드로 데이터를 조작하게 해 주는 기술
    - 코드가 아니라 아이디어(패러다임)이므로 직접 설치하거나 실행할 수 없다.
        - `Book` 클래스 ↔ `book` 테이블
        - 필드 ↔ 컬럼
        - 객체 참조(`category`) ↔ 외래 키(`category_id`)

  #### 2. Java/Spring 생태계: 명세와 구현체의 분리

    - 자바 진영은 기술의 명세(Specification)와 이를 구체화한 구현체(Implementation)를 분리해서 설계하는 철학을 가진다.

    ```
    ORM (개념)
     └─ JPA (자바 표준 명세, 인터페이스)
         └─ Hibernate (JPA 구현체, 실제 SQL 생성·실행)
             └─ Spring Data JPA (JpaRepository 등 편의 기능)
    ```

  **1) JPA (Java Persistence API)**

    - **역할:** 자바 ORM 기술의 표준 명세서(인터페이스)이다.
    - **특징:** "ORM 라이브러리는 이렇게 동작해야 한다"는 규칙과 API 형태만 정의한다. 데이터를 저장하거나 쿼리를 만드는 코드는 내부에 없으므로, 혼자서는 동작하지 않는다.
    - **예시:** `@Entity`, `@Id`, `@ManyToOne`, `EntityManager`

  **2) Hibernate**

    - **역할:** JPA 명세를 실제로 구현한 구현체(라이브러리)이다.
    - **특징:** SQL 생성, 영속성 컨텍스트 관리, 지연 로딩 등 내부 동작을 구체적으로 완성한 핵심 엔진이다.
    - EclipseLink, DataNucleus 같은 다른 JPA 구현체도 있지만, Hibernate가 사실상 업계 표준이며 Spring Boot의 기본값이다.
    - **예시:** `ddl-auto: validate` 설정은 Hibernate의 기능이다.

  **3) Spring Data JPA (참고)**

    - **역할:** 개발자가 Hibernate를 더 쉽게 쓰도록 Spring이 제공하는 추상화 계층이다.
    - **특징:** `JpaRepository`를 상속하면 `save()`, `findById()` 같은 CRUD를 자동으로 제공하고, 내부에서 Hibernate를 호출한다.
    - **예시:** `BookRepository extends JpaRepository<Book, Long>`

  #### 3. Node.js/TypeScript 생태계: TypeORM의 통합 구조

    - TypeORM은 Java 진영과 달리 **명세와 구현체가 분리되어 있지 않다**.
    - **역할:** 그 자체로 규격이자 기능을 갖춘 독립적인 단일 ORM 라이브러리이다.
    - **특징:** JPA처럼 강제되는 외부 표준 인터페이스가 없다. 라이브러리 하나가 엔티티 정의, 쿼리 빌딩, 커넥션 관리, 마이그레이션을 모두 처리한다.
    - TypeScript 데코레이터(`@`)를 적극적으로 활용한다. (`@Entity`, `@Column`, `@ManyToOne` 등)
    - 두 가지 패턴을 모두 지원한다.
        - **Active Record:** 엔티티 자체가 DB 접근 메서드를 가진다.
        - **Data Mapper:** Repository가 데이터를 관리한다. (JPA와 유사)

  | 비교 항목 | JPA / Hibernate (Java) | TypeORM (Node.js/TypeScript) |
      | --- | --- | --- |
  | 아키텍처 | 명세(JPA)와 구현체(Hibernate)가 분리됨 | 단일 라이브러리가 ORM 기능을 통합 제공 |
  | 교체 가능성 | 이론상 코드 수정 없이 구현체를 교체할 수 있음 | 라이브러리 자체가 규격이므로, 교체하려면 다른 ORM(Prisma, Sequelize 등)으로 마이그레이션해야 함 |
  | 설계 패턴 | Data Mapper 중심 (`EntityManager`로 조작) | Active Record와 Data Mapper를 모두 지원 |
  | 영속성 컨텍스트 | 1차 캐시, 쓰기 지연, 변경 감지(Dirty Checking) 등 상태 관리가 정교함 | JPA 수준의 영속성 컨텍스트가 없고, 쿼리 실행이 더 직관적임 |
  | Repository | `JpaRepository` 상속 (Spring Data JPA) | `Repository<T>` 주입 (`@InjectRepository`) |
  | 스키마 검증 설정 | `ddl-auto: validate` | `synchronize: false` |
  | 설정 위치 | `application.yml` | `TypeOrmModule.forRoot()` |
- **Entity Lifecycle과 Persistence Context** — 엔티티를 저장할 때 즉시 SQL이 실행되지 않을 수 있는 이유는 무엇인가?

  #### 1. 영속성 컨텍스트(Persistence Context)란

    - 엔티티를 **영구 저장하는 환경**이라는 뜻이다.
    - 애플리케이션과 DB 사이에서 객체를 보관하는 **가상의 데이터베이스** 역할을 한다.
    - JPA는 `EntityManager`를 통해 영속성 컨텍스트에 접근하고 엔티티를 관리한다.
    - Spring에서는 보통 **트랜잭션 단위**로 생성되고 종료된다. 그래서 `@Transactional` 범위 안에서만 영속성 컨텍스트의 기능이 동작한다.

  #### 2. 엔티티의 생명주기(Entity Lifecycle)

  | 상태 | 설명 | 예시 |
      | --- | --- | --- |
  | 비영속 (New / Transient) | 영속성 컨텍스트와 전혀 관계없는 순수 객체 상태이다. | `Member member = new Member();` |
  | 영속 (Managed) | 영속성 컨텍스트가 관리하는 상태이다. 이 시점에는 DB에 쿼리가 날아가지 않는다. | `em.persist(member);` |
  | 준영속 (Detached) | 영속성 컨텍스트에 저장되었다가 분리된 상태이다. 영속성 컨텍스트가 제공하는 기능을 쓰지 못한다. | `em.detach(member);`, `em.clear();` |
  | 삭제 (Removed) | 실제 DB에서 삭제하기 위해 삭제 예약된 상태이다. | `em.remove(member);` |

    ```
    비영속 --persist()--> 영속 --detach()/clear()/트랜잭션 종료--> 준영속
                            └--remove()--> 삭제 --flush--> DELETE SQL
    ```

    - 영속 상태가 되어야 1차 캐시, 쓰기 지연, 변경 감지의 대상이 된다.
    - 영속 상태가 되었다고 해서 곧바로 DB에 저장되는 것은 아니다.

  #### 3. 즉시 SQL이 실행되지 않는 이유: 쓰기 지연

  `em.persist()`로 엔티티를 영속 상태로 만들어도 INSERT SQL은 바로 실행되지 않는다.

  **1) 쓰기 지연의 동작 방식**

    1. **1차 캐시 저장:** `em.persist(entity)`가 호출되면 엔티티를 영속성 컨텍스트 내부의 1차 캐시에 저장한다.
    2. **SQL 생성 및 저장:** 동시에 엔티티 정보로 INSERT SQL을 생성해 **쓰기 지연 SQL 저장소**에 차곡차곡 쌓아 둔다.
    3. **플러시(Flush) 발생:** 트랜잭션 커밋(`transaction.commit()`)이나 `em.flush()` 호출 시점에 영속성 컨텍스트의 변경 내용이 DB와 동기화된다.
    4. **DB 전송 및 커밋:** 저장소에 모여 있던 쿼리가 한 번에 DB로 전송되고, 이후 실제 트랜잭션이 커밋된다.

    ```
    persist(A) ─┐
    persist(B) ─┼─> [쓰기 지연 SQL 저장소: INSERT A, INSERT B]
    persist(C) ─┘                    │
                              commit / flush
                                     ▼
                         DB로 한 번에 전송 후 커밋
    ```

  #### 4. 쓰기 지연을 사용하는 이유와 장점

  **1) 네트워크 통신 비용 감소와 성능 최적화 (Batch 처리)**

    - DB에 접근해 쿼리를 실행하는 것은 비용이 큰 작업이다.
    - 엔티티를 저장할 때마다 SQL을 바로 보내면 네트워크 통신 횟수가 늘어 성능이 떨어진다.
    - 쓰기 지연을 활용하면 여러 엔티티를 저장하더라도 커밋 직전에 모아 둔 쿼리를 한 번에 전송할 수 있다. (**JDBC Batch** 기능)
    - 이를 통해 DB와의 통신 횟수를 크게 줄여 애플리케이션 성능을 최적화한다.

  **2) 깔끔한 롤백**

    - 중간에 에러가 발생해도 쿼리가 아직 DB에 반영되기 전이므로 롤백 처리가 훨씬 깔끔하다.
- **DTO와 API Contract** — Entity를 그대로 응답하면 어떤 문제가 생길까?

  #### 1. 용어 정리

  **1) API Contract (API 계약)**

    - 클라이언트와 서버 간에 **데이터를 어떻게 주고받을 것인가**에 대한 명시적인 약속이다.
    - 서버는 약속된 형태(JSON 구조, 필드명 등)로 응답해야 한다.
    - 내부 DB 구조가 바뀌더라도 **약속된 API 스펙은 유지**되어야 한다.
    - URL, HTTP 메서드, 요청 필드, 응답 필드, 상태 코드, 오류 형식이 모두 포함된다.
    - 계약이 바뀌면 이를 쓰는 클라이언트(프론트엔드, 앱)가 깨질 수 있으므로 함부로 바꾸지 않는다.

  **2) DTO (Data Transfer Object)**

    - 계층 사이, 특히 클라이언트와 서버 사이에서 **데이터를 전달하기 위한 객체**이다.
    - 로직 없이 필요한 필드만 가진다.
    - 요청용(`BookRequest`)과 응답용(`BookResponse`)을 따로 둔다.

  #### 2. Entity 직접 반환 시 발생하는 문제

  **1) 순환 참조 (Infinite Recursion)**

    - 양방향 연관관계(예: `Member` ↔ `Team`, `Category` ↔ `Book`)를 가진 Entity를 JSON으로 직렬화하면 무한 루프에 빠진다.
    - `Member` 변환 중 `Team` 조회 → `Team` 변환 중 `Member` 리스트 조회 → 다시 `Team` 조회... 로 이어져 `StackOverflowError`가 발생한다.
    - 이를 막으려고 `@JsonIgnore`를 Entity에 덕지덕지 붙이면, Entity가 **프레젠테이션 계층(뷰 로직)에 종속**된다.
    - DTO는 필요한 값만 평평하게 담으므로 순환이 생기지 않는다.

  **2) API 스펙의 붕괴 (유연성 부족)**

    - Entity는 DB 스키마와 1:1로 강하게 결합되어 있다.
    - 기획 변경으로 컬럼명(`username` → `nickname`)을 수정하면 **응답 JSON의 키값도 함께 변한다.**
    - 백엔드 내부 구현 수정이 프론트엔드와 클라이언트에 즉각적인 버그를 유발하고, API Contract가 무의미해진다.
    - 이번 주차 코드라면 `is_available`, `category_id` 같은 DB 내부 구조가 그대로 외부에 드러나는 경우이다.

  **3) 불필요한 데이터 노출과 보안 문제 (Over-fetching)**

    - 화면에는 이름과 이메일만 필요한데 Entity를 통째로 넘기면 비밀번호, 생성일자, 주민번호 같은 **민감 정보까지 네트워크를 타고 전달**된다.
    - 데이터 전송량이 불필요하게 커지고, 심각한 보안 취약점이 된다.
    - DTO는 응답에 필요한 필드만 골라 담는다.

  **4) Entity(Domain)의 오염 (SRP 위배)**

    - Entity는 핵심 비즈니스 로직과 DB 매핑을 담당하는 **순수한 도메인 객체**여야 한다.
    - API 응답 요구사항을 맞추려고 `@JsonFormat`, `@JsonProperty`, `@NotNull` 같은 화면 렌더링용·검증용 어노테이션이 붙기 시작하면 **단일 책임 원칙(SRP)이 깨진다.**
    - DB 매핑과 요청 검증이라는 서로 다른 책임이 한 클래스에 섞여 유지보수가 어려워진다.

  **5) 지연 로딩(LAZY) 오류**

    - `BookEntity`의 `category`는 `FetchType.LAZY`이므로 실제 객체가 아닌 **프록시**일 수 있다.
    - JSON 변환은 Controller 이후, 즉 트랜잭션이 끝난 뒤에 일어난다.
    - 이때 프록시에 접근하면 `LazyInitializationException`이나 직렬화 오류가 발생할 수 있다.
    - 이번 주차는 `open-in-view: false`이므로 이 문제가 더 분명하게 드러난다.
    - DTO는 **트랜잭션 안(Service)에서** 필요한 값만 꺼내 담으므로 이를 피한다.

  **6) 숨은 N+1 쿼리**

    - 응답 변환 과정에서 연관 엔티티에 접근하면 **추가 쿼리가 하나씩** 실행된다.
    - 코드에 드러나지 않아 원인을 찾기 어렵다.
    - 변환 시점을 Service로 고정하면 조회 범위를 의식하며 관리할 수 있다.

  **7) 과도한 값 바인딩 (Mass Assignment)**

    - 요청 본문을 Entity로 바로 받으면 클라이언트가 `bookId`나 `isAvailable` 같은 **의도하지 않은 필드**까지 채워 보낼 수 있다.
    - 요청 DTO는 받을 필드(`categoryId`, `title`, `description`)만 열어 둔다.

  **8) 화면별 응답에 대응하기 어려움**

    - 목록용, 상세용, 관리자용 응답이 서로 다를 수 있다.
    - Entity 하나로는 이를 표현하기 어렵고, Jackson 어노테이션을 덧붙일수록 Entity가 지저분해진다.
    - DTO는 용도별로 여러 개를 만들 수 있다.

    ---

  #### 3. 해결책: DTO의 도입

    - **역할 분리:** Entity는 DB와 통신하는 영속성 계층에서만 쓰고, Controller에서 클라이언트로 반환할 때는 반드시 DTO로 변환한다.
    - **맞춤형 데이터:** 각 API(목록 조회, 상세 조회 등)가 요구하는 스펙에 맞게 DTO를 별도로 만들어 필요한 데이터만 담는다.
- **Validation** — 요청 검증을 Controller 단에서 수행해야 하는 이유는 무엇인가?

  #### 1. Validation이란

    - 요청 데이터가 약속된 규칙(API Contract)을 지키는지 확인하는 과정이다.
    - 이번 주차 `POST /books`의 검증 규칙은 다음과 같다.
        - `categoryId`는 비어 있지 않은 숫자여야 한다.
        - `title`은 비어 있지 않고 100자 이하여야 한다.
        - `description`은 선택 값이다.
    - 규칙을 어긴 요청은 처리하지 않고 오류로 응답한다.

  #### 2. Controller 단에서 검증해야 하는 이유

  **1) 잘못된 요청을 빠르게 차단한다 (Fail Fast)**

    - 요청이 Service에 들어오기 **전에** 검증이 끝난다.
    - 잘못된 값으로 트랜잭션을 열거나 DB를 조회하는 낭비가 없다.
    - 오류를 가장 이른 시점에 발견하므로 원인을 찾기도 쉽다.

  **2) Service는 올바른 데이터만 받는다**

    - Service는 "검증을 통과한 값"이라는 전제로 **핵심 로직에만 집중**할 수 있다.
    - Service마다 `if (title == null || title.isBlank())` 같은 방어 코드를 반복하지 않아도 된다.
    - 검증 규칙이 흩어지지 않고 DTO 한곳에 모인다.

  **3) DB 오류 대신 명확한 4xx 응답을 줄 수 있다**

    - 검증 없이 DB까지 가면 제약 조건 위반이 `DataIntegrityViolationException` 같은 예외로 터지고, 응답은 **500 Internal Server Error**가 된다.
    - 예를 들어 `title`이 100자를 넘으면 `length = 100` 컬럼에서 오류가 난다.
    - 클라이언트 잘못은 **400 Bad Request**로, 서버 잘못은 500으로 구분해야 한다.
    - 검증을 앞단에서 하면 "제목은 100자 이하여야 합니다" 같은 원인을 알 수 있는 응답을 줄 수 있다.

  **4) 클라이언트 검증을 신뢰할 수 없다**

    - 프론트엔드 검증은 사용자 편의를 위한 것이다.
    - Postman이나 직접 만든 요청은 프론트엔드 검증을 우회한다.
    - 그래서 **서버가 반드시 다시 검증**해야 한다. 서버 검증이 진짜 방어선이다.

  **5) 선언적으로 간결하게 작성할 수 있다**

    - DTO 필드에 어노테이션만 붙이면 규칙이 문서처럼 읽힌다.
    - Controller 파라미터에 `@Valid`만 붙이면 실행된다.
    - 검증 코드와 비즈니스 코드가 분리되어 읽기 쉽다.

  **6) 오류 응답을 일관되게 처리할 수 있다**

    - 검증 실패는 `MethodArgumentNotValidException`이라는 정해진 예외로 발생한다.
    - `@RestControllerAdvice`의 `GlobalExceptionHandler` 한곳에서 잡아 **같은 형식의 400 응답**으로 바꿀 수 있다.

  **7) 안전하지 않은 입력을 막는다**

    - 비정상적으로 긴 문자열, 누락된 필수 값, 잘못된 타입 같은 입력이 내부 로직과 DB로 흘러가는 것을 막는다.
    - 입력 검증은 보안의 가장 기본적인 첫 단계이다.
- **N+1 Query** — 관계 데이터를 함께 조회할 때 쿼리가 예상보다 많이 실행되는 이유는 무엇인가?

  #### 1. N+1 Query란

    - 총 쿼리 수가 `1 + N`이 되는 문제이다.
        - **1:** 부모 목록을 조회하는 쿼리
        - **N:** 목록의 각 항목마다 연관 데이터를 조회하는 쿼리
    - 기능은 정상 동작하므로 **겉으로는 문제가 드러나지 않는다.** 데이터가 적을 때는 느낌이 없다가, 데이터가 늘면 응답이 급격히 느려진다.
    - DB 통신 횟수가 늘어나므로 네트워크 비용과 DB 부하가 함께 커진다.

  #### 2. 발생 원리

  **1) 이번 주차 코드에서 보기**

    ```java
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "category_id", nullable = false)
    private CategoryEntity category;
    ```

    ```java
    @Transactional(readOnly = true)
    public List<BookResponse> getBooks() {
        return bookRepository.findAllByOrderByBookIdDesc().stream()
                .map(BookResponse::from)   // 여기서 book.getCategory().getName() 호출
                .toList();
    }
    ```

  **2) 실행되는 쿼리 흐름**

    ```sql
    -- 1번: 도서 목록 조회 (category는 가져오지 않음)
    SELECT * FROM book ORDER BY book_id DESC;
    
    -- N번: 각 도서의 카테고리 이름이 필요할 때마다 실행
    SELECT * FROM category WHERE category_id = 1;
    SELECT * FROM category WHERE category_id = 2;
    SELECT * FROM category WHERE category_id = 3;
    ...
    ```

  **3) 왜 이렇게 되는가**

    1. `findAllByOrderByBookIdDesc()`는 `book` 테이블만 조회한다. 이때 `category`는 실제 객체가 아닌 **프록시(가짜 객체)**로 채워진다.
    2. `BookResponse.from()`에서 `getCategory().getName()`을 호출하는 순간 프록시가 **실제 데이터를 가져오려고 SELECT를 실행**한다.
    3. 도서가 여러 권이면 이 과정이 반복되어 쿼리가 늘어난다.

  **4) 정확하게 알아둘 점**

    - 같은 트랜잭션 안에서는 **1차 캐시**가 동작하므로, 이미 조회한 같은 `category_id`는 다시 쿼리하지 않는다.
    - 따라서 실제 추가 쿼리 수는 N이 아니라 **서로 다른 카테고리의 개수**이다.
    - 도서 100권이 카테고리 3개에 나뉘어 있으면 `1 + 3`번이다. 카테고리가 모두 다르면 `1 + 100`번에 가까워진다.
    - 연관 컬렉션(`Category`의 `List<Book>`)이나 **EAGER 로딩**에서는 더 심하게 나타날 수 있다.

  #### 3. 해결 방법

  **1) fetch join (JPQL)**

    ```java
    @Query("select b from BookEntity b join fetch b.category order by b.bookId desc")
    List<BookEntity> findAllWithCategory();
    ```

    - `book`과 `category`를 **JOIN 하나로 한 번에** 조회한다.
    - 연관 엔티티가 실제 객체로 채워지므로 이후 `getCategory().getName()`에서 추가 쿼리가 나가지 않는다.
    - 쿼리가 `1 + N`번에서 **1번**으로 줄어든다.

  **2) `@EntityGraph`**

    ```java
    @EntityGraph(attributePaths = "category")
    List<BookEntity> findAllByOrderByBookIdDesc();
    ```

    - 메서드 이름 기반 쿼리를 **그대로 유지**하면서 연관 데이터를 함께 가져온다.
    - JPQL을 직접 쓰지 않아도 되어 이번 주차 코드에 가장 적용하기 쉽다.
    - 내부적으로는 fetch join과 같은 효과를 낸다.

  **3) Batch Size**

    ```yaml
    spring:
      jpa:
        properties:
          hibernate:
            default_batch_fetch_size: 100
    ```

    - 건별 쿼리를 `WHERE category_id IN (?, ?, ?, ...)` 형태로 **묶어서** 보낸다.
    - `1 + N`번이 `1 + 1`번 정도로 줄어든다.
    - 컬렉션(1:N) 연관관계에서도 쓸 수 있어 **전역 설정으로 두기 좋다.**

  #### 4) DTO로 직접 조회 (Projection)

    ```java
    @Query("select new com.umc.study.domain.Book.dto.response.BookResponse(" +
           "b.bookId, b.title, b.description, c.name, b.isAvailable) " +
           "from BookEntity b join b.category c order by b.bookId desc")
    List<BookResponse> findAllBooks();
    ```

    - 엔티티를 거치지 않고 **필요한 컬럼만** 바로 조회한다.
    - 쿼리가 1번이고 가져오는 데이터도 줄어든다.
    - 반면 Repository가 응답 DTO를 알게 되어 계층 간 결합이 생긴다.
- **Migration과 synchronize** — 운영 환경에서 자동 스키마 변경이 위험한 이유는 무엇인가?

  #### 1. 용어 정리

  **1) 자동 스키마 변경**

    - 엔티티 정의와 DB 구조를 비교해 **차이를 앱이 알아서 맞추는** 기능이다.
    - 🍃 Spring Boot(Hibernate): `spring.jpa.hibernate.ddl-auto`
    - 🐱 NestJS(TypeORM): `synchronize`

  **2) Migration**

    - 스키마 변경을 **버전이 붙은 스크립트**로 만들어, 정해진 순서대로 실행하는 방식이다.
    - "무엇을, 언제, 어떤 순서로 바꿨는지"가 파일로 남는다.
    - 도구는 다음과 같다.
        - 🍃 Spring Boot: Flyway, Liquibase
        - 🐱 NestJS: TypeORM Migration (`migration:generate`, `migration:run`, `migration:revert`)

    ---

  #### 3. 옵션 정리

  **1) 🍃 Hibernate `ddl-auto`**

  | 값 | 동작 | 운영 사용 |
      | --- | --- | --- |
  | `none` | 아무것도 하지 않는다. | 가능 |
  | `validate` | 엔티티와 테이블이 **맞는지 검증만** 한다. 불일치하면 앱 시작이 실패한다. | 권장 |
  | `update` | 없는 테이블·컬럼을 **추가**한다. 기존 컬럼 삭제나 변경은 하지 않는다. | 위험 |
  | `create` | 시작할 때 기존 테이블을 **삭제하고 새로 만든다.** | 금지 |
  | `create-drop` | `create`와 같고, 종료할 때 테이블도 삭제한다. | 금지 |

  **2) 🐱 TypeORM `synchronize`**

  | 값 | 동작 |
      | --- | --- |
  | `true` | 앱 시작마다 엔티티 기준으로 **DB 스키마를 바꾼다.** 컬럼 삭제와 타입 변경도 포함된다. |
  | `false` | 스키마를 건드리지 않는다. 변경은 Migration으로 한다. |
    - TypeORM 공식 문서도 `synchronize: true`는 **운영 환경에서 쓰지 말라**고 안내한다.

    ---

  #### **4. 운영에서 위험한 이유**

  **1) 데이터 손실**

    - 엔티티에서 컬럼을 삭제하거나 이름을 바꾸면, 자동 동기화는 이를 **"기존 컬럼 삭제 + 새 컬럼 추가"**로 처리할 수 있다.
    - 예를 들어 `is_available`을 `available`로 바꾸면 기존 컬럼의 **데이터가 사라지고** 빈 컬럼이 생긴다.
    - 컬럼 타입 변경도 데이터가 잘리거나 변환에 실패할 수 있다.
    - `create` 계열은 시작 시 **테이블 전체를 삭제**하므로 가장 치명적이다.
    - 한 번 날아간 운영 데이터는 백업이 없으면 복구할 수 없다.

  **2) 변경 내용을 예측하고 검토할 수 없다**

    - 어떤 SQL이 실행될지 배포 전에 **확인할 방법이 없다.**
    - 코드 리뷰에서도 엔티티 변경만 보일 뿐, 실제 DDL은 보이지 않는다.
    - Migration은 SQL 파일이 있어서 리뷰·승인·사전 테스트가 가능하다.

  **3) 이력과 버전 관리가 없다**

    - 누가, 언제, 왜 스키마를 바꿨는지 **기록이 남지 않는다.**
    - 환경(개발·스테이징·운영)마다 DB 구조가 달라져도 알아채기 어렵다.
    - Migration 도구는 적용 이력을 별도 테이블(예: Flyway의 `flyway_schema_history`)에 저장해, 환경 간 상태를 같게 유지한다.

  **4) 롤백이 어렵다**

    - 자동 변경은 **되돌리는 방법이 없다.** 잘못 바뀐 스키마는 직접 수습해야 한다.
    - Migration은 `down`(되돌리기) 스크립트나 보정 스크립트를 함께 준비할 수 있다.
    - 단, 삭제된 데이터 자체는 롤백으로 복구되지 않으므로 **사전 백업**은 별개로 필요하다.

  **5) 앱 시작과 DB 변경이 묶인다**

    - 배포(앱 시작)할 때마다 스키마 변경이 함께 실행된다.
    - 서버가 여러 대이면 **여러 인스턴스가 동시에 스키마를 바꾸려 해** 충돌하거나 중간 상태가 생길 수 있다.
    - 대용량 테이블의 ALTER는 **테이블을 잠그고 오래 걸려** 서비스 장애로 이어질 수 있다.
    - 변경이 실패하면 앱 시작 자체가 실패해 배포가 중단된다.

  **6) 엔티티 실수가 곧바로 운영 DB에 반영된다**

    - 개발자가 엔티티를 잘못 고치거나 브랜치를 잘못 배포해도 **즉시 운영 DB에 적용**된다.
    - 인덱스, 제약 조건, 기본값, 문자셋 같은 세부 설정은 엔티티 기반으로 생성하면 의도와 달라질 수 있다.

  **7) 자동 도구가 못 하는 변경이 있다**

    - 컬럼 이름 변경, 테이블 분리, **기존 데이터 변환·이관**은 자동 비교로 처리할 수 없다.
    - 예를 들어 "`name`을 `first_name`과 `last_name`으로 나누기"는 데이터 이관 SQL이 필요하다.
    - 이런 변경은 사람이 작성한 Migration이 필요하다.

  **8) MySQL의 DDL은 트랜잭션으로 롤백되지 않는다**

    - MySQL은 `CREATE`, `ALTER`, `DROP` 같은 DDL을 실행하면 **암묵적 커밋**이 일어난다.
    - 여러 변경 중 일부만 적용되고 실패하면 **중간 상태로 남을 수 있다.**
    - 그래서 변경을 작은 단위의 Migration으로 나누고, 적용 전 백업과 사전 검증을 하는 것이 중요하다.

    ---

  #### 5. 환경별 권장 설정

  | 환경 | 🍃 `ddl-auto` | 🐱 `synchronize` | 스키마 관리 |
      | --- | --- | --- | --- |
  | 로컬 개발 (빠른 실험) | `update` 또는 `create` | `true` | 자동 허용 |
  | 로컬 DB를 직접 만든 경우 (이번 주차) | `validate` | `false` | Workbench 등으로 직접 |
  | 테스트 | `create-drop` (임베디드·테스트 DB) | `true` (테스트 DB) | 자동 허용 |
  | 스테이징 / 운영 | `validate` 또는 `none` | `false` | **Migration** |
    - 자동 변경을 쓰더라도 **운영 DB에 연결되지 않도록** 접속 정보를 철저히 분리한다.
    - 운영 DB 계정에는 DDL 권한을 제한하는 방법도 있다.