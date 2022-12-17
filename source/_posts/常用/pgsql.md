# 替换json字段

```sql

with tmp as (SELECT id, replace(patient_common_text->'main'->2->'value'->>0, chr(13), '') res FROM registry
WHERE patient_common_text->'main'->2->'value'->>0 like '%'||chr(13)||'%')
update registry t1 set patient_common_text = jsonb_set(patient_common_text, '{main, 2, value, 0}', ('"'||tmp.res||'"')::jsonb ) 
from tmp
WHERE t1.id = tmp.id;
```

# 建表

```sql
CREATE TABLE uni_message_config(
    id int8 NOT NULL,
    business_name varchar(64) NOT NULL,
    business_code varchar(64) NOT NULL,
    notify_type varchar(255) NOT NULL,
    msg_template text NOT NULL,
    update_time timestamp,
    PRIMARY KEY (id)
);

COMMENT ON TABLE uni_message_config IS '消息配置';
COMMENT ON COLUMN uni_message_config.business_name IS '业务名称';
COMMENT ON COLUMN uni_message_config.business_code IS '业务code';
COMMENT ON COLUMN uni_message_config.notify_type IS '通知方式;phone, email,station, 以|分割';
COMMENT ON COLUMN uni_message_config.msg_template IS '消息模板;{参数名称} 中代表参数';
COMMENT ON COLUMN uni_message_config.update_time IS '更新时间';
```



# 建序列

```sql
create sequence uni_message_config_s 
increment  by 1  --步长
minvalue 1     --最小值
maxvalue 9223372036854775807 --最大值
start 1  --起始值
cache 1  --每次生成几个值
cycle;  --到达最大值或最小值循环(不加默认不循环)

```



```
cn.uni.common.util.exception.UniException
```

# 加字段

```sql
COMMENT ON TABLE uni_message_config IS '消息配置';
```

