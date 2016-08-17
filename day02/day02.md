# day02
##  1. 数据库
1. 数据库尽可能保持单表查询，使用冗余解决表的关联问题。
2. mybatis逆向工程，再次生成代码需删除已生成代码，否则被追加新代码。
---- 
## 2. ssm整合
### 2.1.1. Dao
mybatis：sqlMapConfig.xml, 创建一个applicationContext-dao.xml
1. 配置数据源
2. 需要让spring容器管理SqlsessionFactory，单例存在。
3. 把mapper的代理对象放到spring容器中。使用扫描包的方式加载mapper的代理对象。
#### 2.1.2. Service
1. 事务管里。
2. service 实现类对象放到spring容器中管理。
#### 2.1.3. 表现层
1. 配置注解驱动
2. 配置视图解析器
3. 需要扫描controller
#### 2.1.4. Web.xml
1. spring容器配置
2. SpringMVC前端控制器配置
3. Post乱码过滤器
### 2.2. 框架整合
需要把配置文件放到taotao-manager-web工程下。因为此工程为war工程，其他的工程只是一个jar包。
#### 2.2.1 Mybatis整合
**路径taotao-manager/taotao-manager-web/src/main/resources
`SqlMapConfig.xml`
`db.properties`
`applicationContext-dao.xml`
1. 数据库连接池( com.alibaba.druid.pool.DruidDataSource ) db.properties
2.  sqlsessionFactory ( org.mybatis.spring.SqlSessionFactoryBean )
3. 配置扫描包，加载mapper代理对象
#### 2.2.2 Service层
`applicationContext-service.xml`
`applicationContext-trans.xml`
#### 2.2.3 表现层
`springmvc.xml`
`web.xml`
1. spring容器`(   classpath:spring/applicationContext-*.xml   )`
	2. post乱码
	3. springmvc的前端控制器( DispatcherServlet )
