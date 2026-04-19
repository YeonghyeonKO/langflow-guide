# Langflow 사용 가이드

**AI Agent Builder로 시작하는 LLM 애플리케이션 개발**

최종 업데이트: 2026년 4월
문의: **AI/Data Platform 고영현**

> **폐쇄망 환경 가이드:** 본 가이드는 **폐쇄망(Air-gapped) 환경**에서 사용 가능한 컴포넌트를 중심으로 작성되었습니다. 외부 클라우드 API(OpenAI, Anthropic, Google, Amazon Bedrock 등)는 사용할 수 없으며, 사내 Private Cloud에서 호스팅 중인 **vLLM 기반 API**를 **OpenAI Compatible** 컴포넌트로 연동하여 사용합니다.

---

## 1. LLM & AI Agent 기본 이론

### LLM (Large Language Model)이란?

LLM은 **대규모 언어 모델(Large Language Model)**의 약자로, 방대한 텍스트 데이터를 학습하여 자연어를 이해하고 생성할 수 있는 인공지능 모델입니다. Transformer 아키텍처를 기반으로 하며, 대표적인 예로 OpenAI의 GPT 시리즈, Anthropic의 Claude, Google의 Gemini, Meta의 LLaMA 등이 있습니다.

**LLM의 핵심 특징:**

- **텍스트 생성:** 주어진 프롬프트에 대해 자연스러운 텍스트를 생성합니다.
- **문맥 이해:** 긴 문맥을 파악하고 적절한 응답을 생성합니다.
- **다양한 작업:** 번역, 요약, 코드 생성, 질의응답 등 다양한 NLP 작업을 수행합니다.
- **제한점:** 학습 데이터 기반으로만 응답하며, 실시간 정보 접근이나 외부 시스템과의 상호작용은 자체적으로 불가능합니다.

### AI Agent란?

AI Agent는 **LLM + 도구(Tools) + 자율적 추론/행동 루프**를 결합한 시스템입니다. 단순히 텍스트를 생성하는 것을 넘어서, 스스로 판단하고 외부 도구를 호출하여 실제 작업을 수행할 수 있습니다.

**AI Agent 아키텍처:**

```
사용자 입력 → LLM (추론) → Tool 호출 → 결과 분석 → 최종 응답
                  ↑                          |
                  └──── 반복 (루프) ──────────┘
```

Agent는 필요시 Tool 호출과 결과 분석을 반복(루프)합니다.

### LLM vs AI Agent 비교

| 구분 | LLM (단독) | AI Agent |
|------|-----------|----------|
| **동작 방식** | 입력 → 텍스트 출력 | 입력 → 추론 → 도구 호출 → 반복 → 출력 |
| **외부 도구** | 사용 불가 | API 호출, 웹 검색, DB 조회, 코드 실행 등 |
| **자율성** | 단일 응답 생성 | 목표 달성까지 자율적으로 여러 단계 실행 |
| **실시간 정보** | 학습 데이터 한정 | 도구를 통해 최신 정보 접근 가능 |
| **활용 예** | 챗봇, 글 작성, 번역 | 자동화 워크플로, 리서치, 데이터 분석, 코드 자동 수정 |

---

## 2. 왜 Langflow인가

Langflow는 **비주얼 드래그 앤 드롭** 방식으로 LLM 기반 애플리케이션과 AI Agent를 구축할 수 있는 오픈소스 플랫폼입니다. 코드를 최소화하면서도 강력한 AI 파이프라인을 설계할 수 있어, 프로토타이핑부터 프로덕션까지 빠르게 진행할 수 있습니다.

### Langflow의 강점

- **비주얼 빌더:** 드래그 앤 드롭으로 컴포넌트를 연결하여 복잡한 AI 파이프라인을 시각적으로 구성합니다. 코드 한 줄 없이도 작동하는 Flow를 만들 수 있습니다.
- **오픈소스:** Apache 2.0 라이선스로 공개되어 있어 자유롭게 사용, 수정, 배포할 수 있습니다. 커뮤니티 기반의 활발한 개발이 이루어지고 있습니다.
- **Python 확장성:** Custom Component를 Python으로 작성하여 무한히 확장할 수 있습니다. 기존 Python 라이브러리와의 통합이 자유롭습니다.
- **내장 Agent & API:** Agent 컴포넌트와 Tool Mode를 내장하고 있으며, OpenAI 호환 API를 자동 제공하여 외부 시스템과의 연동이 쉽습니다.

### 다른 도구와의 비교

| 기능 | Langflow | Flowise | Dify | n8n |
|------|----------|---------|------|-----|
| **비주얼 빌더** | O | O | O | O |
| **오픈소스** | O (Apache 2.0) | O (MIT) | O (제한적) | O (제한적) |
| **Custom Component (Python)** | O (강력) | 제한적 | 제한적 | JS 기반 |
| **Agent 내장** | O (Tool Mode) | O | O | 제한적 |
| **OpenAI 호환 API** | O (/responses) | 제한적 | O | X |
| **LangChain 통합** | O (네이티브) | O | 부분적 | X |
| **주 사용 목적** | AI Agent / LLM 앱 | 챗봇 | LLM 앱 | 범용 자동화 |

---

## 3. Component 소개

Langflow의 컴포넌트는 Flow를 구성하는 기본 단위입니다. 각 컴포넌트는 특정 기능을 수행하며, 입력(Input)과 출력(Output) 포트를 통해 다른 컴포넌트와 데이터를 주고받습니다.

### 컴포넌트 카테고리

| 카테고리 | 설명 | 대표 컴포넌트 |
|---------|------|-------------|
| **Inputs** | 사용자 입력을 받는 컴포넌트 | Chat Input, Text Input |
| **Outputs** | 결과를 출력하는 컴포넌트 | Chat Output, Text Output |
| **Models** | LLM 모델 연동 | OpenAI Compatible (사내 vLLM 등) |
| **Prompts** | 프롬프트 템플릿 정의 | Prompt Template |
| **Tools** | Agent가 사용할 외부 도구 | Calculator, URL Fetcher, Python REPL, API Request |
| **Agents** | 자율 판단 에이전트 | Agent Component |
| **Data** | 데이터 로딩 및 처리 | File Loader, API Request, URL |
| **Processing** | 텍스트 가공 및 변환 | Text Splitter, Filter Data, Parse Data |
| **Memories** | 대화 기록 저장/관리 | Chat Memory, Message History |
| **Embeddings** | 텍스트를 벡터로 변환 | HuggingFace Embeddings (로컬), OpenAI Compatible Embeddings |
| **Vector Stores** | 벡터 데이터 저장/검색 | Chroma, FAISS, Pinecone |

### 포트(Port) 이해하기

각 컴포넌트의 좌측은 **입력 포트**, 우측은 **출력 포트**입니다. 호환되는 타입의 포트끼리만 연결할 수 있으며, 포트의 색상으로 데이터 타입을 구분합니다.

> **TIP:** 컴포넌트 연결 시 호환되지 않는 포트는 연결이 되지 않으니, 색상이 같은 포트끼리 연결하면 됩니다. 연결 가능한 포트에 마우스를 가져가면 하이라이트가 표시됩니다.

---

## 4. Custom Component 만들기

Langflow에서 제공하는 기본 컴포넌트만으로 부족할 때, Python 코드로 직접 **Custom Component**를 작성할 수 있습니다. `Component` 클래스를 상속받아 입력, 출력, 처리 로직을 정의합니다.

### Custom Component 기본 구조

```python
from langflow.custom import Component
from langflow.io import StrInput, Output
from langflow.schema import Data


class TextProcessor(Component):
    display_name = "Text Processor"
    description = "텍스트를 처리하는 커스텀 컴포넌트"
    icon = "code"

    inputs = [
        StrInput(
            name="input_text",
            display_name="Input Text",
            info="처리할 텍스트를 입력하세요.",
            required=True,
        ),
        StrInput(
            name="prefix",
            display_name="Prefix",
            info="텍스트 앞에 붙일 접두사",
            value="[처리됨]",
        ),
    ]

    outputs = [
        Output(
            display_name="Processed Text",
            name="processed",
            method="process_text",
        ),
    ]

    def process_text(self) -> Data:
        result = f"{self.prefix} {self.input_text}"
        self.status = result  # UI에 상태 표시
        return Data(data={"text": result})
```

### 주요 Input 타입

| Input 클래스 | 용도 | 예시 |
|-------------|------|------|
| `StrInput` | 문자열 입력 | 텍스트, URL, API Key |
| `IntInput` | 정수 입력 | 반복 횟수, 포트 번호 |
| `FloatInput` | 실수 입력 | Temperature, Threshold |
| `BoolInput` | Boolean 입력 | 활성화/비활성화 토글 |
| `DropdownInput` | 선택 목록 | 모델 선택, 옵션 선택 |
| `FileInput` | 파일 업로드 | 문서, 이미지 파일 |
| `MultilineInput` | 여러 줄 텍스트 | 프롬프트, 긴 텍스트 |
| `HandleInput` | 다른 컴포넌트 연결 | LLM, Tool 등 연결 |

### UI에서 Custom Component 추가하기

1. **새 컴포넌트 생성:** 좌측 사이드바 하단의 **+ New Component** 버튼을 클릭하거나, 캔버스에서 빈 공간을 더블클릭하면 코드 에디터가 열립니다.
2. **코드 작성:** 에디터에 Python 코드를 작성합니다. `display_name`, `inputs`, `outputs`, 메서드를 정의합니다.
3. **저장 및 사용:** **Check & Save** 버튼을 누르면 컴포넌트가 검증되고 캔버스에 추가됩니다. 이후 다른 컴포넌트와 연결하여 사용하면 됩니다.

---

## 5. Custom Component로 pip install 하기

Langflow 환경에 필요한 Python 패키지가 설치되어 있지 않을 때, **Custom Component에서 `subprocess`를 사용하여 직접 패키지를 설치**할 수 있습니다.

> **보안 주의:** subprocess로 패키지를 설치하는 것은 보안 위험이 있을 수 있습니다. 신뢰할 수 있는 패키지만 설치하고, 프로덕션 환경에서는 사전에 필요한 패키지를 Docker 이미지에 포함시키는 것을 권장합니다.

### Pip Installer 컴포넌트

```python
import subprocess
import sys
from langflow.custom import Component
from langflow.io import StrInput, Output
from langflow.schema import Data


class PipInstaller(Component):
    display_name = "Pip Installer"
    description = "Python 패키지를 설치하는 컴포넌트"
    icon = "download"

    inputs = [
        StrInput(
            name="package_name",
            display_name="Package Name",
            info="설치할 패키지 이름 (예: requests, pandas)",
            required=True,
        ),
    ]

    outputs = [
        Output(
            display_name="Result",
            name="result",
            method="install_package",
        ),
    ]

    def install_package(self) -> Data:
        try:
            result = subprocess.run(
                [sys.executable, "-m", "pip", "install", self.package_name],
                capture_output=True,
                text=True,
                timeout=120,
            )
            if result.returncode == 0:
                status = "success"
                output = result.stdout
                self.status = f"'{self.package_name}' 설치 완료!"
            else:
                status = "error"
                output = result.stderr
                self.status = f"설치 실패: {result.stderr[:200]}"

            return Data(data={
                "status": status,
                "package": self.package_name,
                "output": output,
            })
        except subprocess.TimeoutExpired:
            self.status = "설치 시간 초과 (120초)"
            return Data(data={
                "status": "timeout",
                "package": self.package_name,
                "output": "설치 시간이 초과되었습니다.",
            })
```

### 사용 방법

1. 위 코드를 Custom Component로 추가합니다.
2. **Package Name** 필드에 설치할 패키지명을 입력합니다. (예: `beautifulsoup4`, `pandas==2.0.0`)
3. 컴포넌트를 실행하면 해당 패키지가 Langflow 환경에 설치됩니다.

> **Docker 환경에서의 주의사항:** Docker 컨테이너 내에서 실행 중인 Langflow의 경우, 컨테이너가 재시작되면 설치된 패키지가 사라집니다. 영구적으로 필요한 패키지는 Dockerfile에서 사전 설치하세요.

```dockerfile
FROM langflowai/langflow:latest

# 추가 패키지 사전 설치
RUN pip install beautifulsoup4 pandas openpyxl
```

---

## 6. 사용 가능한 pip list 확인법

현재 Langflow 환경에 어떤 Python 패키지가 설치되어 있는지 확인하려면, Custom Component를 사용하여 `pip list` 명령을 실행할 수 있습니다.

### Pip List Checker 컴포넌트

```python
import subprocess
import sys
from langflow.custom import Component
from langflow.io import StrInput, Output
from langflow.schema import Data


class PipListChecker(Component):
    display_name = "Pip List Checker"
    description = "설치된 Python 패키지 목록을 확인합니다"
    icon = "list"

    inputs = [
        StrInput(
            name="filter_keyword",
            display_name="Filter Keyword",
            info="특정 키워드로 패키지를 필터링 (빈칸이면 전체 목록)",
            value="",
            advanced=True,
        ),
    ]

    outputs = [
        Output(
            display_name="Package List",
            name="packages",
            method="list_packages",
        ),
    ]

    def list_packages(self) -> Data:
        result = subprocess.run(
            [sys.executable, "-m", "pip", "list", "--format=columns"],
            capture_output=True,
            text=True,
        )
        packages = result.stdout

        if self.filter_keyword:
            lines = packages.split("\n")
            # 헤더(처음 2줄)는 유지하고 나머지 필터링
            header = lines[:2]
            filtered = [
                line for line in lines[2:]
                if self.filter_keyword.lower() in line.lower()
            ]
            packages = "\n".join(header + filtered)
            self.status = f"'{self.filter_keyword}' 포함 패키지: {len(filtered)}개"
        else:
            total = len(result.stdout.strip().split("\n")) - 2
            self.status = f"총 {total}개 패키지 설치됨"

        return Data(data={"packages": packages})
```

### 프로그래밍적 확인 (importlib)

`subprocess` 대신 Python 표준 라이브러리인 `importlib.metadata`를 사용하여 설치된 패키지 정보를 프로그래밍적으로 조회할 수도 있습니다.

```python
from importlib.metadata import distributions

# 설치된 모든 패키지 이름과 버전 출력
for dist in distributions():
    print(f"{dist.metadata['Name']} == {dist.version}")
```

> **TIP:** 특정 패키지가 설치되어 있는지만 확인하고 싶다면 `importlib.metadata.version("패키지명")`을 사용하세요. 설치되지 않은 경우 `PackageNotFoundError`가 발생합니다.

---

## 7. 간단한 Flow 사용법

Flow는 여러 컴포넌트를 연결하여 하나의 AI 파이프라인을 구성한 것입니다. 가장 기본적인 Flow를 만들어 보겠습니다.

### 기본 Flow: 간단한 챗봇

```
Chat Input → Prompt Template → OpenAI Compatible → Chat Output
```

### 단계별 만들기

**1단계. 새 Flow 생성**

Langflow 대시보드에서 **+ New Flow** 버튼을 클릭합니다. "Blank Flow"를 선택하거나, 미리 준비된 템플릿을 사용할 수도 있습니다.

**2단계. 컴포넌트 배치**

좌측 사이드바에서 아래 4개의 컴포넌트를 캔버스로 드래그합니다:

- **Inputs > Chat Input** -- 사용자 메시지 입력
- **Prompts > Prompt Template** -- 시스템 프롬프트 설정
- **Models > OpenAI Compatible** -- 사내 vLLM API 연동
- **Outputs > Chat Output** -- 응답 출력

**3단계. 컴포넌트 연결**

각 컴포넌트의 출력 포트(우측)에서 다음 컴포넌트의 입력 포트(좌측)로 드래그하여 연결합니다:

- Chat Input의 **Message** 출력 → Prompt Template의 **입력 변수**
- Prompt Template의 **Prompt** 출력 → OpenAI Compatible의 **Input**
- OpenAI Compatible의 **Text** 출력 → Chat Output의 **Input**

**4단계. 모델 서버 설정**

OpenAI Compatible 컴포넌트를 클릭하여 설정 패널을 열고, **Base URL** 필드에 사내 vLLM 서버 주소를 입력합니다 (예: `http://your-vllm-server:8000/v1`). 사용할 모델명도 입력합니다.

**5단계. Prompt 작성**

Prompt Template 컴포넌트에서 시스템 프롬프트를 작성합니다. 중괄호 `{}`로 변수를 정의할 수 있습니다:

```
당신은 친절한 한국어 AI 어시스턴트입니다.
사용자의 질문에 명확하고 도움이 되는 답변을 해주세요.

사용자 질문: {user_message}
```

**6단계. 테스트 (Playground)**

우측 하단의 **Playground** 버튼을 클릭하면 채팅 인터페이스가 열립니다. 메시지를 입력하여 Flow가 정상적으로 작동하는지 확인합니다.

> **TIP:** Flow는 상단 메뉴에서 **이름 변경**과 **저장**이 가능합니다. 또한 **Export**를 통해 JSON 파일로 내보내어 팀원과 공유하거나 다른 환경으로 이전할 수 있습니다.

---

## 8. Agent 만들기 (Tool Mode)

Langflow의 **Agent 컴포넌트**는 LLM이 스스로 도구(Tool)를 선택하고 호출하여 작업을 수행하는 자율 에이전트를 구성할 수 있게 해줍니다. **Tool Mode**를 활용하면 기존 컴포넌트를 도구로 변환하여 Agent에 연결할 수 있습니다.

### Agent Flow 구성

```
Chat Input → Agent → Chat Output
               ↑
     ┌─────────┼─────────┐
Calculator   URL Fetcher  Python REPL
  Tool         Tool         Tool
```

### 단계별 구성

**1단계. Agent 컴포넌트 추가**

사이드바에서 **Agents > Agent**를 캔버스에 드래그합니다. Agent 컴포넌트에는 LLM 모델 설정과 시스템 프롬프트를 구성합니다.

**2단계. Tool 컴포넌트 추가**

Agent가 사용할 도구 컴포넌트를 추가합니다. 예를 들어:

- **Calculator** -- 수학 계산
- **URL Fetcher** -- 웹 페이지 내용 가져오기
- **Python REPL** -- Python 코드 실행
- **API Request** -- 외부 API 호출

**3단계. Tool Mode 활성화**

일반 컴포넌트를 Tool로 사용하려면, 해당 컴포넌트의 **헤더 메뉴(...)** 를 클릭한 후 **Tool Mode**를 토글하여 활성화합니다. 활성화하면 컴포넌트에 **Toolset** 출력 포트가 생깁니다.

**4단계. Agent에 Tool 연결**

각 Tool 컴포넌트의 **Toolset** 출력 포트를 Agent 컴포넌트의 **Tools** 입력 포트에 연결합니다. 하나의 Agent에 여러 Tool을 연결할 수 있습니다.

**5단계. Input/Output 연결**

**Chat Input**을 Agent의 입력에, Agent의 출력을 **Chat Output**에 연결하여 전체 Flow를 완성합니다.

> **Tool Mode의 동작 원리:** Tool Mode가 활성화된 컴포넌트는 Agent가 판단에 따라 **선택적으로 호출**할 수 있는 도구가 됩니다. Agent는 사용자 요청을 분석한 후, 필요한 도구를 자동으로 선택하여 실행하고, 그 결과를 바탕으로 최종 응답을 생성합니다.

### Agent 설정 팁

- **시스템 프롬프트:** Agent의 역할과 사용 가능한 도구에 대한 설명을 명확하게 작성하면 더 정확한 결과를 얻을 수 있습니다.
- **모델 선택:** Agent에는 도구 호출(Function Calling)을 지원하는 모델을 사용하세요. (사내 vLLM에서 호스팅 중인 모델 활용)
- **Tool Description:** 각 Tool의 설명을 구체적으로 작성하면 Agent가 도구를 더 적절하게 선택합니다.

---

## 9. API Access (OpenAI 호환)

Langflow는 구축한 Flow를 **OpenAI 호환 API**로 외부에 노출할 수 있습니다. **Share > API Access** 메뉴를 통해 API 엔드포인트를 확인하고, 어떤 OpenAI 호환 클라이언트에서든 해당 Flow를 LLM처럼 호출할 수 있습니다.

### API 접근 설정

1. **API Access 열기:** Flow 편집 화면 상단의 **Share** 버튼 (또는 **API** 버튼)을 클릭한 후 **API Access**를 선택합니다.
2. **엔드포인트 확인:** OpenAI 호환 엔드포인트 URL과 Flow ID, API Key 정보를 확인합니다.

### API 엔드포인트 정보

| 항목 | 값 |
|------|-----|
| **Endpoint** | `POST {LANGFLOW_URL}/api/v1/responses` |
| **인증** | `x-api-key` 헤더 (Bearer 토큰이 아님에 주의) |
| **Model** | Flow ID를 model 이름으로 사용 |

### 사용 예시

#### 1) Python requests로 직접 호출

```python
import requests

LANGFLOW_URL = "http://localhost:7860"
FLOW_ID = "your-flow-id-here"
API_KEY = "your-api-key-here"

response = requests.post(
    f"{LANGFLOW_URL}/api/v1/responses",
    headers={
        "x-api-key": API_KEY,
        "Content-Type": "application/json",
    },
    json={
        "model": FLOW_ID,
        "input": "안녕하세요, 오늘 날씨가 어때요?",
    },
)

result = response.json()
print(result)
```

#### 2) OpenAI SDK 호환 호출

OpenAI Python SDK를 사용하여 Langflow Flow를 호출할 수 있습니다. `base_url`을 Langflow 서버로 지정하고, Flow ID를 model로 사용합니다.

```python
from openai import OpenAI

client = OpenAI(
    base_url="http://localhost:7860/api/v1",
    api_key="your-api-key-here",
    default_headers={"x-api-key": "your-api-key-here"},
)

response = client.responses.create(
    model="your-flow-id-here",
    input="Langflow에 대해 설명해주세요.",
)

print(response.output_text)
```

#### 3) 스트리밍 응답

`stream=True`를 설정하면 실시간으로 응답을 받을 수 있습니다.

```python
from openai import OpenAI

client = OpenAI(
    base_url="http://localhost:7860/api/v1",
    api_key="your-api-key-here",
    default_headers={"x-api-key": "your-api-key-here"},
)

stream = client.responses.create(
    model="your-flow-id-here",
    input="AI Agent의 장점을 알려주세요.",
    stream=True,
)

for event in stream:
    print(event)
```

#### 4) 멀티턴 대화

`previous_response_id`를 사용하면 이전 대화를 이어갈 수 있습니다.

```python
from openai import OpenAI

client = OpenAI(
    base_url="http://localhost:7860/api/v1",
    api_key="your-api-key-here",
    default_headers={"x-api-key": "your-api-key-here"},
)

# 첫 번째 메시지
response1 = client.responses.create(
    model="your-flow-id-here",
    input="안녕하세요! 저는 AI에 관심이 많습니다.",
)
print("응답 1:", response1.output_text)

# 두 번째 메시지 (이전 대화 이어가기)
response2 = client.responses.create(
    model="your-flow-id-here",
    input="방금 제가 관심있다고 한 분야가 뭐였죠?",
    previous_response_id=response1.id,
)
print("응답 2:", response2.output_text)
```

> **TIP:** 이 OpenAI 호환 API를 활용하면, Langflow에서 만든 Flow를 다른 애플리케이션(웹 프론트엔드, 모바일 앱, Slack 봇 등)에서 일반 LLM API처럼 간편하게 호출할 수 있습니다. 기존 OpenAI SDK를 사용하는 코드에서 `base_url`만 변경하면 됩니다.

> **API Key 발급:** Langflow API Key는 프로필 설정 또는 환경변수(`LANGFLOW_API_KEY`)로 설정할 수 있습니다. DataStax Langflow (클라우드 버전)를 사용하는 경우, 대시보드의 Settings에서 API 토큰을 발급받을 수 있습니다.

---

---

## 참고 자료

- [Langflow 공식 문서](https://docs.langflow.org)
- [Langflow GitHub](https://github.com/langflow-ai/langflow)

---

**가이드 관련 문의: AI/Data Platform 고영현**
