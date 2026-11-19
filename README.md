# Week 11 — Attachment Board (Student Sample)

Spring Boot + Thymeleaf + JPA + MariaDB: 게시글 CRUD / 다중 첨부파일 업로드 / 다운로드

## 실행하기
1. Java 17+ 및 MariaDB 준비, `CREATE DATABASE web11 CHARACTER SET utf8mb4;`
2. 환경변수 `DB_URL`, `DB_USERNAME`, `DB_PASSWORD`를 자신의 MariaDB에 맞게 설정. 기본값은 `jdbc:mariadb://localhost:3306/web11`, `webuser`, `webpass`.
3. Gradle이 설치된 환경: `gradle bootRun` / IntelliJ: `AttachmentBoardApplication.main()` 실행.
4. 접속: http://localhost:8080/posts

> Gradle Wrapper는 이 ZIP에 포함되어 있지 않습니다. IntelliJ의 Gradle 지원 또는 설치된 Gradle을 사용하세요. `gradle wrapper` 실행 후 생성된 wrapper 파일은 Git에 포함하는 것을 권장합니다.

## 구현된 기능
- 게시글 목록 / 생성 / 상세 / 수정 / 삭제
- 게시글 생성 시 여러 파일 선택, UUID 이름으로 `uploads/`에 저장
- `attachments` 테이블에 메타데이터 저장
- 첨부파일 다운로드 (`GET /attachments/{id}/download`)

## 학생 과제 (TODO)
1. 코드를 분석하여 업로드/다운로드 요청 흐름 정리
2. 화면이나 정보 표시 3곳 이상 수정
3. **이미지 미리보기** 구현: `GET /attachments/{id}/view` + 상세 화면 `<img>` 표시. 이미지(jpg/jpeg/png/gif)만 허용하고 `Content-Type` 정확히 설정
4. 정상/오류 테스트 후 README 및 Weekly Review 작성

## 코드 위치
- `PostController`: 게시글 화면 / Form
- `AttachmentController`: 다운로드 + 미리보기 TODO
- `PostService`: JPA CRUD와 파일 저장 연결
- `FileStorageService`: 파일 검증 / UUID 저장 / 로드
- `Post`, `Attachment`: 1:N 관계
- `templates/posts/detail.html`: 미리보기 TODO

## 주의사항
- 학습용 프로젝트입니다. 로그인/접근 권한, 업로드 파일의 실제 바이트 검증, 악성 파일 검사, CSRF 보호, 파일 저장과 DB 트랜잭션의 완전한 일관성 보장 등은 구현하지 않았습니다. 인터넷 공개 배포 전에 보강이 필요합니다.
- `spring.jpa.hibernate.ddl-auto=update`는 학습용 설정이며 운영 DB에는 그대로 사용하지 않습니다.
- Render 기본 파일 시스템에 저장된 업로드 파일은 재시작/재배포 시 유지되지 않을 수 있습니다. 이번 과제는 **로컬 파일 처리**를 평가합니다.

## 3. Source Code Exploration & Analysis

### ① Source Code Analysis

#### 1. Project Structure & Spring MVC
* Q1. PostController, PostService, PostRepository는 각각 어떤 역할을 하며, 서로 어떻게 연결되는가?
    * `PostController`는 사용자의 HTTP 요청을 받아 처리, 결과를 화면에 전달 역할을 함.
    * `PostService`는 게시글 작성, 조회, 수정 등의 핵심 비즈니스 로직을 담당함.
    * `PostRepository`는 JPA를 이용해 데이터베이스에 게시글 정보를 저장하거나 조회함.
    * 세 클래스는 Controller -> Service -> Repository 순서로 연결되어 각자의 역할을 분리하여 처리함.

* Q2. 게시글 작성 시 HTML Form의 입력값과 첨부파일은 Controller에 어떻게 전달되는가?
    * HTML Form에서 입력한 게시글 정보는 `name` 속성을 기준으로 HTTP 요청에 포함됨.
    * Controller의 `@ModelAttribute` 또는 `@RequestParam` 등을 통해 전달됨.
    * 첨부파일은 Form의 `enctype="multipart/form-data"` 설정을 통해 전송됨.
    * Controller에서 `MultipartFile` 또는 `List<MultipartFile>` 타입으로 전달받음.
    * 전달받은 데이터는 Service에 전달되어 게시글 저장과 파일 업로드 작업에 사용됨.

#### 2. JPA & Entity Relationship
* Q3. Post와 Attachment Entity는 어떤 관계이며, @ManyToOne과 @OneToMany는 각각 어떤 역할을 하는가?
    * 하나의 Post에 여러 개의 Attachment이 연결될 수 있으므로 두 Entity는 1:N의 관계임.
    * `@OneToMany`는 Post 입장에서 여러 Attachment를 관리할 때 사용하고,`@ManyToOne`은 여러 Attachment가 하나의 Post에 연결될 때 사용함.
  
* Q4. cascade = CascadeType.ALL과 orphanRemoval = true는 어떤 역할을 하며, 실제 디스크에 저장된 파일에도 영향을 주는가?
    * 

#### 3. File Upload
* Q5. MultipartFile은 무엇이며, 업로드된 파일은 어떤 과정을 거쳐 서버에 저장되는가?
    * 

* Q6. 파일을 원본 이름 대신 UUID 기반 파일명으로 저장하는 이유는 무엇인가?
    * 

#### 4. File Download & Security
* Q7. 파일 다운로드 시 Resource, ResponseEntity, Content-Disposition은 각각 어떤 역할을 하는가?
    * 

* Q8. 파일 저장 과정에서 normalize()와 startsWith()를 사용하는 이유는 무엇인가?
    *