![image](https://user-images.githubusercontent.com/68880203/113988217-32caea80-988a-11eb-9c76-533925bb0ccd.png)
# PostgreSQL과 Oracle 차이 정리
    PostgreSQL
	ANSI 표준 SQL을 사용, 오픈소스로 무료로 사용이 가능한다. 
	Oracle
	대규모 데이터베이스 지원, 성능이 좋고 기능이 많은데 비싸다.
	
## 1. TEXT ↔  CLOB (VARCHAR2) 
    (PostgreSQL) TEXT ↔  (Oralcle) CLOB
    LOB은 TEXT, 그래픽, 이미지, 비디오, 사운드 등 구조화되지 않은 대형 데이터를 저장하는데 사용한다. CLOB - Text, BLOB - 파일
    VARCHAR2 를 CLOB으로 삽입시 TO_CLOB(text), 혹은 CLOB을 TO_CHAR(clob) 으로 사용
                                                    참고: https://stepping.tistory.com/30 [디딤돌]

## 2. PIVOT :   STRING_AGG() ↔  LISTAGG()
    행으로 여러개 되어 있는 데이터를 한 줄로 가져오는 것
![image](https://user-images.githubusercontent.com/68880203/113993506-7f64f480-988f-11eb-8020-7a461bebd7d6.png)

    (PostgreSQL) STRING_AGG ↔  (Oralcle) LISTAGG
    
    
    STRING_AGG 함수는 GROUP BY 절과 함께 사용해야 한다. ORDER BY 절을 사용하여 정렬이 가능하며 ORDER BY 절은 생략할 수 있다.
    
    (POSTGRESQL)
    STRING_AGG("합칠컬럼명", "구분자") WITHIN GROUP(ORDER BY "컬럼명")
    (ORACLE)
    listagg (컬럼명, '구분기호') within group (order by 정렬기준컬럼)

                                    출처: https://goddaehee.tistory.com/57 [갓대희의 작은공간]
    예시)
![image](https://user-images.githubusercontent.com/68880203/113992371-5a23b680-988e-11eb-83fd-0a58a1023a28.png)
	
	

## 3. (SELECT RAWTOHEX(SYS_GUID()) FROM dual) ↔ uuid_generate_v4()::text
    고유한 값을 생성 함
    SYS_GUID() : 난수를 발생시켜 고유한 RAW DATA를 생성해내는 Oracle의 함수
    RAWTOHEX() : RAW DATA를 16진수의 문자열로 변경시키는 Oracle의 함수
    uid_generate_v4()::text : 위의 기능을 PostgreSQL에서 사용할 수 있도록 만든 

## 4. DELETE
    (Oracle)
    DELETE [TABLE_NAME] or DELETE FROM [TABLE_NAME]
    (PostgreSQL)
    DELETE FROM [TABLE_NAME]

## 5. MERGE

    (Oracle)
    MERGE INTO [TABLE_NAME]
    USING (
            [SELECT 절]
          ) v
       ON ([조건 절])
     WHEN MATCHED THEN
            UPDATE SET [UPDATE 절]

    (PostgreSQL)
    with v as (
        [SELECT 절]
    ) update [TABLE_NAME]
         set [UPDATE 절]
        from v
       where [조건 절]
       
  ## 6. 기타
  문자열 비교시(Tibero ; 거의 mysql, Oracle 과 유사) Tibero는 a ||'^'||b 로 문자열을 이어 붙이지만,
  Postgresql은 CONCAT으로 이어 붙인다. 

    <Tibero>
    AND g.scprop4||'^'||g.scprop7||'^'||g.scprop9||'^'||g.scprop21||'^'||g.scprop31||'^'||g.scprop32 !=
    g.tcprop4||'^'||g.tcprop7||'^'||g.tcprop9||'^'||g.tcprop21||'^'||g.tcprop31||'^'||g.tcprop32
------------------------------	
    <Postgresql>
    AND CONCAT(g.scprop4, '^', g.scprop7, '^', g.scprop8, '^', g.scprop9, '^', g.scprop23, '^', g.scprop24 ) !=
	CONCAT(g.tcprop4, '^', g.tcprop7, '^', g.tcprop8, '^', g.tcprop9, '^', g.tcprop23, '^', g.tcprop24)

   ## 7. Function
   : Oracle은 계층형 쿼리를 제공해주지만 Postgresql은 따로 제공해주는 것이 없음
   ### Oracle 예시
```
CREATE OR REPLACE FUNCTION MDQ_GET_BIZAREA_PATH (V_BIZ_ID VARCHAR2)
        RETURN VARCHAR2 IS
        RET   VARCHAR2(500) := '';
        BEGIN
            SELECT LISTAGG(x.biz_name, ' > ') WITHIN GROUP(ORDER BY LEVEL DESC)
              INTO RET
              FROM mdq_bizarea x
             START WITH x.biz_id = V_BIZ_ID
               AND x.expireddate = '99991231235959'
           CONNECT BY PRIOR x.biz_pid = x.biz_id
               AND x.expireddate = '99991231235959';
        RETURN RET;
        END MDQ_GET_BIZAREA_PATH;
 ```
     ### Postgresql 예시
```
    CREATE OR REPLACE FUNCTION MDQ_GET_BIZAREA_PATH(v_biz_id character varying)
    RETURNS character varying
    LANGUAGE plpgsql
    STABLE security definer
    AS $function$
    DECLARE
        ret varchar(500) := '';
    BEGIN
        WITH RECURSIVE cte AS (
            SELECT 1 as lvl
                    , biz_name
                    , biz_pid
                    , biz_id
              FROM mdq_bizarea x
             WHERE 1=1
               AND x.biz_id = v_biz_id
               AND x.expireddate = '99991231235959'
             UNION ALL
            SELECT cte.lvl + 1 as lvl
                  , x.biz_name
                  , x.biz_pid
                  , x.biz_id
              FROM mdq_bizarea x
                  JOIN cte cte
                ON cte.biz_pid = x.biz_id
             WHERE x.expireddate = '99991231235959'
        ) SELECT STRING_AGG(biz_name, ' > ' ORDER BY lvl DESC) INTO ret FROM cte;
        RETURN ret;
    END;
    $function$;
```
 
