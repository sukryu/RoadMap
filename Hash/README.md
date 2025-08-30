# 📝 Day 1-2 학습 기록: 해시 함수 완전 마스터

> **학습 기간**: Day 1-2 (4시간)  
> **목표**: DJB2 암기 구현 + 보안 취약점 이해 + 실무 선택 기준 확립  
> **성과**: ✅ 완료 (실무 수준 구현 달성)

## 🧠 **핵심 개념 체화**

### **해시 함수의 본질**

```yaml
정의: 임의 크기 데이터 → 고정 크기 정수 변환
목적: 빠른 검색을 위한 "주소 계산" 함수

핵심 특성 4가지:
✅ 결정적(Deterministic): 같은 입력 = 같은 출력
✅ 균등분포(Uniform): 모든 해시값이 동일한 확률  
✅ 빠른계산(Fast): O(n) 시간복잡도
✅ 눈사태효과(Avalanche): 1비트 변경 → 해시값 크게 변화
```

## 🔍 **3가지 해시 함수 마스터**

### **1. Division Method**

```c
// ❌ 문제점: 아나그램 동일 해시값
unsigned int basic_division_hash(char* key, int table_size) {
    unsigned int hash = 0;
    for (int i = 0; key[i] != '\0'; i++) {
        hash += key[i];  // "abc" = "bca" = "cab"
    }
    return hash % table_size;
}

// ✅ 개선: 위치별 가중치 적용
unsigned int improved_division_hash(char* key, int table_size) {
    unsigned int hash = 0;
    unsigned int multiplier = 1;
    for (int i = 0; key[i] != '\0'; i++) {
        hash += key[i] * multiplier;
        multiplier *= 37;  // 소수 사용
    }
    return hash % table_size;
}
```

### **2. Multiplication Method**

```c
#define GOLDEN_RATIO 0.6180339887  // (√5 - 1) / 2

unsigned int multiplication_hash(char* key, int table_size) {
    unsigned int hash = 0;
    for (int i = 0; key[i] != '\0'; i++) {
        hash = hash * 31 + key[i];  // Horner's method
    }
    double product = hash * GOLDEN_RATIO;
    double fractional = product - (unsigned int)product;
    return (unsigned int)(table_size * fractional);
}

// 장점: table_size 2의 제곱수여도 잘 작동
// 단점: 부동소수점 연산으로 느림
```

### **3. DJB2/DJB2A (실무 표준) ⭐**

```c
// DJB2 (ADD 버전)
unsigned int djb2_hash(char* key) {
    unsigned int hash = 5381;  // 소수 초기값
    int c;
    while ((c = *key++)) {
        hash = ((hash << 5) + hash) + c;  // hash * 33 + c
    }
    return hash;
}

// DJB2A (XOR 버전) - 구현한 버전
unsigned int djb2a_hash(char* key) {
    unsigned int hash = 5381;
    int c;
    while ((c = *key++)) {
        hash = ((hash << 5) + hash) ^ c;  // hash * 33 ^ c
    }
    return hash;
}
```

#### **핵심 암기 포인트**

- **5381**: 소수 초기값 (해시 품질 향상)
- **33**: 홀수 승수 (32 + 1, 비트 시프트 최적화)
- **<< 5**: hash × 32 (빠른 곱셈)
- **XOR vs ADD**: XOR이 더 균등 분포, ADD가 더 빠름

## 🚨 **보안 취약점과 실무 대응**

### **Hash Flooding Attack**

```yaml
문제: 악의적 입력으로 모든 키가 같은 해시값 생성
결과: O(1) 해시테이블 → O(n) 연결리스트로 퇴화
사례: Java 1.7 이전, PHP 해시 DoS 공격

방어법:
✅ 랜덤 시드: 예측 불가능한 해시값
✅ SipHash: 암호학적 안전성 (Redis 방식)  
✅ 입력 검증: 악의적 키 패턴 차단
```

### **실무 선택 기준**

```yaml
보안 중요 (외부 입력):
- SipHash (Redis 방식)
- CityHash + 랜덤시드
- SHA-256 (규제 준수)

성능 중요 (내부 시스템):  
- DJB2/DJB2A (간단함)
- MurmurHash3 (극한 성능)
- FNV-1a (메모리 제약)

Go map의 안전성:
- 런타임마다 랜덤 시드 변경
- 이터레이션 순서 랜덤화
- Hash Flooding 공격 방지
```

## 🧪 **실험 결과**

### **개인 이름 해시 테스트**

```c
// 테스트 문자열: "JinhyeokHong"
String: 'JinhyeokHong'
DJB2  (ADD): 1436037906 (0x559C9F92)
DJB2A (XOR): 1498809622 (0x595D9CE6)

테이블 매핑:
Table 100:   slot 22
Table 1000:  slot 622  
Table 10007: slot 1190 (소수 크기 효과)
```

### **아발란체 효과 분석**

```yaml
테스트: 'g' → 'G' (1비트 차이)
결과: 1/32 비트 변경 (3.1%)
예상: 16/32 비트 변경 (50%)

발견: 문자열 끝 변경에 상대적으로 덜 민감
교훈: 완벽한 해시함수는 없다, "충분히 좋음"이 목표
```

## 💡 **핵심 깨달음**

### **이론적 인사이트**

```yaml
1. "해시함수 = 압축함수"
   - 큰 우주(모든 문자열) → 작은 우주(테이블 인덱스)
   - 비둘기집 원리에 의해 충돌 불가피
   - 좋은 해시함수 = 균등한 압축

2. "보안은 선택이 아닌 필수"
   - 외부 입력 처리 시 Hash Flooding 고려 필수
   - 랜덤 시드 없으면 DoS 공격 취약

3. "성능 vs 보안 트레이드오프"
   - DJB2: 빠름, 예측 가능
   - SipHash: 안전, 상대적 느림
   - 실무에서는 용도에 따라 선택
```

### **실무 연결점**

```yaml
채팅 서버 세션 관리:
- 사용자 ID → 서버 매핑: hash(user_id) % server_count
- 세션 테이블: hash(session_id) → 세션 정보  
- 메시지 중복 제거: hash(message_content + timestamp)

선택 기준:
- 1000명 동시접속 → 테이블 크기 2003 (소수, 2배 여유)
- Load Factor 0.5 목표 (충돌 최소화)
- 외부 API → SipHash, 내부 로직 → DJB2A
```

## ❓ **남은 의문점 → 다음 학습 동기**

```yaml
🤔 해결할 문제들:
1. "JinhyeokHong"과 다른 키가 slot 622에 충돌한다면?
   → 체이닝? 오픈 어드레싱?

2. 해시테이블이 가득 찰 때 어떻게 확장?
   → 동적 리사이징 전략

3. 멀티스레드에서 안전하게 사용하려면?
   → 동시성 제어 방법

4. Redis는 왜 Load Factor 6.5까지 허용?
   → 고급 최적화 기법
```

## 🎯 **다음 학습 계획**

### **Day 3-4: 충돌 해결법**

- 체이닝 vs 오픈 어드레싱 완전 비교
- Robin Hood Hashing 고급 기법
- 메모리 사용량과 성능 트레이드오프

### **예습 과제**

```yaml
생각해볼 시나리오:
1. 카카오톡 사용자 5000만명을 1000대 서버에 분산한다면?
2. 실시간 게임 매치메이킹에서 해시테이블 역할은?
3. API Rate Limiting을 해시테이블로 구현한다면?
```

## 📊 **학습 성과 평가**

### **✅ 달성한 목표**

- [x] DJB2A 화이트보드 구현 능력 (20분 내)
- [x] 해시함수 선택 기준 정립
- [x] 보안 취약점 인식 및 대응법 이해
- [x] 실무 적용 시나리오 구체화

### **🎯 다음 목표**

- [ ] 충돌 해결 방법 2가지 완전 구현
- [ ] 성능 벤치마킹 및 분석
- [ ] 메모리 효율성 최적화 기법
- [ ] 동적 리사이징 알고리즘 이해

-----
