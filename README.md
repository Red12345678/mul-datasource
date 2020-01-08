## 多数据源切换使用说明及范例：

    本文档着重描述如何在一个系统中访问多个数据库[一个主库和多个子库], 在系统访问数据库不指定子库的情况下,
默认访问的是主库, 只有指定了子库才能访问子库

------

###### 后端范例

* 在子系统pom.xml中添加公共模块的datasource组件pom依赖

  ```java
     <dependency>
       <groupId>com.cjkj</groupId>
       <artifactId>cjkj-datasource-spring-boot-starter</artifactId>
     </dependency>
  ```
  
* 在子系统配置文件中配置数据库信息包括一个主库和多个子库
  * 主库只配置一个[master-db]
  * 子库可以配置多个[slave-dbs]
  * 基本信息包括：jdbcUrl: 数据库服务地址，username: 数据库用户名，password: 数据库密码, driverClassName: 连接数据库驱动, enabled: 当前数据源是否可用标识, toDataSourceKey: 切换数据源key，最好这么命名：DB1，DB2，DB3 .....
    
  ```java
    # 多数据来源配置
    dbs:
      master-db:
        jdbcUrl: jdbc:mysql://10.253.96.125:3306/asc_transport?useUnicode=true&characterEncoding=utf-8&useSSL=false&serverTimezone=GMT%2B8
        username: asc_user
        password: Asc!@#123
        driverClassName: com.mysql.cj.jdbc.Driver
      slave-dbs:
        baseDbInfoList: [{jdbcUrl: "jdbc:mysql://10.253.96.125:3306/asc_trade?useUnicode=true&characterEncoding=utf-8&useSSL=false&serverTimezone=GMT%2B8", username: asc_user, password: Asc!@#123, driverClassName: com.mysql.cj.jdbc.Driver, enabled: true, toDataSourceKey: DB1}]
  ```

* 在子系统接口上通过 @DB(DataSourceType.DB1)加到接口上就可以切换至DB1数据源了
  * 如果在接口上不指定数据源, 数据源默认访问的是主库数据源
  * @DB注解中传key值需要传入要给常量[比如：可以定义一个常量叫DataSourceType，属性是DB1, DB2, DB3 ....]
  * @DB注解中的常量Name必须和配置文件中的toDataSourceKey的值[DB1, DB2, DB3 ....]保持一致,否则找不到对应的数据源
  
  ```java
   /**切换数据源key常量*/
   public enum DataSourceType {
  
      /**主数据源*/
      MASTER,
      /**子数据源1*/
      DB1,
      /**子数据源2*/
      DB2
   }
  
  /**切换数据源接口类*/
   @Service
   public class DbServiceImpl implements DbService {  
       @Autowired
       private SysUserOneDao sysUserOneDao;  
       @Autowired
       private SysTwoDao sysTwoDao;  
       @Override
       public SysUserOne selectUser(long id) {
           return sysUserOneDao.selectById(id);
       }  
       @DB(DataSourceType.DB1)
       @Override
       public SysTwo selectTwo(long id) {
           return sysTwoDao.selectById(id);
       }  
   }
  ```
