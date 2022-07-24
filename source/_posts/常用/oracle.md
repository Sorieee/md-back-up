# 查询唯一约束条件

```sql
select a.constraint_name,a.constraint_type,b.column_name,b.table_name
from user_constraints a inner join user_cons_columns b on a.table_name=b.table_name
where a.constraint_name='SYS_C0044117'
```

# oracle修改序列当前值

```sql
SELECT CM_COURSE_CONFIG_MODULE_S.NEXTVAL FROM DUAL;

SELECT MAX(TO_NUMBER(UNIQUEID))  FROM KRIM_ROLE_PERM_T WHERE CM_COURSE_CONFIG_MODULE;
111111772 - 111111791

alter sequence KRIM_ROLE_PERM_ID_S increment by 112,491;
SELECT KRIM_ROLE_PERM_ID_S.NEXTVAL FROM DUAL;
alter sequence KRIM_ROLE_PERM_ID_S increment by 1;

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
create table TT_TIMETABLE_RELEASE
(
  UNIQUEID      varchar2(30) not null, 
  RELEASED    varchar2(1) not null,
	SESSION_ID VARCHAR2(30) NOT NULL,
	DEGREE_TYPE VARCHAR2(20) not null,
	UPDATE_TIME DATE DEFAULT SYSDATE not null
);

comment on table TT_TIMETABLE_RELEASE 
  is '课表发布表';
comment on column TT_TIMETABLE_RELEASE.RELEASED
  is '已发布(Y)/未发布(N)';
comment on column TT_TIMETABLE_RELEASE.SESSION_ID
  is '学期id';
comment on column TT_TIMETABLE_RELEASE.DEGREE_TYPE
  is '本科/研究生';
comment on column TT_TIMETABLE_RELEASE.UPDATE_TIME
  is '保存时间';
	
 alter table TT_TIMETABLE_RELEASE
  add constraint pk_TT_TIMETABLE_RELEASE_id primary key (UNIQUEID);
	
create sequence TT_TIMETABLE_RELEASE_S
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

