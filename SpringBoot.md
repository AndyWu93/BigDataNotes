# 代码片段

- https://blog.csdn.net/weixin_42925715/article/details/84027676?utm_medium=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7EBlogCommendFromMachineLearnPai2%7Edefault-1.vipsorttest&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7EBlogCommendFromMachineLearnPai2%7Edefault-1.vipsorttest

```java
// 不显示为空字段
@JsonInclude(value=JsonInclude.Include.NON_NULL)
public class testVo() {
  
}
```

## 2 调度

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.scheduling.annotation.EnableScheduling;

@SpringBootApplication
@EnableScheduling  // 启动类加上
public class RuozedataPlatformApplication {

    public static void main(String[] args) {
        SpringApplication.run(RuozedataPlatformApplication.class, args);
    }

}
```

## 3 表自动create

- properties

```yaml
spring:
  datasource:
    driver-class-name: com.mysql.jdbc.Driver
    url: jdbc:mysql://hadoop000:3306/test?useSSL=false&useUnicode=true&characterEncoding=utf-8&autoReconnect=true&serverTimezone=Asia/Shanghai
    username: root
    password: Weareme@123

  jpa:
    database: mysql
    show-sql: true
    hibernate:
      ddl-auto: update

```

- dependencies

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>

<dependency>
  <groupId>mysql</groupId>
  <artifactId>mysql-connector-java</artifactId>
  <version>5.1.47</version>
</dependency>
```

- dao

```java
import lombok.Data;

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;
import javax.persistence.Table;

@Entity
@Data
@Table(name = "company_info")
public class CompanyInfo {

    @Id
    @GeneratedValue
    public int id;

    public String companyName;

    public int fund_num;

    public String company_url;

    public String funds_url;
}
```



## 4 Jpa

```java
import com.ruozedata.bigdata.platform.domain.HDFSSummary;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;

import java.util.List;

public interface HDFSSummaryRepository extends JpaRepository<HDFSSummary, Long> {
     /**
     *  使用@Query注解的形式，查询某个班级下某种性别的所有学生的姓名
     *  上面方法是用的是参数的位置来查询的，Spring Data JPA中还支持用
     *  名称来匹配查询使用格式 “:参数名称” 引用
     * @param clazzName
     * @return
     */
    @Query("select s.name from Student s "
            + "where s.clazz.name = :clazzName and s.sex = :sex ")
    List<String> findNameByClazzNameAndSex(@Param("clazzName")String clazzName , @Param("sex")char sex);

    List<HDFSSummary> findByIsDeleteFalseAndCreateTimeBetweenOrderByCreateTimeDesc(Integer start, Integer end);

    HDFSSummary findTop1ByIsDeleteFalseAndCreateTimeLessThanEqualOrderByCreateTimeDesc(Integer time);
}
```

## 5 mapperfacade

```xml
<dependency>
    <groupId>ma.glasnost.orika</groupId>
    <artifactId>orika-core</artifactId>
</dependency>
```



# Lombok

## 1 注解

- @Data
  - 使用这个注解，就不用再去手写Getter,Setter,equals,canEqual,hasCode,toString等方法了，注解后在编译时会自动加进去。
- @AllArgsConstructor
  - 使用后添加一个构造函数，该构造函数含有所有已声明字段属性参数
- @NoArgsConstructor
  - 使用后创建一个无参构造函数
- @Builder
  - 关于Builder较为复杂一些，Builder的作用之一是为了解决在某个类有很多构造函数的情况，也省去写很多构造函数的麻烦，在设计模式中的思想是：用一个内部类去实例化一个对象，避免一个类出现过多构造函数





# [Spring Data](https://spring.io/projects/spring-data)

- [reference](https://docs.spring.io/spring-boot/docs/current/reference/html/dependency-versions.html#dependency-versions)

