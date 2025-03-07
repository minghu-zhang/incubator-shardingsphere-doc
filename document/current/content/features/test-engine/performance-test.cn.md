+++
pre = "<b>3.6.5. </b>"
toc = true
title = "性能测试"
weight = 5
+++

## 目标

根据验证目标将性能测试分为损耗测试和提升测试，从业务角度考虑，insert+update+delete通常用作一个完整的关联操作，可用作损耗/提升测试评估，而select作为查询操作关注分片优化可用作损耗/提升操作性能评估的另一个操作，因此在性能测试设计中，基于基本应用场景（单路由，主从，主从+脱敏+分库分表，全路由）分别以上述操作做测试。为了更好的观察性能效果，设计在已知数据量（200W）的基础上，20并发线程持续压测半小时，进行增删改查性能测试。

## 测试场景

#### 单路由

分库分表，根据ID分为4个库，根据K分为1024个表，在200W数据的基础上，查询操作路由到单库单表。

#### 主从

基本主从场景，设置一主库一从库，查询使用相同的单条数据查询。

#### 主从+脱敏+分库分表

根据ID分为4个库，根据K分为1024个表，C使用aes加密，pad使用md5加密，查询操作路由到单库单表。

#### 全路由

分库分表，根据ID分为4个库，根据K分为1个表，查询操作使用全路由。

## 测试环境搭建

#### 数据库表结构

```shell
CREATE TABLE `press_test` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `k` int(11) NOT NULL DEFAULT 0,
  `c` char(120) NOT NULL DEFAULT '',
  `pad` char(60) NOT NULL DEFAULT '',
  PRIMARY KEY (`id`)
);
```

#### 测试场景配置

Sharding-JDBC使用与Sharding-Proxy一致的配置，MySQL直连一个库用做损耗/提升对比，下面为四个场景的具体配置：

##### 单路由配置

```yaml
schemaName: sharding_db

dataSources:
  press_test_0:
    url: jdbc:mysql://***.***.***.***:****/press_test?serverTimezone=UTC&useSSL=false
    username: test
    password: 
    connectionTimeoutMilliseconds: 30000
    idleTimeoutMilliseconds: 60000
    maxLifetimeMilliseconds: 1800000
    maxPoolSize: 200
  press_test_1:
    url: jdbc:mysql://***.***.***.***:****/press_test?serverTimezone=UTC&useSSL=false
    username: test
    password: 
    connectionTimeoutMilliseconds: 30000
    idleTimeoutMilliseconds: 60000
    maxLifetimeMilliseconds: 1800000
    maxPoolSize: 200
  press_test_2:
    url: jdbc:mysql://***.***.***.***:****/press_test?serverTimezone=UTC&useSSL=false
    username: test 
    password: 
    connectionTimeoutMilliseconds: 30000
    idleTimeoutMilliseconds: 60000
    maxLifetimeMilliseconds: 1800000
    maxPoolSize: 200
  press_test_3:
    url: jdbc:mysql://***.***.***.***:****/press_test?serverTimezone=UTC&useSSL=false
    username: test
    password:
    connectionTimeoutMilliseconds: 30000
    idleTimeoutMilliseconds: 60000
    maxLifetimeMilliseconds: 1800000
    maxPoolSize: 200
shardingRule:
    tables:
      test:
        actualDataNodes: press_test_${0..3}.test${0..1023}
        tableStrategy:
          inline:
            shardingColumn: k
            algorithmExpression: test${k % 1024}
        keyGenerator:
            type: SNOWFLAKE
            column: id
    defaultDatabaseStrategy:
      inline:
        shardingColumn: id
        algorithmExpression: press_test_${id % 4}
    defaultTableStrategy:
      none:
```

##### 主从配置

```yaml
schemaName: sharding_db

dataSources:
  master_ds:
    url: jdbc:mysql://***.***.***.***:****/press_test?serverTimezone=UTC&useSSL=false
    username: test
    password:
    connectionTimeoutMilliseconds: 30000
    idleTimeoutMilliseconds: 60000
    maxLifetimeMilliseconds: 1800000
    maxPoolSize: 200
  slave_ds_0:
    url: jdbc:mysql://***.***.***.***:****/press_test?serverTimezone=UTC&useSSL=false
    username: test
    password:
    connectionTimeoutMilliseconds: 30000
    idleTimeoutMilliseconds: 60000
    maxLifetimeMilliseconds: 1800000
    maxPoolSize: 200
masterSlaveRule:
  name: ms_ds
  masterDataSourceName: master_ds
  slaveDataSourceNames:
    - slave_ds_0
```

##### 主从+脱敏+分库分表配置

```yaml
schemaName: sharding_db

dataSources:
  master_ds_0:
    url: jdbc:mysql://***.***.***.***:****/press_test?serverTimezone=UTC&useSSL=false
    username: test
    password:
    connectionTimeoutMilliseconds: 30000
    idleTimeoutMilliseconds: 60000
    maxLifetimeMilliseconds: 1800000
    maxPoolSize: 200
  slave_ds_0:
    url: jdbc:mysql://***.***.***.***:****/press_test?serverTimezone=UTC&useSSL=false
    username: test
    password:
    connectionTimeoutMilliseconds: 30000
    idleTimeoutMilliseconds: 60000
    maxLifetimeMilliseconds: 1800000
    maxPoolSize: 200
  master_ds_1:
    url: jdbc:mysql://***.***.***.***:****/press_test?serverTimezone=UTC&useSSL=false
    username: test
    password:
    connectionTimeoutMilliseconds: 30000
    idleTimeoutMilliseconds: 60000
    maxLifetimeMilliseconds: 1800000
    maxPoolSize: 200
  slave_ds_1:
    url: jdbc:mysql://***.***.***.***:****/press_test?serverTimezone=UTC&useSSL=false
    username: test
    password:
    connectionTimeoutMilliseconds: 30000
    idleTimeoutMilliseconds: 60000
    maxLifetimeMilliseconds: 1800000
    maxPoolSize: 200
  master_ds_2:
    url: jdbc:mysql://***.***.***.***:****/press_test?serverTimezone=UTC&useSSL=false
    username: test
    password:
    connectionTimeoutMilliseconds: 30000
    idleTimeoutMilliseconds: 60000
    maxLifetimeMilliseconds: 1800000
    maxPoolSize: 200
  slave_ds_2:
    url: jdbc:mysql://***.***.***.***:****/press_test?serverTimezone=UTC&useSSL=false
    username: test
    password:
    connectionTimeoutMilliseconds: 30000
    idleTimeoutMilliseconds: 60000
    maxLifetimeMilliseconds: 1800000
    maxPoolSize: 200
  master_ds_3:
    url: jdbc:mysql://***.***.***.***:****/press_test?serverTimezone=UTC&useSSL=false
    username: test
    password:
    connectionTimeoutMilliseconds: 30000
    idleTimeoutMilliseconds: 60000
    maxLifetimeMilliseconds: 1800000
    maxPoolSize: 200
  slave_ds_3:
    url: jdbc:mysql://***.***.***.***:****/press_test?serverTimezone=UTC&useSSL=false
    username: test
    password:
    connectionTimeoutMilliseconds: 30000
    idleTimeoutMilliseconds: 60000
    maxLifetimeMilliseconds: 1800000
    maxPoolSize: 200
shardingRule:
  tables:
    test:
      actualDataNodes: ms_ds_${0..3}.test${0..1023}
      databaseStrategy:
        inline:
          shardingColumn: id
          algorithmExpression: ms_ds_${id % 4}
      tableStrategy:
        inline:
          shardingColumn: k
          algorithmExpression: test${k % 1024}
      keyGenerator:
        type: SNOWFLAKE
        column: id
  bindingTables:
    - test
  defaultDataSourceName: master_ds_1
  defaultTableStrategy:
    none:
  masterSlaveRules:
    ms_ds_0:
      masterDataSourceName: master_ds_0
      slaveDataSourceNames:
        - slave_ds_0
      loadBalanceAlgorithmType: ROUND_ROBIN
    ms_ds_1:
      masterDataSourceName: master_ds_1
      slaveDataSourceNames:
        - slave_ds_1
      loadBalanceAlgorithmType: ROUND_ROBIN
    ms_ds_2:
      masterDataSourceName: master_ds_2
      slaveDataSourceNames:
        - slave_ds_2
      loadBalanceAlgorithmType: ROUND_ROBIN
    ms_ds_3:
      masterDataSourceName: master_ds_3
      slaveDataSourceNames:
        - slave_ds_3
      loadBalanceAlgorithmType: ROUND_ROBIN
encryptRule:
  encryptors:
    encryptor_aes:
      type: aes
      props:
        aes.key.value: 123456abc
    encryptor_md5:
      type: md5
  tables:
    sbtest:
      columns:
        c:
          plainColumn: c_plain
          cipherColumn: c_cipher
          encryptor: encryptor_aes
        pad:
          cipherColumn: pad_cipher
          encryptor: encryptor_md5    
```

##### 全路由

```yaml
schemaName: sharding_db

dataSources:
  press_test_0:
    url: jdbc:mysql://***.***.***.***:****/press_test?serverTimezone=UTC&useSSL=false
    username: test
    password:
    connectionTimeoutMilliseconds: 30000
    idleTimeoutMilliseconds: 60000
    maxLifetimeMilliseconds: 1800000
    maxPoolSize: 200
  press_test_1:
    url: jdbc:mysql://***.***.***.***:****/press_test?serverTimezone=UTC&useSSL=false
    username: test
    password:
    connectionTimeoutMilliseconds: 30000
    idleTimeoutMilliseconds: 60000
    maxLifetimeMilliseconds: 1800000
    maxPoolSize: 200
  press_test_2:
    url: jdbc:mysql://***.***.***.***:****/press_test?serverTimezone=UTC&useSSL=false
    username: test
    password:
    connectionTimeoutMilliseconds: 30000
    idleTimeoutMilliseconds: 60000
    maxLifetimeMilliseconds: 1800000
    maxPoolSize: 200
  press_test_3:
    url: jdbc:mysql://***.***.***.***:****/press_test?serverTimezone=UTC&useSSL=false
    username: test
    password:
    connectionTimeoutMilliseconds: 30000
    idleTimeoutMilliseconds: 60000
    maxLifetimeMilliseconds: 1800000
    maxPoolSize: 200
shardingRule:
  tables:
    test:
      actualDataNodes: press_test_${0..3}.test1
      tableStrategy:
        inline:
          shardingColumn: k
          algorithmExpression: test1
      keyGenerator:
          type: SNOWFLAKE
          column: id
  defaultDatabaseStrategy:
    inline:
      shardingColumn: id
      algorithmExpression: press_test_${id % 4}
  defaultTableStrategy:
    none:  
```

## 测试结果验证

#### 压测语句

```shell
Insert+Update+Delete语句：
Insert into press_test(k, c, pad) values (1, "###", "###")
Update press_test set c="###-#", pad="###-#" where id=**
Delete from press_test where id=**
全路由查询语句：
select max(id) from test where id%4=1
单路由查询语句：
select id, k from test where id=1 and k=1
```

#### 压测类

```shell
参考https://github.com/apache/incubator-shardingsphere-benchmark/tree/master/shardingsphere-benchmark
```

#### 编译

```shell
git clone https://github.com/apache/incubator-shardingsphere-benchmark.git
cd incubator-shardingsphere-benchmark/shardingsphere-benchmark
mvn clean install
```

#### 压测执行

```shell
cp target/shardingsphere-benchmark-1.0-SNAPSHOT-jar-with-dependencies.jar apache-jmeter-4.0/lib/ext
jmeter –n –t test_plan/test.jmx
test.jmx参考https://github.com/apache/incubator-shardingsphere-benchmark/tree/master/report/script/test_plan/test.jmx
```

#### 压测结果处理

注意修改为上一步生成的result.jtl的位置。
```shell
sh shardingsphere-benchmark/report/script/gen_report.sh
```