- 트랜잭션(Transaction)

  ### 1. 트랜잭션이란

    - 여러 SQL 작업을 하나의 묶음으로 처리하는 단위이다.
    - MySQL InnoDB에서는 모든 사용자 활동이 트랜잭션 안에서 일어난다.
    - autocommit이 켜져 있으면 SQL 문장 하나가 하나의 트랜잭션이 된다.
    - 여러 문장을 묶으려면 `START TRANSACTION` 또는 `BEGIN`으로 시작하고, `COMMIT` 또는 `ROLLBACK`으로 끝낸다.

  ### 2. commit / rollback

  | 구분 | 의미 |
      | --- | --- |
  | commit | 현재 트랜잭션의 변경을 영구 반영하고 다른 세션에서도 볼 수 있게 한다. |
  | rollback | 현재 트랜잭션이 수행한 모든 수정을 취소한다.  |
    - commit과 rollback 모두 해당 트랜잭션이 잡고 있던 InnoDB 락을 해제한다.

  ### 3. 원자성(Atomicity)과의 관계

    - 트랜잭션은 ACID 중 원자성과 가장 직접 연결된다.
    - MySQL 문서는 원자성과 관련된 기능으로 autocommit 설정, COMMIT, ROLLBACK을 든다.
    - 하나의 트랜잭션 안의 작업은 모두 반영되거나 모두 취소된다는 뜻이다.

  ### 4. Spring `@Transactional` 핵심

  | 항목 | 내용 |
      | --- | --- |
  | 정상 종료 | 메서드가 끝나면 commit한다. |
  | 롤백 조건 | 기본 설정에서는 런타임(unchecked) 예외일 때만 롤백 대상으로 표시한다. |
  | 체크 예외 | 기본 설정에서는 롤백을 일으키지 않는다. |
  | 규칙 변경 | `rollbackFor`, `noRollbackFor` 속성으로 지정한다. |
  | 주의점 | 프록시를 통해 들어오는 외부 메서드 호출만 가로챈다. 같은 객체 안의 self-invocation은 `@Transactional`이 붙어 있어도 실제 트랜잭션이 만들어지지 않는다.  |

  ### 5. 이번 백엔드 코드 예시: `MovieRecordService`

    ```java
    @Service
    @RequiredArgsConstructor
    public class MovieRecordService {
    
        private final MemberRepository memberRepository;
        private final RatingRepository ratingRepository;
        private final FavoriteRepository favoriteRepository;
    
        @Transactional  // 이 메서드 전체가 하나의 트랜잭션
        public MovieRecordResponse create(Long memberId, CreateMovieRecordRequest request) {
            Member member = memberRepository.findById(memberId)
                    .orElseThrow(() -> new ApiException(HttpStatus.NOT_FOUND, "MEMBER404_1",
                            "해당 회원을 찾을 수 없습니다."));
    
            // 사전 검사: 이미 평점이 있으면 RATING409_1
            if (ratingRepository.existsByMember_MemberIdAndMovieId(memberId, request.movieId())) {
                throw new ApiException(HttpStatus.CONFLICT, "RATING409_1",
                        "이미 이 영화에 평점을 남기셨습니다.");
            }
    
            // 사전 검사: 즐겨찾기를 함께 저장할 때 이미 있으면 FAVORITE409_1
            if (request.addToFavorites()
                    && favoriteRepository.existsByMember_MemberIdAndMovieId(memberId, request.movieId())) {
                throw new ApiException(HttpStatus.CONFLICT, "FAVORITE409_1",
                        "이미 이 영화를 즐겨찾기하셨습니다.");
            }
    
            // 1) 평점 저장
            Rating savedRating = ratingRepository.save(Rating.builder()
                    .member(member).movieId(request.movieId())
                    .score(request.score()).comment(request.comment()).build());
    
            // 2) 즐겨찾기 저장 (addToFavorites가 true일 때만)
            FavoriteResponse favoriteResponse = null;
            if (request.addToFavorites()) {
                Favorite savedFavorite = favoriteRepository.save(Favorite.builder()
                        .member(member).movieId(request.movieId()).build());
                favoriteResponse = FavoriteResponse.from(savedFavorite);
            }
    
            return new MovieRecordResponse(RatingResponse.from(savedRating), favoriteResponse);
        }
    }
    ```

    - `create()`에 붙은 `@Transactional` 하나로 평점 저장과 즐겨찾기 저장이 같은 트랜잭션에 들어간다.
    - 메서드가 정상적으로 끝나면 두 저장이 함께 commit된다.
    - 2번 저장 중 예외가 나면 이미 저장한 1번 평점도 함께 rollback된다.
    - `RatingService`와 `FavoriteService`를 연달아 호출하지 않고 한 Service에서 직접 처리하기 때문에 트랜잭션 경계가 하나로 유지된다.

  ### 6. 트랜잭션 유무 비교

  | 상황 | 트랜잭션이 있을 때 | 트랜잭션이 없을 때 |
      | --- | --- | --- |
  | 평점 저장 성공, 즐겨찾기 저장 실패 | 평점도 rollback되어 둘 다 저장되지 않는다. | 평점만 저장되어 사용자가 의도한 기록과 달라진다. |
  | 둘 다 성공 | 함께 commit된다. | 각각 저장된다. |
- commit / rollback

  ### 1. commit / rollback이란

  | 구분 | 의미 |
      | --- | --- |
  | commit | 현재 트랜잭션의 변경을 영구 반영하고, 다른 세션에서도 볼 수 있게 한다. |
  | rollback | 현재 트랜잭션이 수행한 모든 수정을 취소한다.  |
    - 트랜잭션은 `START TRANSACTION` 또는 `BEGIN`으로 시작하고, `COMMIT` 또는 `ROLLBACK`으로 끝난다.
    - commit과 rollback 모두 해당 트랜잭션이 잡고 있던 InnoDB 락을 해제한다.
    - 트랜잭션 안의 변경은 commit하기 전까지 확정되지 않은 상태이다.

  ### 2. SQL로 보는 흐름

  **성공하면 commit**

    ```sql
    START TRANSACTION;
    INSERT INTO rating ...;     -- 아직 확정 전
    INSERT INTO favorite ...;   -- 아직 확정 전
    COMMIT;                     -- 두 저장이 함께 확정
    ```

  **실패하면 rollback**

    ```sql
    START TRANSACTION;
    INSERT INTO rating ...;     -- 성공
    INSERT INTO favorite ...;   -- 실패
    ROLLBACK;                   -- 평점 저장도 취소, 처음 상태로 복귀
    ```

  ### 3. Spring에서는 누가 commit / rollback 하는가

    - `@Transactional`을 쓰면 직접 `COMMIT`, `ROLLBACK`을 쓰지 않아도 Spring이 메서드 결과에 따라 대신 처리한다.

  | 메서드 결과 | 처리 |
      | --- | --- |
  | 정상 종료 | commit |
  | 런타임(unchecked) 예외가 밖으로 나감 | rollback |
  | 체크 예외가 밖으로 나감 | 기본 설정에서는 rollback 안 함 |
    - 기본 설정에서는 런타임 예외일 때만 롤백 대상으로 표시하고, `rollbackFor`로 규칙을 바꿀 수 있다.

  ### 4. 이번 코드 적용: `POST /movie-records`

  | 상황 | 결과 |
      | --- | --- |
  | 평점 저장 → 즐겨찾기 저장 모두 성공 | 메서드 정상 종료, 둘 다 commit |
  | 사전 검사에서 `RATING409_1` 예외 | 저장 전에 예외가 나므로 DB 변경 없음 |
  | 평점 저장 후 즐겨찾기 저장 중 예외 | 이미 저장한 평점도 rollback |
    - 두 저장이 같은 트랜잭션 안에 있어서, 하나라도 실패하면 둘 다 취소되고 둘 다 성공해야 commit된다.
- 원자성(Atomicity)

  ### 1. 원자성이란

    - ACID 중 A에 해당하며, 트랜잭션 안의 작업이 **모두 반영되거나 모두 취소되는 성질**이다.
    - 하나의 작업 묶음을 더 쪼갤 수 없는 단위(원자)처럼 다룬다는 의미이다.

  ### 2. MySQL에서 원자성과 연결된 기능

  | 관련 기능 | 역할 |
      | --- | --- |
  | autocommit 설정 | 문장마다 자동 commit할지 정한다. |
  | COMMIT | 트랜잭션의 변경을 확정한다. |
  | ROLLBACK | 트랜잭션의 변경을 모두 취소한다. |
    - 즉 commit과 rollback이 있어서 "전부 반영 또는 전부 취소"가 가능해지고, 이것이 원자성을 실현하는 수단이다.

  ### 3. 원자성이 있을 때와 없을 때

  | 상황 | 원자성이 있을 때 | 원자성이 없을 때 |
      | --- | --- | --- |
  | 작업 A 성공 후 작업 B 실패 | A도 취소되어 처음 상태로 돌아간다. | A만 반영된 어중간한 상태가 남는다. |
  | 모든 작업 성공 | 함께 반영된다. | 각각 반영된다. |
    - 예시로 카드 결제와 주문 생성은 함께 성공해야 한다. 결제만 되고 주문이 없거나, 주문만 생기고 결제가 안 되면 문제가 된다.

  ### 4. 이번 미션 코드에 적용

    - `POST /movie-records`는 평점 저장과 즐겨찾기 저장을 하나의 트랜잭션으로 묶어 원자성을 보장한다.
    - 즐겨찾기 저장이 실패하면 이미 저장한 평점도 rollback되어 둘 다 저장되지 않는다.
    - 두 저장이 모두 성공해야 commit되어 둘 다 반영된다.
    - 원자성은 오류를 없애 주는 기능이 아니라, 오류가 났을 때 데이터가 어중간한 상태로 남지 않게 하는 성질이다.
- ACID

  ### 1. ACID란

    - ACID 모델은 업무 데이터와 핵심 애플리케이션에서 중요한 신뢰성을 강조하는 데이터베이스 설계 원칙의 집합이다.
    - Atomicity(원자성), Consistency(일관성), Isolation(격리성), Durability(지속성)의 앞 글자를 딴 이름이다.
    - InnoDB는 이 모델을 따라서 소프트웨어 크래시나 하드웨어 오작동 같은 예외 상황에서도 데이터가 손상되지 않도록 한다.

  ### 2. 네 가지 속성

  | 속성 | 의미 | MySQL 문서에서 연결한 기능 |
      | --- | --- | --- |
  | 원자성 (Atomicity) | 트랜잭션 안의 작업이 모두 반영되거나 모두 취소된다. | InnoDB 트랜잭션이 중심이며, autocommit 설정, COMMIT, ROLLBACK과 관련된다. |
  | 일관성 (Consistency) | 트랜잭션 전후로 데이터가 항상 유효한 상태를 유지한다. | 크래시로부터 데이터를 보호하는 InnoDB 내부 처리가 중심이며, doublewrite buffer 등이 관련된다. |
  | 격리성 (Isolation) | 동시에 실행되는 트랜잭션이 서로 간섭하지 않는다. | InnoDB 트랜잭션, 그중에서도 각 트랜잭션에 적용되는 격리 수준이 중심이다. |
  | 지속성 (Durability) | commit된 데이터는 장애가 나도 유지된다. | MySQL 소프트웨어 기능이 실제 하드웨어 구성과 맞물려 동작하는 부분이다.  |
    - 각 속성의 "의미" 칸은 일반적인 정의이고, 오른쪽 칸은 MySQL 문서가 해당 속성에 연결한 기능이다.

  ### 3. 지속성은 하드웨어 환경의 영향도 받는다

    - MySQL 문서는 지속성에 영향을 주는 요소로 저장 장치의 쓰기 버퍼, 배터리 백업 캐시, 운영체제, 백업 전략 등을 든다.
    - 즉 지속성은 DB 설정만으로 결정되지 않고 서버 환경에도 달려 있다.

  ### 4. 이번 미션 코드와 연결

  | 속성 | 이번 코드에서의 예 |
      | --- | --- |
  | 원자성 | `POST /movie-records`에서 평점과 즐겨찾기를 한 트랜잭션으로 묶어 함께 저장하거나 함께 취소한다. |
  | 일관성 | `favorite`의 `(member_id, movie_id)` UNIQUE 제약으로 같은 회원의 같은 영화 중복 저장을 막는다. |
  | 격리성 | 같은 요청이 동시에 들어와도 트랜잭션끼리 서로 간섭하지 않도록 DB가 격리 수준에 따라 처리한다. |
  | 지속성 | commit된 즐겨찾기는 서버를 재시작해도 DB에 남아 있다. |
    - 이번 주 학습의 중심은 원자성이고, 나머지 세 속성은 DB가 함께 보장하는 부분이다.
- UNIQUE 제약 조건

  ### 1. UNIQUE 제약 조건이란

    - 지정한 컬럼(들)의 값이 테이블 안에서 중복되지 않도록 DB가 강제하는 규칙이다.
    - UNIQUE 인덱스는 인덱스 안의 모든 값이 서로 달라야 한다는 제약을 만든다.
    - 이미 있는 행과 같은 키 값을 가진 새 행을 추가하려고 하면 오류가 발생한다.
    - 이 검사는 애플리케이션 코드와 무관하게 DB가 직접 수행한다.

  ### 2. 단일 컬럼과 복합 컬럼

  | 구분 | 의미 | 예시 |
      | --- | --- | --- |
  | 단일 UNIQUE | 한 컬럼의 값이 중복되면 안 된다. | `email` |
  | 복합 UNIQUE | 여러 컬럼의 **조합**이 중복되면 안 된다. | `(member_id, movie_id)` |
    - 복합 UNIQUE에서는 각 컬럼 값 자체는 반복되어도 되고, 컬럼 값의 조합이 같을 때만 막힌다.

  ### 3. NULL 처리

    - MySQL의 UNIQUE 인덱스는 NULL을 허용하는 컬럼에 여러 개의 NULL 값을 허용한다.
    - 이번 `favorite`의 `member_id`, `movie_id`는 `nullable = false`라서 이 경우와 무관하다.

  ### 4. 위반하면 어떻게 되는가

    - DB에서 중복 삽입 오류가 발생한다.
    - Spring은 제약 위반을 `DataIntegrityViolationException`으로 변환해 던진다.
    - 이 예외는 `RuntimeException`의 하위 클래스이다.
    - `DuplicateKeyException`은 기본키나 UNIQUE 제약 위반을 나타내는 하위 예외이다.
    - `DataIntegrityViolationException`은 런타임 예외라서, `@Transactional` 메서드 밖으로 나가면 기본 설정에서 rollback 대상이 된다.

  ### 5. JPA에서 설정하는 방법

    ```java
    @Entity
    @Table(
            name = "favorite",
            uniqueConstraints = @UniqueConstraint(columnNames = {"member_id", "movie_id"})
    )
    public class Favorite extends BaseTimeEntity { ... }
    ```

    - `columnNames`는 제약을 구성하는 컬럼 이름들이다.
    - `name`은 선택이며, 생략하면 JPA 구현체가 이름을 만든다.
    - 이 어노테이션은 생성되는 DDL에 UNIQUE 제약을 포함시키는 정보를 지정한다.
    - DDL 자동 생성을 쓰지 않고 SQL로 직접 테이블을 만든 경우에는, 어노테이션만으로 DB에 제약이 생기지는 않는다.

  ### 6. 서버 사전 검사 vs DB UNIQUE 제약

  | 구분 | 서버 사전 검사 (`existsBy...`) | DB UNIQUE 제약 |
      | --- | --- | --- |
  | 목적 | 친절한 오류(`FAVORITE409_1`)를 주기 위해서 | 데이터 자체를 지키기 위해서 |
  | 동작 시점 | 저장 전에 조회로 확인 | 저장 시도 때 DB가 검사 |
  | 동시 요청 | 같은 요청 두 개가 모두 검사를 통과할 수 있다. | 동시 요청에서도 한쪽만 저장되도록 막는다. |
  | 역할 | 사용자 경험 | 마지막 안전망 |
    - 둘 중 하나만 있으면 부족하고, 함께 두는 것이 맞다.

  ### 7. 이번 미션 코드와 연결

    - `favorite` 테이블에 `(member_id, movie_id)` 복합 UNIQUE 제약이 있어서, 같은 회원이 같은 영화를 두 번 즐겨찾기하는 일을 DB도 막는다.
    - `movie_id`만 UNIQUE로 두면 한 영화가 서비스 전체에서 한 번만 즐겨찾기될 수 있어서, 반드시 두 컬럼을 한 쌍으로 묶어야 한다.
    - 서버는 `existsByMember_MemberIdAndMovieId`로 먼저 확인해 `FAVORITE409_1`을 응답한다.
- 낙관적 잠금(Optimistic Locking)

  ### 1. 낙관적 잠금이란

    - 데이터를 읽은 뒤 수정하는 사이에 다른 트랜잭션이 같은 데이터를 바꾸지 않았다고 **낙관적으로 가정**하고 진행하다가, 저장하는 시점에 충돌 여부를 확인하는 방식이다.
    - JPA 구현체는 기본적으로 낙관적 잠금을 사용하며, 변경을 commit하기 전에 데이터를 읽은 뒤 다른 트랜잭션이 수정하거나 삭제하지 않았는지 확인한다.
    - 버전 번호나 타임스탬프로 충돌하는 업데이트를 감지해서 lost update(갱신 손실)를 막는다.
    - lost update는 두 사용자가 같은 데이터를 읽고 각각 수정해서 저장할 때, 나중에 저장한 쪽이 먼저 저장한 변경을 덮어써 사라지게 하는 문제이다.

  ### 2. 동작 방식 (version 컬럼)

    - 엔티티에 version 컬럼을 두고, 수정할 때마다 값을 증가시킨다.
    - 데이터가 수정되면 version 값이 증가하고, 원래 트랜잭션은 version을 확인해서 다른 트랜잭션이 데이터를 수정했다면 `OptimisticLockException`이 발생한다.

  | 단계 | 사용자 A | 사용자 B | DB의 version |
      | --- | --- | --- | --- |
  | 1 | 평점 읽음 (version 1) | 평점 읽음 (version 1) | 1 |
  | 2 | 수정 저장 → 성공 |  | 2 |
  | 3 |  | 수정 저장 시도 → version이 달라서 실패 | 2 |

  저장할 때 실행되는 SQL은 version 조건이 붙는 형태이다. (예시)

    ```sql
    UPDATE rating
    SET score = 3, version = 2
    WHERE rating_id = 1 AND version = 1;   -- 0행이 수정되면 충돌로 판단
    ```

  ### 3. JPA에서 설정하는 방법

    ```java
    @Entity
    public class Rating extends BaseTimeEntity {
    
        @Id @GeneratedValue
        private Long ratingId;
    
        @Version              // 이 필드가 낙관적 잠금의 version
        private Long version;
    
        ...
    }
    ```

    - `@Version`은 엔티티의 version 필드를 선언하며, 낙관적 잠금 충돌을 감지하는 데 쓰인다.
    - JPA는 순차 숫자 version 방식과 타임스탬프 방식, 두 가지 전략을 지원한다.
    - 애플리케이션은 version 값을 읽을 수는 있지만 직접 수정해서는 안 된다.

  ### 4. 충돌했을 때 발생하는 예외

    - 낙관적 잠금 실패는 엔티티 상태를 읽은 시점과 업데이트를 시도하는 시점 사이에 DB의 version이나 타임스탬프가 바뀌었을 때 발생한다.
    - JPA에서는 영속성 제공자가 충돌이 생기면 `OptimisticLockException`을 던진다.
    - Spring에서는 이를 `ObjectOptimisticLockingFailureException`으로 감싸서 던지며, 이는 런타임 예외이다.
    - 낙관적 잠금 실패는 보통 DB 자체가 아니라 ORM 도구나 애플리케이션 코드가 감지한다.
    - 충돌이 나면 사용자에게 다시 시도하도록 안내하거나, 코드에서 재시도하는 식으로 처리한다.

  ### 5. 낙관적 잠금 vs 비관적 잠금

  | 구분 | 낙관적 잠금 | 비관적 잠금 |
      | --- | --- | --- |
  | 가정 | 충돌이 드물다. | 충돌이 자주 난다. |
  | 방식 | 락을 걸지 않고 저장 시점에 version으로 충돌을 감지한다. | 읽는 시점에 DB 락을 걸어 다른 트랜잭션의 접근을 막는다. |
  | 충돌 시 | 예외가 발생하고 재시도한다. | 다른 트랜잭션이 락 해제를 기다린다. |
  | 적합한 경우 | 충돌이 드문 일반적인 웹 서비스 | 충돌이 잦고 반드시 순서를 보장해야 하는 경우 |
    - Hibernate도 락 전략을 낙관적과 비관적, 두 가지로 구분한다.

  ### 6. 이번 미션과의 관계

    - 이번 주 코드는 낙관적 잠금을 직접 사용하지 않는다.
    - 즐겨찾기 중복 등록은 **같은 행을 수정하는 충돌이 아니라 같은 행을 두 번 삽입하는 문제**라서, 낙관적 잠금이 아니라 UNIQUE 제약이 막는다.
    - 낙관적 잠금은 평점 수정처럼 **이미 있는 행을 동시에 수정**하는 상황에서 의미가 있다. 예를 들어 `Rating`에 `@Version`을 두면 두 요청이 같은 평점을 동시에 수정할 때 나중 요청을 감지할 수 있다.