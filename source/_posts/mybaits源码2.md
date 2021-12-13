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

