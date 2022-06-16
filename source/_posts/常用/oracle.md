# 查询唯一约束条件

```sql
select a.constraint_name,a.constraint_type,b.column_name,b.table_name
from user_constraints a inner join user_cons_columns b on a.table_name=b.table_name
where a.constraint_name='SYS_C00124765'
```

# oracle修改序列当前值

```sql
SELECT CM_COURSE_FORMAT_S.NEXTVAL FROM DUAL;
SELECT MAX(TO_NUMBER(UNIQUEID)) FROM CM_COURSE_FORMAT;

alter sequence CM_COURSE_FORMAT_S increment by 106;
SELECT CM_COURSE_FORMAT_S.NEXTVAL FROM DUAL;
alter sequence CM_COURSE_FORMAT_S increment by 1;
```

50014





26279354
