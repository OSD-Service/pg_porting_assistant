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

## 주의사항
1. 모든 결과물은 사용자의 추가 검증이 필요합니다.
2. RockyLinux 8 환경에서만 테스트가 진행되었습니다.
