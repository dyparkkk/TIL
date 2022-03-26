# DB 정규화

## 목적

- 불필요한 데이터를 제거 → 저장공간 더 잘 사용
- 논리적이고 직관적인 데이터
- 이상현상을 없애고 더 적은 쿼리로 작동가능

![Untitled](https://raw.githubusercontent.com/dyparkkk/TIL/main/DB/img/nor01.png)

삽입이상 : 삽입시 중복된 데이터 저장

삭제이상: 삭제시 삭제되서는 안될 데이터 삭제

수정이상 : 수정시 많은 row를 수정해야함 → 비효율적

## 1차 정규화

- 각 row마다 값이 하나씩만 있어야 함 → 원자값 (atomic value)
- 정규화 방법
    - 여러 값이 있어야 한다면 테이블을 나눔
    - 일대다 관계 만듬

## 2차 정규화

테이블의 모든 컬럼이 기본키에 종속되어야함 → 기본키 일부(특정 칼럼) 에 종속 X (부분 종속)

- 기본키 일부가 수정시 종속된 칼럼도 수정 필요 → 수정이상
- 키본키 일부 생성시 종속된 칼럼도 생성됨 (기본키가 아님 ...) → 생성이상
- 정규화 방법
    - 기본키 일부를 fk 로 놓음 → 다른 테이블 생성
    - 다대일 관계 만듬

정규화 전

![Untitled](https://raw.githubusercontent.com/dyparkkk/TIL/main/DB/img/nor02.png)

정규화 후

![Untitled](https://raw.githubusercontent.com/dyparkkk/TIL/main/DB/img/nor03.png)

## 3차 정규화

- 2차에서 이행적 종속을 없애도록 테이블 분해 ( **특정 칼럼이 다른 칼럼에 종속됨** )
- 이행적 종속 : a → b, b → c 일 때 a → c가 성립( 당연한게 칼럼은 기본키에 무조건 종속됨)
- 갱신 이상 발생을 막음 ex) 데이터베이스를 수정했는데 수강료도 같이 수정해야함

전

![Untitled](https://raw.githubusercontent.com/dyparkkk/TIL/main/DB/img/nor04.png)

후

![Untitled](https://raw.githubusercontent.com/dyparkkk/TIL/main/DB/img/nor05.png)

## BCNF 정규화

- 3차에서 모든 결정자(다른 컬럼 결정)가 후보키가 되도록 분해
- 이상현상
    - 삽입, 삭제 이상 : 교수가 특강을 개설했는데 학생이 없으면 데이터를 삽입 할 수 없음

전

![Untitled](https://raw.githubusercontent.com/dyparkkk/TIL/main/DB/img/nor06.png)

후

![Untitled](https://raw.githubusercontent.com/dyparkkk/TIL/main/DB/img/nor07.png)

---

참고 자료 :

[https://mangkyu.tistory.com/110](https://mangkyu.tistory.com/110)

[https://www.youtube.com/watch?v=pMcv0Zhh3J0](https://www.youtube.com/watch?v=pMcv0Zhh3J0) - 동영상