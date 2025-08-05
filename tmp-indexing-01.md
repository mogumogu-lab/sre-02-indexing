실무에서 많이 비교하는 실험들 추천해드릴게요.

(1) Partial Index
	•	자주 조회되는 값(is_active = true 등)에만 인덱싱
	•	용량↓, 속도↑ (조건이 자주 맞으면 더 효율적)

(2) Expression Index
	•	함수/가공 데이터에 인덱스 (예: LOWER(email))

(3) Multi-column(Composite) Index
	•	2~3개 복합 인덱스에서, 칼럼 순서별로 실행계획 변화 관찰

(4) Covering Index (INCLUDE)
	•	PostgreSQL 11부터 가능,
SELECT에서 조회하는 컬럼이 인덱스에 다 들어있으면 Index Only Scan 발생(실제 테이블 데이터 안 봐도 돼서 속도↑)

(5) Other Types
	•	GIN: 배열, jsonb, full-text에서 실험(특정 칼럼 jsonb/text 넣어서 → GIN vs B-Tree)
	•	BRIN: 범위가 넓고, 거의 append only한 timestamp, bigserial 등에서 효과(큰 테이블일수록 체감 큼)
	•	Hash: 거의 안 씀. 실무에서 잘 안 써요. 정확히 등호만 지원