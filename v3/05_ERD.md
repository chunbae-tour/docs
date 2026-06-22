# 05_ERD

> **문서 버전**: v3.0
> **최신 반영일**: 2026-06-22  
> **반영 기준**: 최종 ERD 이미지 + 기존 상세 Mermaid ERD

## 0. 전체 상세 ERD
<img width="5427" height="5475" alt="image" src="https://github.com/user-attachments/assets/a2a742a6-a84a-419a-90a4-4bd469244679" />

아래 이미지는 전체 테이블과 관계를 한 번에 볼 수 있는 전체 상세 ERD이다.


## 0-1. 도메인 ERD 개요
<img width="4100" height="3025" alt="image" src="https://github.com/user-attachments/assets/f5297cff-65e8-45db-bd45-4a14b441cbf5" />


## 0-2. 도메인별 상세 ERD


### 1. 인증·유저·제재 도메인
<img width="2450" height="1850" alt="image" src="https://github.com/user-attachments/assets/eb4ae726-9a29-4d3f-9dbd-c3bd7d003b96" />


### 2. 커뮤니티 도메인
<img width="2400" height="2250" alt="image" src="https://github.com/user-attachments/assets/226e1fb0-e258-4b1d-9228-10d1a059db64" />


### 3. 동행·리뷰·채팅 도메인
<img width="3525" height="2850" alt="image" src="https://github.com/user-attachments/assets/3e5eb8ef-47ea-4fb8-ac10-61eebbed87f0" />


### 4. 관광지·전통시장·축제 도메인
<img width="3200" height="2350" alt="image" src="https://github.com/user-attachments/assets/06aac9b8-9ae4-4443-aa55-9de3bf720717" />


### 5. 가게·상인 도메인
<img width="3600" height="2675" alt="image" src="https://github.com/user-attachments/assets/af3115cc-5545-49a5-93f6-8647f6777902" />


### 6. 결제·스토어·엽전 도메인
<img width="3575" height="2750" alt="image" src="https://github.com/user-attachments/assets/9708ec5b-13ef-4143-a0f6-58805450e5ba" />


### 7. CS·신고·감사·알림·기타 도메인
<img width="1179" height="1008" alt="image" src="https://github.com/user-attachments/assets/01f58137-7c7e-4f95-8311-d7f04a7e2475" />



## 0-3. ERD 해석 기준

| 구분 | 설명 |
| --- | --- |
| 실선 관계 | 실제 DB FK 또는 aggregate 내부 강한 관계 |
| 점선 관계 | 도메인 간 논리 참조, DB FK 없이 ID 값으로 연결 |
| 사용자 참조 | 대부분 도메인이 `users.id`를 논리참조하며, 일부는 실제 FK 대신 인덱스/서비스 검증으로 보장 |
| 다형 참조 | 신고, 알림, 좋아요, 댓글 등은 target type + target id로 여러 도메인을 연결 |
| Redis 데이터 | 검색 랭킹, 최근 검색어, 지도 Geo, 캐시, write-behind counter는 DB ERD가 아닌 운영 데이터 구조로 별도 관리 |

## 1. 도메인별 테이블 목록

| 도메인 | 테이블 | 담당 |
| --- | --- | --- |
| 회원/인증 | users, merchant_applications | 정민교 |
| 관리자 | faqs, banners, suspensions, admin_action_logs | 정민교 |
| 관광지/지도 | places, place_sync_state, user_likes | 김인목 |
| 전통시장 | traditional_markets | 김인목 |
| 검색 | (Redis 중심, DB 보조) | 김인목 |
| 축제/캘린더 | festivals | 박경화 |
| 커뮤니티 | free_posts, free_post_images, companion_posts, comments | 박경화 |
| 채팅/동행 | chat_rooms, chat_room_members, join_requests, messages, companions, companion_participants, companion_reviews | 임하은 |
| CS/알림 | notifications, support_rooms, support_messages | 임하은 |
| 번역/로그 | translation_cache, common_error_logs | 임하은 |
| 제재 | user_sanction | 정민교 |
| 결제/엽전 | wallets, payment_orders, yeopjeon_histories, refunds, qr_pay_requests | 신현민 |
| 스토어 | products, store_orders, user_items | 신현민 |
| 상인/가게 | shops, shop_images, shop_notices, menus, shop_wallets, settlements, ad_applications, store_certifications | 신현민 |
| 관광지 리뷰 | place_reviews | 신현민 |

---

## 2. 전체 ERD (Mermaid)

> 첫 번째 러프 ERD는 제거하고, enum/json/decimal 기반 정제 ERD만 유지한다.
> 

```mermaid
erDiagram

  users {
    bigint id PK
    varchar email
    varchar password
    varchar name
    varchar phone
    varchar phone_hash
    date birthdate
    varchar oauth_provider
    varchar oauth_id
    varchar nickname
    varchar profile_image_url
    varchar language
    float companion_score
    int companion_review_count
    enum role
    enum status
    timestamp suspended_until
    varchar suspended_reason
    varchar sanction_type
    timestamp sanction_end_at
    timestamp created_at
    timestamp updated_at
    timestamp deleted_at
  }

  merchant_applications {
    bigint id PK
    bigint user_id FK
    varchar shop_name
    varchar business_number
    varchar category
    varchar address
    decimal lat
    decimal lng
    varchar phone
    text description
    enum status
    varchar reject_reason
    varchar active_flag
    timestamp created_at
    timestamp updated_at
  }

  suspensions {
    bigint id PK
    bigint user_id FK
    bigint admin_id FK
    text reason
    timestamp suspended_until
    timestamp created_at
  }

  places {
    bigint id PK
    varchar name
    enum category
    text description
    varchar address
    point location
    varchar thumbnail_url
    json image_urls
    varchar operating_hours
    varchar closed_days
    varchar phone
    varchar admission_fee
    int rating
    int review_count
    int like_count
    int view_count
    json tags
    enum status
    varchar external_id
    varchar source
    varchar external_modified_time
    int enrich_attempt_count
    varchar sido
    varchar sigungu
    bigint version
    timestamp created_at
    timestamp updated_at
  }

  user_likes {
    bigint id PK
    bigint user_id FK
    varchar target_type
    bigint target_id
    timestamp created_at
  }

  festivals {
    bigint id PK
    varchar name
    text description
    varchar region
    varchar location
    decimal lat
    decimal lng
    date start_date
    date end_date
    varchar thumbnail_url
    json image_urls
    enum status
    varchar source
    varchar category
    varchar external_id
    timestamp created_at
    timestamp updated_at
  }

  free_posts {
    bigint id PK
    bigint author_id FK
    varchar title
    text content
    bigint view_count
    enum status
    timestamp created_at
    timestamp updated_at
    timestamp deleted_at
  }

  free_post_images {
    bigint post_id FK
    varchar image_url
    int sort_order
  }

  place_reviews {
    bigint id PK
    bigint place_id FK
    bigint user_id FK
    int rating
    varchar content
    json image_urls
    varchar status
    timestamp created_at
    timestamp updated_at
  }

  comments {
    bigint id PK
    bigint author_id FK
    bigint post_id FK
    enum post_type
    bigint parent_comment_id FK
    varchar content
    enum status
    timestamp created_at
    timestamp updated_at
    timestamp deleted_at
  }

  reports {
    bigint id PK
    bigint reporter_id FK
    bigint reported_user_id FK
    enum target_type
    bigint target_id
    enum reason
    varchar description
    enum status
    enum action
    text admin_note
    varchar resolved_by
    timestamp resolved_at
    bigint version
    timestamp created_at
    timestamp updated_at
  }

  chat_rooms {
    bigint id PK
    bigint post_id FK
    bigint owner_id FK
    varchar title
    text description
    int max_members
    int current_members
    enum status
    bigint version
    timestamp created_at
    timestamp updated_at
  }

  chat_room_members {
    bigint id PK
    bigint chat_room_id FK
    bigint user_id FK
    enum member_state
    timestamp joined_at
    timestamp left_at
  }

  join_requests {
    bigint id PK
    bigint chat_room_id FK
    bigint user_id FK
    text message
    bigint pending_key
    enum status
    timestamp created_at
    timestamp updated_at
  }

  messages {
    bigint id PK
    bigint chat_room_id FK
    bigint sender_id FK
    enum message_type
    text content
    varchar file_url
    varchar file_name
    bigint file_size
    text translated_content
    varchar translate_lang
    timestamp sent_at
  }
  
  notifications {
    bigint id PK
    bigint user_id FK
    enum type
    varchar title
    text message
    enum reference_type
    bigint reference_id
    boolean is_read
    timestamp created_at
    timestamp deleted_at
  }

  support_rooms {
    bigint id PK
    bigint user_id FK
    bigint admin_id FK
    enum status
    text summary
    timestamp created_at
    timestamp updated_at
    timestamp closed_at
  }

  support_messages {
    bigint id PK
    bigint support_room_id FK
    bigint sender_id FK
    enum sender_role
    enum message_type
    text content
    varchar file_url
    varchar file_name
    bigint file_size
    timestamp sent_at
  }

  companion_reviews {
    bigint id PK
    bigint reviewer_id FK
    bigint target_user_id FK
    bigint chat_room_id FK
    int score
    text content
    timestamp created_at
  }

  wallets {
    bigint id PK
    bigint user_id FK
    bigint balance
    timestamp created_at
    timestamp updated_at
  }

  payment_orders {
    bigint id PK
    bigint user_id FK
    varchar order_uid
    varchar idempotency_key
    bigint amount
    enum payment_method
    enum status
    varchar pg_order_id
    varchar pg_transaction_id
    timestamp created_at
    timestamp updated_at
  }

  yeopjeon_histories {
    bigint id PK
    bigint user_id FK
    bigint payment_order_id FK
    bigint shop_id FK
    varchar pay_request_id
    enum type
    bigint amount
    bigint balance_snapshot
    text description
    timestamp created_at
  }

  refunds {
    bigint id PK
    bigint user_id FK
    bigint payment_order_id FK
    bigint amount
    varchar reason
    enum status
    varchar reject_reason
    int retry_count
    timestamp next_retry_at
    timestamp created_at
    timestamp updated_at
  }

  qr_pay_requests {
    bigint id PK
    varchar pay_request_id
    varchar pending_key
    bigint user_id FK
    bigint shop_id FK
    bigint amount
    json menu_items
    enum status
    text reject_reason
    timestamp expired_at
    timestamp completed_at
    timestamp created_at
    timestamp updated_at
  }

  products {
    bigint id PK
    varchar name
    text description
    varchar category
    bigint price
    bigint original_price
    varchar merchant_name
    int stock
    int original_stock
    int max_per_person
    json image_urls
    int validity_days
    varchar redemption_scope
    enum status
    timestamp created_at
    timestamp updated_at
  }

  store_orders {
    bigint id PK
    bigint user_id FK
    bigint product_id FK
    varchar product_name
    bigint product_price
    int quantity
    bigint total_price
    enum status
    timestamp created_at
    timestamp updated_at
  }

  user_items {
    bigint id PK
    bigint user_id FK
    bigint product_id FK
    varchar product_name
    bigint order_id FK
    bigint used_shop_id FK
    enum status
    date expires_at
    timestamp used_at
    timestamp created_at
    timestamp updated_at
  }

  traditional_markets {
    bigint id PK
    varchar name
    varchar address
    decimal lat
    decimal lng
    varchar market_type
    varchar phone_number
    varchar homepage_url
    int establish_year
    varchar sido
    varchar sigungu
    timestamp created_at
    timestamp updated_at
  }

  shop_notices {
    bigint id PK
    bigint shop_id FK
    varchar title
    text content
    timestamp created_at
    timestamp updated_at
  }

  shop_images {
    bigint id PK
    bigint shop_id FK
    varchar object_key
    varchar type
    int sort_order
    boolean is_primary
    timestamp created_at
    timestamp updated_at
  }

  companions {
    bigint id PK
    bigint chat_room_id FK
    enum status
    date trip_start_date
    date trip_end_date
    timestamp started_at
    timestamp ended_at
    timestamp created_at
    timestamp updated_at
  }

  companion_participants {
    bigint id PK
    bigint companion_id FK
    bigint user_id FK
    timestamp added_at
    timestamp ended_at
    timestamp created_at
    timestamp updated_at
  }

  shops {
    bigint id PK
    bigint user_id FK
    bigint application_id FK
    bigint place_id FK
    bigint traditional_market_id FK
    varchar shop_name
    varchar category
    varchar address
    decimal lat
    decimal lng
    varchar phone
    text description
    varchar operating_hours
    varchar closed_days
    varchar qr_nonce
    boolean is_certified
    float rating
    int review_count
    enum status
    bigint version
    timestamp created_at
    timestamp updated_at
  }

  menus {
    bigint id PK
    bigint shop_id FK
    varchar name
    text description
    bigint price
    varchar image_url
    boolean is_available
    timestamp created_at
    timestamp updated_at
    timestamp deleted_at
  }

  shop_wallets {
    bigint id PK
    bigint shop_id FK
    bigint balance
    bigint version
    varchar bank_name
    varchar account_number
    varchar account_holder
    timestamp created_at
    timestamp updated_at
  }

  settlements {
    bigint id PK
    bigint shop_id FK
    bigint amount
    enum status
    varchar reject_reason
    varchar account_holder
    varchar account_number
    varchar bank_name
    timestamp created_at
    timestamp updated_at
  }

  ad_applications {
    bigint id PK
    bigint shop_id FK
    enum ad_type
    date start_date
    date end_date
    bigint cost
    varchar reject_reason
    enum status
    timestamp created_at
  }

  store_certifications {
    bigint id PK
    bigint shop_id FK
    enum status
    text reject_reason
    bigint reviewed_by FK
    timestamp reviewed_at
    timestamp created_at
  }

  companion_posts {
    bigint id PK
    bigint author_id FK
    varchar title
    text content
    varchar target_type
    bigint target_id
    varchar target_name
    varchar region
    date meeting_date
    int max_members
    int current_members
    bigint view_count
    enum status
    timestamp created_at
    timestamp updated_at
    timestamp deleted_at
  }

  admin_action_logs {
    bigint id PK
    bigint admin_user_id FK
    varchar action_type
    varchar target_type
    bigint target_id
    varchar reason
    varchar before_status
    varchar after_status
    timestamp created_at
  }

  user_sanction {
    bigint id PK
    bigint user_id FK
    bigint report_id FK
    varchar target_type
    varchar sanction_type
    varchar reason
    timestamp started_at
    timestamp ended_at
    timestamp released_at
    bigint released_by FK
    timestamp created_at
  }

  translation_cache {
    bigint id PK
    char content_hash
    varchar target_language
    text translated_content
    timestamp created_at
    timestamp updated_at
  }

  place_sync_state {
    bigint id PK
    varchar last_modified_time
    timestamp updated_at
  }

  common_error_logs {
    bigint id PK
    varchar domain
    varchar error_type
    text message
    text detail
    varchar external_provider
    timestamp created_at
  }

  faqs {
    bigint id PK
    varchar question
    text answer
    varchar category
    boolean is_active
    timestamp created_at
    timestamp updated_at
  }

  banners {
    bigint id PK
    varchar title
    varchar image_url
    varchar link_url
    enum status
    int sort_order
    timestamp start_at
    timestamp end_at
    timestamp created_at
  }

  users ||--o{ merchant_applications : "신청"
  users ||--o{ suspensions : "정지 이력"
  users ||--o| wallets : "보유"
  users ||--o{ user_likes : "찜"
  users ||--o{ free_posts : "작성"
  users ||--o{ comments : "작성"
  users ||--o{ place_reviews : "작성"
  users ||--o{ reports : "신고"
  users ||--o{ chat_room_members : "참여"
  users ||--o{ join_requests : "신청"
  users ||--o{ messages : "전송"
  users ||--o{ notifications : "수신"
  users ||--o{ support_rooms : "상담 요청"
  users ||--o{ support_messages : "상담 메시지"
  users ||--o{ companion_reviews : "작성"
  users ||--o{ payment_orders : "결제"
  users ||--o{ yeopjeon_histories : "내역"
  users ||--o{ refunds : "환불"
  users ||--o{ store_orders : "구매"
  users ||--o{ user_items : "보유"
  users ||--o{ companion_posts : "작성"
  users ||--o{ user_sanction : "제재 이력"
  users ||--o| shops : "운영"

  places ||--o{ place_reviews : "리뷰"

  free_posts ||--o{ free_post_images : "이미지"
  free_posts ||--o{ comments : "댓글"
  companion_posts ||--o{ comments : "댓글"
  companion_posts ||--o| chat_rooms : "채팅방"

  chat_rooms ||--o{ chat_room_members : "멤버"
  chat_rooms ||--o{ join_requests : "신청"
  chat_rooms ||--o{ messages : "메시지"
  chat_rooms ||--o| companions : "동행"
  chat_rooms ||--o{ companion_reviews : "리뷰"

  companions ||--o{ companion_participants : "참여자"

  support_rooms ||--o{ support_messages : "메시지"

  shops ||--o{ menus : "메뉴"
  shops ||--o{ shop_images : "이미지"
  shops ||--o{ shop_notices : "공지"
  shops ||--o| shop_wallets : "지갑"
  shops ||--o{ settlements : "정산"
  shops ||--o{ ad_applications : "광고"
  shops ||--o{ store_certifications : "인증"
  shops ||--o{ qr_pay_requests : "QR 결제"

  merchant_applications ||--o| shops : "생성"

  payment_orders ||--o{ yeopjeon_histories : "이력"
  payment_orders ||--o{ refunds : "환불"

  store_orders ||--o{ user_items : "아이템"
  products ||--o{ user_items : "보유"
  products ||--o{ store_orders : "구매"
```

---

## 3. 테이블 상세 설계

### 3.1 users

| 컬럼 | 타입 | 제약 | 설명 |
| --- | --- | --- | --- |
| id | BIGINT | PK, AUTO_INCREMENT | 사용자 ID |
| email | VARCHAR(255) | UNIQUE, NULL | 이메일 (소셜 가입은 NULL 가능) |
| password | VARCHAR(255) | NULL | BCrypt 비밀번호 (소셜 가입은 NULL) |
| name | VARCHAR(50) | NULL | 실명 (소셜 로그인 연동) |
| phone | VARCHAR(255) | NULL | 전화번호 (AES-GCM 암호화 저장) |
| phone_hash | VARCHAR(64) | UNIQUE, NULL | 전화번호 HMAC-SHA256 해시 (중복 가입 방지) |
| birthdate | DATE | NULL | 생년월일 |
| oauth_provider | VARCHAR(20) | NULL | kakao / naver |
| oauth_id | VARCHAR(100) | NULL | 소셜 고유 ID |
| nickname | VARCHAR(50) | UNIQUE, NOT NULL | 닉네임 |
| profile_image_url | VARCHAR(500) | NULL | 프로필 이미지 S3 URL |
| language | VARCHAR(10) | NOT NULL, DEFAULT ‘ko’ | 사용자 언어 설정 (ko, en, ja, zh 등) |
| companion_score | FLOAT | DEFAULT 0 | 동행 평균 점수 |
| companion_review_count | INT | DEFAULT 0 | 동행 리뷰 수 |
| role | ENUM | NOT NULL | USER / MERCHANT / ADMIN |
| status | ENUM | NOT NULL, DEFAULT ACTIVE | 계정 상태 |
| suspended_until | TIMESTAMP | NULL | 정지 해제 일시 |
| suspended_reason | VARCHAR(500) | NULL | 정지 사유 |
| sanction_type | VARCHAR(20) | NULL | 현재 제재 유형 |
| sanction_end_at | TIMESTAMP | NULL | 현재 제재 종료 일시 |
| created_at | TIMESTAMP | NOT NULL | 가입일 |
| updated_at | TIMESTAMP | NOT NULL | 수정일 |
| deleted_at | TIMESTAMP | NULL | 탈퇴일 (Soft Delete) |

**인덱스**

```sql
INDEX idx_users_email (email)
INDEX idx_users_nickname (nickname)
INDEX idx_users_status (status)
```

---

### 3.2 user_likes

| 컬럼 | 타입 | 제약 | 설명 |
| --- | --- | --- | --- |
| id | BIGINT | PK, AUTO_INCREMENT | |
| user_id | BIGINT | FK, NOT NULL | 사용자 ID |
| target_type | VARCHAR(50) | NOT NULL | PLACE / MARKET / FESTIVAL |
| target_id | BIGINT | NOT NULL | 찜 대상 ID |
| created_at | TIMESTAMP | NOT NULL | 찜 일시 |

**제약**
```sql
UNIQUE KEY uk_user_likes_type_target (user_id, target_type, target_id)
INDEX idx_user_likes_target (target_type, target_id)
```

> V202606101155 마이그레이션으로 `place_id` 단일 FK → `target_type + target_id` 다형 참조로 전환됨. PLACE·MARKET·FESTIVAL 3종 모두 동일 테이블에서 관리.

---

### 3.4 traditional_markets

| 컬럼 | 타입 | 제약 | 설명 |
| --- | --- | --- | --- |
| id | BIGINT | PK, AUTO_INCREMENT | |
| name | VARCHAR(100) | NOT NULL | 시장명 |
| address | VARCHAR(255) | NOT NULL | 주소 |
| lat | DECIMAL(10,7) | NOT NULL | 위도 |
| lng | DECIMAL(10,7) | NOT NULL | 경도 |
| market_type | VARCHAR(50) | NULL | 시장 유형 (공공데이터 분류) |
| phone_number | VARCHAR(20) | NULL | 전화번호 |
| homepage_url | VARCHAR(500) | NULL | 홈페이지 |
| establish_year | INT | NULL | 개설 연도 |
| created_at | TIMESTAMP | NOT NULL | |
| updated_at | TIMESTAMP | NOT NULL | |

> 공공데이터포털 API 자동 수집. `shops.traditional_market_id`로 입점 가게와 연결.

---

### 3.5 places

| 컬럼 | 타입 | 제약 | 설명 |
| --- | --- | --- | --- |
| id | BIGINT | PK | 장소 ID |
| name | VARCHAR(100) | NOT NULL | 장소명 |
| category | ENUM | NOT NULL | TOURIST_SPOT / TRADITIONAL_MARKET |
| description | TEXT | NULL | 소개 |
| address | VARCHAR(255) | NOT NULL | 주소 |
| lat | DECIMAL(10,7) | NOT NULL | 위도 |
| lng | DECIMAL(10,7) | NOT NULL | 경도 |
| thumbnail_url | VARCHAR(500) | NULL | 대표 이미지 |
| image_urls | JSON | NULL | 이미지 URL 배열 |
| operating_hours | VARCHAR(100) | NULL | 운영시간 |
| closed_days | VARCHAR(100) | NULL | 휴무일 |
| phone | VARCHAR(20) | NULL | 전화번호 |
| admission_fee | VARCHAR(50) | NULL | 입장료 정보 |
| rating | INT | DEFAULT 0 | 평균 별점 ×10 정수 (4.5점=45) |
| review_count | INT | DEFAULT 0 | 리뷰 수 |
| like_count | INT | DEFAULT 0 | 찜 수 |
| view_count | INT | DEFAULT 0 | 조회수 |
| tags | JSON | NULL | 태그 배열 |
| status | ENUM | NOT NULL, DEFAULT ACTIVE | ACTIVE / HIDDEN / DELETED |
| external_id | VARCHAR(64) | UNIQUE, NULL | KorService2 contentid (MANUAL은 NULL) |
| source | VARCHAR(20) | NOT NULL, DEFAULT 'MANUAL' | MANUAL / API_FETCH |
| external_modified_time | VARCHAR(14) | NULL | 외부 API modifiedtime (yyyyMMddHHmmss) |
| enrich_attempt_count | INT | DEFAULT 0 | 상세 수집 재시도 횟수 |
| sido | VARCHAR(20) | NULL | 시도 |
| sigungu | VARCHAR(30) | NULL | 시군구 |
| version | BIGINT | DEFAULT 0 | 낙관적 락 (@Version) |
| created_at | TIMESTAMP | NOT NULL | 등록일 |
| updated_at | TIMESTAMP | NOT NULL | 수정일 |

> `lat/lng` (DECIMAL) → `location POINT SRID 4326` (V202606041400). Spatial Index로 위치 쿼리. `rating`은 float → INT×10 (정수 정렬 최적화).

**인덱스**

```sql
SPATIAL INDEX idx_places_location (location)
INDEX idx_places_active_rating_id (status, rating DESC, id DESC)
INDEX idx_places_active_category_rating_id (status, category, rating DESC, id DESC)
FULLTEXT INDEX idx_places_name (name)
UNIQUE INDEX uk_places_external_id (external_id)
```

---

### 3.3 wallets

| 컬럼 | 타입 | 제약 | 설명 |
| --- | --- | --- | --- |
| id | BIGINT | PK | 지갑 ID |
| user_id | BIGINT | FK, UNIQUE | 사용자 ID (1:1) |
| balance | BIGINT | NOT NULL, DEFAULT 0 | 엽전 잔액 |
| created_at | TIMESTAMP | NOT NULL | 생성일 |
| updated_at | TIMESTAMP | NOT NULL | 수정일 |

> ⚠️ **balance는 절대 음수가 되면 안 됨** → CHECK(balance >= 0) 제약 추가 권장
동시성 제어는 분산 락 + 비관적 락으로 보호 (동시성 제어 설계서 참조)
> 

---

### 3.4 chat_rooms

| 컬럼 | 타입 | 제약 | 설명 |
| --- | --- | --- | --- |
| id | BIGINT | PK | 채팅방 ID |
| post_id | BIGINT | FK | 연결된 동행 게시글 |
| owner_id | BIGINT | FK | 방장 userId |
| title | VARCHAR(50) | NOT NULL | 채팅방 제목 |
| description | TEXT | NULL | 채팅방 소개 |
| max_members | INT | NOT NULL | 최대 인원 (2~50) |
| current_members | INT | NOT NULL, DEFAULT 1 | 현재 인원 |
| status | ENUM | NOT NULL, DEFAULT OPEN | OPEN / FULL / CLOSED |
| version | BIGINT | NOT NULL, DEFAULT 0 | 낙관적 락 (@Version) |
| created_at | TIMESTAMP | NOT NULL | 생성일 |
| updated_at | TIMESTAMP | NOT NULL | 수정일 |

**인덱스**

```sql
INDEX idx_chat_rooms_status (status)
INDEX idx_chat_rooms_owner (owner_id)
```

---

### 3.5 messages

| 컬럼 | 타입 | 제약 | 설명 |
| --- | --- | --- | --- |
| id | BIGINT | PK | 메시지 ID |
| chat_room_id | BIGINT | FK | 채팅방 ID |
| sender_id | BIGINT | FK | 발신자 userId |
| message_type | ENUM | NOT NULL | TEXT / IMAGE / FILE / SYSTEM |
| content | TEXT | NULL | 텍스트 메시지 내용 또는 파일/이미지 캡션 |
| file_url | VARCHAR(500) | NULL | IMAGE/FILE 메시지의 S3 URL |
| file_name | VARCHAR(255) | NULL | 원본 파일명 |
| file_size | BIGINT | NULL | 파일 크기(byte) |
| translated_content | TEXT | NULL | 번역된 메시지 |
| translate_lang | VARCHAR(10) | NULL | 번역 대상 언어 |
| sent_at | TIMESTAMP | NOT NULL | 전송 시각 |

**인덱스**

```sql
INDEX idx_messages_chat_room (chat_room_id, sent_at DESC)  -- 메시지 목록 조회
```

> ⚠️ 메시지는 대용량 데이터로 증가하므로, 향후 파티셔닝 또는 NoSQL(MongoDB) 이관 검토 권장
> 

---

### 3.6 payment_orders

| 컬럼 | 타입 | 제약 | 설명 |
| --- | --- | --- | --- |
| id | BIGINT | PK | 결제 ID |
| user_id | BIGINT | FK | 사용자 ID |
| order_uid | VARCHAR(100) | UNIQUE | 주문 번호 |
| idempotency_key | VARCHAR(100) | UNIQUE | 멱등성 키 |
| amount | BIGINT | NOT NULL | 결제 금액 (원) |
| payment_method | ENUM | NOT NULL | CARD / FOREIGN_CARD / KAKAO_PAY / TOSS_PAY |
| status | ENUM | NOT NULL | PENDING / COMPLETED / FAILED / CANCELLED / PARTIAL_CANCELLED / ADJUSTMENT_REQUIRED / REFUNDED |
| pg_order_id | VARCHAR(100) | NULL | PG사 주문 ID |
| pg_transaction_id | VARCHAR(100) | NULL | PG사 트랜잭션 ID |
| created_at | TIMESTAMP | NOT NULL | 요청일 |
| updated_at | TIMESTAMP | NOT NULL | 수정일 |

**인덱스**

```sql
UNIQUE INDEX idx_payment_idempotency (idempotency_key)  -- 중복 결제 방지
INDEX idx_payment_user (user_id, created_at DESC)
```

---

### 3.7 products

| 컬럼 | 타입 | 제약 | 설명 |
| --- | --- | --- | --- |
| id | BIGINT | PK | 상품 ID |
| name | VARCHAR(100) | NOT NULL | 상품명 |
| description | TEXT | NULL | 상품 설명 |
| category | VARCHAR(50) | NOT NULL | ADMISSION_TICKET / TOUR_PASS / EXPERIENCE / DISCOUNT_COUPON / LOCAL_PRODUCT |
| price | BIGINT | NOT NULL | 판매가 (엽전) |
| original_price | BIGINT | NULL | 정가 (할인 전) |
| merchant_name | VARCHAR(100) | NULL | 제공 상인명 (비정규화) |
| stock | INT | NOT NULL | 현재 재고 |
| original_stock | INT | NOT NULL | 초기 재고 |
| max_per_person | INT | NULL | 인당 최대 구매 수량 |
| image_urls | JSON | NULL | 이미지 URL 배열 |
| validity_days | INT | NULL | 유효기간 (일) |
| redemption_scope | VARCHAR(50) | NULL | 사용 가능 가게 범위 |
| status | ENUM | NOT NULL | ON_SALE / SOLD_OUT / HIDDEN |
| created_at | TIMESTAMP | NOT NULL | 등록일 |
| updated_at | TIMESTAMP | NOT NULL | 수정일 |

> ⚠️ **stock은 절대 음수가 되면 안 됨** → CHECK(stock >= 0) 제약 추가 권장
동시성 제어는 Redis 가점유 + 분산 락 + 비관적 락으로 보호
> 

---

### 3.8 shops

| 컬럼 | 타입 | 제약 | 설명 |
| --- | --- | --- | --- |
| id | BIGINT | PK | 가게 ID |
| user_id | BIGINT | FK | 상인 userId |
| application_id | BIGINT | FK | 승인된 신청 ID |
| place_id | BIGINT | FK, NULL | 연결 관광지 (선택) |
| traditional_market_id | BIGINT | FK, NULL | 입점 전통시장 (선택) |
| shop_name | VARCHAR(50) | NOT NULL | 가게명 |
| category | VARCHAR(50) | NOT NULL | 음식/잡화 등 |
| address | VARCHAR(255) | NOT NULL | 주소 |
| lat | DECIMAL(10,7) | NOT NULL | 위도 |
| lng | DECIMAL(10,7) | NOT NULL | 경도 |
| phone | VARCHAR(20) | NULL | 전화번호 |
| description | TEXT | NULL | 가게 소개 |
| operating_hours | VARCHAR(100) | NULL | 운영시간 |
| closed_days | VARCHAR(100) | NULL | 휴무일 |
| is_certified | BOOLEAN | NOT NULL, DEFAULT FALSE | 인증 마크 여부 |
| rating | FLOAT | DEFAULT 0 | 평균 별점 |
| review_count | INT | DEFAULT 0 | 리뷰 수 |
| status | ENUM | NOT NULL | ACTIVE / SUSPENDED / CLOSED |
| version | BIGINT | NOT NULL, DEFAULT 0 | 낙관적 락 (@Version) |
| created_at | TIMESTAMP | NOT NULL | 등록일 |
| updated_at | TIMESTAMP | NOT NULL | 수정일 |

> `image_urls JSON` 컬럼은 `shop_images` 테이블로 완전 이전됨 (ADR-003, KAN-315).

### 3.x shop_images

| 컬럼 | 타입 | 제약 | 설명 |
| --- | --- | --- | --- |
| id | BIGINT | PK | |
| shop_id | BIGINT | FK, NOT NULL | 가게 ID |
| object_key | VARCHAR(512) | NOT NULL | S3 오브젝트 키 |
| type | VARCHAR(20) | NOT NULL | PROFILE / GALLERY |
| sort_order | INT | NOT NULL | 정렬 순서 |
| is_primary | BOOLEAN | NOT NULL | 대표 이미지 여부 |
| created_at | TIMESTAMP | NOT NULL | |
| updated_at | TIMESTAMP | NOT NULL | |

```sql
UNIQUE KEY uq_shop_images_shop_type_sort_order (shop_id, type, sort_order)
```

### 3.x shop_notices

| 컬럼 | 타입 | 제약 | 설명 |
| --- | --- | --- | --- |
| id | BIGINT | PK | |
| shop_id | BIGINT | FK, NOT NULL | 가게 ID |
| title | VARCHAR(100) | NOT NULL | 공지 제목 |
| content | TEXT | NOT NULL | 공지 내용 |
| created_at | TIMESTAMP | NOT NULL | |
| updated_at | TIMESTAMP | NOT NULL | |

### 3.x companions / companion_participants

**companions**

| 컬럼 | 타입 | 제약 | 설명 |
| --- | --- | --- | --- |
| id | BIGINT | PK | |
| chat_room_id | BIGINT | FK, UNIQUE | 채팅방 1:1 |
| status | ENUM | NOT NULL | ONGOING / ENDED |
| started_at | TIMESTAMP | NOT NULL | 동행 시작 일시 |
| ended_at | TIMESTAMP | NULL | 동행 완료 일시 |
| created_at | TIMESTAMP | NOT NULL | |
| updated_at | TIMESTAMP | NOT NULL | |

**companion_participants**

| 컬럼 | 타입 | 제약 | 설명 |
| --- | --- | --- | --- |
| id | BIGINT | PK | |
| companion_id | BIGINT | FK, NOT NULL | 동행 ID |
| user_id | BIGINT | FK, NOT NULL | 참여자 userId |
| added_at | TIMESTAMP | NOT NULL | 참여 일시 |
| created_at | TIMESTAMP | NOT NULL | |
| updated_at | TIMESTAMP | NOT NULL | |

```sql
UNIQUE KEY uq_companion_participant (companion_id, user_id)
```

### 3.9 free_posts / free_post_images (자유게시판)

**free_posts**

| 컬럼명 | 데이터 타입 | 제약 조건 | 설명 |
| --- | --- | --- | --- |
| id | BIGINT | PK, AUTO_INCREMENT | 게시글 ID |
| author_id | BIGINT | FK, NOT NULL | 작성자 userId |
| title | VARCHAR(255) | NOT NULL | 제목 |
| content | TEXT | NOT NULL | 본문 |
| view_count | BIGINT | NOT NULL, DEFAULT 0 | 조회수 |
| status | ENUM | NOT NULL | ACTIVE / HIDDEN / DELETED |
| created_at | TIMESTAMP | NOT NULL | |
| updated_at | TIMESTAMP | NOT NULL | |
| deleted_at | TIMESTAMP | NULL | Soft Delete |

**free_post_images**

| 컬럼명 | 데이터 타입 | 제약 조건 | 설명 |
| --- | --- | --- | --- |
| post_id | BIGINT | PK(복합), FK | 게시글 ID |
| image_url | VARCHAR(500) | NOT NULL | S3 이미지 URL |
| sort_order | INT | PK(복합), NOT NULL | 정렬 순서 |

```sql
PRIMARY KEY (post_id, sort_order)
```

> 자유게시글 이미지는 JSON 배열이 아닌 별도 테이블로 정규화됨.

### 3.10 comments

| 컬럼명 | 데이터 타입 | 제약 조건 | 설명 |
| --- | --- | --- | --- |
| id | BIGINT | PK, NOT NULL | 댓글 고유 식별자 |
| author_id | BIGINT | FK, NOT NULL | 댓글 작성자 userId |
| post_id | BIGINT | FK, NOT NULL | 대상 게시글 ID (free_posts 또는 companion_posts) |
| post_type | ENUM | NOT NULL | COMPANION / FREE (다형 참조) |
| parent_comment_id | BIGINT | FK, NULL | 부모 댓글 ID (대댓글인 경우) |
| content | VARCHAR(1000) | NOT NULL | 댓글 내용 |
| status | ENUM | NOT NULL | ACTIVE / DELETED / HIDDEN |
| created_at | TIMESTAMP | NOT NULL | 등록 일시 |
| updated_at | TIMESTAMP | NOT NULL | 수정 일시 |
| deleted_at | TIMESTAMP | NULL | 삭제 일시 (Soft Delete) |

### 3.x place_reviews (관광지 리뷰)

| 컬럼명 | 데이터 타입 | 제약 조건 | 설명 |
| --- | --- | --- | --- |
| id | BIGINT | PK, AUTO_INCREMENT | 리뷰 ID |
| place_id | BIGINT | FK, NOT NULL | 관광지 ID |
| user_id | BIGINT | FK, NOT NULL | 작성자 userId |
| rating | INT | NOT NULL | 별점 (1~5) |
| content | VARCHAR(500) | NOT NULL | 리뷰 내용 |
| image_urls | JSON | NULL | 이미지 URL 배열 |
| status | VARCHAR(10) | NOT NULL | ACTIVE / DELETED |
| created_at | TIMESTAMP | NOT NULL | |
| updated_at | TIMESTAMP | NOT NULL | |

```sql
UNIQUE KEY uk_place_review_active (user_id, place_id, active_status)  -- 1인 1ACTIVE 리뷰
INDEX idx_place_reviews_place (place_id)
INDEX idx_place_reviews_user (user_id)
```

> 1인 1리뷰(ACTIVE 상태 기준) MySQL 가상 컬럼 + UNIQUE 제약으로 DB 레벨 보장.

---

### 3.11 festivals

| 컬럼명 | 데이터 타입 | 제약 조건 | 설명 |
| --- | --- | --- | --- |
| id | BIGINT | PK, NOT NULL | 축제 고유 식별자 |
| name | VARCHAR(255) | NOT NULL | 축제 이름 |
| description | TEXT | NULL | 축제 상세 설명 |
| region | VARCHAR(100) | NOT NULL | 축제 개최 지역 (행정 구역 단위) |
| location | VARCHAR(255) | NOT NULL | 축제 개최 상세 장소 (주소 등) |
| lat | DECIMAL(10,7) | NOT NULL | 장소 위도 (Latitude) |
| lng | DECIMAL(11,7) | NOT NULL | 장소 경도 (Longitude) |
| start_date | DATE | NOT NULL | 축제 시작일 |
| end_date | DATE | NOT NULL | 축제 종료일 |
| thumbnail_url | VARCHAR(512) | NULL | 대표 썸네일 이미지 URL |
| image_urls | JSON | NULL | 축제 관련 추가 이미지 URL 배열 |
| status | ENUM('ACTIVE', 'HIDDEN', 'DELETED') | NOT NULL | 축제 상태 (노출, 숨김, 삭제) |
| created_at | TIMESTAMP | NOT NULL | 등록 일시 |
| updated_at | TIMESTAMP | NOT NULL | 수정 일시 |

---

## 4. Redis 데이터 구조 정의

> MySQL ERD와 별도로 Redis에서 관리하는 데이터 구조를 정의한다.
> 

| Key 패턴 | 자료구조 | TTL | 용도 |
| --- | --- | --- | --- |
| `refresh:{userId}` | String | 7일 | Refresh Token 저장 |
| `blacklist:{token}` | String | 토큰 잔여 만료시간 | 로그아웃 토큰 블랙리스트 |
| `geo:places` | Geospatial | 없음 | 관광지 위치 인덱스 |
| `place:{placeId}` | String (JSON) | 10분 | 관광지 상세 캐시 |
| `place:view:{placeId}` | String (INCR) | 없음 | 관광지 조회수 |
| `place:like:{placeId}` | String (INCR) | 없음 | 관광지 찜 수 |
| `stock:{productId}` | String (DECR) | 10분 | 상품 재고 가점유 |
| `search:ranking` | ZSet | 없음 (일간 초기화) | 인기 검색어 |
| `search:ranking:prev` | ZSet | 없음 | 이전 인기 검색어 (순위 변동 계산) |
| `search:recent:{userId}` | List | 30일 | 사용자 최근 검색어 |
| `recommend:popular` | ZSet | 1시간 | 인기 관광지 랭킹 |
| `purchase:lock:{userId}` | String (Redisson) | 5초 | 상품 구매 분산 락 |
| `payment:lock:{userId}` | String (Redisson) | 5초 | 엽전 결제 분산 락 |
| `chatroom:lock:{chatRoomId}` | String (Redisson) | 5초 | 채팅방 참여 분산 락 |
| `chat:{chatRoomId}` | Pub/Sub | - | 채팅 메시지 브로커 |
| `notification:{userId}` | Pub/Sub | - | 사용자 알림 브로커 |
| `post:liked:{postId}:{userId}` | String (SETNX) | 없음 | 게시글 좋아요 중복 방지 |
| `post:like:{postId}` | String (INCR) | 없음 | 게시글 좋아요 수 |

---

## 5. 주요 설계 결정 사항

### 5.1 Soft Delete 전략

삭제 시 실제 DELETE 대신 `deleted_at` 컬럼 설정.
적용 테이블: `users`, `free_posts`, `companion_posts`, `comments`, `place_reviews`

- `free_posts/companion_posts.status = HIDDEN`: 신고 처리로 숨김
- `free_posts/companion_posts.status = DELETED`: 작성자 또는 관리자 삭제

**이유**
- 신고 처리 시 삭제된 게시글 원문 조회 필요
- 사용자 탈퇴 후 결제 내역 보존 필요 (법적 의무)
- QueryDSL에서 `deleted_at IS NULL` 조건으로 자동 필터링

---

### 5.2 JSON 컬럼 사용 기준

`image_urls`, `tags`, `menu_items` 등 리스트형 데이터는 JSON 컬럼으로 저장.

**이유**
- 1:N 테이블 생성 시 JOIN 비용 발생
- 이미지 URL, 태그는 단순 조회만 사용하므로 JSON으로 충분
- MySQL 8.4 JSON 컬럼 지원 및 JSON_CONTAINS 등 함수 활용 가능

---

### 5.3 balance BIGINT 선택 이유

- INT(최대 약 21억)는 엽전 잔액 충분하나, 서비스 확장 시 안전을 위해 BIGINT 사용

---

### 5.4 메시지 테이블 확장 고려

현재 MySQL messages 테이블에 저장하되, 트래픽 증가 시 MongoDB 또는 Cassandra 이관 고려.
`sent_at DESC` 파티셔닝으로 조회 성능 최적화 예정.

---

### 5.5 companion_reviews 복합 UNIQUE 제약

동일 채팅방 내에서 같은 사람에게 중복 리뷰를 작성하는 것을 DB 레벨에서 방지한다.

```sql
UNIQUE KEY uq_companion_review (reviewer_id, target_user_id, chat_room_id)
```

**이유**
- 애플리케이션 레벨 검사만으로는 동시 요청 시 중복 삽입이 발생할 수 있음
- DB UNIQUE 제약으로 최종 정합성 보장

---

### 5.6 merchant_applications.shop_name 길이 제한

`merchant_applications.shop_name`은 최대 **50자**로 제한한다.

**이유**
- 승인 후 `shops.shop_name`(VARCHAR 50)으로 복사되므로 신청 단계부터 동일한 길이 제한 적용
- 에러코드 `SHOP_006` (“가게명은 최대 50자까지 입력 가능합니다.”)와 일치

---

### 3.x companion_posts (동행 게시판)

| 컬럼 | 타입 | 제약 | 설명 |
| --- | --- | --- | --- |
| id | BIGINT | PK | |
| author_id | BIGINT | FK, NOT NULL | 작성자 userId |
| title | VARCHAR(255) | NOT NULL | 제목 |
| content | TEXT | NOT NULL | 본문 |
| target_type | VARCHAR(20) | NOT NULL | PLACE / MARKET / FESTIVAL |
| target_id | BIGINT | NOT NULL | 만남 장소 대상 ID |
| target_name | VARCHAR(100) | NOT NULL | 만남 장소명 (비정규화) |
| region | VARCHAR(50) | NULL | 지역 필터용 |
| meeting_date | DATE | NULL | 동행 희망 날짜 |
| max_members | INT | NULL | 최대 모집 인원 |
| current_members | INT | NULL | 현재 참여 인원 |
| view_count | BIGINT | NOT NULL, DEFAULT 0 | 조회수 (V202606171100) |
| status | ENUM | NOT NULL | ACTIVE / CLOSED / DELETED / HIDDEN |
| created_at | TIMESTAMP | NOT NULL | |
| updated_at | TIMESTAMP | NOT NULL | |
| deleted_at | TIMESTAMP | NULL | Soft Delete |

> V202606181201 마이그레이션으로 `place_id/place_name` → `target_type/target_id/target_name` 다형 참조로 전환됨. 이미지 없음(companion_posts는 image 미지원).

---

### 3.x admin_action_logs

| 컬럼 | 타입 | 제약 | 설명 |
| --- | --- | --- | --- |
| id | BIGINT | PK | |
| admin_user_id | BIGINT | NOT NULL | 액션 수행 관리자 |
| action_type | VARCHAR(64) | NOT NULL | USER_SUSPEND / USER_UNSUSPEND 등 |
| target_type | VARCHAR(64) | NOT NULL | USER / MERCHANT / SHOP 등 13종 |
| target_id | BIGINT | NOT NULL | 대상 엔티티 PK |
| reason | VARCHAR(500) | NULL | 처리 사유 |
| before_status | VARCHAR(64) | NULL | 변경 전 상태 |
| after_status | VARCHAR(64) | NULL | 변경 후 상태 |
| created_at | TIMESTAMP | NOT NULL | |

> Append-only. 수정·삭제 없음. 감사(audit) 전용.

---

### 3.x user_sanction

| 컬럼 | 타입 | 제약 | 설명 |
| --- | --- | --- | --- |
| id | BIGINT | PK | |
| user_id | BIGINT | NOT NULL | 제재 대상 사용자 |
| report_id | BIGINT | NOT NULL | 트리거된 신고 ID |
| target_type | VARCHAR(20) | NOT NULL | 제재 대상 유형 |
| sanction_type | VARCHAR(20) | NOT NULL | 제재 종류 |
| reason | VARCHAR(500) | NOT NULL | 제재 사유 |
| started_at | TIMESTAMP | NOT NULL | 제재 시작 |
| ended_at | TIMESTAMP | NULL | 제재 만료 (NULL=영구) |
| released_at | TIMESTAMP | NULL | 조기 해제 일시 |
| released_by | BIGINT | NULL | 해제 관리자 ID |
| created_at | TIMESTAMP | NOT NULL | |

> `users.sanction_type / sanction_end_at` 은 현재 활성 제재 요약 (빠른 조회용). 이력은 이 테이블에서 관리.

---

### 3.x translation_cache

| 컬럼 | 타입 | 제약 | 설명 |
| --- | --- | --- | --- |
| id | BIGINT | PK | |
| content_hash | CHAR(64) | NOT NULL | SHA-256 해시 (원문 지문) |
| target_language | VARCHAR(10) | NOT NULL | 번역 대상 언어 |
| translated_content | TEXT | NOT NULL | 번역 결과 |
| created_at | TIMESTAMP | NOT NULL | |
| updated_at | TIMESTAMP | NOT NULL | |

```sql
UNIQUE KEY uq_translation_cache_hash_lang (content_hash, target_language)
```

---

### 3.x place_sync_state

| 컬럼 | 타입 | 제약 | 설명 |
| --- | --- | --- | --- |
| id | BIGINT | PK | 단일 행 (id=1 고정) |
| last_modified_time | VARCHAR(14) | NULL | 마지막 동기화 시각 (yyyyMMddHHmmss) |
| updated_at | TIMESTAMP | NOT NULL | |

> 관광지 공공데이터 증분 동기화용. 전수 재적재 없이 변경분만 수집.

---

*본 문서는 SA 설계 과정에서 지속적으로 업데이트됩니다.*

