## Mybatis-Plus 指南


### 常用注解

@TableName

描述：表名注解，标识实体类对应的表

使用位置：实体类


@TableId

描述：主键注解

使用位置：实体类主键字段

@TableField

描述：字段注解（非主键）

@Version

描述：乐观锁注解、标记 @Verison 在字段上


@EnumValue

描述：普通枚举类注解(注解在枚举字段上)

@TableLogic

描述：表字段逻辑处理注解（逻辑删除）


### 条件构造器

```
QueryWrapper<User> qw = new QueryWrapper<>();
qw.select("id","name").between("age",20,25)
        .orderByAsc("role_id","id");
List<User> plainUsers = userMapper.selectList(qw);

LambdaQueryWrapper<User> lwq = new LambdaQueryWrapper<>();
lwq.select(User::getId,User::getName).between(User::getAge,20,25)
        .orderByAsc(User::getRoleId,User::getId);
List<User> lambdaUsers = userMapper.selectList(lwq);
```

### 多数据源

```
# 多主多从                      纯粹多库（记得设置primary）                   混合配置
spring:                               spring:                               spring:
  datasource:                           datasource:                           datasource:
    dynamic:                              dynamic:                              dynamic:
      datasource:                           datasource:                           datasource:
        master_1:                             mysql:                                master:
        master_2:                             oracle:                               slave_1:
        slave_1:                              sqlserver:                            slave_2:
        slave_2:                              postgresql:                           oracle_1:
        slave_3:
```

使用 @DS 切换数据源。

@DS 可以注解在方法上或类上，同时存在就近原则 方法上注解 优先于 类上注解


### 分页插件

```
@Configuration
public class MybatisPlusConfig {
    @Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor() {
        MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
        interceptor.addInnerInterceptor(new PaginationInnerInterceptor(DbType.H2));
        return interceptor;
    }
}
```

```
@Test
void lambdaPagination() {
    Page<User> page = new Page<>(1, 3);
    Page<User> result = mapper.selectPage(page, Wrappers.<User>lambdaQuery().ge(User::getAge, 1).orderByAsc(User::getAge));
    assertThat(result.getTotal()).isGreaterThan(3);
    assertThat(result.getRecords().size()).isEqualTo(3);
  }
```


### 乐观锁插件
```
@Bean
public MybatisPlusInterceptor mybatisPlusInterceptor() {
    MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
    interceptor.addInnerInterceptor(new OptimisticLockerInnerInterceptor());
    return interceptor;
}
```

```
@Version
private Integer version;
```

