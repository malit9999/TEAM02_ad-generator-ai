협업일지 링크: https://app.notion.com/p/38ffc56d6c708069b591e59de1beebc7

# 🎨 AI 상세페이지 자동 생성 서비스 (Service 01)

> **제품 사진, 정보, 스타일 선택만으로 B2B/B2C 커머스용 상세페이지(긴 이미지)와 썸네일을 3분 만에 자동 생성하는 End-to-End AI 파이프라인입니다.**

---

## 🌟 주요 기능 (Key Features)

* **🤖 LLM 기반 카피라이팅 & 이미지 렌더링 프롬프트 생성**: `gpt-5-mini`를 사용하여 제품 카테고리와 스펙에 최적화된 마케팅 문구 및 컷별 이미지 생성 프롬프트를 자동 조립합니다.
* **🖼️ 멀티 컷 이미지 생성 & 합성 (Composite)**: `gpt-image-1-mini`로 연출 배경을 생성하고, `rembg` 기반 배경 제거(누끼) 알고리즘으로 원본 제품컷을 자연스럽게 합성합니다.
* **📐 PIL 기반 상세페이지/썸네일 조립 (Composer)**: 레이아웃 규격, 폰트 스타일, 무드, 톤앤매너에 맞게 카피와 이미지를 원스톱으로 배치하여 통합 긴 이미지 및 메인/부가 썸네일을 출력합니다.
* **🛡️ Fallback & Resilience 매커니즘**: AI 연출 제안이 검증을 통과하지 못할 경우 안전한 기본 템플릿/패스스루 경로로 전환하여 100% 생성 완주를 보장합니다.
* **🔒 보안 & 휘발성 파일 관리**: 업로드된 제품컷 및 중간 생성물은 처리 후 자동 삭제되며, 외부로 노출되지 않는 임시 스토리지 기반으로 작동합니다.

---

## 🏗️ 시스템 아키텍처 (System Architecture)

[ Frontend ]               [ Worker / Backend ]                   [ AI Model Pipeline ]
 Streamlit (8501)  ────>  FastAPI (8010/8000)  ────>  model_v2 Pipeline
  (UI / 미리보기)            (API / 파이프라인 수행)          ├── LLM Copywriter (gpt-5-mini)
                                                            ├── Image Generator (gpt-image-1-mini)
                                                            ├── Background Remover (rembg)
                                                            └── Page Composer (PIL)

---

## 📂 프로젝트 구조 (Directory Structure)

TEAM02_ad-generator-ai/
├── frontend/                     # Streamlit 기반 프론트엔드 UI
│   ├── app.py                    # V1 애플리케이션 진입점
│   └── model_v2_app.py           # V2 멀티 파이프라인 지원 UI
├── backend/                      # FastAPI 백엔드 API 서버
│   ├── main.py                   # V1 FastAPI 진입점
│   ├── main_v2.py                # V2 Worker 서비스 진입점
│   ├── schemas.py                # Request/Response 규격 정의
│   ├── requirements.txt          # 백엔드/워커 파이썬 의존성
│   └── Dockerfile                # 백엔드/워커 전용 도커 빌드 파일
├── model_v2/                     # AI 핵심 파이프라인 & 데이터
│   ├── baseline/                 # 아키타입, LLM 프롬프트, 카피/이미지 생성기
│   ├── composer/                 # PIL 기반 상세페이지 & 썸네일 합성 레이아웃
│   ├── image/                    # 임시 업로드/생성 이미지 폴더 (gitignore)
│   └── .env                      # API 키 및 환경변수 (gitignore)
├── docker-compose.yml            # 멀티 컨테이너 통합 오케스트레이션
└── README.md                     # 프로젝트 안내 문서

---

## 🚀 시작하기 (Quick Start)

### 1. 환경변수 설정 (.env)

model_v2/.env (또는 model/.env) 파일을 생성하고 OpenAI API 키를 입력합니다.

$ cp model_v2/.env.example model_v2/.env
OPENAI_API_KEY=sk-proj-...

---

### 2. Docker Compose로 실행 (권장) 🐳

도커 환경으로 구동하면 패키지 및 C++ 런타임 종속성(rembg, opencv, numba 등)의 꼬임 없이 환경이 단권화되어 한 번에 켜집니다.

# 1. 이전 컨테이너 정리 & 캐시 없이 완전히 깨끗하게 재빌드 구동
$ docker compose up -d --build

# 2. 서비스 실행 상태 확인 (status가 Up 인지 체크)
$ docker compose ps

# 3. 실시간 워커 로그 모니터링
$ docker compose logs -f worker-v2

> 🌐 서비스 접속 주소
> * 프론트엔드 (UI): http://localhost:8501
> * Worker-v2 백엔드 (API): http://localhost:8010
> * Main 백엔드 (API): http://localhost:8000

---

### 3. 로컬 가상환경에서 실행 (선택사항) 🐍

# 1. 가상환경 생성 및 활성화
$ python3.10 -m venv .venv
$ source .venv/bin/activate

# 2. 필수 패키지 설치
$ pip install --upgrade pip
$ pip install -r backend/requirements.txt

# 3. 백엔드 실행 (터미널 1)
$ uvicorn backend.main_v2:app --host 0.0.0.0 --port 8010 --reload

# 4. 프론트엔드 실행 (터미널 2)
$ streamlit run frontend/model_v2_app.py --server.port=8501

---

## 💡 사용 방법 (User Guide)

1. 사이트 설정 (사이드바): 타겟 플랫폼 규격, 이미지 무드/밝기, 톤앤매너, 아키타입 선택
2. 제품 정보 입력: 상품명, 카테고리, 핵심 스펙/주요 특징 입력
3. 이미지 업로드:
   * 대표 제품컷 (필수): 단독 제품 촬영컷 (배경 제거 및 Hero 컷 합성용)
   * 응용 참고컷 (선택): 실제 사용 공간, 인테리어 배치, 디테일 컷 (Lifestyle/Build 연출용)
4. [상세페이지 생성] 클릭:
   * Hero / Build / Connectivity / Lifestyle 4개 컷과 스펙 표, 메인/부가 썸네일 자동 생성 및 다운로드

---

## 🔧 기술 스택 & 종속성 (Tech Stack)

| 구분 | 기술 스택 | 설명 |
| :--- | :--- | :--- |
| Frontend | Streamlit | 대화형 인터랙티브 웹 UI 구현 |
| Backend | FastAPI, Uvicorn | 비동기 백엔드 API 서비스 및 파이프라인 게이트웨이 |
| LLM & Copy | LangChain, OpenAI gpt-5-mini | 구조화된 마케팅 카피 및 프롬프트 제안 생성 |
| Image Gen | OpenAI gpt-image-1-mini | High-quality 상품 연출 배경 및 요소 생성 |
| Vision & PIL | rembg, onnxruntime, Pillow, OpenCV | 제품 누끼(배경 제거), 알파 매팅, 텍스트/이미지 레이아웃 합성 |
| DevOps | Docker, Docker Compose | 멀티 컨테이너 단권화 배포 환경 구축 |

---

## 💰 예상 소요 비용 (Cost Estimation)

* 생성 1회 당 비용: 약 $0.06 ~ $0.08
  * Text API (gpt-5-mini): 카피 생성 4~5회 호출 (~$0.005)
  * Image API (gpt-image-1-mini): 컷당 ~$0.02 (3~4컷 생성 시 약 $0.06)
# 터미널 1 — 백엔드 (FastAPI, 포트 8000)
cd backend && uvicorn main:app --reload

# 터미널 2 — 프론트 (Streamlit, 포트 8501)
streamlit run frontend/app.py
```
> 프론트가 백엔드(`localhost:8000`)에 요청 → 상세페이지·썸네일을 받아 표시.
> 업로드/결과는 서버에 저장되지 않음(임시폴더 자동 삭제 — 휘발성).

## 사용
1. 사이드바에서 스타일(사이트규격·밝기·무드·톤 등) 선택
2. 상품명·카테고리·스펙 입력 + 제품 이미지 업로드 (+응용 이미지 선택)
3. **[상세페이지 생성]** → 상세이미지 + 썸네일 출력·다운로드

> 비용: 생성마다 이미지 컷 수 × ~$0.02 (gpt-image-1-mini).
