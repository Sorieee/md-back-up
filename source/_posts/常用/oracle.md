# 查询唯一约束条件

```sql
select a.constraint_name,a.constraint_type,b.column_name,b.table_name
from user_constraints a inner join user_cons_columns b on a.table_name=b.table_name
where a.constraint_name='SYS_C0044117'
```

# oracle修改序列当前值

```sql
SELECT CM_COURSE_FORMAT_S.NEXTVAL FROM DUAL;
SELECT MAX(TO_NUMBER(UNIQUEID)) FROM CM_COURSE_FORMAT;

39250
alter sequence CM_COURSE_FORMAT_S increment by 10,869;
SELECT CM_COURSE_FORMAT_S.NEXTVAL FROM DUAL;
alter sequence CM_COURSE_FORMAT_S increment by 1;

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

DELETE FROM UNITIME_INSTRUCTOR;
DELETE FROM  KRIM_ENTITY_T;
DELETE FROM  KRIM_ENTITY_ENT_TYP_T;
DELETE FROM KRIM_PRNCPL_T;
DELETE FROM KRIM_ROLE_MBR_T;
