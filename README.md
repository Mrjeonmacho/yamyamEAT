# yamyamEAT

## 1. 프로젝트 개요

**yamyamEAT**은 사용자가 주변 맛집을 검색하고, 상세 정보를 확인하며, 자신만의 맛집 리스트(위시리스트)를 관리할 수 있는 웹 애플리케이션입니다.

Spring Boot 백엔드가 REST API를 제공하고, Vue.js 기반의 프론트엔드가 사용자 인터페이스를 담당합니다. 초기 맛집 데이터는 Naver 지도를 크롤링하여 DB에 저장합니다.

## 2. 주요 기능

- **사용자 관리**:
    - 회원가입 및 로그인 기능
    - 사용자별 맛집 위시리스트 추가 및 삭제
- **맛집 정보**:
    - 전체 맛집 목록 조회
    - 맛집 상세 정보 확인 (이름, 주소, 평점 등)
- **데이터 수집**:
    - Selenium을 활용하여 Naver 지도의 맛집 정보를 크롤링하여 데이터베이스에 초기 데이터를 구축

## 3. 기술 스택

### Backend

- **Java 17**
- **Spring Boot 3.x**
- **Spring Data JPA**: 데이터베이스 연동
- **MySQL**: 데이터베이스
- **Selenium**: 웹 크롤링
- **Gradle**: 빌드 및 의존성 관리

### Frontend

- **Vue.js 3**
- **TypeScript**
- **Vite**: 프론트엔드 개발 서버 및 빌드 툴
- **Pinia**: 상태 관리
- **Vue Router**: 라우팅
- **Axios**: HTTP 통신

## 4. 프로젝트 구조

```
yamyamEAT/
├── build.gradle              # 프로젝트의 모든 의존성과 빌드 설정
├── src/
│   ├── main/
│   │   ├── frontend/         # Vue.js 프론트엔드 소스코드
│   │   │   ├── src/
│   │   │   ├── package.json
│   │   │   └── vite.config.ts
│   │   ├── java/ssafy/eat/demo/
│   │   │   ├── controller/   # API 엔드포인트를 정의하는 컨트롤러
│   │   │   ├── service/      # 비즈니스 로직
│   │   │   ├── repository/   # JPA 레포지토리 (DB 접근)
│   │   │   ├── domain/       # JPA 엔티티 (데이터 모델)
│   │   │   ├── crawling/     # Selenium 크롤링 로직
│   │   │   └── DemoApplication.java # Spring Boot 메인 애플리케이션
│   │   └── resources/
│   │       ├── application.properties # Spring 설정 (DB 연결 정보 등)
│   └── test/
```

- **통합 빌드**: Gradle을 사용하여 Vue.js 프론트엔드를 빌드한 후, 생성된 정적 파일(HTML, CSS, JS)을 Spring Boot 애플리케이션의 `static` 리소스로 복사합니다. 이를 통해 별도의 웹 서버 없이 백엔드 서버가 프론트엔드까지 모두 서빙합니다.

## 5. 데이터베이스 스키마

`ssafy_eat` 데이터베이스 사용. 주요 테이블은 다음과 같습니다.

- **`users`**: 사용자 정보 (id, password, name)
- **`restaurants`**: 맛집 정보 (id, name, address 등)
- **`naver_rating`, `kakao_rating`, `google_rating`**: 각 플랫폼별 평점 정보
- **`user_wishlist`**: 사용자와 맛집 간의 위시리스트 관계 (Many-to-Many)

자세한 스키마는 `src/main/java/ssafy/eat/demo/sql/table_schema.sql` 파일에서 확인할 수 있습니다.

## 6. API 주요 엔드포인트

### User Controller (`/api/users`)

- `POST /register`: 회원가입
- `POST /login`: 로그인
- `POST /{userId}/wishlist/{restaurantId}`: 위시리스트 추가
- `DELETE /{userId}/wishlist/{restaurantId}`: 위시리스트 삭제
- `GET /{userId}/wishlist`: 사용자의 위시리스트 목록 조회

### Restaurant Controller (`/api/restaurants`)

- `GET /`: 전체 맛집 목록 조회
- `GET /{id}`: 특정 맛집의 상세 정보 조회

## 7. 시작 가이드

### 사전 요구사항

- **Java 17** 설치
- **Node.js 18+** 설치
- **MySQL** 서버 실행 및 `ssafy_eat` 스키마 생성

### 1) 데이터베이스 설정

- `src/main/resources/application.properties` 파일에 자신의 MySQL DB 연결 정보를 수정합니다.

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/ssafy_eat?serverTimezone=UTC&characterEncoding=UTF-8
spring.datasource.username=[DB계정]
spring.datasource.password=[DB비밀번호]
```

### 2) 백엔드 실행

프로젝트 루트 디렉토리에서 다음 명령어를 실행하여 Spring Boot 애플리케이션을 시작합니다.

```bash
./gradlew bootRun
```

### 3) 프론트엔드 실행 (개발 시)

백엔드와 별개로 프론트엔드 개발 서버를 실행하여 빠른 UI 개발이 가능합니다.

```bash
cd src/main/frontend
npm install
npm run dev
```

**[중요]** 개발 시 프론트엔드(`localhost:5173`)와 백엔드(`localhost:8080`) 간의 API 요청은 CORS(Cross-Origin Resource Sharing) 문제를 발생시킬 수 있습니다. 이를 해결하려면 `src/main/frontend/vite.config.ts` 파일에 프록시 설정을 추가하거나, Spring Boot에 `@CrossOrigin` 어노테이션을 추가해야 합니다.

### 8. 초기 데이터 구축 (크롤링)

애플리케이션을 처음 실행할 때 Naver 지도 크롤링을 통해 맛집 데이터를 DB에 저장할 수 있습니다.

**주의**: 크롤링은 시간이 오래 걸릴 수 있으며, 대상 웹사이트의 구조가 변경되면 작동하지 않을 수 있습니다.

다음과 같이 `crawling` 프로파일을 활성화하여 애플리케이션을 실행합니다.

1.  프로젝트 빌드
    ```bash
    ./gradlew build
    ```

2.  `crawling` 프로파일로 실행
    ```bash
    java -jar -Dspring.profiles.active=crawling build/libs/demo-0.0.1-SNAPSHOT.jar
    ```
