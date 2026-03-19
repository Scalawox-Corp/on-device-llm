# [스칼라웍스](https://www.scalawox.com/)의 ASUS GX10 On-Premise LLM 서버를 위한 구성 명세서

이 문서는 **현재 ASUS GX10 장비에 설치/구동 중인 구성**을 정리한 **명세서**입니다. (설치 매뉴얼/운영 절차는 포함하지 않음)

> ⚠️ **중요**: 이 장비는 **Ubuntu ARM64(aarch64)** 환경입니다. 관련 소프트웨어는 **x64가 아닌 ARM64** 기준으로 구성합니다.

---

## 1. 장비 및 시스템 정보

| 항목 | 값 |
|---|---|
| 장비 모델명 | ASUS GX10 |
| 아키텍처 | aarch64 (ARM64) |
| OS | Ubuntu 24.04.3 LTS |
| 커널 | 6.14.0-1015-nvidia |
| 통합 메모리 | 128GB |
| 저장장치 | 1TB |
| GPU | NVIDIA GB10 |
| NVIDIA Driver | 580.95.05 |
| CUDA Toolkit (nvcc) | 13.0 (V13.0.88) |

---

## 2. 설치/구동 소프트웨어 버전

| 항목 | 값 |
|---|---|
| [Visual Studio Code](https://code.visualstudio.com/download) | 2.3.41 (arm64) |
| [Continue](https://marketplace.visualstudio.com/items?itemName=Continue.continue) (VS Code Extension) | 1.2.17 |
| [LM Studio](https://lmstudio.ai/download) | 0.4.6 (Build 1) |
| [Open WebUI](https://github.com/open-webui/open-webui) | v0.8.10 |
| [Page Assist](https://chromewebstore.google.com/detail/page-assist-a-web-ui-for/jfgfiigpkhlkbnfnbobbkinehhgmcaeeh) (Chrome Extension) | 1.5.57 |
| [OpenClaw](https://openclaw.ai/) | 2026.3.13 |
| uv | 0.9.7 |

### LM Studio 연동 소프트웨어별 권장 LLM 모델

| 소프트웨어 | 권장 모델 | 선택 이유 |
|---|---|---|
| [Continue](https://marketplace.visualstudio.com/items?itemName=Continue.continue) | `qwen/qwen3-coder-next` (1순위), `qwen/qwen3.5-27b` (범용 보조) | 코드 자동완성·리뷰 중심 — 코딩 특화 MoE로 빠른 응답, 범용 dense로 추론 보완 |
| [Page Assist](https://chromewebstore.google.com/detail/page-assist-a-web-ui-for/jfgfiigpkhlkbnfnbobbkinehhgmcaeeh) | `nvidia/nemotron-3-nano-30b-a3b`, `openai/gpt-oss-20b`, `qwen/qwen3.5-27b` | 브라우저 사이드바는 응답 속도 우선 — 30–50 t/s 이상 모델 권장 |
| [OpenClaw](https://openclaw.ai/) | `qwen/qwen3.5-27b`, `openai/gpt-oss-120b` | 파일·셸·브라우저 자율 조작 에이전트 — 지시 이행 정확도가 핵심, 대형 dense 모델 우선 |

---

## 3. 서비스 엔드포인트(접속 주소/포트)

> 📝 **IP 표기 규칙**
> - `<DEVICE_LAN_IP>`: 출고/현장 네트워크에서 장비에 할당된 **LAN IP** (환경마다 달라짐)
> - `127.0.0.1`: **장비 자기 자신**(로컬호스트)

| 서비스 | 바인딩 | 접속 URL |
|---|---|---|
| Open WebUI | `0.0.0.0:8080` | `http://<DEVICE_LAN_IP>:8080` 또는 `http://localhost:8080` (장비 로컬) |
| LM Studio (OpenAI 호환 API) | `127.0.0.1:1234` | `http://127.0.0.1:1234/v1` 또는 `http://localhost:1234/v1` (장비 로컬) |

> 📝 **참고**
> - LM Studio API는 현재 `127.0.0.1`에 바인딩되어 있어 **동일 장비에서만 직접 접근** 가능합니다.
> - 하지만 Open WebUI처럼 **동일 장비에서 LM Studio API를 호출**하는 애플리케이션은, 애플리케이션 자체를 `0.0.0.0`로 바인딩해 **외부(로컬망) 공개**가 가능합니다.

---

### 권장 LLM 모델

ASUS Ascent GX10 AI 슈퍼컴퓨터를 위한 LLM 모델
(lmstudio app 내에서 검색 후 결과 중 보라색 아이콘에 해당하는 것으로 설치 권장)

#### 범용 모델

| 모델명 (lms ls 기준) | 제작사 | 크기 | 설명 | 컨텍스트 | 권장 양자화 | VRAM | 초당 토큰 |
|---|---|---|---|---|---|---|---|
| [qwen/qwen3.5-27b](https://huggingface.co/Qwen/Qwen3.5-27B) | Qwen | 27B | 코딩·추론·대화 전반에 걸쳐 균형 잡힌 범용 LLM. GPT-5 mini와 SWE-bench 동점 | 262144 | Q5_K_M | ~20 GB | ~18–22 t/s |
| [qwen/qwen3.5-122b-a10b](https://huggingface.co/Qwen/Qwen3.5-122B-A10B) | Qwen | 122B·10B활성 | 하이브리드 MoE로 122B급 품질을 74GB대에서 구현하는 최상위 범용 LLM | 262144 | Q4_K_M | ~74 GB | ~12–18 t/s |
| [nvidia/nemotron-3-nano-30b-a3b](https://huggingface.co/nvidia/NVIDIA-Nemotron-3-Nano-30B-A3B-BF16) | NVIDIA | 30B·3B활성 | 에이전트 다중 작업에 최적화된 초고속 경량 MoE. 동급 대비 처리량 3배 이상 | 1048576 | Q4_K_M | ~18 GB | ~30–50 t/s |
| [nvidia/nemotron-3-super-120b-a12b](https://huggingface.co/nvidia/NVIDIA-Nemotron-3-Super-120B-A12B-BF16) | NVIDIA | 120B·12B활성 | Mamba 하이브리드로 1M 컨텍스트를 목표로 한 에이전트 추론 중심 MoE | 262144 ⚠️ | Q4_K_M | ~73 GB | ~15–22 t/s |
| [openai/gpt-oss-120b](https://openai.com/ko-KR/index/introducing-gpt-oss/) | OpenAI | 120B | 핵심 추론 벤치마크에서 OpenAI o4-mini와 거의 동등한 결과를 달성 | 8192 | Q4_K_M | ~73 GB | ~5–7 t/s |
| [openai/gpt-oss-20b](https://openai.com/ko-KR/index/introducing-gpt-oss/) | OpenAI | 20B | 일반 벤치마크에서 OpenAI o3-mini와 비슷한 결과를 달성, 빠른 반복 작업에 적합 | 32768 | Q5_K_M | ~15 GB | ~22–28 t/s |

> ⚠️ `nemotron-3-super`의 1M 컨텍스트는 llama.cpp GGUF 기준 현재 262K까지 안정 동작 확인.

#### 코딩 특화 모델

| 모델명 (lms ls 기준) | 제작사 | 크기 | 설명 | 컨텍스트 | 권장 양자화 | VRAM | 초당 토큰 |
|---|---|---|---|---|---|---|---|
| [qwen/qwen3-coder-next](https://huggingface.co/Qwen/Qwen3-Coder-Next) | Qwen | 80B·3B활성 | 에이전트 코딩 특화 MoE. 3B 활성 파라미터로 경량 동작하면서 SWE-bench 상위권 | 262144 | Q4_K_M | ~48 GB | ~35–50 t/s |

### 임베딩 모델

| 모델명 (`lms ls` 기준) | 제작사 | 크기 | 설명 |
|------------------------|--------|------|------|
| [`text-embedding-bge-m3`](https://huggingface.co/BAAI/bge-m3) | BAAI | 567M | 검색/유사도/벡터DB 구축용 임베딩 모델 |  |
| [`text-embedding-nomic-embed-text-v1.5`](https://huggingface.co/nomic-ai/nomic-embed-text-v1.5) | Nomic | - | 경량 임베딩 모델(문서/질문 임베딩) |  |
| [`text-embedding-qwen3-embedding-8b`](https://huggingface.co/Qwen/Qwen3-Embedding-8B) | Qwen | 8B | 고성능 임베딩 모델(대규모 문서 임베딩/검색) |  |

### 비전-언어(VL) 모델

| 모델명 (lms ls 기준) | 제작사 | 크기 | 설명 | 컨텍스트 | 권장 양자화 | VRAM | 초당 토큰 |
|---|---|---|---|---|---|---|---|
| [qwen/qwen3-vl-30b](https://huggingface.co/Qwen/Qwen3-VL-30B-A3B-Instruct) | Qwen | 30B·3B활성 | 이미지+텍스트 입력을 함께 다루는 비전-언어(VL) 모델 | 65536 | Q4_K_M | ~18 GB | ~20–30 t/s |

> 📝 **양자화 및 속도 참고**
> - VRAM·초당 토큰은 LM Studio에서 해당 양자화 포맷 선택 시 기준, GB10 NVLink-C2C 대역폭(~450 GB/s) 기준 추정값입니다.
> - **Q5_K_M**: 20–50B dense 모델의 품질·속도 최적 지점
> - **Q4_K_M**: 70B 이상 또는 MoE 전체 가중치가 큰 모델에 권장 (크기 우선)
> - MoE 모델은 전체 가중치를 메모리에 올리되 토큰당 활성 파라미터만 연산하므로 dense 대비 빠름
> - 배치 크기 1 기준 추정값이며, 동시 요청·컨텍스트 길이에 따라 달라집니다.
> 모델 목록 확인: `lms ls`

### LLM 메모리(통합 메모리) 운영 권장 상한

이 기기의 **LLM 메모리 상한은 LM Studio 사양이 아니라 기기(통합 메모리 128GB) 제약**입니다. 운영 시에는 안정성을 위해 아래처럼 잡는 것을 권장합니다.

- **권장**: 총 128GB 중 **최소 16–24GB는 OS/캐시/안전 마진(미리 비워둘 여유 메모리)로 남겨두고**, 나머지 **약 104–112GB 이내**를 LLM 프로세스가 점유 가능한 최대 범위(권장)로 간주
- **주의**: **컨텍스트 길이를 크게 설정할수록**, 그리고(Open WebUI 등) **동시 실행 서비스/프로세스가 많아질수록** 메모리 사용이 늘어 **미리 비워둘 여유 메모리(안전 마진)를 더 크게 남겨야** 하며, 그만큼 **LLM에 할당 가능한 상한(104–112GB)은 더 낮춰** 잡는 것이 안전합니다.

---

## 4. Python/uv 운영 정책(프로젝트별 가상환경 직접 구성)

이 장비에서는 **프로젝트 폴더마다 Python 버전을 고정**하고, 해당 폴더에서 **가상환경을 직접 생성**한 뒤 라이브러리를 구성합니다.

### uv 버전

- `uv 0.9.7`

### 프로젝트별 Python 버전 고정 및 가상환경 생성 예시

```bash
# 프로젝트 루트에서 Python 버전 고정(예시)
echo "3.11.14" > .python-version

# 프로젝트별 가상환경 생성(예: .venv)
uv venv --python 3.11.14

# 이후 설치는 프로젝트 규칙에 맞게 수행
# - requirements.txt 기반이면:
#   uv pip install -r requirements.txt

```

