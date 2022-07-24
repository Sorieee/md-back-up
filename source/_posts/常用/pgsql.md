# 替换json字段

```sql

with tmp as (SELECT id, replace(patient_common_text->'main'->2->'value'->>0, chr(13), '') res FROM registry
WHERE patient_common_text->'main'->2->'value'->>0 like '%'||chr(13)||'%')
update registry t1 set patient_common_text = jsonb_set(patient_common_text, '{main, 2, value, 0}', ('"'||tmp.res||'"')::jsonb ) 
from tmp
WHERE t1.id = tmp.id;
```

