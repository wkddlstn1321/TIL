# MySql

[내장함수](#내장함수)

## 내장함수

reference 사이트를 보면 다 나와있지만 자주 쓸만한것들은 따로 빼서 정리해놓으면 나중에 참고하기 편하겠다.

https://dev.mysql.com/doc/refman/8.0/en/built-in-function-reference.html


**집계 함수**

달리 명시하지 않는 한 집계 함수는 NULL값을 무시

* COUNT() : 행의 수 반환
* SUM() : 값의 합을 반환
* AVG() : 값의 평균을 반환
* MAX() : 최대값 반환
* MIN() : 최솟값 반환

**문자열 함수**

* LIKE : 단순 패턴 매칭
* NOT LIKE : 단순 패턴 매칭 부정
	
	와일드 카드 '%' 사용가능 0 ~ 모든 수의 문자와 일치

	`SELECT 'hello' LIKE 'h%e%ll%'; -> 1`

* CONCAT(): 문자열 연결

* LENGTH(): 문자열 길이 반환

**NULL 함수**

* IFNULL(expr1, expr2) : expr1이 NULL 이면 expr2를 반환 NULL이 아니면 expr1반환

* COALESCE(expr1, expr2, ...) : 처음으로 NULL이 아닌 값을 반환
  
* IS NULL: 특정 값이 NULL 인지 확인

* ISNULL() : 인수가 NULL 인지 확인

* IS NOT NULL: 특정 값이 NULL이 아닌지 확인

* NULLIF(expr1, expr2): expr1과 expr2가 같으면 NULL을 반환하고, 그렇지 않으면 expr1을 반환

**DATE 함수**

* NOW(): 현재 날짜와 시간을 반환

* CURDATE(): 현재 날짜를 반환

* CURTIME(): 현재 시간을 반환

* DATE(): 날짜 값에서 날짜 부분만 추출

* TIME(): 날짜 값에서 시간 부분만 추출

**숫자 함수**

* ABS(): 절대값을 반환

* ROUND(): 반올림된 값을 반환

* CEIL(): 주어진 숫자 이상의 가장 작은 정수를 반환

* FLOOR(): 주어진 숫자 이하의 가장 큰 정수를 반환