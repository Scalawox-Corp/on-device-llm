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
| VS Code | 2.3.41 (arm64) |
| Cline (VS Code Extension) | saoudrizwan.claude-dev 3.55.0 |
| LM Studio | 0.3.31+7 |
| Open WebUI | 0.6.33 |
| uv | 0.9.7 |

> 📝 **Cline 버전 관련**: 동일 확장이 복수 버전으로 설치돼 있을 수 있으나, 설치 목록 기준 최신(3.55.0)을 표기합니다.

---

## 3. 서비스 엔드포인트(접속 주소/포트)

> 📝 **IP 표기 규칙**
> - `<DEVICE_LAN_IP>`: 출고/현장 네트워크에서 장비에 할당된 **LAN IP** (환경마다 달라짐)
> - `127.0.0.1`: **장비 자기 자신**(로컬호스트)

| 서비스 | 바인딩 | 접속 URL |
|---|---|---|
| Open WebUI | `0.0.0.0:8080` | `http://<DEVICE_LAN_IP>:8080` |
| LM Studio (OpenAI 호환 API) | `127.0.0.1:1234` | `http://127.0.0.1:1234/v1` |

> 📝 **참고**
> - LM Studio API는 현재 `127.0.0.1`에 바인딩되어 있어 **동일 장비에서만 직접 접근** 가능합니다.
> - 하지만 Open WebUI처럼 **동일 장비에서 LM Studio API를 호출**하는 애플리케이션은, 애플리케이션 자체를 `0.0.0.0`로 바인딩해 **외부(로컬망) 공개**가 가능합니다.

---

## 현재 설치된 모델 목록

총 11개 모델, 약 198GB 디스크 사용

### 권장 LLM 모델

ASUS Ascent GX10 AI 슈퍼컴퓨터를 위한 LLM 모델
(lmstudio app 내에서 검색 후 결과 중 보라색 아이콘에 해당하는 것으로 설치 권장)

| 모델명 (`lms ls` 명령어 기준) | 제작사 | 크기 | 설명 | 컨텍스트 길이 |
|------------------------|--------|------|------|---------------|
| [`exaone-4.0-32b`](https://huggingface.co/LGAI-EXAONE/EXAONE-4.0-32B) | LGAI | 32B | 한국어 중심 작업에 활용하기 좋은 범용 텍스트 LLM | 32768 |
| [`google/gemma-3-27b`](https://huggingface.co/google/gemma-3-27b-it) | Google | 27B | 지시 따르기/대화형 작업에 사용하는 범용 텍스트 LLM | 32768 |
| [`openai/gpt-oss-120b`](https://openai.com/ko-KR/index/introducing-gpt-oss/) | OpenAI | 120B | 핵심 추론 벤치마크에서 OpenAI o4-mini와 거의 동등한 결과를 달성 | 8192 |
| [`openai/gpt-oss-20b`](https://openai.com/ko-KR/index/introducing-gpt-oss/) | OpenAI | 20B | 일반 벤치마크에서 OpenAI o3‑mini와 비슷한 결과를 달성했으며 빠른 반복 작업에 적합 | 32768 |
| [`qwen/qwen3-30b-a3b-2507`](https://huggingface.co/Qwen/Qwen3-30B-A3B-Instruct-2507) | Qwen | 30B-A3B | 생성 품질과 효율을 균형 있게 쓰기 위한 MoE 계열 텍스트 LLM | 32768 |
| [`qwen/qwen3-8b`](https://huggingface.co/Qwen/Qwen3-8B) | Qwen | 8B | 빠른 응답과 반복 작업에 적합한 경량 텍스트 LLM | 32768 |
| [`qwen/qwen3-vl-30b`](https://huggingface.co/Qwen/Qwen3-VL-30B-A3B-Instruct) | Qwen | 30B-A3B | 이미지+텍스트 입력을 함께 다루는 비전-언어(VL) 모델 | 65536 |
| [`qwen3-32b`](https://huggingface.co/Qwen/Qwen3-32B) | Qwen | 32B | 고정밀 텍스트 생성/분석을 위한 32B급 텍스트 LLM | 32768 |

### 임베딩 모델

| 모델명 (`lms ls` 기준) | 제작사 | 크기 | 설명 |
|------------------------|--------|------|------|
| [`text-embedding-bge-m3`](https://huggingface.co/BAAI/bge-m3) | BAAI | 567M | 검색/유사도/벡터DB 구축용 임베딩 모델 |  |
| [`text-embedding-nomic-embed-text-v1.5`](https://huggingface.co/nomic-ai/nomic-embed-text-v1.5) | Nomic | - | 경량 임베딩 모델(문서/질문 임베딩) |  |
| [`text-embedding-qwen3-embedding-8b`](https://huggingface.co/Qwen/Qwen3-Embedding-8B) | Qwen | 8B | 고성능 임베딩 모델(대규모 문서 임베딩/검색) |  |

> 📝 **참고**: 위 컨텍스트 길이는 이 기기에서 LM Studio에 설정된 값입니다.  
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

