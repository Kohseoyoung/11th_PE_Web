- 미션 1

  ### 1. Repository — SQL 작성

  `BookRepository.java`에 메서드 추가:

    ```java
    public List<Map<String, Object>> findByCategoryId(Long categoryId) {
        String sql = "SELECT * FROM book WHERE category_id = ?";
        return jdbcTemplate.queryForList(sql, categoryId);
    }
    ```

  ### 2. Service — Repository 호출

  `BookService.java`에 메서드 추가:

    ```java
    public List<Map<String, Object>> getBooksByCategory(Long categoryId) {
        return bookRepository.findByCategoryId(categoryId);
    }
    ```

  ### 3. Controller — Path Variable 받기

  `BookController.java`에 메서드 추가:

    ```java
    import org.springframework.web.bind.annotation.PathVariable;
    
    // GET http://localhost:8080/books/category/1
    @GetMapping("/category/{categoryId}")
    public List<Map<String, Object>> getBooksByCategory(@PathVariable Long categoryId) {
        return bookService.getBooksByCategory(categoryId);
    }
    ```

  `{categoryId}`는 URL 경로의 일부를 변수처럼 받는 것이다. `@PathVariable`이 URL의 `{categoryId}` 자리에 들어온 값을 메서드 파라미터로 넘겨준다. 이름이 같으면(`{categoryId}` ↔ `Long categoryId`) 자동으로 매칭된다.

  ### 전체 흐름 정리

    ```
    GET /books/category/1
        ↓
    Controller: categoryId = 1 로 받음
        ↓
    Service: getBooksByCategory(1) 호출
        ↓
    Repository: SELECT * FROM book WHERE category_id = 1
    ```

  ### 테스트

  앱 재시작 후 Postman에서:

    - 메서드: **GET**
    - URL: `http://localhost:8080/books/category/1`

  category_id가 1인 책들만 응답으로 돌아오면 성공

    ![image (5).png](image%20%285%29.png)

- 미션 2

  ### 1. Repository — SQL 작성

  `RentalRepository.java` 새로 생성:

    ```java
    package com.umc.study.domain.repository;
    
    import lombok.RequiredArgsConstructor;
    import org.springframework.jdbc.core.JdbcTemplate;
    import org.springframework.stereotype.Repository;
    
    import java.util.Map;
    
    @Repository
    @RequiredArgsConstructor
    public class RentalRepository {
    
        private final JdbcTemplate jdbcTemplate;
    
        public void save(Map<String, Object> body) {
            String sql = "INSERT INTO rental (user_id, book_id, rented_at, due_at) " +
                         "VALUES (?, ?, NOW(), DATE_ADD(NOW(), INTERVAL 7 DAY))";
            jdbcTemplate.update(
                    sql,
                    body.get("userId"),
                    body.get("bookId")
            );
        }
    }
    ```

    - `NOW()`, `DATE_ADD(...)`는 사용자 입력이 아니라 DB가 실행 시점에 계산하는 함수라서 `?` 자리에 넣지 않고 SQL에 그대로 적는다.
    - `?`는 오직 **외부에서 받은 값**(`userId`, `bookId`)에만 사용한다.

  ### 2. Service

  `RentalService.java` 새로 생성:

    ```java
    package com.umc.study.domain.service;
    
    import com.umc.study.domain.repository.RentalRepository;
    import lombok.RequiredArgsConstructor;
    import org.springframework.stereotype.Service;
    
    import java.util.Map;
    
    @Service
    @RequiredArgsConstructor
    public class RentalService {
    
        private final RentalRepository rentalRepository;
    
        public void createRental(Map<String, Object> body) {
            rentalRepository.save(body);
        }
    }
    ```

  ### 3. Controller

  `RentalController.java` 새로 생성:

    ```java
    package com.umc.study.domain.controller;
    
    import com.umc.study.domain.service.RentalService;
    import lombok.RequiredArgsConstructor;
    import org.springframework.web.bind.annotation.PostMapping;
    import org.springframework.web.bind.annotation.RequestBody;
    import org.springframework.web.bind.annotation.RequestMapping;
    import org.springframework.web.bind.annotation.RestController;
    
    import java.util.Map;
    
    @RestController
    @RequestMapping("/rentals")
    @RequiredArgsConstructor
    public class RentalController {
    
        private final RentalService rentalService;
    
        // POST http://localhost:8080/rentals
        @PostMapping
        public String createRental(@RequestBody Map<String, Object> body) {
            rentalService.createRental(body);
            return "대여 기록이 등록되었습니다!";
        }
    }
    ```

  ### 테스트

  앱 재시작 후 Postman에서:

    - 메서드: **POST**
    - URL: `http://localhost:8080/rentals`
    - Body → raw → JSON:

    ![image (6).png](image%20%286%29.png)