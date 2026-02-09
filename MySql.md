
**Mysql**

0617
Install => Custom / 
MySQL 가장 마지막 => 오른쪽으로
Applications 가장 마지막 오른쪽으로
Next => Execute => Next => Next
Port: 3306
Next =>
Accounts and rules: MySQL password: ezen1234
=> Next
=> Next
No, I will, permissions after ~_~
=> execute
  
0618
mysql은 실행 중이어야 한다. 
Cmd => mysql -u root -p
안 되면 bin 폴더가 있는 루트 찾아서 위의 명령어 다시 실행하기

사용자계정:root
비밀번호:ezen1234
Create database eduDB;
Create user ‘ezen’@‘%’identified by ‘1234’;
Create user ‘even’@‘localhost’ identified by ‘1234’

Grant all on eduDB.* to ‘ezen’@‘%;
Fulsh privileges;

Grant all on eduDB.* to ‘ezen’@‘localhost;
Fulsh privileges;

권한 조회
Show grants for ‘ezen’@‘localhost

현재 유저 확인
Select user(), current_user();
Quit

——————————
이젠 ezen으로 접속하기

Mysql -u ezen -p
1234
Select user(), current_user();
Use eduDB;
———————————

DBMS: mySQL, Oracle, MS Server, H2,.. => 관계형 데이터베이스
NoSQL => MongoDB, ..

데이터베이스명: eduDB
테이블(여러 개의 컬럼을 갖는 레코드들로 구성) 
DB설계
테이블 구성


1 : N 관계라고 한다.  
회원 —포스트에 글을 쓴다—포스트(게시판)


Id(PK)                              id
Name                              title
Email(unique)                content
…                                     writer (FK) => members테이블의 email을 레퍼런스


DDL (Data Definition Language) => create, drop, alter, —
Create table members(

);


조건을 줄 수 있음
CREATE TABLE IF NOT EXISTS members ( 
	id INT AUTO_INCREMENT PRIMARY KEY, -- 회원번호 name VARCHAR(50) NOT NULL, email VARCHAR(100) UNIQUE NOT NULL, password VARCHAR(100) NOT NULL, role VARCHAR(20) DEFAULT 'USER', -- USER, ADMIN indate DATETIME DEFAULT CURRENT_TIMESTAMP, refreshtoken VARCHAR(255) 
	);


MySql Workbench 
