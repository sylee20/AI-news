# AI 뉴스 데일리 브리핑 (루틴 작업 지침)

매일 아침 클라우드 루틴이 이 지침대로 **전날(KST 기준)** AI 기사 브리핑 메일을 Gmail로 보낸다.

## 기준 날짜
- `TZ=Asia/Seoul date -d yesterday +%F` 로 구한 날짜(= 전날, 한국 시간)를 사용한다.

## 소스 1 — AI타임스 (전날 기사 전체)
- 목록 페이지(https://www.aitimes.com/news/articleList.html?view_type=sm)는 서버가 `page=N` 파라미터를 무시해 1페이지만 내려준다. 페이지 넘김으로는 수집 불가.
- 대신 다음 방식으로 수집한다 (curl + `-A "Mozilla/5.0"`):
  1. RSS https://www.aitimes.com/rss/allArticle.xml (최신 50건, pubDate 포함)에서 기준 날짜 기사를 모은다.
  2. 기사 번호(idxno)는 대체로 순차 증가한다. RSS에 나온 가장 큰 idxno부터 1씩 내려가며
     `https://www.aitimes.com/news/articleView.html?idxno=N` 을 조회하고,
     `<meta property="article:published_time" content="YYYY-MM-DDTHH:MM:SS+09:00">` 로 날짜를 확인한다.
     기준 날짜 기사면 수집, 기준 날짜보다 이전 기사가 연속 30개 나오면 멈춘다. (없는 번호/삭제 기사는 건너뜀)
- 기사마다: 제목, 링크, 등록 시각, 1~2문장 한국어 요약 (og:description 또는 본문 기반).

## 소스 2 — AINews (news.smol.ai)
- https://news.smol.ai/ 에 날짜별 이슈 링크가 올라온다.
- 기준 날짜에 해당하는 이슈 링크를 찾아 열고 내용을 읽는다. (같은 날짜 이슈가 여러 개면 모두)
- 해당 날짜 이슈가 없으면 "해당 날짜 이슈 없음"으로 표기한다.
- 주요 헤드라인/토픽을 한국어로 정리(5~10개 bullet) + 원문 링크.

## 메일
- 받는 사람: 루틴 소유자의 Gmail (루틴 프롬프트에 지정)
- 제목: `[AI 뉴스 브리핑] YYYY-MM-DD`
- 본문(HTML): 
  1. 오늘의 핵심 3~5줄 요약
  2. AI타임스 — 기사 전체 목록 (총 N건, 제목 링크 + 요약)
  3. AINews (smol.ai) — 해당 날짜 이슈 요약 + 링크
- 사이트 접속 실패 시에도 메일은 보내고, 실패한 소스와 오류를 본문에 명시한다.
