# 查询唯一约束条件

```sql
select a.constraint_name,a.constraint_type,b.column_name,b.table_name
from user_constraints a inner join user_cons_columns b on a.table_name=b.table_name
where a.constraint_name='SYS_C0014310'
```

# oracle修改序列当前值

```sql
alter sequence UNITIME_COURSE_S increment by 239;
SELECT UNITIME_COURSE_S.NEXTVAL FROM DUAL;
alter sequence UNITIME_COURSE_S increment by 1;
```

