# 핏센스 (FitSense)

내 키·몸무게와 옷의 실측 사이즈표를 비교해 **어떤 사이즈가 맞을지 추천**하는 의류 사이즈 추천 서비스의 웹 프로토타입입니다.
후기에 쓰인 체형·구매 사이즈·체감(작은 편/정사이즈/큰 편)을 모아, 나와 비슷한 체형의 후기를 근거로 보여줍니다.

## 실행 방법
JSON을 `fetch`로 읽기 때문에 `index.html`을 더블클릭(`file://`)하면 데이터가 안 뜹니다.
- **GitHub Pages**: 저장소 Settings → Pages → Branch `main` / root → 발급된 주소로 접속
- **로컬**: 이 폴더에서 `python -m http.server 8000` 실행 후 http://localhost:8000

## 폴더
```
index.html              화면 틀 + 시작 코드(데이터 fetch → checkData → 화면)
css/style.css           스타일 (모바일 우선, PC에서는 중앙 정렬)
js/data.js              data/*.json → 앱 형식으로 변환(mapDB), 상품 이미지 경로 규칙(IMG_ON)
js/check.js             checkData(): REQUIRED / ALLOWED / TYPES / ID 참조 검사
js/core.js              핏 계산(evalSize·recommend), 상태(state)·localStorage, 라우터, SVG
js/home_measure.js      홈 · 사이즈 측정(① 내 신체 ② 옷 입력) · 결과
js/browse.js            브랜드 · 상품 목록(검색/필터/정렬) · 상품 상세 · 후기
js/records_my.js        기록 · 내 옷장 · 찜 · 알림 · 마이 · 문의
data/                   brands / products / reviews .json (+ test_cases.json: 테스트 전용, 앱 기본 화면은 읽지 않음)
README.md               이 문서 (데이터 명세 · 출처/실제·더미 구분표 포함)
```

## 주요 기능
| 기능 | 설명 |
|---|---|
| 사이즈 측정 | 키·몸무게(+선택 실측) 입력 → 상품 사이즈표와 비교해 부위별 타이트/적당/여유 판정, 추천 사이즈 |
| 브랜드·상품 탐색 | 성별·카테고리·세부 카테고리, 검색, 필터(핏·소매·신축성·가격·색상), 정렬 |
| 후기 | 나와 비슷한 체형(키 ±6cm·몸무게 ±7kg) 후기만 보기, 사이즈·키워드 필터 |
| 기록 · 내 옷장 | 분석 기록, 입어본 옷 피드백 → 다음 추천에 반영(마이핏드백) |
| 마이 | 프로필, 선호 스타일, 알림, 비밀번호·개인정보, 찜 |

## 데이터: JSON과 LocalStorage의 구분
| 저장 위치 | 내용 |
|---|---|
| `data/*.json` (읽기 전용) | 브랜드 64 · 상품 725 · 후기 4,713 (상품·후기는 전부 더미) |
| 브라우저 LocalStorage (`fitsense_wf_v5`) | 내 신체 치수, 프로필, 선호 스타일, 찜, 내 옷장·기록, 알림, 사용자가 쓴 후기·답글 |

자세한 필드·허용값과 출처·실제/더미 구분은 아래 **데이터 명세**와 **출처·구분표**를 보세요.

## 데이터 검증
브라우저에서 `checkData()`가 페이지를 열 때마다 실행되고, 문제는 콘솔(`[checkData] …`)과 `window.DATA_ISSUES`에 남습니다. 정상이면 `이상 없음 · brands 64 / products 725 / reviews 4718`이 표시됩니다.

## 알려진 한계
- 상품·후기는 **더미**입니다. 실제 상품 정보나 실제 구매자 후기가 아닙니다.
- 브랜드 사이트 링크는 `meta.site_verified`가 `false`인 항목이 있습니다(직접 접속해 확인 필요).
- 상품 이미지는 아직 없습니다. 넣는 방법: 저장소에 `asset/products/{product_id}.jpg`를 올리고 `js/data.js`의 `IMG_ON`을 `true`로 바꿉니다.
- 로그인·후기·좋아요 등 사용자 데이터는 서버가 아니라 **내 브라우저에만** 저장됩니다(심화 트랙에서 DB 연동 예정).

## 데이터 명세

공통 규칙: ID는 문자열·중복 없음 / 날짜 `YYYY-MM-DD` / 불리언 `true|false` / 값이 없으면 `null`(빈 문자열·"N/A" 금지) / 사이즈표 단위 cm(가슴·허리·힙·허벅지는 **둘레**).

### brands.json (64행)
| 필드 | 타입 | 필수 | 허용값/예시 |
|---|---|---|---|
| brand_id | string | ✔ | `BR001` |
| name_kr / name_en | string | ✔ | `유니클로` / `UNIQLO` |
| site_url | string | ✔ | 공식 사이트 |
| size_guide_url | string/null | | |
| genders | string[] | ✔ | `여성`, `남성` (공용 상품이 있으면 둘 다) |
| categories | string[] | ✔ | `상의` `하의` `아우터` `원피스` |
| styles | string[] | ✔ | 1~3개: 캐주얼·시티·시티보이·클래식·에스닉·걸리시·고프코어·미니멀·프레피·리조트·레트로·로맨틱·스포티·스트릿·워크웨어 |
| size_tendency | string | ✔ | `작게 나옴` `정사이즈` `크게 나옴` |
| price_tier | string | ✔ | `저가` `중가` `고가` `프리미엄` |
| meta | object | ✔ | source, source_url, real[], dummy[], site_verified, api, collected_at |

### products.json (725행, 브랜드당 10~13개)
| 필드 | 타입 | 필수 | 허용값/예시 |
|---|---|---|---|
| product_id | string | ✔ | `P0001` |
| brand_id | string | ✔ | brands.json 참조 |
| name | string | ✔ | `(가상) 베이직 티셔츠` |
| category | string | ✔ | `상의` `하의` `아우터` `원피스` (신발 없음) |
| sub_category | string | ✔ | 티셔츠·니트·데님 팬츠·패딩·롱 원피스 등 |
| gender | string | ✔ | `여성` `남성` `공용` |
| fit_type | string | ✔ | `슬림` `레귤러` `오버` |
| sleeve_type | string/null | | `반소매` `긴소매` `민소매` / 하의는 null |
| stretch | string | ✔ | `없음` `약간` `좋음` |
| price | number | ✔ | 원 단위 정수 |
| colors / sizes | string[] | ✔ | 색상명 / `XS`~`XL` |
| size_chart | object/null | | `{ "M": { "어깨":.., "가슴":.. } }` 부위는 아래 |
| release_date | string | ✔ | 최신순 정렬 |
| meta | object | ✔ | |

size_chart 부위: 상의·아우터 `어깨 가슴 소매 총장` / 하의 `허리 힙 허벅지 총장` / 원피스 `어깨 가슴 허리 총장`. 사이즈가 커질수록 값이 커져야 합니다.

### reviews.json (4,713행, 상품당 5~16개)
| 필드 | 타입 | 필수 | 허용값/예시 |
|---|---|---|---|
| review_id / product_id | string | ✔ | `R00001` / products 참조 |
| reviewer_id / reviewer_nick / reviewer_handle | string | ✔ | `U0001` / 닉네임 / 공개용 ID |
| review_date | string | ✔ | `2026-10-24` |
| rating | number | ✔ | 1~5 정수 |
| height_cm / weight_kg | number | ✔ | |
| fit_preference | string | | `슬림 핏` `기본 핏` `여유 핏` |
| purchased_size / fitted_size | string | ✔ | 상품 sizes 중 하나 |
| size_opinion | string | ✔ | `작은 편` `정사이즈` `큰 편` |
| length_opinion | string | ✔ | `짧은 편` `적당` `긴 편` |
| fit_details | object | | 부위별 `타이트` `적당` `여유` |
| keywords | string[] | | 앱 키워드 목록 중 최대 6개 |
| color / review_text | string | | |
| data_type | string | ✔ | `dummy` |
| meta | object | ✔ | |

### test_cases.json
`[테스트]` 접두어 상품(사이즈표 없음, 매우 긴 이름)과 **0건이 나오는 필터 조합** 목록. 앱 기본 화면에서는 읽지 않습니다.

## 출처 · 실제/더미 구분표

> 생성일 2026-11-16 · 생성 스크립트 `tools/gen_data.py` (seed 고정, 같은 결과 재현)

| 데이터 | 실제 / 더미 | 출처 | 비고 |
|---|---|---|---|
| 브랜드명(`name_kr`, `name_en`) | **실제** | 브랜드 공식 표기 | 무신사 랭킹 등에 자주 등장하는 브랜드 64개 |
| 브랜드 사이트(`site_url`) | **실제** (일부 확인 전) | 브랜드 공식 사이트 | 18개는 팀이 사전 정리한 링크(`source: official_site`), 46개는 `manual`. `site_verified:false`는 아직 접속 확인 전 → `tools/check_links.py`로 확인 |
| 브랜드 스타일 / 사이즈 경향 / 가격대 | 더미 | 팀 판단(임의 지정) | 실제 브랜드 평가가 아님 |
| 상품(이름·가격·색상·사이즈표 등 전체) | **더미** | `tools/gen_data.py` 생성 | 이름 앞 "(가상)". 실제 판매 상품이 아님 |
| 상품 개별 링크 | 두지 않음 | - | 더미 상품이라 개별 페이지가 없어, 상품 화면에서는 브랜드 공식 홈페이지로 연결 |
| 상품 이미지 | 없음 | - | 자리표시. 넣을 때는 `asset/README.md`의 출처 규칙 준수 |
| 후기 / 후기 작성자 / 키·몸무게 | **더미(합성)** | `tools/gen_data.py` 생성 | 실제 구매자 후기 아님. 크롤링하지 않음 |
| 알림·기록·내 옷장 예시 | 더미 | 앱 내 샘플 | `sample:true` 표시 |
| 핏 판정 기준값(TARGET/TOL 등) | 가정값 | 수업 실습용 가정 | 실측 연구 결과가 아님 |

- 외부 API: **사용하지 않음** (`meta.api: false`)
- 모든 데이터 항목은 `meta`(source, real, dummy, api)를 가지며, 화면의 출처·실제/더미 표시와 이 표가 일치합니다.
- 웹 크롤링·후기/지도/사진 수집은 하지 않았습니다.
