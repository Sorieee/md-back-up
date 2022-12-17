# 查询唯一约束条件

```sql
select a.constraint_name,a.constraint_type,b.column_name,b.table_name
from user_constraints a inner join user_cons_columns b on a.table_name=b.table_name
where a.constraint_name='SYS_C0028653'
```

# oracle修改序列当前值

```sql
SELECT TT_CLASS_TIMETABLE_INSTR_S.NEXTVAL FROM DUAL;

SELECT MAX(TO_NUMBER(TT_CLASS_TIMETABLE_INSTR))  FROM UNITIME_CLASS_INSTRUCTOR;


alter sequence CM_PREREQUISITE_COURSE_S increment by 6400;
SELECT CM_PREREQUISITE_COURSE_S.NEXTVAL FROM DUAL;
alter sequence CM_PREREQUISITE_COURSE_S increment by 1;

```

# 查询执行计划

```
explain plan for ;

select * from table(dbms_xplan.dispaly);
```

# 添加索引

```
create index idx_address on

employee(address);
```

# 创建序列

```sql
CREATE SEQUENCE TEST_TAB 
INCREMENT BY 1 
START WITH 1
MINVALUE 0  
NOCYCLE 
CACHE 20
```



# 建表

```sql
CREATE TABLE UT_OFFERING_EXAM_FORM(
    UNIQUEID VARCHAR2(30) NOT NULL,
    OBJ_ID VARCHAR2(36) NOT NULL,
    COURSE_OFFERING_ID VARCHAR2(30) NOT NULL,
    EXAM_FORM VARCHAR2(255) NOT NULL,
    EXAM_TYPE VARCHAR2(255) NOT NULL,
    CREATE_BY VARCHAR2(32),
    CREATE_TIME DATE NOT NULL,
    UPDATE_BY VARCHAR2(32),
    UPDATE_TIME DATE NOT NULL,
    PRIMARY KEY (UNIQUEID)
);

COMMENT ON TABLE UT_OFFERING_EXAM_FORM IS '考试组织形式';
COMMENT ON COLUMN UT_OFFERING_EXAM_FORM.UNIQUEID IS 'ID';
COMMENT ON COLUMN UT_OFFERING_EXAM_FORM.COURSE_OFFERING_ID IS '开课课程id';
COMMENT ON COLUMN UT_OFFERING_EXAM_FORM.EXAM_FORM IS '考试形式;1 线下，2 线上，3 线上加线下';
COMMENT ON COLUMN UT_OFFERING_EXAM_FORM.EXAM_TYPE IS '考试类型;正考、补考、缓考';
COMMENT ON COLUMN UT_OFFERING_EXAM_FORM.CREATE_BY IS '创建人';
COMMENT ON COLUMN UT_OFFERING_EXAM_FORM.CREATE_TIME IS '创建时间';
COMMENT ON COLUMN UT_OFFERING_EXAM_FORM.UPDATE_BY IS '更新人';
COMMENT ON COLUMN UT_OFFERING_EXAM_FORM.UPDATE_TIME IS '更新时间';

	
create UT_OFFERING_EXAM_FORM_S
START WITH 1000
    maxvalue 10000000000;
```

## 创建序列



## 创建表空间

select file#,status,name from v$datafile;

```
create tablespace tab_name
datafile 'ccibe'
size n
[autoextend on next n1 maxsize m /of]

create tablespace CCIBE datafile 'C:\SOFT\DB\ORACLE\ORADATA\ORCL\CCIBE.DBF' size 5G autoextend on next 1G maxsize unlimited;

create user CCIBE  identified by cquisse default tablespace CCIBE
grant connect,resource,dba to CCIBE;
```

# 查询序列创建

```sql
select 'create sequence '||sequence_name||  
       ' minvalue '||min_value||  
       ' maxvalue '||max_value||  
       ' start with '||last_number||  
       ' increment by '||increment_by||  
       (case when cache_size=0 then ' nocache' else ' cache '||cache_size end) ||';'  --这个是实时的Sequence
from user_sequences 
```

# 视图



# 回滚数据

```sql
alter table CLIENT_DETAILS enable row movement;

FLASHBACK TABLE CLIENT_DETAILS TO timestamp to_timestamp('2020-8-28 09:30:00' , 'yyyy-mm-dd hh24:mi:ss');

ALTER TABLE CLIENT_DETAILS DISABLE ROW MOVEMENT;

select uniqueid, NVL(RELEASE_FLAG, 'N') RELEASE_FLAG from EXAM_ITEM_TASK as of timestamp to_Date('2022-10-20 13:30:00', 'yyyy-mm-dd hh24:mi:ss')
```
