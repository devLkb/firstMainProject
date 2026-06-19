# 🅿️ SafeParking

현재 위치 기반으로 주정차 가능 여부를 판단하고, 금지 구역일 경우 근처 주차장을 안내하는 서비스입니다.

## 프로젝트 개요

| 항목 | 내용 |
|------|------|
| 개발 기간 | 2026.02.07 ~ 2026.02.14 (1주) |
| 팀 규모 | 4인 (프론트 2 / 백엔드 2) |
| 담당 역할 | 주정차 판별 백엔드 API, 즐겨찾기 API |
| 프론트엔드 | React Native + Expo |
| 백엔드 | Spring Boot, MySQL |

## 백엔드 아키텍처

GPS 좌표를 받아 주정차 금지 여부를 판별하는 파이프라인 구조입니다.

```
프론트엔드 (GPS 좌표 x, y)
  │
  ▼
ParkingStopCheckController   ← POST /api/pas/parking-stop-check
  │
  ▼
ParkingStopCheckService
  │
  ├─ 1) Kakao Geocoding API 호출 (x, y → 도로명주소 변환)
  │     - road_address에서 도로명, 시/도, 시/군/구 파싱
  │     - road_address 없을 경우 address로 fallback
  │
  ├─ 2) 공공데이터포털 「전국주정차금지(지정)구역표준데이터」 API 호출
  │     - 시도명(ctprvnNm) + 시군구명(signguNm) + 도로명(rdnmadr) 조합으로 조회
  │     - 데이터 존재 → prohibited: true / 없음 → false
  │
  └─ 3) ParkingStopCheckResponse (prohibited, roadAddress) 반환
```

## 내가 담당한 코드

```
backend/src/main/java/com/backend/backend/
├── pas/                          ← 주정차 판별 (내 담당)
│   ├── controller/
│   │   └── ParkingStopCheckController.java
│   ├── service/
│   │   └── ParkingStopCheckService.java
│   └── dto/
│       ├── ParkingStopCheckRequest.java
│       ├── ParkingStopCheckByAddressRequest.java
│       └── ParkingStopCheckResponse.java
├── favorite/                     ← 즐겨찾기 (내 담당)
│   ├── controller/
│   │   └── FavoriteController.java
│   ├── service/
│   │   └── FavoriteService.java
│   ├── entity/
│   │   └── Favorite.java
│   ├── repository/
│   │   └── FavoriteRepository.java
│   └── dto/
│       ├── FavoriteRequest.java
│       └── FavoriteResponse.java
└── config/
    ├── KakaoApiProperties.java   ← 외부 API 설정
    ├── ParkingStopApiProperties.java
    └── ParkingApiProperties.java
```

## API 명세

### 주정차 판별

| Method | Endpoint | 설명 |
|--------|----------|------|
| POST | `/api/pas/parking-stop-check` | GPS 좌표(x, y) → 주정차 금지 여부 판별 |
| POST | `/api/pas/parking-stop-check-by-address` | 주소 직접 입력 → 주정차 금지 여부 판별 (테스트용) |

**요청 예시**
```json
{
  "x": 127.0276,
  "y": 37.4979
}
```

**응답 예시**
```json
{
  "prohibited": true,
  "roadAddress": "경기도 성남시 분당구 불정로 6"
}
```

### 즐겨찾기

| Method | Endpoint | 설명 |
|--------|----------|------|
| POST | `/api/favorite/add` | 즐겨찾기 추가 |
| DELETE | `/api/favorite/remove?parkingName=` | 즐겨찾기 삭제 |
| GET | `/api/favorite/list` | 즐겨찾기 목록 조회 |
| GET | `/api/favorite/check?parkingName=` | 즐겨찾기 여부 확인 |

## Trouble Shooting

**공공데이터 API http/https 문제**

공공데이터포털 API 엔드포인트가 공식 문서에 `https`로 안내되어 있었으나, 실제로는 `http`로 요청해야 정상 동작하는 문제가 발생했습니다. 콘솔 로그(`System.out.println`)를 통해 요청 URL과 응답 상태를 추적하여 원인 범위를 좁힌 뒤, `http`로 전환하여 해결했습니다.

## 기술 스택

| 분류 | 기술 |
|------|------|
| 백엔드 | Spring Boot, Java |
| 데이터베이스 | MySQL |
| 외부 API | Kakao Geocoding API, 공공데이터포털 주정차금지구역 API |
| 프론트엔드 | React Native, Expo |
| 협업 | GitHub |

## 실행 방법

### 백엔드

```bash
cd backend
# application.properties에 API 키 설정 필요
./mvnw spring-boot:run
```

### 프론트엔드

프론트엔드 실행 방법은 `gps/` 디렉토리 내 설정을 참고해 주세요.
