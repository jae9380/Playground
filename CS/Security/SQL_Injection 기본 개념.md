# SQL-Injection

![](https://i.postimg.cc/tC3Wv3rt/image.png)    

`SQL-Injection`은 SQL문법을 이용하여 임의의 쿼리문을 주입하여 데이터베이스가 비정상적인 동작을 하도록 조작을 하여 공격을 하는 형태의 행위이다.

해당 방식의 공격은 OWASP TOP 10에서 오랜 기간 동안 1위 자리를 굳건히 지켰던 방법이다.

`SQL-Injection`에서는 SQL언어를 이용하여 공격하는 형태를 이야기 한다. 여기서 `SQL(Structured Query Language)`은 관계형 데이터베이스 관리 시스템이 데이터를 관리하기 위하여 개발된 언어이다.  

해당 내용은 WebGoat로 실습하면서 글을 작성할 것이다.

![](https://i.postimg.cc/MGBNMrZJ/SQL-Injection.png)

WebGoat에서 SQL-Injection 섹션에서의 목표

- SQL이 어떤 방식으로 동작을 하는지 그리고 어떻게 사용되는지에 대하여 기본적인 이해
- SQL-Injection이 무엇인지, 어떻게 동작을 하는지에 대한 기본적인 이해
- 다음에 대한 지식을 시연한다
  - DML, DDL 및 DCL
  - 문자열 SQL Injection
  - 숫자 SQL Injection
  - SQL Injection이 CIA 삼원칙을 어떻게 위반하는지

## What is SQL??

SQL배경 지식이 있다고 가정하여 쿼리에 대한 직접적인 사용 방법에 대하여는 이야기 하지 않고, 간단하게 알아보고 넘어갈 것이다.

SQL 쿼리는 그케 DML, DDL, DCL 이렇게 세 가지로 나뉜다.  
 처음에 확인할 것은 `DML(Data Manipulation Language)`은 데이터 조작어라고 칭하며 데이터 조작과 관련된 기능을 하는 문법이다. 어떠한 데이터를 찾기위해 조회하는 Select와 데이터를 추가하는 Insert, 수정하는 Update 마지막으로 삭제하는 Delete쿼리 등이 있다.

`DDL(Data Definition Language)`은 데이터 정의어라고 부른다. 이는 데이터 구조와 관련된 문법이다.  
 데이터 구조(Database, table 등)를 생성하는 Create, 구조를 수정하는 Alter, 구조를 삭제하는 Drop 등이 있다. 특이하게도 Truncate 쿼리도 DDL에 포함되어 있는데 해당 쿼리는 DML인 Delte와 유사하게 테이블의 데이터를 삭제하지만 특정 데이터만 삭제할 수는 없어기 때문에 전체 행을 삭제하여, Delete는 데이터 삭제 후ㅠ 테이블이 점뮤하고 있는 공간을 그대로 방치하는 방면, Truncate는 Delete와 달리 해당 공간 모두 제거한다는 특징이 있다.

`DCL(Data Control Language)`은 데이터 제어와 관련된 문법이다. 사용자에게 권한을 부여하는 Grant와 권한을 제거하는 Revoke, 작업결과를 저장하는 쿼리, 작업한 결과를 원복하는 쿼리 등이 있다.

![](https://i.postimg.cc/13TZjKwz/SQL-Injection-9.png)

해당 문제는 임직원 BoB Franco의 부서 정보를 조회하면 된다. 이때 사용자는 이미 모든 데이터에 대한 접근할 수 있는 관리자 권한을 얻은 상태이다.

```sql
select department from Employees where first_name='Bob' and last_name='Franco'
```

# WebGoat SQL-Injection

![](https://i.postimg.cc/Qx2wJ7S1/SQL-Injection.png)   
`SQL Injection`공격은 개발자가 설계한 SQL쿼리문에 공격자가 악의적인 공격 구문을 삽입하여 공격자의 의도대로 실행되도록 하는 공격을 이야기 한다.   

이를 통하여 공격자는 데이터베이스에 저장된 주요한 데이터를 외부로 유출하여 데이터 기밀성을 해칠 수 있으며, 임의로 수정, 삭제, 삽입을 통하여 무결성을 해칠 수 있다. 또한 무결성이 침해되면 자연스럽게 정합성도 떨어지기 때문에 웹 서비스의 장애를 야기할 수 있어 가용성까지 추가적으로 해치게 된다.    

이렇게 정보보안의 3대 요소인 **기밀성** **무결성** **가용성** 모두를 해칠 수 있는 치명적인 무기가 된다.

해당 공격은 개발자가 설계한 부분에서 어떠한 취약점에서 공격이 들어오냐면 **사용자에게 받은 데이터를 그대로 쿼리문 작성에 사용**하는 빈틈으로 공격을 한다.   

## String SQL Injection

왜냐하면 개발자가 설계한 쿼리문의 일부에 사용자의 입력값을 이어 붙여 실행하도록 구현을 한 경우  DBMS에서는 어디부터 어디까지가 입력값인지 정확한 범위를 모르기 때문에 공격이 가능해진다.   

```sql
Select * from user where id='[입력값]' and pw='[입력값]'
```
위 쿼리를 사용하여 로그인을 할 경우 각 값에 입력값으로 쿼리를 보낸다.   

```sql
Where id='aa' and pw=aa' or '1'='1
```
이렇게 쿼리를 보내게 된다면 어떻게 될까?   

```sql
Select * from user where id='aa' and pw='aa' or '1'='1'
```
싱글쿼터가 문자열의 범위를 강제로 종료시키면서 이어지는 입력값은 문자열의 범위가 종료가 되었기 때문에 쿼리의 한 부분으로 동작을 하게된다.   

일단 id와 pw의 값이 aa인 데이터가 없다고 가정을 한다.   
id와 pw의 입력값은 일단 aa로 동작을 하게된다. 다음으로 or구문이 해석이 될 것이다. `'1'='1'`은 참이되는 조건이기 때문에 테이블에서 존재하는 모든 행을 조회하는데 성공을 하게된다.   

이렇게 공격자는 id와 pw를 모르는 상황에서 임의의 계정으로 로그인이 가능해진다.   


WebGoat/SQL Injection 에서 9번 항목을 보자
![](https://i.postimg.cc/y8SPnrW8/SQL-Injection-2.png)

문제 문구를 확인을 하면 개발자가 쿼리를 Concatenating String(문자열을 이어붙이는)형태로 실행하여 문자열을 이용한 공격이 가능하다고 말을 해준다.   

일단 해당 문제의 Select Box에 어떤 값이 있는지 확인을 하자   
`Smith`, `'Smith`, `''`, `Smith'`, `'Smith'`   
`and`, `or`, `and not`   
`1=1`, `1=2`, `1'='2`, `'1'='1` ,...   
이렇게 선택지가 있다.   


앞에서 간단히 확인한 내용을 바탕으로 쉽게 해결이 가능하다. 싱글쿼터를 사용하여 문자열의 범위를 강제로 종료시키고 or구문을 추가하면 앞이 false이지만 뒤에 따라오는 조건인 true가 되면 전체가 참이기 때문에 뒤에는 참이되는 구문을 추가하면 쉽게 해결이 가능하다.
```sql
SELECT * FROM user_data WHERE first_name = 'John' and last_name = '' or '1' = '1'	
```

## Numeric SQL Injection

![](https://i.postimg.cc/fRHT8Zqx/SQL-Injection-3.png)   
10번 항목의 문제이다. 해당 문제에서는 `Login_Count`와 `User_Id` 중 한 곳에만 공격이 가능하다고 알려준다.   

문제에서 쿼리문은 문자열을 모두 이어붙이는 형태로 구성되어 있다. 이번에는 쿠ㅓ리에 싱글쿼터와 더블쿼터와 같은 문자열 범위를 설정하는 특수문자의 사용은 없다. 현재 쿼리문은 숫자 형태의 값을 비교한다고 판단할 수 있으며 숫자의 범위는 문자열과 다르게 공백을 기준으로 한다는 부분이다.   

일단 어느 부분에서 공격이 가능한지 확인하기 위하여 공백과 특수문자를 이용하여 제출을 해보자.

![](https://i.postimg.cc/yx9yTLVZ/SQL-Injection-4.png)   

카운트 부분에서 숫자의 형태인지 검증을 하고있다는 것을 확인 가능하다. 그렇기 때문에 ID를 타겟을 잡고 공격을 하면 되겠다.   

해당 문제에서도 or구문을 추가하고 뒤에는 참이되는 조건을 추가하면 된다.
```sql
SELECT * From user_data WHERE Login_Count = 123 and userid= 123 or 1=1
```

---

## How do you attack confidentiality, integrity, and availability in the way of SQL Injection?

이제는 SQL Injection 기본적인 공격 방법으로 어떻게 기밀성과 무결성, 가용성을 공격하는지 알아보자   

11번 항목을 보자   
![](https://i.postimg.cc/QtvRtY49/SQL-Injection-5.png)   
이번에는 "나는 기업에 근무하는 'John Smith'이며 'TAN'이라는 내부 시스템을 통하여 조직 및 월급 등 개인적인 정보를 확인하고 있다. 그런데 어느 날 회사의 전 직원의 월급을 확인하고 싶어졌다."라고 명확한 상황을 제시해준다. 

그리고 쿼리문을 확인하면 '+'로 문자열을 이어붙인 형태의 쿼리문이다. 이런 형태의 쿼리문은 DBMS에서는 사용자의 입력값과 개발자가 의도한 쿼리를 명확한 구분이 어렵다고 말을 했었다.

이번에는 `Employee Name`을 타겟으로 공격을 시도하겠다.   

타겟의 입력값을 `asd' or 1=1--`으로 작성하고 다른 입력값에는 대충 아무런 값이나 넣어서 제출을 하면 쉽게 공격이 가능하다. 

이러한 공격이 가능한 이유는 싱글쿼터로 문자열의 범위를 종료시켜주고 or구문으로 뒤에는 참이되는 조건을 작성하고 뒤에 이어서 오는 쿼리 부분은 주석을 통하여 `Authentication TAN`검증을 무효화를 했기 때문에 간단하게 공격이 가능해져서 우리 'John Smith'씨는 쉽게 회사 직원들의 개인정보를 확인하여 기밀성을 해쳤다.   

다음으로 무결성 측면에서 공격하는 방법을 확인하기 위해 12번 항목으로 넘어가자  

![](https://i.postimg.cc/HxJhqG0c/SQL-Injection-6.png)   
이번에는 'John Smith'씨가 자신의 월급이 회사 내 가장 많은 월급을 받는 사람이 되고싶어 한다고 말을 한다. 어서 도와주자   

여기서 `SQL query chaining`은 무엇이냐? SQL쿼리를 연쇄적으로 실행한다는 의미이다. 쿼리를 마친다는 표현인 세미콜론을 붙여서 쿼리를 종료하고 뒤에 또 다른 쿼리를 이어서 작성을 해주면 두개의 쿼리가 연속적으로 실행이 된다. 이것이 `SQL query chaining`이다.   

앞에서 사내 직원들의 개인정보를 확인하는 방법에서 세미콜론을 이용하여 쿼리문 하나를 종료시키고 뒤에는 자신의 월급을 수정하는 쿼리문을 작성하고 그 뒤에는 주석을 작성하면 된다.

![](https://i.postimg.cc/52hgXH80/SQL-Injection-7.png)   
이제 우리의 'John Smith'씨는 한 달 몸값이 100만 달러가 넘는 엄청난 회사원이 되었다.    

마지막으로 가용성 부분을 해치는 방법을 확인하겠다.   

가용성 부분은 여러가지 방법이 있다. 예를 들어서 `SQL query chaining`방법을 이용하여 어떠한 계정의 아이디나 비밀번호를 변경하여 서비스 이용을 막거나, 임의로 권한을 변경시켜 쿼리가 정상적인 동작을 못 하게 만들거나 등 여러가지 방법이 존재한다.   

13번 예제에서는 공격자의 행동이 기록되고 있는 `access_log`의 테이블을 삭제를 통하여 정상적으로 해당 테이블을 이용하지 못하게 만들어 데이터 가용성을 해치는 것이 이번 항목에서의 목표이다.   

![](https://i.postimg.cc/BZRBYLwC/SQL-Injection-8.png)   

이번에도 같이 `SQL query chaining`을 이용하여 `'; drop table access_log--`을 입력하여 간단하고 손 쉽게 해당 테이블을 제거하여 우리의 'John Smith'의 완전범죄를 마무리 지었다.   

이렇게 WebGoat실습으로 간단한 예제의 SQL_Injection의 공격방법을 간단하게 확인을 했다.   

사실 이러한 공격 방법은 실직적으로 영향을 주기에는 어렵다. 공격할 댜상의 환경과 상황 또는 해커의 목표에 따라 더욱 다양한 공격 기법이 사용되어야 하기 때문에 다음에는 좀 더 심화 개념을 알아보고 다양한 기법에 대해서 확인을 하겠다.