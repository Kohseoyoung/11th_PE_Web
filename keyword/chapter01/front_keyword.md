- TypeScript와 정적 타입 검사
    - JavaScript와 비교했을 때 TypeScript는 오류를 언제, 어떤 방식으로 확인하나요?

      ### 1. JavaScript — 실행 시점

        - 코드를 실행해야 오류를 알 수 있다.
        - 실행되지 않은 코드의 문제는 발견할 수 없고, 문제가 있어도 오류 없이 넘어가는 경우가 많다.

        ```jsx
        add("1", "2")   // "12" — 에러 없이 이상한 값 반환
        ```

      ### 2. TypeScript — 작성 시점

        - `tsc`가 코드를 정적으로 분석해 타입이 맞지 않으면 컴파일을 거부한다.
        - 에디터에서는 타이핑하는 즉시 표시된다.

      #### 비교

      |  | JavaScript | TypeScript |
              | --- | --- | --- |
      | 시점 | 실행 중 | 작성 / 컴파일 시 |
      | 방식 | 해당 코드가 실행되어야 발견 | 전체 코드를 정적 분석 |
      | 범위 | 실행된 경로만 | 실행되지 않는 경로까지 |
    - 컴파일 타임 오류와 런타임 오류는 어떤 차이가 있나요?

      ### 1. 컴파일 타임 오류

        - 코드를 변환하는 단계에서 발생.
        - EX) 타입 불일치, 없는 프로퍼티 접근, 인자 개수 불일치 등.

        ```jsx
        error TS2345: Argument of type 'string' is not assignable to parameter of type 'number'.
        ```

      ### 2. 런타임 오류

        - 프로그램 실행 도중 발생.
        - EX) `undefined` 접근, 네트워크 실패, 잘못된 입력값 등.

        ```jsx
        TypeError: Cannot read properties of null (reading 'addEventListener')
        ```

      #### 비교

      |  | 컴파일 타임 | 런타임 |
              | --- | --- | --- |
      | 발생 시점 | 실행 전 | 실행 중 |
      | 영향 | 개발자만 | 사용자까지 |
      | 수정 비용 | 낮음 | 높음 (로그 확인 → 재현 → 수정 → 재배포) |
    - TypeScript의 타입 정보가 실행되는 JavaScript에 남지 않는 이유는 무엇일까요?

      ### 1. 브라우저와 Node.js는 JavaScript만 실행한다

        - 타입 문법은 JavaScript에 존재하지 않는다. 그대로 두면 문법 오류가 나므로 컴파일 과정에서 제거된다.

        ```tsx
        const name: string = "유리";   // TypeScript
        ```

        ```jsx
        const name = "유리";           // 컴파일 결과
        ```

      ### 2. 검사가 이미 끝났다

        - 컴파일 시점에 타입 검사가 완료되므로 실행 중에 다시 확인할 필요가 없다. 남겨두면 검사 코드가 추가되어 성능만 떨어진다.

      **장점**

        - 런타임 동작에 전혀 영향을 주지 않으므로, 기존 JavaScript 프로젝트에 점진적으로 도입할 수 있다.

      **단점**

        - 실행 중에는 타입 정보가 없어 타입을 이용한 판단이 불가능하다.

        ```jsx
        function check(value: string | number) {
          // ❌ 런타임에 타입 정보가 없어 불가능
          // ✅ JavaScript 문법으로 확인해야 함
          if (typeof value === "string") { }
        }
        ```

- 타입 추론과 타입 모델링
    - TypeScript가 타입을 추론하도록 두는 경우와 타입을 직접 작성하는 경우는 각각 언제 알맞을까요?

      ### 1. 추론에 맡기는 경우

      #### 1) 초기값만으로 타입이 명확할 때

        ```jsx
        const name = "유리";              // string
        const level = 1;                  // number
        const skills = ["TS", "React"];   // string[]
        ```

        - 굳이 `const name: string = "유리"`라고 쓰면 중복이다. 값을 보면 이미 알 수 있는 정보를 반복해서 적는 셈이라, 오히려 코드만 길어진다.

      #### 2) 함수의 반환 타입

        ```jsx
        function createCard(member: StudyMember) {
          return member.name + "님";   // string으로 추론
        }
        ```

        - 내부 로직이 바뀌면 반환 타입도 자동으로 따라가므로 수정할 곳이 줄어든다.

      ### 2. 직접 작성하는 경우

      #### 1) **함수의 매개변수**

        - 추론이 불가능하다. 무엇이 들어올지 TypeScript가 알 방법이 없으므로 반드시 적어야 한다.

        ```tsx
        function createCard(member: StudyMember) { }
        ```

      #### 2) **선언과 할당이 분리될 때**

        ```tsx
        let status: Status;   // 값이 없어 추론 불가
        ```

      #### 3) **추론 결과보다 좁히고 싶을 때**

        ```tsx
        let role = "LEADER";               // string으로 추론됨let role: Role = "LEADER";         // "LEADER" | "MEMBER"로 제한
        ```

        - `string`이면 아무 문자열이나 들어가지만, 유니언으로 명시하면 오타가 잡힌다.

      #### 4) **API 응답, 외부 데이터**

        - 형태를 명확히 선언해야 이후 코드에서 자동완성과 검사를 받을 수 있다.

      #### 5) **공개 함수의 반환 타입**

        - 명시하면 의도치 않게 반환 타입이 바뀌는 것을 막을 수 있다. 문서 역할도 한다.

      #### 기준

      > **값에서 알 수 있으면 추론에 맡기고, 값만으로 알 수 없거나 더 엄격하게 제한하고 싶으면 직접 쓴다.**
    >
    - 객체와 함수의 형태를 타입으로 표현하면 어떤 실수를 미리 찾을 수 있을까요?

      **객체에서**

      #### 1) 필수 프로퍼티 누락

        ```tsx
        const member: StudyMember = { name: "유리", level: 1 };// error TS2741: Property 'isLeader' is missing
        ```

      #### **2) 타입 불일치**

        ```tsx
        { level: "1" }   // string은 number에 할당 불가
        ```

      #### 3) 오타

        ```tsx
        member.nmae   // error: Property 'nmae' does not exist
        ```

      #### **4) 허용되지 않은 값**

        ```tsx
        type Role = "LEADER" | "MEMBER";const role: Role = "ADMIN";   // ❌ 오타와 잘못된 값 모두 차단
        ```

      #### 5) 선택적 값 미처리

        ```tsx
        member.githubId.toUpperCase();// error: 'member.githubId' is possibly 'undefined'
        ```

        - `?`가 붙은 프로퍼티는 확인 없이 쓰면 경고한다. 런타임 오류의 대표적 원인을 미리 막는다.

      #### 함수에서

      **인자 개수 불일치**

      #### 1) **인자 개수 불일치**

        ```tsx
        createCard();   // error: Expected 1 argument, but got 0
        ```

      #### 2) **인자 순서 바꿔 전달**

        ```tsx
        introduce(1, "유리");   // 타입이 다르면 즉시 발견
        ```

      #### 3) **반환값 오용**

        ```tsx
        const result: number = createCard(member);   // ❌ string을 number에 할당
        ```

      #### 4) **반환값이 없을 가능성**

        ```tsx
        function find(id: number): StudyMember | undefined { }const m = find(1);m.name;   // ❌ undefined일 수 있음
        ```

        - 찾지 못한 경우를 처리하지 않으면 컴파일되지 않는다.
    - `type`과 `interface`는 표현 범위와 확장 방식에서 어떤 차이가 있을까요?

      ### 1. 표현 범위

      **`interface`는 객체 형태만 표현할 수 있다.** `type`은 그 외의 것도 표현한다.

        ```tsx
        // type만 가능type Role = "LEADER" | "MEMBER";          // 유니언type ID = string | number;type Point = [number, number];            // 튜플type Callback = (value: string) => void;  // 함수 타입
        ```

        ```tsx
        // 둘 다 가능type Member = { name: string };interface Member { name: string }
        ```

        - 객체를 표현할 때는 차이가 없고, 그 밖의 타입에서는 `type`만 쓸 수 있다.

        ---

      ### 2. 확장 방식

      **`interface` — `extends`**

        ```tsx
        interface Member { name: string }interface Leader extends Member { teamName: string }
        ```

      **`type` — 교차 타입 `&`**

        ```tsx
        type Member = { name: string };type Leader = Member & { teamName: string };
        ```

        - 결과는 같지만 문법이 다르다. `extends`는 상속처럼 읽히고, `&`는 두 타입을 합치는 연산이다.

        ---

      ### 3. 선언 병합

        - **`interface`만의 특징.** 같은 이름으로 여러 번 선언하면 자동으로 합쳐진다.

        ```tsx
        interface Member { name: string }interface Member { level: number }// → { name: string; level: number }
        ```

        - `type`은 같은 이름을 다시 선언하면 오류가 난다.
        - 이 특성은 **외부 라이브러리 타입을 확장할 때** 유용하다. 원본 코드를 수정하지 않고 프로퍼티를 추가할 수 있기 때문이다. 반대로 의도치 않게 타입이 합쳐질 수 있어 예측 가능성은 떨어진다.

        ---

      ### 비교

      |  | `type` | `interface` |
              | --- | --- | --- |
      | 객체 표현 | ✅ | ✅ |
      | 유니언 | ✅ | ❌ |
      | 튜플 | ✅ | ❌ |
      | 함수 타입 | ✅ | △ (가능하나 어색) |
      | 확장 | `&` | `extends` |
      | 선언 병합 | ❌ | ✅ |
        
      ---

      ### 선택 기준

        - 객체 형태만 다루고 확장 가능성이 있다면 → `interface`
        - 유니언, 튜플 등 객체가 아닌 타입 → `type`
        - 라이브러리 타입을 확장해야 한다면 → `interface`
        - 객체 형태만 표현 할 수 있다.
- 유니언 타입과 타입 좁히기
    - 여러 타입 중 하나가 될 수 있는 값을 유니언 타입으로 표현하면 어떤 장점이 있나요?

      #### 1) 허용 범위를 정확히 제한한다

        ```tsx
        let status: string;                          // 아무 문자열이나 가능
        let status: "CHALLENGING" | "COMPLETED";     // 두 값만 가능
        ```

        - `string`으로 두면 `"COMPLETE"`, `"완료"` 같은 오타가 그대로 통과한다. 유니언으로 제한하면 **오타가 작성 시점에 잡힌다.**
        - DB의 ENUM과 같은 역할이다.

      #### 2) any보다 안전하다

        ```tsx
        let id: any;              // 타입 검사 포기
        let id: string | number;  // 둘 중 하나임을 보장
        ```

        - `any`는 무엇이든 허용해 TypeScript를 쓰는 의미가 사라진다. 유니언은 유연하면서도 검사를 유지한다.

      #### 3) 처리 누락을 방지한다

        ```tsx
        function find(id: number): StudyMember | undefined { }
        
        const m = find(1);
        m.name;   // ❌ 'm' is possibly 'undefined'
        ```

        - "못 찾을 수도 있다"를 타입에 명시하면, **그 경우를 처리하지 않으면 컴파일되지 않는다.** 런타임 오류의 대표 원인을 미리 차단한다.

      #### 4) 자동완성이 동작한다

        - 선택지가 정해져 있으므로 에디터가 가능한 값을 띄워준다. 문서를 찾아볼 필요가 줄어든다.
    - 조건문, `typeof`와 판별 프로퍼티는 타입을 어떻게 좁히나요?
        - 유니언 타입은 **여러 타입의 공통 기능만** 사용할 수 있다.

        ```tsx
        function print(value: string | number) {  value.toUpperCase();   // ❌ number에는 없는 메서드}
        ```

        - 그래서 사용 전에 타입을 하나로 확정해야 한다. 이를 **타입 좁히기(Narrowing)** 라고 한다.

      #### 1) typeof — 원시 타입 구분

        ```tsx
        function print(value: string | number) {
          if (typeof value === "string") {
            value.toUpperCase();   // 여기선 string
          } else {
            value.toFixed(2);      // 여기선 number
          }
        }
        ```

        - `string`, `number`, `boolean`, `undefined`, `function` 등에 사용한다.

      #### 2) 조건문 — null / undefined 제거

        ```tsx
        const member = findMember(id);
        
        if (member === undefined) {
          return "회원을 찾을 수 없습니다.";
        }
        
        member.name;   // 여기서는 StudyMember로 확정
        ```

        - 먼저 걸러내고 빠져나가는 방식(early return)이 중첩을 줄여 읽기 좋다.

      #### 3) 판별 프로퍼티 — 객체 구분

        - 각 타입에 **공통 이름의 리터럴 타입 프로퍼티**를 두고 그 값으로 구분한다.

        ```tsx
        type Success = { status: "success"; data: string };
        type Failure = { status: "error"; message: string };
        
        function handle(result: Success | Failure) {
          if (result.status === "success") {
            result.data;      // Success로 확정
          } else {
            result.message;   // Failure로 확정
          }
        }
        ```

      객체는 `typeof`로 구분되지 않으므로(둘 다 `"object"`) 이 방식을 쓴다. API 응답 처리에 자주 사용된다.

      #### 그 외

      | 방법 | 용도 |
              | --- | --- |
      | `typeof` | 원시 타입 |
      | `instanceof` | 클래스 인스턴스 |
      | `in` | 특정 프로퍼티 존재 여부 |
      | `Array.isArray()` | 배열 여부 |
    - `null`과 `undefined`가 포함된 값을 안전하게 다룰 때 옵셔널 체이닝과 널 병합 연산자는 어떤 역할을 하나요?

      **옵셔널 체이닝 `?.`**

        - 앞의 값이 `null` 또는 `undefined`면 **접근을 멈추고 `undefined`를 반환한다.**

        ```tsx
        member?.githubId
        ```

      | `member` | 결과 |
              | --- | --- |
      | 객체 | `member.githubId` |
      | `null` / `undefined` | `undefined` (오류 없음) |
        - 기존 방식과 비교하면 차이가 분명하다.

        ```tsx
        // 이전
        const id = member && member.profile && member.profile.githubId;
        
        // 옵셔널 체이닝
        const id = member?.profile?.githubId;
        ```

        - `Cannot read properties of undefined` 오류를 막는 것이 핵심 역할이다.

      **함수와 배열에도 쓸 수 있다.**

        ```tsx
        callback?.();      // 함수가 있을 때만 호출list?.[0];         // 배열이 있을 때만 접근
        ```

      **널 병합 연산자 `??`**

        - 왼쪽이 `null` 또는 `undefined`일 때만 오른쪽 값을 사용한다.

        ```tsx
        const github = member.githubId ?? "미등록";
        ```

      **`||`와의 차이**

        - `||`는 falsy한 값 전부(`0`, `""`, `false`)를 대체한다.

        ```tsx
        const count = 0;
        
        count || 10;   // 10  ← 0이 유효한 값인데 무시됨
        count ?? 10;   // 0   ← 의도대로 동작
        ```

        - **값이 없는 것과 값이 0·빈 문자열인 것을 구분해야 할 때 `??`를 써야 한다.** 포인트, 개수, 가격처럼 0이 의미를 갖는 데이터에서 중요하다.

      #### 함께 쓰기

        ```tsx
        const github = member?.profile?.githubId ?? "미등록";
        ```

        - `?.`로 안전하게 접근하고, 결과가 없으면 `??`로 기본값을 채운다. 자주 쓰이는 조합이다.

      #### 타입 좁히기와의 관계

        ```tsx
        // 값을 꺼내 쓸 때 — 연산자
        const name = member?.name ?? "이름 없음";
        
        // 분기 처리가 필요할 때 — 조건문
        if (member === undefined) {
          return "회원 없음";
        }
        ```

        - **값을 안전하게 꺼내는 것이 목적이면 연산자**, **없을 때 다른 로직을 실행해야 하면 조건문**을 쓴다. 옵셔널 체이닝은 타입을 좁히는 것이 아니라 접근을 건너뛰는 것이므로, 이후 코드에서 타입이 확정되지는 않는다.
- 타입 안전성과 제네릭
    - `any`와 `unknown`은 타입 검사를 허용하는 방식이 어떻게 다른가요?

      #### `any` — 검사를 꺼버린다

        ```tsx
        let value: any = "hello";
        
        value.toUpperCase();   // ✅ 통과
        value.toFixed(2);      // ✅ 통과 (실행하면 터짐)
        value.아무거나();       // ✅ 통과
        ```

        - TypeScript가 아예 검사를 포기한다. 오류를 없애주는 게 아니라 **런타임으로 미루는 것**이다.

      #### `unknown` — 확인을 강제한다

        ```tsx
        let value: unknown = "hello";
        
        value.toUpperCase();   // ❌ 에러
        
        if (typeof value === "string") {
          value.toUpperCase();   // ✅ 좁힌 뒤에는 가능
        }
        ```

        - 타입을 확인하기 전에는 아무것도 못 한다.

      #### 비교

      |  | `any` | `unknown` |
              | --- | --- | --- |
      | 값 할당 | 자유 | 자유 |
      | 사용 | 검사 없음 | 좁혀야 가능 |
      | 안전성 | 낮음 | 높음 |
        - **"타입을 모르겠다"면 `any`가 아니라 `unknown`을 쓴다.** API 응답, JSON 파싱 결과처럼 형태를 확신할 수 없는 값에 적합하다.
    - 여러 타입을 받을 때 `unknown`과 제네릭은 각각 어떤 상황에 알맞을까요?

      #### `unknown` — 타입을 모르고, 알 필요도 없을 때

        ```tsx
        function log(value: unknown) {
          console.log(value);
        }
        ```

        - 받아서 확인만 하고 끝나는 경우. **들어온 타입 정보가 나갈 때 필요 없다.**

      #### 제네릭 — 타입을 그대로 돌려줘야 할 때

        ```tsx
        function first<T>(list: T[]): T {
          return list[0];
        }
        
        first([1, 2, 3]);       // number
        first(["a", "b"]);      // string
        ```

        - `unknown`으로 만들면 이렇게 된다.

        ```tsx
        function first(list: unknown[]): unknown {
          return list[0];
        }
        
        const n = first([1, 2, 3]);
        n.toFixed(2);   // ❌ unknown이라 못 씀
        ```

        - **타입 정보가 사라져버린다.**

      #### 기준

      | 상황 | 선택 |
              | --- | --- |
      | 받기만 하고 끝 | `unknown` |
      | 받은 타입을 반환해야 함 | 제네릭 |
    - 제네릭은 입력 타입과 출력 타입의 관계를 어떻게 유지하나요?

      #### `T`는 **호출할 때 결정되는 타입 자리**다.

        - 직접 적지 않아도 전달한 값에서 추론된다.

        ```tsx
        function first<T>(list: T[]): T { }
        
        first([1, 2, 3]);
        //    ↑ number[] → T = number → 반환 number
        ```

        - 입력에서 확정된 `T`가 출력에도 그대로 쓰이므로 **"넣은 타입 = 나온 타입"이 보장된다.**

      #### **여러 타입 사이 관계도 표현할 수 있다**

        ```tsx
        function pair<K, V>(key: K, value: V): [K, V] {
          return [key, value];
        }
        
        pair("id", 1);   // [string, number]
        ```

      #### 제약 걸기

        ```tsx
        function getName<T extends { name: string }>(item: T): string {
          return item.name;
        }
        ```

        - `T`가 무엇이든 `name`은 있어야 한다고 제한한다. 제네릭의 유연함을 유지하면서 필요한 최소 조건만 요구한다.

      #### 실제 예시

        ```tsx
        type ApiResponse<T> = {
          status: number;
          data: T;
        };
        
        const res: ApiResponse<StudyMember> = { status: 200, data: member };
        res.data.name;   // ✅ StudyMember로 확정
        ```

        - 응답 껍데기는 공통이고 `data`만 달라질 때 쓰는 대표 패턴이다.