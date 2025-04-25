# pg_porting_assistant
Oracle -> PostgreSQL 쿼리 포팅 보조 유틸리티

## 사전 환경구성
#### 폴더 구성
```bash
- pg_porting_assistant
- fastagent.config.yaml
- fastagent.secrets.yaml # 필수사항 아님
- ASIS
  └── team
      └── user1.sql
  └── team2
      └── user2.sql
```

## 사용법
```bash
./pg_porting_assistant
```

## 설정
#### fastagent.config.yaml
기본적인 설정을 진행할 수 있는 yaml포멧 설정파일로, 실행프로그램과 동일한 디렉토리에 위치해야합니다.  
만약 해당 설정파일이 누락되었을 경우 실행파일 실행 시 예시파일이 자동생성됩니다.

[샘플파일](../config_sample/fastagent.config.yaml)
```yaml
default_model: openai.gpt-4o-mini             ## 기본값은 openai의 모델이며, openai의 API키 설정이 필요합니다.

generic:
  base_url: "http://localhost:11434/v1"

logger:
  level: info                                 ## info / debug
  type: console                               ## file / console
  path: ./logs/pg_porting_assistant.log       ## type이 file인 경우 로그 위치
  progress_display: true
  show_chat: true
  show_tools: true
  truncate_tools: true

mcp:
  servers:
    fetch:
      command: "uvx"
      args: ["mcp-server-fetch"]
    filesystem:
      command: "npx"
      args: ["-y", "@modelcontextprotocol/server-filesystem", "."]

sqlporter:
  models:                                      ## 컨버터별 에이전트 모델 설정
    converter_1: "generic.gemma3:4b"
    converter_2: "generic.llama3.2:1b"
    converter_3: "openai.gpt-4o-mini"

  instructions:                                ## 컨버터별 변환규칙 추가
    converter_1: ""
    converter_2: |
      Convert the Oracle SQL to PostgreSQL using Llama3-specific syntax rules.
      Respond ONLY with JSON as described in the default instruction.
    converter_3: ""

  settings:
    max_refinements: 3                         ## 최대 쿼리 정제 횟수
    min_rating: "EXCELLENT"                    ## 쿼리품질 [ EXCELLENT / GOOD ]
    retry_limit: 3                             ## 최대 재시도 횟수
    comment_prefix: "--"
```

#### fastagent.secrets.yaml
API키 등 민감정보를 설정하는 설정파일입니다.  
600권한을 권장합니다.

[샘플파일](../config_sample/fastagent.secrets.yaml)
```yaml
openai:
    api_key: sk-abcdefg ## opanai API Key 입력
mcp:
    servers:
        brave:
            env:
                BRAVE_API_KEY: <your_api_key_here>
```

#### ASIS 디렉토리
ASIS 디렉토리는 하위구조를 가질 수 있는 디렉토리로, 원본 Oracle sql파일이 위치하는 장소입니다.
pg_porting_assistant 실행파일과 동일한 위치에 존재해야하며, 동작 시 하위 디렉토리의 모든 파일들을 변환하려고 시도하게 됩니다.

#### TOBE 디렉토리
TOBE 디렉토리는 포팅된 sql파일들이 위치하는 장소로, ASIS의 하위구조를 모방하여 생성됩니다.
pg_porting_assistant 실행파일과 동일한 위치에 존재해야하며, 없는경우 생성 후 하위 디렉토리에 변환된 sql파일을 적재시키게 됩니다.

## 주의사항
1. 모든 결과물은 사용자의 추가 검증이 필요합니다.
2. RockyLinux 8 환경에서만 테스트가 진행되었습니다.
