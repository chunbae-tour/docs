# 06_API_명세서

> **문서 버전**: v2.0  
> **최신 반영일**: 2026-06-22  
> **온라인 API 문서**: https://chunbae-tour-api.netlify.app/

이 문서는 첨부된 `최종_API_명세서 (1).md`를 기준으로 반영한 최종 API 명세이다.  
프론트엔드, QA, 발표 자료에서는 위 온라인 API 문서를 우선 확인하고, 상세 필드/에러/인증 조건은 아래 명세를 함께 참조한다.

---
# 최종 API 명세서

Swagger 어노테이션 + 컨트롤러/서비스/DTO 코드 직접 확인해서 작성. 도메인별 섹션 분리.

작성 진행 상태: 아래 목차에 ✅ 표시된 도메인만 작성 완료.

## 목차

- [x] Chat / CompanionReview / Merchant / Report / Notification / Translation
- [x] CS / Community / Search
- [x] Auth / Payment / Festival
- [x] Place / Store
- [x] Admin / Market / Yeopjeon
- [x] Shop

---

## Shop 도메인

### AdApplicationController
경로: `src/main/java/com/chunbaetour/domain/shop/controller/AdApplicationController.java`

#### POST /api/v1/merchants/me/ads — 광고 신청
| 항목 | 내용 |
|---|---|
| Method | POST |
| Endpoint | `/api/v1/merchants/me/ads` |
| 설명 | 광고 신청 생성 |
| 인증 | ✅ MERCHANT |
| 응답 코드 | 201 Created |

**Request Body — `AdApplicationRequest`**
| 필드 | 타입 | 필수 | 검증 | 설명 |
|---|---|---|---|---|
| shopId | Long | ✅ | `@NotNull` | 광고할 가게 ID |
| adType | AdType | ✅ | `@NotNull` | 광고 타입 (현재: BANNER) |
| startDate | LocalDate | ✅ | `@NotNull` | 광고 시작일 |
| endDate | LocalDate | ✅ | `@NotNull` | 광고 종료일 |

**Response — `ApiResponse<AdApplicationResponse>`**
| 필드 | 타입 | Nullable | 설명 |
|---|---|---|---|
| data.applicationId | Long | - | 신청 ID |
| data.shopId | Long | - | 가게 ID |
| data.adType | AdType | - | 광고 타입 |
| data.startDate | LocalDate | - | 광고 시작일 |
| data.endDate | LocalDate | - | 광고 종료일 |
| data.cost | long | - | 광고 비용(엽전) |
| data.status | AdApplicationStatus | - | PENDING/APPROVED/REJECTED |
| data.createdAt | LocalDateTime | - | 생성 시각(UTC) |

#### POST /api/v1/merchants/me/ads/{adId}/extend — 광고 연장
| 항목 | 내용 |
|---|---|
| Method | POST |
| Endpoint | `/api/v1/merchants/me/ads/{adId}/extend` |
| 설명 | 광고 연장 (엽전 차감) |
| 인증 | ✅ MERCHANT |
| 응답 코드 | 200 OK |

**Path Variable**: adId(Long, 필수) — 연장할 광고 ID

**Request Body — `AdExtendRequest`**
| 필드 | 타입 | 필수 | 검증 | 설명 |
|---|---|---|---|---|
| extensionDays | int | ✅ | `@Positive`, `@Max(365)` | 연장 일수 (1~365) |

**Response**: `ApiResponse<AdApplicationResponse>` — 동일 구조(연장된 endDate)

---

### AdminAdApplicationController
경로: `src/main/java/com/chunbaetour/domain/shop/controller/AdminAdApplicationController.java`

#### GET /api/v1/admin/ads — 광고 신청 목록 조회
| 항목 | 내용 |
|---|---|
| Method | GET |
| Endpoint | `/api/v1/admin/ads` |
| 설명 | 광고 신청 목록 커서 페이징 조회 |
| 인증 | ✅ ADMIN |
| 응답 코드 | 200 OK |

**Query**: cursor(String, 선택), size(int, 선택, 1~100, 기본 20), status(AdApplicationStatus, 선택)

**Response — `ApiResponse<CursorPageResponse<AdminAdApplicationResponse>>`**
| 필드 | 타입 | 설명 |
|---|---|---|
| data.content[].applicationId | Long | 신청 ID |
| data.content[].shopId | Long | 가게 ID |
| data.content[].adType | AdType | 광고 타입 |
| data.content[].startDate | LocalDate | 시작일 |
| data.content[].endDate | LocalDate | 종료일 |
| data.content[].cost | long | 광고 비용 |
| data.content[].status | AdApplicationStatus | 신청 상태 |
| data.content[].rejectReason | String | 거절 사유(거절 시만) |
| data.content[].createdAt | LocalDateTime | 생성 시각 |
| data.nextCursor | String | 다음 페이지 커서 |
| data.hasNext | boolean | 다음 페이지 존재 여부 |
| data.size | int | 요청 페이지 크기 |

#### GET /api/v1/admin/ads/{adId} — 광고 신청 단건 조회
Path: adId(Long, `@Positive`). Response: `ApiResponse<AdminAdApplicationResponse>` 단건.

#### PATCH /api/v1/admin/ads/{adId}/approve — 광고 승인
Path: adId(Long, `@Positive`). Response: data null.

#### PATCH /api/v1/admin/ads/{adId}/reject — 광고 거절
Path: adId(Long, `@Positive`).
**Request Body — `AdRejectRequest`**: reason(String, 필수, `@NotBlank @Size(max=500)`) — 거절 사유
Response: data null.

---

### AdminSettlementController
경로: `src/main/java/com/chunbaetour/domain/shop/controller/AdminSettlementController.java`

#### GET /api/v1/admin/settlements — 정산 목록 조회
Query: cursor, size(1~100, 기본 20), status(SettlementStatus, 선택)

**Response — `ApiResponse<CursorPageResponse<AdminSettlementResponse>>`**
| 필드 | 타입 | 설명 |
|---|---|---|
| data.content[].settlementId | Long | 정산 ID |
| data.content[].shopId | Long | 가게 ID |
| data.content[].amount | long | 정산액(엽전) |
| data.content[].status | SettlementStatus | 상태 |
| data.content[].rejectReason | String | 거절 사유 |
| data.content[].bankName | String | 은행명 |
| data.content[].accountNumber | String | 계좌번호(마스킹 미적용, 실계좌) |
| data.content[].accountHolder | String | 예금주 |
| data.content[].createdAt | LocalDateTime | 생성 시각 |

#### PATCH /api/v1/admin/settlements/{settlementId}/approve — 정산 승인
Path: settlementId(Long). Response: data null.

#### PATCH /api/v1/admin/settlements/{settlementId}/reject — 정산 거절
Path: settlementId(Long). Request Body `SettlementRejectRequest`: reason(String, `@NotBlank @Size(max=500)`). Response: data null.

---

### AdminShopController (shop 패키지)
경로: `src/main/java/com/chunbaetour/domain/shop/controller/AdminShopController.java`
> 주의: `admin/shop/controller/AdminShopController.java`(admin 도메인)에도 동명 클래스 별도 존재 — 다른 섹션 참고.

#### PATCH /api/v1/admin/shops/{shopId}/status — 가게 상태 변경
Path: shopId(Long, `@Positive`). Request Body `AdminShopStatusRequest`: status(ShopStatus, `@NotNull`, ACTIVE/SUSPENDED). Response: data null.

#### PATCH /api/v1/admin/shops/{shopId}/place — 가게-장소 수동 연결 (KAN-217)
Path: shopId(Long, `@Positive`). Request Body `AdminShopPlaceRequest`: placeId(Long, 선택, `@Positive` — null이면 해제).

**Response — `ApiResponse<AdminShopPlaceResponse>`**
| 필드 | 타입 | Nullable | 설명 |
|---|---|---|---|
| data.shopId | Long | - | 가게 ID |
| data.placeId | Long | ✅ | 연결된 장소 ID |
| data.placeName | String | ✅ | 연결된 장소명 |
| data.linked | boolean | - | 연결 여부 |

#### PATCH /api/v1/admin/shops/{shopId}/market — 가게-전통시장 수동 연결 (KAN-268)
Path: shopId(Long, `@Positive`). Request Body `AdminShopMarketRequest`: traditionalMarketId(Long, 선택, `@Positive` — null이면 해제).

**Response — `ApiResponse<AdminShopMarketResponse>`**: shopId, traditionalMarketId(Nullable), marketName(Nullable), linked(boolean)

---

### MenuController
경로: `src/main/java/com/chunbaetour/domain/shop/controller/MenuController.java`

#### GET /api/v1/merchants/me/shops/{shopId}/menus — 내 가게 메뉴 목록 조회 (KAN-267)
Path: shopId(Long). 비활성 포함, 삭제 제외.

**Response — `ApiResponse<List<MenuResponse>>`**
| 필드 | 타입 | 설명 |
|---|---|---|
| data[].id | Long | 메뉴 ID |
| data[].shopId | Long | 가게 ID |
| data[].name | String | 메뉴명 |
| data[].description | String | 설명 |
| data[].price | Long | 가격(원) |
| data[].imageUrl | String | 이미지 URL |
| data[].isAvailable | boolean | 주문 가능 여부 |

#### POST /api/v1/merchants/me/shops/{shopId}/menus — 메뉴 등록
Path: shopId(Long).
**Request Body — `MenuCreateRequest`**
| 필드 | 타입 | 필수 | 검증 | 설명 |
|---|---|---|---|---|
| name | String | ✅ | `@NotBlank @Size(max=100)` | 메뉴명 |
| description | String | ❌ | `@Size(max=500)` | 설명 |
| price | Long | ✅ | `@NotNull @Min(1) @Max(9999999)` | 가격(원) |
| imageUrl | String | ❌ | `@URL @Size(max=500)` | 이미지 URL |

Response: `ApiResponse<MenuResponse>` 단건.

#### PATCH /api/v1/merchants/me/shops/{shopId}/menus/{menuId} — 메뉴 수정
부분 수정(null 필드 미수정). Path: shopId, menuId(Long).
**Request Body — `MenuUpdateRequest`**: name(선택,공백거부,max100), description(선택,max500), price(선택,1~9999999), imageUrl(선택,`@URL`,max500), isAvailable(Boolean,선택)
Response: `ApiResponse<MenuResponse>` 단건.

#### DELETE /api/v1/merchants/me/shops/{shopId}/menus/{menuId} — 메뉴 삭제
Path: shopId, menuId(Long). 응답 204, body 없음.

---

### MerchantHomeController
경로: `src/main/java/com/chunbaetour/domain/shop/controller/MerchantHomeController.java`

#### GET /api/v1/merchants/me/home — 상인 홈 대시보드 조회
오늘/어제 매출, 시간대별 분포, 미완료 결제 건수, 최근 결제 목록.

**Response — `ApiResponse<MerchantHomeResponse>`**
| 필드 | 타입 | 설명 |
|---|---|---|
| data.todaySalesAmount | long | 오늘(KST 영업일) 완료 매출 합계(원) |
| data.yesterdaySalesAmount | long | 어제(KST) 완료 매출 합계(원) |
| data.todaySalesDate | LocalDate | 오늘 기준 영업일(KST) |
| data.hourlySales | List | KST 0~23시 시간대별 매출(항상 24개) — {hour, amount} |
| data.missedPaymentCount | long | 오늘(KST) 미완료(거절+만료) 결제 건수 |
| data.recentPayments | List | 최근 완료 결제 — {payRequestId, shopId, amount, completedAt} |

타임존 주의: hourlySales는 KST 기준, completedAt은 UTC 저장값.

---

### MerchantQrController
경로: `src/main/java/com/chunbaetour/domain/shop/controller/MerchantQrController.java`

#### GET /api/v1/merchants/me/shops/{shopId}/qr — 내 가게 QR 코드 조회
Response: `ApiResponse<QrCodeResponse>` — shopId, shopName, qrPayload(`YEOPJEON_PAY:SHOP:{shopId}:{qrNonce}`)

#### POST /api/v1/merchants/me/shops/{shopId}/qr/reissue — QR 재발급
분실/도용 시 nonce 회전, 옛 QR 무효화. ACTIVE 가게만 가능. Response 동일 구조(새 qrPayload).

---

### MerchantQrPayController
경로: `src/main/java/com/chunbaetour/domain/shop/controller/MerchantQrPayController.java`

#### GET /api/v1/merchants/me/qr-payments/pending — 대기중 QR 결제 목록 조회
**Response — `ApiResponse<List<PendingQrPayResponse>>`**
| 필드 | 타입 | 설명 |
|---|---|---|
| data[].payRequestId | String | 결제 ID |
| data[].shopId | Long | 가게 ID |
| data[].amount | Long | 결제액(원) |
| data[].menuItems | List | {menuId, menuName, quantity, unitPrice} |
| data[].createdAt | LocalDateTime | 결제 생성 시각(UTC) |
| data[].expiredAt | LocalDateTime | 결제 만료 시각(UTC) |

---

### SettlementController
경로: `src/main/java/com/chunbaetour/domain/shop/controller/SettlementController.java`

#### POST /api/v1/merchants/me/shops/{shopId}/settlements — 정산 신청
**Response — `ApiResponse<SettlementResponse>`**: settlementId, shopId, amount(정산액/엽전), status(PENDING), rejectReason(Nullable), bankName, accountNumber(마스킹: 앞3/뒤4 외 `*`), accountHolder, createdAt

#### GET /api/v1/merchants/me/shops/{shopId}/settlements — 내 정산 내역 조회
Query: cursor, size(1~100, 기본 20). Response: `ApiResponse<CursorPageResponse<SettlementResponse>>`

---

### ShopAccountController
경로: `src/main/java/com/chunbaetour/domain/shop/controller/ShopAccountController.java`

#### PUT /api/v1/merchants/me/shops/{shopId}/account — 정산 계좌 등록/변경
**Request Body — `ShopAccountRequest`**
| 필드 | 타입 | 필수 | 검증 | 설명 |
|---|---|---|---|---|
| bankName | String | ✅ | `@NotBlank @Size(max=50)` | 은행명 |
| accountNumber | String | ✅ | `@NotBlank @Pattern(\d+[\d-]*) @Size(10~30)` | 계좌번호(숫자+하이픈) |
| accountHolder | String | ✅ | `@NotBlank @Pattern([가-힣a-zA-Z ]+) @Size(max=50)` | 예금주 |

Response: `ApiResponse<ShopAccountResponse>` — shopId, bankName, accountNumber(마스킹: 뒤4자리 외 `*`), accountHolder

---

### ShopController
경로: `src/main/java/com/chunbaetour/domain/shop/controller/ShopController.java`

#### GET /api/v1/merchants/me/shops — 내 가게 목록 조회
**Response — `ApiResponse<List<ShopResponse>>`**
| 필드 | 타입 | Nullable | 설명 |
|---|---|---|---|
| data[].id | Long | - | 가게 ID |
| data[].userId | Long | - | 사용자 ID |
| data[].placeId | Long | ✅ | 연결된 장소 ID |
| data[].shopName | String | - | 가게명 |
| data[].category | String | - | 카테고리 |
| data[].address | String | - | 주소 |
| data[].lat / lng | BigDecimal | - | 위도/경도 |
| data[].phone | String | - | 전화번호 |
| data[].description | String | - | 소개글 |
| data[].imageUrls | String | - | 이미지 URL JSON 배열(presigned GET URL) |
| data[].operatingHours / closedDays | String | - | 운영시간/휴무일 |
| data[].isCertified | boolean | - | 인증 여부 |
| data[].rating | BigDecimal | - | 평점 |
| data[].reviewCount | int | - | 리뷰 수 |
| data[].status | ShopStatus | - | ACTIVE/SUSPENDED/CLOSED |

#### GET /api/v1/merchants/me/shops/{shopId} — 내 가게 단건 조회
Response: `ApiResponse<ShopResponse>` 단건.

#### PATCH /api/v1/merchants/me/shops/{shopId} — 내 가게 정보 수정
부분 수정, 위치 수정 불가.
**Request Body — `ShopUpdateRequest`**: shopName(선택,1~50), category(선택,1~50), phone(선택,`@Pattern(\d{2,3}-\d{3,4}-\d{4})`), description(선택,max500), operatingHours(선택,1~100), closedDays(선택,1~100), imageUrls(선택,max2000, S3 객체 키 JSON, 전체교체)
Response: `ApiResponse<ShopResponse>`

#### PATCH /api/v1/merchants/me/shops/{shopId}/status — 내 가게 상태 직접 전환
Request Body `MerchantShopStatusRequest`: status(ShopStatus, `@NotNull`, ACTIVE/CLOSED만 허용). Response: data null.

---

### ShopImageController
경로: `src/main/java/com/chunbaetour/domain/shop/controller/ShopImageController.java`

#### POST /api/v1/merchants/me/shops/{shopId}/images — 가게 사진 업로드
Content-Type: `multipart/form-data`. JPEG/PNG/WebP, magic-byte 검증, ≤5MB.
Request: file(MultipartFile, 필수). Response: `ApiResponse<ShopImageResponse>` — objectKey(S3 객체 키, 예: `shops/10/uuid.jpg`)

---

### ShopNoticeController
경로: `src/main/java/com/chunbaetour/domain/shop/controller/ShopNoticeController.java`

#### POST /api/v1/merchants/me/shops/{shopId}/notices — 가게 공지 등록
Request `ShopNoticeCreateRequest`: title(`@NotBlank @Size(max=100)`), content(`@NotBlank @Size(max=1000)`)
Response: `ApiResponse<ShopNoticeResponse>` — noticeId, title, content, createdAt

#### GET /api/v1/merchants/me/shops/{shopId}/notices — 가게 공지 목록 조회
커서 페이징, 최신순. Query: cursor, size(1~100, 기본 20). Response: `ApiResponse<CursorPageResponse<ShopNoticeResponse>>`

#### DELETE /api/v1/merchants/me/shops/{shopId}/notices/{noticeId} — 가게 공지 삭제
Path: shopId, noticeId(Long). Response: data null.

---

### ShopPublicController
경로: `src/main/java/com/chunbaetour/domain/shop/controller/ShopPublicController.java`

#### GET /api/v1/shops/{shopId} — 가게 공개 정보 조회
인증: ❌ PUBLIC. 가게 공개 정보 + 메뉴 목록.

**Response — `ApiResponse<ShopInfoResponse>`**
| 필드 | 타입 | Nullable | 설명 |
|---|---|---|---|
| data.shopId / shopName / category / address / phone / description | - | - | 기본 정보 |
| data.operatingHours / closedDays | - | - | 운영시간/휴무일 |
| data.rating | BigDecimal | - | 평점 |
| data.reviewCount | int | - | 리뷰 수 |
| data.isCertified | boolean | - | 인증 여부 |
| data.status | ShopStatus | - | ACTIVE/SUSPENDED/CLOSED |
| data.businessStatus | BusinessStatus | - | 실시간 영업여부(OPEN/CLOSED/UNKNOWN) |
| data.menus | List\<MenuResponse\> | - | 메뉴 목록 |
| data.representativeImageUrl | String | ✅ | PROFILE 대표사진 presigned GET URL (없으면 null) (KAN-323) |
| data.images | List\<ShopImageItemResponse\> | - | GALLERY 이미지 목록 presigned URL (KAN-323) |

---

### ShopWalletController
경로: `src/main/java/com/chunbaetour/domain/shop/controller/ShopWalletController.java`

#### GET /api/v1/merchants/me/shops/{shopId}/wallet — 가게 수익 지갑 조회
Response: `ApiResponse<ShopWalletResponse>` — shopId, balance(잔액/엽전), updatedAt(마지막 수익 적립 시각, 없으면 생성 시각)

---

### Shop 도메인 주요 에러코드
| HTTP | code | message |
|---|---|---|
| 400 | SHOP_001 | 존재하지 않는 가게입니다. |
| 403 | SHOP_002 | 본인 가게 정보만 수정할 수 있습니다. |
| 404 | SHOP_004 | 존재하지 않는 메뉴입니다. |
| 409 | SHOP_006 | 이미 동일한 이름의 메뉴가 존재합니다. |
| 404 | SHOP_008 | 존재하지 않는 정산 요청입니다. |
| 409 | SHOP_009 | 이미 처리 대기 중인 정산 요청이 있습니다. |
| 400 | SHOP_011 | 정산 가능한 잔액이 없습니다. |
| 400 | SHOP_012 | 정산 신청 최소 금액은 5,000엽전입니다. |
| 404 | SHOP_013 | 존재하지 않는 광고 신청입니다. |
| 409 | SHOP_014 | 이미 처리 대기 중인 광고 신청이 있습니다. |
| 400 | SHOP_017 | 파일 크기가 최대 허용 용량(5MB)을 초과합니다. |
| 400 | SHOP_018 | 지원하지 않는 이미지 형식입니다. (허용: JPEG, PNG, WebP) |
| 404 | SHOP_022 | 존재하지 않는 가게 공지입니다. |
| 409 | PAY_029 | 만료된 QR 코드입니다. 최신 QR로 다시 시도해주세요. |

---

## Place 도메인 — 길찾기 API

| 항목 | 내용 |
|---|---|
| Method | GET |
| Endpoint | `/api/v1/directions` |
| 설명 | 카카오맵 길찾기 URL 생성 (비로그인 허용) |
| 인증 | ❌ 불필요 |

Query: originLat/originLng/destLat/destLng (BigDecimal, 필수, 위경도 범위 검증)
Response — `ApiResponse<DirectionResponse>`: provider("KAKAO"), redirectUrl
에러: 429 COMMON_006(분당 10회 초과). 응답 헤더 `X-RateLimit-Remaining`, `Retry-After`. Rate Limit: IP기준 분당 10회.

---

## Place 도메인 — 지오코딩 API

| 항목 | 내용 |
|---|---|
| Method | GET |
| Endpoint | `/api/v1/places/geocoding` |
| 설명 | 주소 → 좌표 변환 (카카오 API) |
| 인증 | ✅ USER |

Query: query(String, `@NotBlank @Size(2~100)`)
Response — `ApiResponse<GeocodingResponse>`: addressName, lat, lng
에러: 400 COMMON_002, 401 AUTH_006, 503 COMMON_007

---

## Place 도메인 — 관광지 목록 조회 API

| 항목 | 내용 |
|---|---|
| Method | GET |
| Endpoint | `/api/v1/places` |
| 설명 | 카테고리/지역 필터 + 커서 페이징 (정렬: rating DESC, id DESC) |
| 인증 | ❌ 불필요 |

Query: category(선택), region(선택), cursor(Long,선택), cursorRating(Float,선택,0.0~5.0), size(선택,1~50,기본20)
Response — `ApiResponse<PlaceListResponse>`: items[]{id,name,category,address,thumbnailUrl,rating,reviewCount}, hasNext, nextCursorId, nextCursorRating
에러: 400 COMMON_011(cursor/cursorRating 쌍 불일치)

---

## Place 도메인 — 지도 마커 조회 API

| Method | GET | Endpoint | `/api/v1/places/map-markers` | 인증 | ❌ |
|---|---|---|---|---|---|

Query: swLat/swLng/neLat/neLng(BigDecimal, 필수)
Response — `ApiResponse<MapMarkerPageResponse>`: markers[](최대30){id,name,category,lat,lng,thumbnailUrl}, truncated, limit
에러: 400 COMMON_004(지도 span 2.0도 초과)

---

## Place 도메인 — 주변 관광지 조회 API

| Method | GET | Endpoint | `/api/v1/places/nearby` | 인증 | ❌ |
|---|---|---|---|---|---|

Query: lat/lng/radius(Double,필수,100~20000m), cursor/cursorDistance(선택), size(선택,1~50,기본10)
Response — `ApiResponse<NearbyPlacePageResponse>`: places[]{placeId,name,category,imageUrl,latitude,longitude,rating,reviewCount,distanceMeters}, hasNext
에러: 400 COMMON_011

---

## Place 도메인 — 관광지 상세 조회 API

| Method | GET | Endpoint | `/api/v1/places/{placeId}` | 인증 | ❌(로그인 시 찜여부 반영) |
|---|---|---|---|---|---|

Response — `ApiResponse<PlaceDetailResponse>`: placeId,name,description?,category,address,latitude,longitude,imageUrls[],operatingHours?,closedDays?,admissionFee?,phone?,rating,reviewCount,likeCount,isLiked
에러: 404 PLACE_001

---

## Place 도메인 — 관광지 찜 추가/취소 API

| POST `/api/v1/places/{placeId}/like` | ✅ USER | data null | 에러: 401 AUTH_006, 404 PLACE_001, 409 PLACE_010(이미 찜) |
| DELETE `/api/v1/places/{placeId}/like` | ✅ USER | 204 | 에러: 401 AUTH_006, 404 PLACE_011(찜 안함) |
|---|---|---|---|

---

## Place 도메인 — 관광지 기반 추천 API

| Method | GET | Endpoint | `/api/v1/places/{placeId}/recommend` | 인증 | ❌ |
|---|---|---|---|---|---|

설명: 동일 카테고리 + 거리순 TOP5(자신 제외)
Response — `ApiResponse<List<RecommendPlaceResponse>>`: placeId,name,category,thumbnailUrl,latitude,longitude,rating,reviewCount,distanceMeters(null)
에러: 404 PLACE_001

---

## Place 도메인 — 관광지 주변 카테고리 장소 검색 API

| Method | GET | Endpoint | `/api/v1/places/{placeId}/nearby-places` | 인증 | ❌ |
|---|---|---|---|---|---|

설명: 카카오 카테고리 API 연동(맛집/카페/숙박 등)
Query: category(NearbyCategory,필수), radius(선택,1~20000,기본500)
Response — `ApiResponse<NearbyCategoryPlacesResponse>`: items[]{id(카카오ID),placeName,categoryName,phone?,addressName,roadAddressName,lat,lng,placeUrl,distance}, hasNext
에러: 404 PLACE_001, 503 COMMON_007

---

## Place 도메인 — 관광지 주변 상점 조회 API

| Method | GET | Endpoint | `/api/v1/places/{placeId}/nearby-shops` | 인증 | ❌ |
|---|---|---|---|---|---|

Query: limit(선택,1~50,기본5)
Response — `ApiResponse<List<NearbyShopResponse>>`: shopId,shopName,category,address,lat,lng,distanceMeters,rating,reviewCount,imageUrls[]
에러: 404 PLACE_001

---

## Place 도메인 — 리뷰 작성/목록 조회 API

POST `/api/v1/places/{placeId}/reviews` (✅USER, 201) — Request `ReviewCreateRequest`: rating(1~5,필수), content(`@NotBlank @Size(max=500)`), imageUrls(선택,max5)
Response — `ApiResponse<PlaceReviewResponse>`: reviewId,authorId,authorNickname,authorProfileImageUrl?,rating,content,imageUrls[],createdAt
에러: 401 AUTH_006, 404 PLACE_001, 409 PLACE_012(이미 작성). 비고: 저장 후 관광지 rating/reviewCount 즉시 재계산.

GET `/api/v1/places/{placeId}/reviews` (❌, 200) — Spring Data Pageable(page,size,sort=createdAt desc;id desc)
Response — `ApiResponse<Page<PlaceReviewResponse>>` (content[], totalPages, totalElements, size, number)
에러: 404 PLACE_001

---

## Place 도메인 — 추천 API (인기/위치기반/카테고리별)

| Endpoint | 설명 | Query |
|---|---|---|
| GET `/api/v1/recommend/popular` | 인기 관광지 TOP10 | - |
| GET `/api/v1/recommend/nearby` | 위치기반(반경 랜덤 샘플링) | lat,lng(필수), radius(선택,1~50km,기본5), limit(선택,1~20,기본5) |
| GET `/api/v1/recommend/category` | 카테고리별 | category(필수) |

전부 인증 불필요. Response — `ApiResponse<List<RecommendPlaceResponse>>`: placeId,name,category,thumbnailUrl,latitude,longitude,rating,reviewCount,distanceMeters(nearby만 값 있음, 나머지 null)

---

## Place 도메인 — Reverse Geocoding API

| Method | GET | Endpoint | `/api/v1/places/region` | 인증 | ✅ USER |
|---|---|---|---|---|---|

Query: lat,lng(Double,필수)
Response — `ApiResponse<RegionResponse>`: depth1(시/도),depth2(구/군),depth3?(동/읍/면),fullAddress
에러: 401 AUTH_006, 503 COMMON_007

---

## Store 도메인 — 상품 목록/상세 조회 API

GET `/api/v1/store/products` (❌, HIDDEN 제외) — Query: category(선택), cursor(Base64,선택), size(선택,1~100,기본20)
Response — `ApiResponse<CursorPageResponse<ProductSummaryResponse>>`: items[]{productId,name,category{code,label},price,originalPrice?,imageUrl,merchantName,stock,soldCount,status}, cursor?
에러: 400 COMMON_008(유효하지 않은 커서)

GET `/api/v1/store/products/{productId}` (❌) — Response `ApiResponse<ProductDetailResponse>`: productId,name,description?,category{code,label},price,originalPrice?,imageUrls[],merchantName,stock,soldCount,validityDays?,status
에러: 404 STORE_001

---

## Store 도메인 — 상품 구매 / 내 주문 내역 API

POST `/api/v1/store/orders` (✅USER, 201) — Request `StorePurchaseRequest`: productId(필수), quantity(필수,1~99)
Response `ApiResponse<StoreOrderResponse>`: orderId,productId,productName,productPrice,quantity,totalPrice,status,createdAt
에러: 401 AUTH_006, 404 STORE_001, 409 STORE_002(품절), 400 STORE_004(최대구매수량초과)

GET `/api/v1/store/orders` (✅USER) — Query: cursor,size(1~100,기본20)
Response `ApiResponse<CursorPageResponse<StoreOrderResponse>>`
에러: 401 AUTH_006

---

## Store 도메인 — 내 보유 아이템 / QR 발급·사용 API

GET `/api/v1/users/me/items` (✅USER) — Response `ApiResponse<CursorPageResponse<UserItemResponse>>`: items[]{itemId,orderId,productId,productName,status(AVAILABLE/USED/EXPIRED),expiresAt?}, cursor?

GET `/api/v1/users/me/items/{itemId}/qr` (✅USER) — Response `ApiResponse<UserItemQrResponse>`: token(5분TTL),expiresAt
에러: 401 AUTH_006, 404 STORE_008, 409 STORE_010(이미사용), 409 STORE_011(만료)
비고: `Cache-Control: no-store`

POST `/api/v1/merchants/me/shop/items/use` (✅MERCHANT) — Request `UserItemUseRequest`: shopId(필수), token(필수)
Response `ApiResponse<UserItemUseResponse>`: itemId,productId,productName,status(USED),usedAt,usedShopId
에러: 401 AUTH_006, 404 STORE_008, 409 STORE_010/011, 401 STORE_012(QR만료), 401 STORE_013(QR무효)
비고: 응답에 userId 노출 안 함(고객 추적 방지)

---

## Store 도메인 (관리자) — 상품 CRUD API

GET `/api/v1/admin/store/products` (✅ADMIN, HIDDEN 포함) — Query: status,category,cursor,size
POST `/api/v1/admin/store/products` (✅ADMIN, 201) — Request `AdminProductCreateRequest`: name(필수),description?,category(필수),price(필수,≥1),originalPrice?(≥0),stock(필수,≥0),imageUrls?(max10),merchantName?,validityDays?,maxPerPerson(필수,≥1)
Response `ApiResponse<ProductDetailResponse>` / 에러: 400 COMMON_004(originalPrice<price)

PATCH `/api/v1/admin/store/products/{productId}` (✅ADMIN) — 부분수정(null 필드 유지, clear 미지원)
에러: 404 STORE_001, 400 COMMON_004

DELETE `/api/v1/admin/store/products/{productId}` (✅ADMIN) — soft delete(status=HIDDEN), 200 OK
에러: 404 STORE_001

---

## Admin 도메인 — 배너 관리

GET `/api/v1/admin/banners` — status 필터 + cursor 페이징. Response: content[]{id,title,imageUrl,priority,startDate?,endDate?,status,createdAt}, nextCursor?, hasNext, size
POST `/api/v1/admin/banners` (201) — Request `AdminBannerCreateRequest`: title(필수,max100), imageUrl(필수,max500), linkUrl?, priority?, startDate?, endDate?(`@AssertTrue startDate≤endDate`)
PATCH `/api/v1/admin/banners/{bannerId}` — 부분수정, 전필드 null이면 400. 에러: 404 RESOURCE_NOT_FOUND, 409 BANNER_ALREADY_DELETED
DELETE `/api/v1/admin/banners/{bannerId}` — soft delete, 204. 에러: 404, 409 BANNER_ALREADY_DELETED

---

## Admin 도메인 — 상인 인증 관리

GET `/api/v1/admin/shop-certifications` — status 필터+cursor. content[]{id,shopId,status,submittedAt,processedAt?}
GET `/api/v1/admin/shop-certifications/{certificationId}` — 상세(submittedReason,rejectReason?,cancelReason?,processedBy?,isCertified)
PATCH `.../approve` — 승인+가게 인증마크 부여. 에러: 404, 409 SHOP_CERTIFICATION_INVALID_STATUS, 409 SHOP_ALREADY_CERTIFIED
PATCH `.../reject` — Request: reason(필수,max500). 에러: 400 MISSING_REQUIRED_FIELD, 404, 409
PATCH `.../cancel` — APPROVED 인증 회수, Request: reason(필수). 에러: 404, 409(APPROVED 아님)

---

## Admin 도메인 — 대시보드

GET `/api/v1/admin/dashboard` — Response `AdminDashboardResponse`: totalUsers,newUsersToday,suspendedUsers,totalShops?,pendingCertifications?,pendingMerchantApplications?,totalPlaces?,totalBanners?,activeBanners?

---

## Admin 도메인 — 전통시장 관리

GET `/api/v1/admin/traditional-markets` — keyword/sido 필터+cursor. content[]{id,name,marketType,sido,sigungu,address}
GET `/api/v1/admin/traditional-markets/{marketId}` — 상세(lat,lng,phoneNumber?,homepageUrl?,establishYear?,createdAt,updatedAt). 에러: 404
POST `/api/v1/admin/traditional-markets/sync` — 공공데이터 즉시동기화. Response: insertedCount,updatedCount,skippedCount,message. 에러: 409 PLACE_SYNC_IN_PROGRESS

---

## Admin 도메인 — 관광지 관리

GET `/api/v1/admin/places` — keyword/category 필터+cursor. content[]{id,name,category,address,status,createdAt}
POST `/api/v1/admin/places` (201) — Request `AdminPlaceCreateRequest`: name,category,description?,address,lat,lng(필수, 범위검증),thumbnailUrl?,imageUrls?(JSON배열),operatingHours?,closedDays?,phone?,admissionFee?,tags?(JSON배열)
PATCH `/api/v1/admin/places/{placeId}` — 부분수정(전필드 빈공백 거부 패턴 검증). 에러: 400(빈요청), 404, 409 PLACE_ALREADY_DELETED
DELETE `/api/v1/admin/places/{placeId}` — soft delete, 204. 에러: 404, 409 PLACE_ALREADY_DELETED
POST `/api/v1/admin/places/sync` — 한국관광공사 KorService2 비동기 동기화, 202 Accepted, data=메시지문자열. 에러: 409 PLACE_SYNC_IN_PROGRESS

---

## Admin 도메인 — 가게 관리
경로: `src/main/java/com/chunbaetour/domain/admin/shop/controller/AdminShopController.java`
> 주의: `shop/controller/AdminShopController.java`(shop 도메인, 본 문서 상단 Shop 섹션)와 동명 별도 클래스.

GET `/api/v1/admin/shops` — keyword/status 필터+cursor. content[]{id,userId,shopName,category,address,phone,isCertified,status,placeId?,placeName?,traditionalMarketId?,traditionalMarketName?,createdAt}
GET `/api/v1/admin/shops/{shopId}` — 상세(lat,lng,description?,imageUrls?,operatingHours?,closedDays?,rating,reviewCount,updatedAt). 에러: 404
PATCH `/api/v1/admin/shops/{shopId}` — Request: status?(ACTIVE/SUSPENDED만, CLOSED 거부), description?, phone?, operatingHours?. 에러: 400(빈요청 또는 CLOSED 시도), 404

---

## Admin 도메인 — 제재 이력 관리

GET `/api/v1/admin/users/{userId}/sanctions` — Response `ApiResponse<List<UserSanctionResponse>>`: sanctionId,reportId,targetType,sanctionType,reason,startedAt,endedAt?,releasedAt?,releasedBy?,createdAt. 에러: 404
DELETE `/api/v1/admin/users/{userId}/sanctions/{sanctionId}` — 조기해제, 204. 에러: 404, 409 REPORT_NOT_FOUND

---

## Admin 도메인 — 사용자 관리

GET `/api/v1/admin/users` — keyword/status/role 필터+cursor. content[]{id,email?,nickname,role,status,suspendedUntil?,createdAt}
GET `/api/v1/admin/users/{userId}` — 상세(profileImageUrl?,suspendedReason?,reportCount,updatedAt). 에러: 404
POST `/api/v1/admin/users/{userId}/suspensions` (201) — Request: reason(필수,max500), durationDays?(≥1, null=무기한). 에러: 400 MISSING_REQUIRED_FIELD, 400 AUTH_016(탈퇴계정), 404
DELETE `/api/v1/admin/users/{userId}/suspensions` — 정지해제, 204. 에러: 404

---

## 전통시장 도메인 — 공개 API

GET `/api/v1/traditional-markets/nearby` (❌비인증) — Query: lat,lng(필수), radius(선택,100~50000m,기본3000), page(선택,기본0), size(선택,1~100,기본20)
Response: markets[]{id,name,address,lat,lng,marketType,distanceMeters,imageUrl?,targetType}, page,size,hasNext. 에러: 400 INVALID_INPUT_VALUE
POST/DELETE `/api/v1/traditional-markets/{marketId}/like` (✅USER) — 에러: 404, 409 LIKE_ALREADY_EXISTS / 404 LIKE_NOT_FOUND, 403 AUTH_012(정지계정)

---

## 엽전 도메인 — 잔액·이력 조회

GET `/api/v1/yeopjeon/balance` (✅USER) — Response: balance(long). 에러: 404 PAY_012, 403 AUTH_012
GET `/api/v1/yeopjeon/histories` (✅USER) — cursor 페이징. content[]{id,type(CHARGE/PAYMENT/RECEIVED_PAYMENT/REFUND),amount,balanceSnapshot,description?,createdAt}. 에러: 403 AUTH_012

> 참고: 본 배치(Admin/Market/Yeopjeon) 에러코드 표기 중 `RESOURCE_NOT_FOUND`/`INVALID_INPUT_VALUE`/`MISSING_REQUIRED_FIELD` 등은 에이전트가 ErrorCode enum 상수명 그대로 기재한 것으로, 실제 응답 `code` 문자열(예: COMMON_005 등)과 다를 수 있음 — 정확한 문자열은 `ErrorCode.java` 재확인 필요.

---

## Auth 도메인 — 회원가입/로그인

POST `/api/v1/users/auth/signup` (❌, 201) — Request `SignupRequest`: email(`@Email`,max255), password(max72,영문+숫자+특수문자 8자+), nickname(2~20,한글/영문/숫자/_-)
Response `SignupResponse`: userId,email,nickname,role(USER),status(ACTIVE)
에러: 400 AUTH_010/AUTH_011, 409 AUTH_008(이메일중복)/AUTH_009(닉네임중복)

POST `/api/v1/users/auth/login` / `/api/v1/merchants/auth/login` / `/api/v1/admin/auth/login` (❌, 200) — Request `LoginRequest`: email,password
Response `LoginResponse`: accessToken,role. 응답헤더 Set-Cookie(Refresh HttpOnly)
에러: 401 AUTH_001, 403 AUTH_012(정지)/AUTH_007(역할불일치, merchant·admin만)

POST `/api/v1/auth/reissue` (❌, 200) — Refresh Cookie 자동추출. Response: accessToken,role(권한변화 즉시반영). 에러: 401 AUTH_005/AUTH_004
POST `/api/v1/auth/logout` (✅인증, 204) — AccessToken 블랙리스트+Refresh삭제+Cookie만료. 에러: 401 AUTH_006/AUTH_002/AUTH_003

---

## Auth 도메인 — 소셜 로그인(카카오/네이버)

POST `/api/v1/users/auth/oauth/{provider}` (❌, 200) — Request `OauthLoginRequest`: code,redirectUri(필수)
Response `OauthLoginResponse` — 신규: needSignup=true,signupTicket,email?,nickname? / 기존: needSignup=false,accessToken,role(USER)+Set-Cookie
에러: 400 AUTH_018(미지원provider)/AUTH_023(인가코드무효), 402 AUTH_019(공급자오류), 409 AUTH_021(중복가입경쟁)

POST `/api/v1/users/auth/oauth/signup` (❌, 201) — Request `OauthSignupRequest`: ticket,name,phone,birthdate(`@Past`),nickname(`@SocialNickname`)
Response: needSignup=false,accessToken,role(USER)+Set-Cookie
에러: 400 AUTH_020(티켓무효)/AUTH_022(이메일미제공), 409 AUTH_017(전화번호중복)

POST `/api/v1/merchants/auth/oauth/{provider}` (❌, 200) — 상인 소셜 로그인 (MerchantOAuthController)
Request `OauthLoginRequest`: code,redirectUri(필수) — USER OAuth와 동일 구조
Response: needSignup=false,accessToken,role(MERCHANT)+Set-Cookie (needSignup 분기 없음 — MERCHANT 계정만 허용)
에러: 400 AUTH_018(미지원provider)/AUTH_023(인가코드무효), 403 ACCESS_DENIED(USER·미가입 계정)

---

## Auth 도메인 — 마이페이지

GET `/api/v1/users/me` (✅USER) — Response `UserMeResponse`: userId,email?,nickname,profileImageUrl?,language,companionScore,companionReviewCount,role,status,createdAt
PATCH `/api/v1/users/me` (✅USER) — Request `PatchUserMeRequest`(전부 선택): nickname?,language?(ko/en/ja),profileImageUrl?
에러: 400 COMMON_002(빈요청), 409 AUTH_009(닉네임중복)
GET `/api/v1/users/me/home` (✅USER) — Response: profile(UserMeResponse), wallet.balance
GET `/api/v1/users/me/likes` (✅USER) — Query: type(PLACE/MARKET/FESTIVAL,기본PLACE), page,size,sort(createdAt|id). Response `Page<UserLikedTargetResponse>`. 에러: 400 COMMON_002(정렬필드 화이트리스트 외)
GET `/api/v1/users/me/reviews` (✅USER) — Query: page,size,sort. Response `Page<UserReviewResponse>`
DELETE `/api/v1/users/me` (✅USER, 204) — soft delete+토큰무효화. 에러: 401 AUTH_006, 403 AUTH_007

POST `/api/v1/users/me/profile-image` (✅USER, 201) — 프로필 이미지 업로드 (ProfileImageController, KAN-320)
Content-Type: multipart/form-data. Request: `file`(MultipartFile, JPEG/PNG/WebP, magic-byte 검증, ≤5MB)
Response `ProfileImageUploadResponse`: previewUrl(presigned GET URL). 업로드 즉시 프로필 반영.
에러: 400 SHOP_017(5MB초과)/SHOP_018(미지원형식), 401 AUTH_006

---

## Payment 도메인 — 충전/환불

GET `/api/v1/payments/history` (✅USER) — cursor 페이징. content[]{id,orderUid,amount,paymentMethod,status(PENDING/COMPLETED/FAILED/CANCELLED/PARTIAL_CANCELLED/ADJUSTMENT_REQUIRED/REFUNDED),createdAt,updatedAt}
POST `/api/v1/payments/charge` (✅USER) — Header: Idempotency-Key(필수). Request `ChargeRequest`: amount(5000~100000,1000원단위),paymentMethod(CARD/FOREIGN_CARD/KAKAO_PAY/TOSS_PAY)
Response `ChargeResponse`: orderUid,paymentId,storeId,channelKey,orderName,totalAmount,currency(KRW),payMethod
에러: 400 PAY_002/PAY_003/PAY_004/PAY_030(일일한도), 409 PAY_007(중복Idempotency), 503 PAY_005
POST `/api/v1/payments/{orderId}/cancel` (✅USER, 204) — PENDING 주문만 취소. 에러: 404 PAY_009, 409 PAY_021

POST `/api/v1/payments/{orderId}/refund` (✅USER) — Request: reason(필수,max500). Response: refundId,paymentOrderId,amount,status(PENDING),createdAt
에러: 400 PAY_010(기간경과)/PAY_015(완료건만), 404 PAY_009, 409 PAY_016(중복요청)
PATCH `/api/v1/payments/refund/{refundId}/cancel` (✅USER, 204) — 에러: 404 PAY_018, 409 PAY_019
GET `/api/v1/payments/refunds` (✅USER) — status 필터+cursor. content[]{refundId,paymentOrderId,amount,status,reason,rejectReason?,createdAt}

GET `/api/v1/admin/refunds` (✅ADMIN) — content[]{...+userId}
PATCH `/api/v1/admin/refunds/{refundId}/approve` — 에러: 404 PAY_018, 409 PAY_020, 400 PAY_017(시스템엽전부족)
PATCH `/api/v1/admin/refunds/{refundId}/reject` — Request: reason?(선택). 에러: 404, 409

---

## Payment 도메인 — QR 결제

POST `/api/v1/payments/qr` (✅USER) — Request `QrPayCreateRequest`: shopId,qrNonce(필수),menuItems[]{menuId,quantity}(NotEmpty,max50)
Response `QrPayCreateResponse`: payRequestId,shopId,shopName,totalAmount,menuItems[],expiredAt(5분후)
에러: 400 PAY_024(금액≤0)/PAY_023(본인가게), 404 SHOP_001/SHOP_004, 409 PAY_029(만료QR)/PAY_022(중복대기)
GET `/api/v1/payments/qr/{payRequestId}/status` (✅USER) — Response: status(PENDING/COMPLETED/REJECTED/EXPIRED/CANCELLED),amount,shopId,menuItems[],expiredAt,completedAt?. 에러: 404 PAY_026
PATCH `/api/v1/payments/qr/{payRequestId}/confirm` (✅MERCHANT) — Request: action(APPROVE/REJECT),rejectReason?. 에러: 403 PAY_028, 404 PAY_026, 409 PAY_025, 400 PAY_001(엽전부족)
POST `/api/v1/payments/qr/{payRequestId}/cancel` (✅USER) — 에러: 404 PAY_026, 409 PAY_025

POST `/api/v1/payments/webhook` (❌, PortOne 콜백) — Header: webhook-id/signature/timestamp. 에러: 400 COMMON_002, 401 PAY_014(서명무효)

---

## Festival 도메인

ADMIN: GET/POST `/api/v1/admin/festivals`, PUT/DELETE `/{festivalId}`, POST `/fetch`(공공데이터 즉시수집, 결과:fetched/created/updated/skipped). 에러: 404 FESTIVAL_001, 409 FESTIVAL_002/FESTIVAL_004, 502 EXTERNAL_SERVICE_ERROR
공개: GET `/api/v1/festivals`(date/region 필터+cursor), GET `/{festivalId}`(progressStatus 동적계산: BEFORE/ONGOING/ENDED), POST/DELETE `/{festivalId}/like`. 에러: 404 FESTIVAL_001, 409 PLACE_010/404 PLACE_011
캘린더: GET `/api/v1/calendar`(year,month 필수) — markedDates[], events{date:festivals[]}. GET `/api/v1/calendar/daily`(date 필수)

비고(이 배치 공통): 비밀번호 필드는 타입/검증규칙만 기록, 예시값은 더미. Refresh Token은 HttpOnly Cookie로만 전달.

---

## CS 도메인 — FAQ (Admin/User)

GET/POST `/api/v1/admin/faqs` (✅ADMIN) — content[]{faqId,question,answer,category,isActive,createdAt,updatedAt} / POST Request `FaqCreateRequest`: question(max500),answer(max5000),category(max50)
PATCH `/api/v1/admin/faqs/{faqId}` — 부분수정. DELETE — soft delete(isActive=false), 204. 에러: 404 FAQ_001
GET `/api/v1/faqs` (✅USER) — isActive=true만, category 필터+cursor
GET `/api/v1/faqs/{faqId}/translation` (✅USER) — Query: targetLanguage(필수). Response: faqId,question(번역),answer(번역),targetLanguage. 에러: 404 FAQ_001

---

## CS 도메인 — 상담방 (USER·MERCHANT)

POST `/api/v1/support/rooms` (201) — Request: initialMessage?(max1000). Response `SupportRoomResponse`: supportRoomId,userId,adminId?,status(WAITING/IN_PROGRESS/CLOSED),summary?(ADMIN만 노출),closedAt?,createdAt. 에러: 409 CS_004(이미진행중)
GET `/api/v1/support/rooms/me` — status 필터+cursor
GET `/api/v1/support/rooms/{supportRoomId}/messages` — 본인방만. content[]{messageId,senderId,senderRole,messageType,content?,fileUrl?,fileName?,fileSize?,sentAt}. 에러: 404 CS_001, 403 CS_003
POST `/api/v1/support/rooms/{supportRoomId}/files` (multipart, 201) — file(이미지JPEG/PNG/WebP≤5MB, 문서PDF/DOCX/XLSX/PPTX/HWP≤10MB)
Response: fileUrl(S3키),fileName,fileSize,contentType. 에러: 404 CS_001, 403 CS_003, 400 CS_006/CS_007/CS_008/CS_009, 400 COMMON_012

---

## CS 도메인 — Admin 상담방 관리

GET `/api/v1/admin/support/rooms` — content[]{supportRoomId,userId,userNickname,status,lastMessage{content?,sentAt?},createdAt}
GET `/api/v1/admin/support/rooms/{supportRoomId}/messages` — ADMIN은 소유자무관 접근. 에러: 404 CS_001
POST `.../assign` — 호출 ADMIN 배정, WAITING→IN_PROGRESS. 에러: 404 CS_001, 409 CS_005(이미배정)
POST `.../close` — Request: summary?(max1000). 에러: 404 CS_001, 409 CS_002(이미종료)
POST `.../files` — 배정된 ADMIN만. 에러 동일 패턴(CS_006~009, COMMON_012)

STOMP `/pub/support/rooms/{supportRoomId}/messages` (구독: `/sub/support/rooms/{supportRoomId}`) — Request `SupportSendMessageRequest`: messageType?(기본TEXT),content(필수),fileUrl/fileName/fileSize(IMAGE/FILE 필수)
에러(`/user/queue/errors`): AUTH_006/AUTH_003/CS_001/CS_003/COMMON_001

---

## Community 도메인 — 댓글

POST `/api/v1/community/posts/{postType}/{postId}/comments` (✅, 201) — Request: content(`@NotBlank max1000`),parentCommentId?(대댓글)
Response: commentId,parentCommentId?,content,writer{userId,nickname,profileImageUrl,companionScore=null},createdAt
에러: 404 COMMUNITY_001/COMMUNITY_005, 403 COMMUNITY_009/AUTH_024, 400 COMMUNITY_008(대댓글에 답글금지)
GET `.../comments` (❌) — 루트댓글 cursor페이징. content[]{commentId,parentCommentId?,content("(삭제된 댓글)" 처리),writer(삭제시 전체null),createdAt,updatedAt?,replyCount?,deleted}
GET `.../comments/{commentId}/replies` (❌) — 대댓글 전체. 에러: 404 COMMUNITY_001/005
PATCH/DELETE `.../comments/{commentId}` (✅작성자만) — 에러: 404, 400 COMMUNITY_007(이미삭제), 403 COMMUNITY_006

---

## Community 도메인 — 동행 게시글

POST `/api/v1/community/posts/companions` (✅USER, 201) — Request `CompanionPostCreateRequest`
| 필드 | 타입 | 필수 | 검증 | 설명 |
|---|---|---|---|---|
| title | String | ✅ | `@NotBlank @Size(max=200)` | 제목 |
| content | String | ✅ | `@NotBlank @Size(max=5000)` | 본문 |
| targetType | CompanionTargetType | ✅ | `@NotNull` | PLACE / MARKET / FESTIVAL |
| targetId | Long | ✅ | `@NotNull @Positive` | 만남 장소 대상 ID |
| targetName | String | ✅ | `@NotBlank @Size(max=100)` | 만남 장소명 |
| region | String | ❌ | `@Size(max=50)` | 지역 필터용 |
| meetingDate | LocalDate | ✅ | `@NotNull @FutureOrPresent` | 동행 희망 날짜 |
| maxMembers | int | ✅ | `@NotNull @Min(2) @Max(50)` | 최대 모집 인원 |

Response: postId,title,content,targetType,targetId,targetName,region?,meetingDate,maxMembers,currentMembers,status,writer,createdAt
에러: 403 AUTH_024(제재)

GET 목록(❌) — region/meetingDate 필터+cursor. content[]{postId,chatRoomId,title,targetName,region?,meetingDate,maxMembers,currentMembers,status,writer,createdAt}
GET 단건(❌) — +chatRoomStatus,content,targetType,targetId,targetName,updatedAt. 에러: 404 COMMUNITY_001
PUT (✅작성자만) — Request `CompanionPostUpdateRequest`: title?,content?,targetType/targetId/targetName(3개 묶음 변경),region?,meetingDate?,maxMembers? — 에러: 404, 403 COMMUNITY_003/004
DELETE (✅작성자만) — 에러: 404, 403 COMMUNITY_003/004

---

## Community 도메인 — 자유 게시글

POST `/api/v1/community/posts/free/images` (✅USER, 201) — 이미지 업로드 (PostImageController, KAN-317)
Content-Type: multipart/form-data. Request: `file`(MultipartFile, JPEG/PNG/WebP, ≤10MB)
Response `PostImageUploadResponse`: objectKey(S3 객체 키). 글 작성 시 imageUrls에 담아 전송(최대 5장, 순서 유지).

POST `/api/v1/community/posts/free` (✅USER, 201) — Request: title(max200),content(max5000),imageUrls?(S3 objectKey 배열, max5장, 각max500)
GET 목록/단건(❌) — content[]{postId,title,imageUrls,writer,createdAt} / 단건+content,updatedAt. 에러: 404 COMMUNITY_001
PATCH/DELETE (✅작성자만) — 에러: 404, 403 COMMUNITY_003/004, 403 AUTH_024(제재유저 작성시도)

---

## Search 도메인 — v1

GET `/api/v1/search` (❌) — q?,type(ALL/PLACE/SHOP/MENU/FESTIVAL,기본ALL),cursor,size,track(기본true),source?. Response: content[]{targetType,...타입별필드}, nextCursor,hasNext
GET `/api/v1/search/popular` (❌) — TOP10: rank,keyword,searchCount,changeType(SAME/UP/DOWN/NEW)
GET `/api/v1/search/places` (❌,로그인시 선호반영) — q?,category?,region?,cursor(Long),size(기본10),track,source?
Response `TypoCorrectedSearchResponse<SearchPlaceResponse>`: content[]{placeId,name,category,address,imageUrl,rating,reviewCount}, nextCursor,hasNext,didYouMean?(오타교정)
GET `/api/v1/search/festivals` (❌, v1: location/thumbnailUrl 키) — q?,startDate?,endDate?,region?,cursor,size,track,source?. didYouMean 지원
GET `/api/v1/search/suggest` (❌) — q(필수,최소1자) → 자동완성 최대5개. 에러: 400 PLACE_005/PLACE_006(50자초과)
POST `/api/v1/search` (✅USER) — Request: keyword(max50) 최근검색어 저장
GET/DELETE `/api/v1/search/recent` (✅USER) — 목록(최대10,최신순) / 단건·전체삭제(keyword? 파라미터)

## Search 도메인 — v2

GET `/api/v2/search/festivals` (❌, v2: address/imageUrl 키) — v1과 동일 파라미터, 응답 키명만 차이(address 대신 location 아님 — address/imageUrl 사용)

---

## Chat 도메인 — 채팅방

POST `/api/v1/chat/rooms` (✅USER, 201) — Request `CreateChatRoomRequest`: postId(필수),title(max50),description?,maxMembers(2~50). Response: chatRoomId. 에러: 404 COMMON_005, 409 CHAT_014(게시글당 1방)
GET `/api/v1/chat/rooms` (✅USER) — cursor 페이징(기본10). items[]{chatRoomId,title,currentMembers,maxMembers,lastMessage?(미구현null),unreadCount(미구현0),status}
GET `/api/v1/chat/rooms/{roomId}` (✅USER) — Response `ChatRoomDetailResponse`: postId,description?,ownerId,maxMembers,status,myMemberState,members[]{userId,nickname,profileImageUrl?,memberState,joinedAt}. 에러: 404 CHAT_001, 403 CHAT_005
PATCH `/api/v1/chat/rooms/{roomId}/owner` (✅USER,204) — Request: newOwnerId(필수). 에러: 404 CHAT_001, 403 CHAT_006/CHAT_019, 409 CHAT_013
PATCH `/api/v1/chat/rooms/{roomId}/close` (✅USER,204,방장전용) — 에러: 404, 403 CHAT_006
GET `/api/v1/chat/rooms/{roomId}/members` (✅USER) — Response: List{userId,nickname,profileImageUrl?,companionScore,memberState,joinedAt}
DELETE `/api/v1/chat/rooms/{roomId}/members/{targetUserId}` (✅USER,204,강퇴,방장전용) — 에러: 404, 403 CHAT_006/CHAT_017(방장강퇴불가), 409 CHAT_016
DELETE `/api/v1/chat/rooms/{roomId}/members/me` (✅USER,204,퇴장) — 에러: 404, 403 CHAT_015(방장은위임후퇴장), 409 CHAT_010
GET `/api/v1/chat/rooms/{roomId}/messages` (✅USER) — cursor(기본50,idDESC). items[]{messageId,senderId?,senderNickname?,senderProfileImageUrl?,messageType(TEXT/IMAGE/FILE/SYSTEM),content,fileUrl?,fileName?,fileSize?,sentAt}. 에러: 404, 403 CHAT_005
POST `/api/v1/chat/rooms/{roomId}/files` (multipart,201) — file(이미지≤5MB,문서≤10MB). Response: fileUrl,fileName,fileSize,contentType. 에러: 404, 400 CHAT_020/021/022/023, 403 CHAT_005

## Chat 도메인 — STOMP 메시지 전송 (WebSocket)
> 빠졌던 항목 — 추가 (`ChatMessageController.java`, `@Controller`+`@MessageMapping`, REST 아님)

| 항목 | 내용 |
|---|---|
| Method | STOMP @MessageMapping |
| Endpoint | `/pub/chat/rooms/{chatRoomId}/messages` |
| 설명 | 채팅 메시지 수신 → 멤버 검증·저장·Redis Pub/Sub 발행 |
| 구독 경로 | `/sub/chat/rooms/{chatRoomId}` (Redis Pub/Sub → SimpMessagingTemplate 브로드캐스트) |
| 인증 | ✅ USER (Principal — StompChannelInterceptor가 SEND 프레임에서 검증) |

### Request — `ChatSendMessageRequest`
| 필드 | 타입 | 필수 | 설명 |
|---|---|---|---|
| messageType | MessageType | ❌ | TEXT(기본값)/IMAGE/FILE — SYSTEM은 클라이언트 지정 불가 |
| content | String | TEXT는 필수 | 메시지 내용 또는 IMAGE/FILE 캡션 |
| fileUrl | String | IMAGE/FILE 필수 | 업로드 API(`POST /api/v1/chat/rooms/{roomId}/files`)가 반환한 객체 키 |
| fileName | String | FILE 필수 | 업로드 API가 반환한 원본 파일명 |
| fileSize | Long | FILE 필수 | 업로드 API가 반환한 파일 크기(bytes) |

### 에러 응답 — `/user/queue/errors` (`StompErrorResponse`: code, message)
| code | HTTP 의미 | 설명 | 발생 조건 |
|---|---|---|---|
| AUTH_006 | 401 | 인증 필요 | principal=null (정상 흐름에서 도달 불가, 방어코드) |
| AUTH_003 | 401 | 토큰 유효하지 않음 | principal.getName() Long 파싱 실패 |
| COMMON_006 | 429 | 요청 과다 | rate limit 초과 |
| CHAT_005 | 403 | 채팅방 미참여 | 멤버 아님 |
| AUTH_015 | 404 | 사용자 미존재 | userId로 Account 조회 실패 |
| COMMON_002 | 400 | 잘못된 요청 | messageType별 필수 필드(content/fileUrl/fileName) 누락 |
| CHAT_024 | 403 | 파일 소유권 불일치 | fileUrl이 해당 채팅방에 업로드된 파일 아님(IDOR 방지) |
| CHAT_001 | 404 | 채팅방 미존재 | chatRoomId 없음 |

### 비고
- IMAGE/FILE 전송 시 `fileUrl`은 반드시 같은 채팅방에 먼저 업로드 API로 받은 객체 키여야 함 — 다른 방 파일 키를 넣으면 CHAT_024로 차단(IDOR 방지).
- 예상치 못한 예외는 `INTERNAL_SERVER_ERROR`로 치환해 전송, 상세 스택트레이스는 서버 로그에만 보존.

---

## Chat 도메인 — 참여 신청
POST `/api/v1/chat/rooms/{chatRoomId}/join-requests` (✅USER,201,신청 생성) — Request `CreateJoinRequestRequest`: message?(String,`@Size(max=500)`,선택). Response `CreateJoinRequestResponse`: joinRequestId,chatRoomId,writer{userId,nickname,profileImageUrl,companionScore},message,status(PENDING),createdAt. 에러: 404 CHAT_001, 403 CHAT_010(강퇴재참여), 409 CHAT_002(정원초과)/CHAT_003(이미참여)/CHAT_004(중복신청)/CHAT_013(종료된방)

GET `/api/v1/chat/rooms/join-requests/me` (✅USER) — 본인 전체신청(전상태) cursor페이징. items[]{joinRequestId,chatRoomId,chatRoomTitle?(방삭제시null),message?,status,createdAt}
GET `/api/v1/chat/rooms/{chatRoomId}/join-requests` (✅USER,방장전용) — List{joinRequestId,applicant{userId,nickname,profileImageUrl?,companionScore},message?,status,createdAt}. 에러: 404, 403 CHAT_006
POST `.../{requestId}/approve` (방장전용) — Response: joinRequestId,chatRoomId,status(APPROVED),currentMembers. 에러: 404 CHAT_001/CHAT_011, 403 CHAT_006, 409 CHAT_002(정원초과)/CHAT_012(이미처리)
POST `.../{requestId}/reject` (방장전용) — Response: status(REJECTED). 에러: 동일패턴
DELETE `.../{requestId}` (본인,204) — 에러: 404, 403 CHAT_018(본인만), 409 CHAT_012

---

## Companion 도메인

POST `/api/v1/chat/rooms/{roomId}/companion` (✅USER,201,방장전용) — Request `CompanionCreateRequest`: participantUserIds[](필수,빈배열허용),tripStartDate,tripEndDate(필수)
Response: companionId,chatRoomId,status(ONGOING),participantUserIds[],createdAt,tripStartDate,tripEndDate
에러: 404 CHAT_001, 403 CHAT_006/CHAT_005, 400 COMMON_004(날짜역전), 409 CHAT_013/CR_004/CR_007/CR_010(기간겹침)
POST `.../companion/participants` (201,방장전용) — Request: userIds[](NotEmpty,max50). Response: companionId,addedUserIds[](중복제거). 에러: 404 CHAT_001/CR_005, 403 CHAT_006/CHAT_005, 409 CHAT_013/CR_006/CR_008/CR_010
DELETE `.../companion` (204,방장전용,취소) — 하드삭제(이력미보존), 취소후 재생성가능. 에러: 404, 403 CHAT_006, 409 CR_006
PATCH `.../companion/participation/end` (204) — 여행종료후 본인참여종료(리뷰작성전제). 에러: 404, 403 CR_013, 409 CR_014(미종료)/CR_015(중복호출)

## Companion 도메인 — 리뷰

POST `/api/v1/companion-reviews` (✅USER,201) — Request: chatRoomId,targetUserId,score(1~5),content?(max1000)
Response: reviewId,chatRoomId,writerUserId,targetUserId,score,content,createdAt
에러: 404 CR_005, 400 CR_002(자기자신), 403 CR_003/CR_012(정지계정), 409 CR_001(중복)/CR_004(미종료)/CR_011(양쪽미종료)
GET `/api/v1/users/{userId}/companion-score` (❌공개) — Response: userId,nickname,averageScore,reviewCount,scoreDistribution{1~5:count}. 에러: 404 AUTH_015
GET `/api/v1/users/{userId}/companion-reviews` (✅USER) — cursor(idDESC). items[]{reviewId,reviewerId?,reviewerNickname,reviewerProfileImageUrl?,score,content?,createdAt}. 에러: 404 AUTH_015

---

## Merchant 도메인

POST `/api/v1/merchants/apply` (✅USER,201) — Request: shopName,businessNumber(`@Pattern`10자리),category,address,lat,lng(필수),phone?,description?
Response: applicationId,userId,shopName,businessNumber,category,address,status(PENDING),rejectReason(null),createdAt
에러: 400 MERCHANT_002, 409 MERCHANT_001(중복신청)/MERCHANT_004(중복사업자번호)

GET `/api/v1/admin/merchant-applications` (✅ADMIN) — cursor+status필터(기본PENDING)
GET `.../{applicationId}` — 상세. 에러: 404 MERCHANT_006
PATCH `.../approve` — Request: placeId?(선택,Shop-Place연결). 에러: 404, 409 MERCHANT_005
PATCH `.../reject` — Request: rejectReason(필수,max200). 에러: 404, 409 MERCHANT_005

---

## Report 도메인

POST `/api/v1/reports` (✅USER,201) — Request: targetType(POST_FREE/POST_COMPANION/COMMENT/REVIEW/USER/MERCHANT),targetId,reason(HARASSMENT/SPAM/INAPPROPRIATE/ETC),description?(max500)
Response: reportId,targetType,targetId,reason,status(PENDING),createdAt
에러: 404 REPORT_001, 400 REPORT_003(자기신고)/REPORT_004(신고불가대상), 409 REPORT_002(중복신고)
GET `/api/v1/reports/me` / `/{reportId}` (✅USER) — 본인 신고이력. 에러: 404 REPORT_005

GET `/api/v1/admin/reports/pending-count` (✅ADMIN) — count
GET `/api/v1/admin/reports` (✅ADMIN) — status/targetType/reason/reportedUserId 필터+cursor. content[]{...,action?,reporterNickname,adminNote?,resolvedBy?,resolvedAt?,reportedUserStatus?,reportedUserSanctionType?}
GET `/api/v1/admin/reports/{reportId}` — 상세+targetTitle?/targetContent?/targetImageUrls?[]+reportedUserSanction{accountStatus,accountSanctionType?,accountSanctionEndAt?,activeDomainSanctions[]}. 에러: 404 REPORT_005
POST `.../resolve` (콘텐츠신고) — Request: action(WARNING/SUSPEND/DELETE/DISMISS),adminNote?. 에러: 404, 400 REPORT_007(가게신고는 별도엔드포인트), 409 REPORT_006
POST `.../resolve/merchant` (가게신고) — Request: action(HIDE_SHOP/REVOKE_MERCHANT/DISMISS),adminNote?. 에러: 404, 400 REPORT_007, 409 REPORT_006
PATCH `.../status` (오판정정,RESOLVED→DISMISSED만) — Request: status(DISMISSED고정),adminNote?. 에러: 404, 400 REPORT_010, 409 REPORT_006

---

## Notification 도메인

GET `/api/v1/notifications` (✅USER) — cursor(idDESC,기본20,max50). items[]{notificationId,type,title,message,referenceId?,referenceType?,isRead,createdAt}
PATCH `/api/v1/notifications/read-all` (204) — 전체읽음
PATCH `/api/v1/notifications/{notificationId}/read` (204) — 단건읽음. 에러: 404 NOTIFICATION_001
DELETE `/api/v1/notifications/{notificationId}` (204) — 에러: 404 NOTIFICATION_001

---

## Translation 도메인

POST `/api/v1/translations` (✅USER) — Request `TranslationRequest`: content(max1000),targetLanguage,sourceType(필수, FAQ는 거부)
Response: translatedContent,targetLanguage. 에러: 400 COMMON_002(sourceType=FAQ→전용엔드포인트 사용), 503 COMMON_007

---

## 공통 에러 응답 포맷

```json
{ "code": "ERROR_CODE", "message": "에러 메시지", "data": null }
```
HTTP Status: 200/201/204(성공) · 400(검증·비즈니스위반) · 401(인증) · 403(권한) · 404(미존재) · 409(상태충돌) · 503(외부서비스장애)

---

