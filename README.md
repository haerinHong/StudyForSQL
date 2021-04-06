2021-04-06) postgre -> Oracle로 DDL 변환, 패키지 쿼리 오류 수정
2021-04-05) 큐브리드 정리 

![image](https://user-images.githubusercontent.com/68880203/113540798-dc587480-961b-11eb-9446-336ce582be90.png)
# CUBRID
    네이버의 DBMS
	CUBRID는 객체 관계형 데이터베이스 관리 시스템으로서, 데이터베이스 서버, 브로커, CUBRID 매니저로 구성된다. 
	CUBRID는 인터넷 데이터 서비스에 최적화된 데이터베이스 시스템이며, 사용자가 편리하게 사용할 수 있는 다양한 기능을 제공한다. 
## 테이블
	CREATE TABLE olympic2 (
    host_year        INT    NOT NULL PRIMARY KEY,
    host_nation      VARCHAR(40) NOT NULL,
    host_city        VARCHAR(20) NOT NULL,
    opening_date     DATE        NOT NULL,
    closing_date     DATE        NOT NULL,
    mascot           VARCHAR(20),
    slogan           VARCHAR(40),
    introduction     VARCHAR(1500)
    );
    -- BASIC
	CREATE TABLE manager2 (full_name VARCHAR(40), age INT );
	-- NOT NULL 
	CREATE TABLE const_tbl2(id INT NOT NULL PRIMARY KEY, phone VARCHAR);
	-- Unique 

## 테이블 INSERT
	CREATE TABLE const_tbl6(id INT, phone VARCHAR, CONSTRAINT UNIQUE (id, phone));
	INSERT INTO const_tbl6 VALUES (1, NULL), (2, NULL), (1, '000-0000'), (1, '111-1111');
	SELECT * FROM const_tbl6;
	
		
## 테이블 ALTER
동의어 (TABLE = CLASS)
동의어 (VIEW = VCLASS)
동의어(COLUMN = ATTRIBUTE)

## 컬럼 ALTER
	CREATE TABLE a_tbl;
	ALTER TABLE a_tbl ADD COLUMN age INT DEFAULT 0 NOT NULL;
	ALTER TABLE a_tbl ADD COLUMN name VARCHAR FIRST;

## 테이블 INSERT
	CREATE TABLE const_tbl6(id INT, phone VARCHAR, CONSTRAINT UNIQUE (id, phone));
	INSERT INTO const_tbl6 VALUES (1, NULL), (2, NULL), (1, '000-0000'), (1, '111-1111');
	SELECT * FROM const_tbl6;
	
		
## 테이블 ALTER
동의어 (TABLE = CLASS)
동의어 (VIEW = VCLASS)
동의어(COLUMN = ATTRIBUTE)




## 컬럼
	- 초기값 설정
		- SHARED : 모든 행 동일, 다른 새로운 값을 INSERT 하면 모든 행에서 새로운 값으로 갱신된다. (전채 다 변경)
		- DEFAULT : 새로운 행 사입시 컬럼 지정하지 않으면 DEFAULT로 설정된 값 저장. -> MODIFY 로 설정을 변경할 수 있다. (전체 다 변경은 아님) 
		 SHARED 또는 DEFAULT 속성을 동시에 정의 불가 X
## 컬럼 ALTER
	CREATE TABLE a_tbl;
	ALTER TABLE a_tbl ADD COLUMN age INT DEFAULT 0 NOT NULL;
	ALTER TABLE a_tbl ADD COLUMN name VARCHAR FIRST;
	ALTER TABLE a_tbl ADD COLUMN id INT NOT NULL AUTO_INCREMENT UNIQUE FIRST;
	INSERT INTO a_tbl(age) VALUES(20),(30),(40);

	ALTER TABLE a_tbl ADD COLUMN phone VARCHAR(13) DEFAULT '000-0000-0000' AFTER name;
	SELECT * FROM a_tbl;


## AUTO_INCREMENT 사용
	: 테이블 내 한개만 존재
	CREATE TABLE table_name (id INT AUTO_INCREMENT[(seed, increment)]);
	CREATE TABLE table_name (id INT AUTO_INCREMENT) AUTO_INCREMENT = seed ;

## 인덱스
	- CREATE INDEX
		내림차순으로 정렬된 인덱스 생성
		CREATE INDEX gold_index ON participant(gold DESC);
		다중 컬럼 인덱스 생성
		CREATE INDEX name_nation_idx ON athlete(name, nation_code);
		
	- ADD INDEX
		ALTER TABLE a_tbl ADD INDEX i1(age ASC), ADD INDEX i2(phone DESC);
		
	- ALTER INDEX
		: 인덱스 삭제 후 재생성
		CREATE INDEX i_game_medal ON game(medal);
		ALTER INDEX i_game_medal ON game REBUILD; -> 인덱스 재생성1
		ALTER INDEX char_idx ON athlete(gender, nation_code) WHERE gender='M' AND nation_code='USA' REBUILD; -> 인덱스 재생성2
	
	- DROP INDEX
		DROP INDEX i_game_medal ON game;
		UNIQUE인 경우) DROP UNIQUE INDEX i_game_medal ON game;

## PK
	CREATE TABLE pk_tbl (a INT, b INT, PRIMARY KEY (a, b DESC));

	CREATE TABLE const_tbl7 (
		id INT NOT NULL,
		phone VARCHAR,
		CONSTRAINT pk_id PRIMARY KEY (id)
	);

	- PK 생성 제약키 -- CONSTRAINT keyword
	CREATE TABLE const_tbl8 (
		id INT NOT NULL PRIMARY KEY,
		phone VARCHAR
	);

	- pk키 다중 생성 -- primary key is defined on multiple columns 
	CREATE TABLE const_tbl8 (
		host_year    INT NOT NULL,
		event_code   INT NOT NULL,
		athlete_code INT NOT NULL,
		medal        CHAR (1)  NOT NULL,
		score        VARCHAR (20),
		unit         VARCHAR (5),
		PRIMARY KEY (host_year, event_code, athlete_code, medal)
	);
	
## 제약키
	- UNIQUE 제약 : 고유한 값, 동일한 컬럼값 추가시 에러
		CREATE TABLE const_tbl5(id INT UNIQUE, phone VARCHAR);
		
	- KEY 또는 INDEX 는 서로 동일, 해다 칼럼을 키로 하는 인덱스 
		CREATE TABLE const_tbl4(id INT, phone VARCHAR, KEY i_key(id DESC, phone ASC));
		
	
	
## Change / MODIFY
	change : 컬럼의 이름, 타입, 크기 및 속성을 변경, 기존 칼럼의 이름과 새 칼럼의 이름이 같으면 타입, 크기 및 속성만 변경
	MODIFY : 칼럼의 타입, 크기 및 속성을 변경, 칼럼의 이름은 변경X
	
	- 예시)
    CREATE TABLE t1 (a INTEGER);

	-- changing column a's name into a1
	ALTER TABLE t1 CHANGE a a1 INTEGER;
	-- changing column a1's constraint
	ALTER TABLE t1 CHANGE a1 a1 INTEGER NOT NULL;
	- or
		ALTER TABLE t1 MODIFY a1 INTEGER NOT NULL;

	-- changing column col1's type - "DEFAULT 1" constraint is removed.
	CREATE TABLE t1 (col1 INT DEFAULT 1);
	ALTER TABLE t1 MODIFY col1 BIGINT;

	-- changing column col1's type - "DEFAULT 1" constraint is kept.
	CREATE TABLE t1 (col1 INT DEFAULT 1, b VARCHAR(10));
	ALTER TABLE t1 MODIFY col1 BIGINT DEFAULT 1;
	-- changing column b's size
	ALTER TABLE t1 MODIFY b VARCHAR(20);

### Create as select 
	a_tbl 
	  (1,'111-1111')
	  (2,'222-2222')
	  (3, '333-3333')
	  
	CREATE TABLE new_tbl1 AS SELECT * FROM a_tbl;
	SELECT * FROM new_tbl1;
	
	
## 뷰 (VIEW 와 VCLASS 는 동의어)
	a_tbl 
	  (1,'111-1111')
	  (2,'222-2222')
	  (3, '333-3333')
	CREATE VIEW b_view AS SELECT * FROM a_tbl WHERE phone IS NOT NULL WITH CHECK OPTION;
	SELECT * FROM b_view;
	- CHANGE QUERY
		ALTER VIEW b_view ADD QUERY SELECT * FROM a_tbl WHERE id = 3;
	- DROP QUERY
		ALTER VIEW b_view DROP QUERY 2,3;
	- RENAME VIEW
		RENAME VIEW game_2004 AS info_2004;
	- DROP VIEW
		DROP VIEW b_view;
	 
## ORACLE의 시퀀스 SEQUENCE
기존 Oracle의 Sequence 차이
시퀀스란 자동으로 순차적으로 증가하는 순번을 반환하는 데이터베이스 객체.
 기존 문법
    - CREATE SEQUENCE [시퀀스명]
    - INCREMENT BY [증감숫자]
    - START WITH [시작숫자] 
    - NOMINVALUE OR MINVALUE [최솟값]
    - NOMAXVALUE OR MAXVALUE [최대값] 
    - CYCLE OR NOCYCLE 
    - CACHE OR NOCACHE 
       : CACHE 설정시 메모리에 시퀀스 값을 미리 할당하고 NOCACHE 설정시 시퀀스값을 메로리에 할당하지 않음

    (예시)
    --(CREATE)
    CREATE SEQUENCE EX_SEQ --시퀀스이름 EX_SEQ
    INCREMENT BY 1 --증감숫자 1
    START WITH 1 --시작숫자 1
    MINVALUE 1 --최소값 1
    MAXVALUE 1000 --최대값 1000
    NOCYCLE --순한하지않음
    CACHE; --메모리에 시퀀스값 미리할당
    (Read)
    SELECT EX_SEQ.CURRVAL FROM DUAL --해당 시퀀스 값 
    SELECT * FROM USER_SEQUENCES  --전체 시퀀스 조회
    (Update)
    ALTER SEQUENCE [시퀀스명]
    INCREMENT BY [증가값]
    (Delete)
    DROP SEQUENCE [시퀀스명]
    
   ## CUBRID의 시퀀스 SEQUENCE 
    - CREATE SEQUENCE
		CREATE SERIAL serial_name
		[ START WITH initial ]
		[ INCREMENT BY interval ]
		[ MINVALUE min | NOMINVALUE ]
		[ MAXVALUE max | NOMAXVALUE ]
		[ CACHE cached_num | NOCACHE ]
	
		
		CREATE SERIAL order_no START WITH 10000 INCREMENT BY 2 MAXVALUE 20000;
				code  name
		===================================
				10000  'Park'
				10002  'Kim'
				10004  'Choo'
				10004  'Lee'
				
	- ALTER SEQUENCE
		ALTER SERIAL order_no START WITH 100 MINVALUE 100 INCREMENT BY 2;
		ALTER SERIAL order_no CACHE 5;
		--altering serial to operate in common mode
		ALTER SERIAL order_no NOCACHE;
	
	- DROP SEQUENCE
		: IF EXISTS 절을 함께 지정하는 경우, 대상 시리얼이 없어도 에러가 발생 X
		
		DROP SERIAL order_no;
		DROP SERIAL IF EXISTS order_no;

    
    



	
	

