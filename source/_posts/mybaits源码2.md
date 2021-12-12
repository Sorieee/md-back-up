# MybatisAutoConfiguration

提供了如下bean

```
SqlSessionFactory
SqlSessionTemplate

```

## MapperProxy

```
PlainMethodInvoker.invoke
	MapperMethod.execute
		sqlSessionProxy(DefaultSqlSession).selectOne()
			
```

```
shared-configs:
  -
    dataId:
      application-${spring.profiles.active}.${spring.cloud.nacos.config.file-extension}
    group: dev
```