# Scrapbook 탭 설계

보고 듣고 읽은 것(영상·영화·책·음악)을 한두 줄씩 쌓아가는 아카이브 탭.
블로그 글 목록과는 다른 느낌(타임라인 / 위키)으로 보여준다.

## 목표 / 비목표

- 목표: 항목 추가가 YAML 몇 줄로 끝날 것. 글 목록과 시각적으로 구분될 것.
- 비목표: 트윗 아카이빙, 임베드 플레이어, 항목별 댓글(블로그 전체 댓글 비활성), 긴 감상(→ 별도 포스트로 쓰고 `post:` 로 연결).

## 데이터 — `_data/scrapbook.yml`

최상위가 항목 리스트. 파일 내 순서는 무관(렌더 시 `date` 내림차순 정렬).

| 필드 | 필수 | 설명 |
|---|---|---|
| `date` | ✅ | `YYYY-MM-DD`. 본/들은/읽은 날 |
| `type` | ✅ | `video` \| `movie` \| `book` \| `music` |
| `title` | ✅ | 제목 |
| `by` | | 채널 / 감독 / 저자 / 아티스트 |
| `url` | | 외부 링크. 있으면 제목이 링크가 됨 |
| `image` | | 포스터·표지·앨범아트 경로 (`/assets/img/scrapbook/…`) |
| `note` | | 한두 줄 감상 |
| `post` | | 관련 블로그 글 URL (`/posts/slug/`) |

알 수 없는 `type` 은 렌더에서 건너뛴다(빌드 실패 없음).
파일이 비어 있거나 없으면 "아직 모아둔 게 없어요" 문구만 표시.

## 탭 — `_tabs/scrapbook.md`

- `layout: scrapbook`, `icon: fas fa-paperclip`, `order: 2`
- 기존 탭 order 를 하나씩 밀어 순서: Categories(1) · Scrapbook(2) · Tags(3) · Archives(4) · About(5)
- `_data/locales/en.yml` 의 `tabs:` 에 `scrapbook: Scrapbook` 추가 (현재 UI 가 en 폴백)
- URL: `/scrapbook/`

## 레이아웃 — `_layouts/scrapbook.html`

`layout: page` 를 상속. 두 보기를 빌드 타임에 모두 생성하고 JS 로 표시만 전환.

1. **상단 토글** `[타임라인 | 종류별]`
   - 인라인 JS 로 `data-view` 전환, 선택을 `localStorage` 에 저장(try/catch)
   - JS 없으면 타임라인만 보임(종류별은 기본 `hidden`)
2. **타임라인 보기**: 연-월(`2026.09`) 헤더로 묶고, 세로선 + 점 + 카드. 카드에 종류 아이콘·날짜.
3. **종류별 보기**: 영상 · 영화 · 책 · 음악 섹션(비어 있는 섹션은 생략), 상단에 섹션 앵커 목차, 섹션 내부는 최신순.

### 카드 (`_includes/scrapbook-item.html`, 두 보기가 공유)

- 종류 아이콘(Font Awesome, Chirpy 에 이미 로드됨): video `fa-play`, movie `fa-film`, book `fa-book`, music `fa-music`
- 썸네일 우선순위: `image` → (`type: video` 이고 YouTube URL 이면) `https://i.ytimg.com/vi/<ID>/mqdefault.jpg` → 없음
  - YouTube ID 추출: `youtu.be/<id>`, `youtube.com/watch?v=<id>`, `youtube.com/shorts/<id>` 를 Liquid `split` 으로 처리, `?`·`&` 뒤 제거
  - `loading="lazy"`
- 제목(`url` 있으면 새 탭 링크), `by`, `note`, `post` 있으면 "📝 글 보기" 링크

## 스타일 — `_sass/custom/_scrapbook.scss`

- `assets/css/jekyll-theme-chirpy.scss` 에 `@use 'custom/scrapbook';` 추가
- Chirpy CSS 변수(`--main-bg`, `--text-color`, `--border-color`, `--link-color` 등)만 사용 → 라이트/다크 자동
- 종류별 강조색 4개는 라이트·다크 각각 정의
- 모바일(≤576px): 타임라인 좌측 여백 축소, 썸네일은 카드 위로 쌓임. 가로 스크롤 없음

## 문서

- `WRITING_DOOBY.md` 에 "Scrapbook 항목 추가" 절 추가 (필드 표 + 예시)

## 건드리지 않는 것

- 기존 Archives 탭

(`_posts/playlist/` 빈 폴더는 Scrapbook 이 대체하므로 삭제함)

## 검증

- `bundle exec jekyll build` 성공, 경고 없음
- 임시 샘플 4개(종류별 1개; YouTube URL 1개, `image` 1개, `post` 1개 포함)로 로컬 서버에서 확인:
  - 두 보기 전환 + 새로고침 후 선택 유지
  - 다크모드, 375px 폭
  - JS 비활성 시 타임라인 표시
- 알 수 없는 `type` 항목 1개 넣어 빌드가 깨지지 않는지 확인
- 샘플은 사용자가 실제 항목을 주면 그걸로 교체, 아니면 커밋 전 제거
