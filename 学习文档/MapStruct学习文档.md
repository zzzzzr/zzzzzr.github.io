# mapstruct入门教程

> 这篇文章用于记录学习了解`mapstruct` 框架时的内容。

## mapstruct简介

`mapstruct`是一个实体映射框架。当需要在DO与PO之间、VO与DO之间相互赋值时，我们通常会写一个converter，通过get set的方式进行赋值。`mapstruct` 能够简化我们的工作，让我们不必再去写这些“无意义”的代码。

`BeanUtils` 、`Orika` 等框架可以视作`mapstruct`的竞品，但`mapstruct`有比他们更高的效率。`mapstruct` 通过调用实体类的getter、setter方法实现赋值逻辑，这与我们手写代码的方式一致。相对的，`BeanUtils` 使用反射实现赋值，因此`mapstruct` 效率更高。

`mapstruct` 使用了`java apt` 技术，可以在代码编译时生成转换器类，当代码编译完成后，就可以在项目的`sources/generated-classes/annotations` 目录下看到转换器类的实现类。

## mapstruct的安装

使用`mapstruct` 时的`pom.xml`文件配置如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.example</groupId>
    <artifactId>test_map</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
        <org.mapstruct.version>1.5.2.Final</org.mapstruct.version>
        <org.projectlombok.version>1.18.24</org.projectlombok.version>
        <lombok-`mapstruct`-binding.version>0.2.0</lombok-`mapstruct`-binding.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.mapstruct</groupId>
            <artifactId>mapstruct</artifactId>
            <version>${org.mapstruct.version}</version>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>${org.projectlombok.version}</version>
            <scope>provided</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.8.1</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                    <annotationProcessorPaths>
                        <path>
                            <groupId>org.mapstruct</groupId>
                            <artifactId>mapstruct-processor</artifactId>
                            <version>${org.mapstruct.version}</version>
                        </path>
                        <path>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok</artifactId>
                            <version>${org.projectlombok.version}</version>
                        </path>
                        <path>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok-mapstruct-binding</artifactId>
                            <version>${lombok-mapstruct-binding.version}</version>
                        </path>
                    </annotationProcessorPaths>
                </configuration>
            </plugin>
        </plugins>
    </build>

</project>
```

需要注意的是，`lombok` 也使用了`java apt` 技术，因此在同时使用两个工具时，需要在`maven` 或者`gradle` 里进行配置，否则会出现`mapstruct` 找不到`lombok` 生成的getter、setter方法的情况。
这一配置体现在上面的pom文件的`<annotationProcessorPaths>` 项中。
相关链接：
<https://github.com/mapstruct/mapstruct-examples/blob/main/mapstruct-lombok/pom.xml>
<https://mapstruct.org/faq/>

## mapstruct的使用

### 基本使用

`mapstruct` 是实体映射框架，这里首先定义2个实体类：UserDO、UserPO

```java
@Getter
@Setter
@ToString(callSuper = true)
public class UserDo {

    private Long userId;

    private String userName;

}

@Getter
@Setter
public class UserPo {

    private Long id;

    private String name;

}
```

然后使用`mapstruct` 提供的注解，定义转换器接口UserConverter

```java
import org.mapstruct.Mapper;
import org.mapstruct.factory.Mappers;
import org.mapstruct.Mapping;

@Mapper
public interface UserConverter {

    UserConverter INSTANCE = Mappers.getMapper( UserConverter.class );

    @Mapping(source = "id", target = "userId")
    @Mapping(source = "name", target = "userName")
    UserDo po2do(UserPo po);

    @Mapping(source = "userId", target = "id"),
    @Mapping(source = "userName", target = "name")
    UserPo do2po(UserDo ddo);

}
```

最后的主函数如下：

```java
public class BasicMain {
    public static void main(String[] args) {
        UserDo userDo = new UserDo();

        userDo.setUserId(1111L);
        userDo.setUserName("test");

        System.out.println(userDo);

        UserPo userPo = UserConverter.INSTANCE.do2po(userDo);
        System.out.println(userPo);

        UserDo userDo2 = UserConverter.INSTANCE.po2do(userPo);
        System.out.println(userDo2);
    }
}
```

运行结果为：

```text
UserDo(super=test`mapstruct`.demo3.UserDo@15db9742, userId=1111, userName=test)
UserPo(super=test`mapstruct`.demo3.UserPo@3d4eac69, userId=1111, userName=test)
UserDo(super=test`mapstruct`.demo3.UserDo@42a57993, userId=1111, userName=test)
```

**快速理解**
可以看到，我们在converter接口上使用了`@Mapper` 注解，将其声明为一个`mapstruct` 转换器。

然后在其中声明了两个接口方法：`UserDo po2do(UserPo po);` 和 `UserPo do2po(UserDo ddo);` ，这两个方法会被`mapstruct` 实现为对应的转换方法，实现方法会生成在项目的`sources/generated-classes/annotations` 目录下。声明方法时名称可以自定，只要保证入参和出参分别是目标的转换类即可。

在接口方法上，我们使用`@Mapping` 注解，声明两个实体类中属性的映射关系。其中`source` 属性用于指定源实体的属性名，即转换方法的入参实体的属性名。而`target` 属性用于指定目标实体的属性名，即转换方法的出参实体的属性名。
以其中一条举例：`UserDo po2do(UserPo po)@Mapping(source = "id", target = "userId")` 。这条注解的含义是，把传入的PO实体的id 属性赋值到一个新的DO的userId 属性中。这里暗含的一条约束是：id 和userId 属性要是同类型的。相对的，如果要在不同类型的属性间相互转换，则需要额外配置，这一点后面会提到。

可以看到，我们在接口中定义一个了一个属性`INSTANCE` ，通过调用`Mappers.getMapper()` 获取了一个`UserConverter`的实例。要注意的是，作为接口的域，`INSTANCE` 默认带有`public staitc final` 修饰符，因此我们可以在外部通过`UserConverter.INSTANCE` 的方式访问这个域，并调用该转换器的转换方法。

### 详细使用

#### 对属性名相同、类型相同的属性的赋值

这种情况下，不需要声明任何注解，转换方法会自动进行赋值。

#### 对属性名不同、类型相同的属性的赋值

这种情况下，需要像上面的例子一样，声明源实体和目标实体的属性名。举例：

```Java
@Mapping(source = "id", target = "userId")
```

#### 对属性名相同、类型不同的属性的赋值

这种情况下，需要通过在接口中编写default 方法的方式为两个实体中的对应字段做转换。举例如下：
类定义

```java
import lombok.Getter;
import lombok.Setter;
import lombok.ToString;

@Getter
@Setter
@ToString(callSuper = true)
public class UserDo {

    private Long userId;

    private String userName;

    private Long number;

}

// ------------------------------------------

import lombok.Getter;
import lombok.Setter;
import lombok.ToString;

@Getter
@Setter
@ToString(callSuper = true)
public class UserPo {

    private Long id;

    private String name;

    private String number;

}
```

转换器

```java
import org.mapstruct.Mapper;
import org.mapstruct.Mapping;
import org.mapstruct.Mappings;
import org.mapstruct.factory.Mappers;

@Mapper
public interface UserConverter {

    UserConverter INSTANCE = Mappers.getMapper( UserConverter.class );

    @Mapping(source = "id", target = "userId")
    @Mapping(source = "name", target = "userName")
    UserDo po2do(UserPo po);

      @Mapping(source = "userId", target = "id")
      @Mapping(source = "userName", target = "name")
    UserPo do2po(UserDo ddo);

    default String num2str(Long num){
        return num.toString()+" test";
    }

    default Long str2num(String str){
        return Long.valueOf(str.split(" ")[0]);
    }

}
```

主函数

```java
public class BasicMain {
    public static void main(String[] args) {
        UserDo userDo = new UserDo();

        userDo.setUserId(1111L);
        userDo.setUserName("test");
        userDo.setNumber(1111L);

        System.out.println(userDo);

        UserPo userPo = UserConverter.INSTANCE.do2po(userDo);
        System.out.println(userPo);

        UserDo userDo2 = UserConverter.INSTANCE.po2do(userPo);
        System.out.println(userDo2);
    }
}
```

输出

```text
UserDo(super=test`mapstruct`.demo3.UserDo@15db9742, userId=1111, userName=test, number=1111)
UserPo(super=test`mapstruct`.demo3.UserPo@3d4eac69, id=1111, name=test, number=1111 test)
UserDo(super=test`mapstruct`.demo3.UserDo@42a57993, userId=1111, userName=test, number=1111)
```

可以看到，在转换方法上我们并没有通过`@Mapping` 注解声明字段的转换关系，但字段仍然按照期望的方式完成了转换（转换为字符串时出现了" test"的后缀，且在转换为数字时没有因为后缀而报错）

值得注意的是：这里为了展示，我特意在转换方法中增加了逻辑（指字符串增加后缀）。实际上，对于Long和String这样的原生类型，`mapstruct`是提供支持的。也就是说，如果没有特殊逻辑，即使不书写`str2num`和`num2str`两个默认方法，`mapstruct`依然能够完成转换（因为有`Long.toString()`和`Long.valueOf(String)`方法）。

#### 对属性名不同、类型不同的属性的赋值

这种情况下，需要通过在接口中编写`default`方法的方式为两个实体中的对应字段做转换。并且还需要在方法之上通过`@Mapping`注解来声明两个字段的对应关系。举例如下：
类定义

```Java
import java.util.Date;

import lombok.Getter;
import lombok.Setter;
import lombok.ToString;

@Getter
@Setter
@ToString(callSuper = true)
public class UserDo {

    private Long userId;

    private String userName;

    private Long number;

    private Date date;

}

// ------------------------------------------------

import lombok.Getter;
import lombok.Setter;
import lombok.ToString;

@Getter
@Setter
@ToString(callSuper = true)
public class UserPo {

    private Long id;

    private String name;

    private String number;

    private Long timeStamp;

}
```

转换器

```Java
import org.mapstruct.Mapper;
import org.mapstruct.Mapping;
import org.mapstruct.Mappings;
import org.mapstruct.factory.Mappers;

import java.util.Date;

@Mapper
public interface UserConverter {

    UserConverter INSTANCE = Mappers.getMapper( UserConverter.class );

    @Mapping(source = "id", target = "userId")
    @Mapping(source = "name", target = "userName")
    @Mapping(source = "timeStamp", target = "date")
    UserDo po2do(UserPo po);

    @Mapping(source = "userId", target = "id")
    @Mapping(source = "userName", target = "name")
    @Mapping(source = "date", target = "timeStamp")
    UserPo do2po(UserDo ddo);

    default String num2str(Long num){
        return num.toString()+" test";
    }

    default Long str2num(String str){
        return Long.valueOf(str.split(" ")[0]);
    }

    default Date stamp2date(Long timeStamp) {
        return new Date(timeStamp + 1000* 60 * 60L);
    }

    default Long date2stamp(Date date) {
        return date.getTime() + 1000 * 60 * 60L;
    }

}
```

主函数

```Java
import java.util.Date;

public class BasicMain {
    public static void main(String[] args) {
        UserDo userDo = new UserDo();

        userDo.setUserId(1111L);
        userDo.setUserName("test");
        userDo.setNumber(1111L);
        Long current = System.currentTimeMillis();
        userDo.setDate(new Date(current));

        System.out.println(current);
        System.out.println(userDo);

        UserPo userPo = UserConverter.INSTANCE.do2po(userDo);
        System.out.println(userPo);

        UserDo userDo2 = UserConverter.INSTANCE.po2do(userPo);
        System.out.println(userDo2);
    }
}
```

输出

```text
1664100691389
UserDo(super=test`mapstruct`.demo3.UserDo@15db9742, userId=1111, userName=test, number=1111, date=Sun Sep 25 18:11:31 CST 2022)
UserPo(super=test`mapstruct`.demo3.UserPo@232204a1, id=1111, name=test, number=1111 test, timeStamp=1664104291389)
UserDo(super=test`mapstruct`.demo3.UserDo@4aa298b7, userId=1111, userName=test, number=1111, date=Sun Sep 25 20:11:31 CST 2022)
```

#### 指定赋值方法

有时在一个类中会出现多个A类别属性到B类型属性的赋值。如果这些赋值的逻辑一致，那么写一个`default`方法甚至不写就可以完成赋值。
但如果这些对应关系中出现了多种不同的赋值逻辑，那么我们需要为这些赋值逻辑分别书写`default`方法，并使用`@Named`注解声明其名称，最后在对应的`@Mapping`注解中通过`qualifiedByName`属性声明使用到的`default`方法的名称。举例如下：
类定义

```Java
import lombok.Getter;
import lombok.Setter;
import lombok.ToString;

@Getter
@Setter
@ToString(callSuper = true)
public class UserPo {

    private Long id;

    private String name;

    private String number;

    private Long timeStamp;

    private Long preTimeStamp;

}

/*-----------------------------------------------*/

import java.util.Date;

import lombok.Getter;
import lombok.Setter;
import lombok.ToString;

@Getter
@Setter
@ToString(callSuper = true)
public class UserDo {

    private Long userId;

    private String userName;

    private Long number;

    private Date date;

    private Date preDate;

}
```

转换器

```Java
import org.mapstruct.Mapper;
import org.mapstruct.Mapping;
import org.mapstruct.Mappings;
import org.mapstruct.Named;
import org.mapstruct.factory.Mappers;

import java.util.Date;

@Mapper
public interface UserConverter {

    UserConverter INSTANCE = Mappers.getMapper( UserConverter.class );

    @Mapping(source = "id", target = "userId")
    @Mapping(source = "name", target = "userName")
    @Mapping(source = "timeStamp", target = "date", qualifiedByName = "stamp2date1")
    @Mapping(source = "preTimeStamp", target = "preDate", qualifiedByName = "stamp2date2")
    UserDo po2do(UserPo po);

    @Mapping(source = "userId", target = "id")
    @Mapping(source = "userName", target = "name")
    @Mapping(source = "date", target = "timeStamp", qualifiedByName = "date2stamp1")
    @Mapping(source = "preDate", target = "preTimeStamp", qualifiedByName = "date2stamp2")
    UserPo do2po(UserDo ddo);

    default String num2str(Long num){
        return num.toString()+" test";
    }

    default Long str2num(String str){
        return Long.valueOf(str.split(" ")[0]);
    }

    @Named("stamp2date1")
    default Date stamp2date(Long timeStamp) {
        return new Date(timeStamp + 1000* 60 * 60L);
    }

    @Named("date2stamp1")
    default Long date2stamp(Date date) {
        return date.getTime() + 1000 * 60 * 60L;
    }

    @Named("stamp2date2")
    default Date stamp2date2(Long timeStamp) {
        return new Date(timeStamp - 1000* 60 * 60L);
    }

    @Named("date2stamp2")
    default Long date2stamp2(Date date) {
        return date.getTime() - 1000 * 60 * 60L;
    }
```

主函数

```Java
import java.util.Date;

public class BasicMain {
    public static void main(String[] args) {
        UserDo userDo = new UserDo();
        Long current = System.currentTimeMillis();

        userDo.setUserId(1111L);
        userDo.setUserName("test");
        userDo.setNumber(1111L);
        userDo.setDate(new Date(current));
        userDo.setPreDate(new Date(current));

        System.out.println(current);
        System.out.println(userDo);

        UserPo userPo = UserConverter.INSTANCE.do2po(userDo);
        System.out.println(userPo);

        UserDo userDo2 = UserConverter.INSTANCE.po2do(userPo);
        System.out.println(userDo2);
    }
}
```

输出

```text
1664106359883
UserDo(super=test`mapstruct`.demo3.UserDo@15db9742, userId=1111, userName=test, number=1111, date=Sun Sep 25 19:45:59 CST 2022, preDate=Sun Sep 25 19:45:59 CST 2022)
UserPo(super=test`mapstruct`.demo3.UserPo@232204a1, id=1111, name=test, number=1111 test, timeStamp=1664109959883, preTimeStamp=1664102759883)
UserDo(super=test`mapstruct`.demo3.UserDo@4aa298b7, userId=1111, userName=test, number=1111, date=Sun Sep 25 21:45:59 CST 2022, preDate=Sun Sep 25 17:45:59 CST 2022)
```

**注意**
这个类里有2个Long到Date的转换逻辑，因此我首先书写了2组转换方法，然后通过`@Named`注解为这两组方法分别命名，最后在`@Mapping`注解里通过`qualifiedByName`属性将方法与属性对应起来。控制台输出表明，两组属性确实按照我们希望的赋值逻辑进行了对应（UserPO/UserDO中的2个时间戳/Date的值不同，且由于赋值逻辑的存在，其差值在拉大）。

如果愿意，可以只给第2组赋值方法用`@Named`注解命名，而不给第1组赋值方法命名。同时在第1组`@Mapping`里不使用`qualifiedByName`指定方法。这样的话，`mapstruct` 会自动选择没有命名的赋值方法。
举例如下：

```Java
import org.mapstruct.Mapper;
import org.mapstruct.Mapping;
import org.mapstruct.Mappings;
import org.mapstruct.Named;
import org.mapstruct.factory.Mappers;

import java.util.Date;

@Mapper
public interface UserConverter {

    UserConverter INSTANCE = Mappers.getMapper( UserConverter.class );

    @Mapping(source = "id", target = "userId")
    @Mapping(source = "name", target = "userName")
    @Mapping(source = "timeStamp", target = "date")
    @Mapping(source = "preTimeStamp", target = "preDate", qualifiedByName = "stamp2date2")
    UserDo po2do(UserPo po);

    @Mapping(source = "userId", target = "id")
    @Mapping(source = "userName", target = "name")
    @Mapping(source = "date", target = "timeStamp")
    @Mapping(source = "preDate", target = "preTimeStamp", qualifiedByName = "date2stamp2")
    UserPo do2po(UserDo ddo);

    default String num2str(Long num){
        return num.toString()+" test";
    }

    default Long str2num(String str){
        return Long.valueOf(str.split(" ")[0]);
    }

    default Date stamp2date(Long timeStamp) {
        return new Date(timeStamp + 1000* 60 * 60L);
    }

    default Long date2stamp(Date date) {
        return date.getTime() + 1000 * 60 * 60L;
    }

    @Named("stamp2date2")
    default Date stamp2date2(Long timeStamp) {
        return new Date(timeStamp - 1000* 60 * 60L);
    }

    @Named("date2stamp2")
    default Long date2stamp2(Date date) {
        return date.getTime() - 1000 * 60 * 60L;
    }
}
```

这种方式在以下场景中会比较有用：类A与类B的转换中有多个类P到类Q的赋值逻辑，且其中的大多数都使用同一种逻辑，仅有少数几种例外。则此时可以用上面的方式实现这样的效果：为类P到类Q的转换定义一种通用逻辑，满足大多数情况；为其中的特殊情况定义特殊逻辑并通过命名的方式指定，实现精准定位。
但是，考虑到赋值逻辑的明确，建议在遇到这种有多个转换逻辑的场景时，对所有类P到类Q的转换都使用`@Named`注解进行命名，避免后续维护时的疏忽。此外，如果为所有类P到类Q的转换方法进行了命名，那么在使用`@Mapping`注解时是必须使用`qualifiedByName`进行方法指定的，这可以视作一种错误提示。


#### 对List的赋值

如果在`classA`、`classB`的转换器接口中已经实现了`classA`到`classB`的转换方法，那么不需要再为`List<classA>`到`List<classB>`额外书写转换逻辑，只需要声明一个对应的方法即可。`mapstruct` 会自动生成一个转换方法，通过`forEach` 的方式，逐个调用转换方法完成赋值。举例如下：

```Java
@Mappings({
  @Mapping(source = "userId", target = "id"),
  @Mapping(source = "userName", target = "name"),
  @Mapping(source = "date", target = "timeStamp", qualifiedByName = "date2stamp1"),
  @Mapping(source = "preDate", target = "preTimeStamp", qualifiedByName = "date2stamp2")
})
UserPo do2po(UserDo ddo);

List<UserPo> dolist2polist(List<UserDo> doList);
```

#### 使用抽象类定义转换器

除了接口，`mapstruct` 还支持使用抽象类来定义转换逻辑。
使用抽象类定义转换器的方式与使用接口基本相同，主要的区别在于java语法层面对接口和抽象类的不同要求。具体有以下几点：

* 抽象类中不能使用`UserConverter INSTANCE = Mappers.getMapper( UserConverter.class );` 来定义转换器实例，而应该使用 `public static final UserConverter INSTANCE = Mappers.getMapper( UserConverter.class );` 来定义。（接口的域自带`public static final` 修饰符）

* 抽象类中的转换方法要以`abstract` 修饰。（接口的方法自带`abstract` 修饰符）

* 抽象类中的字段转换方法不能带`default` 修饰符。（`default` 修饰符只用在接口中）

以下是具体代码：

```Java
import org.mapstruct.Mapper;
import org.mapstruct.Mapping;
import org.mapstruct.Mappings;
import org.mapstruct.Named;
import org.mapstruct.factory.Mappers;

import java.util.Date;
import java.util.List;

@Mapper
public abstract class UserConverter2 {

    public static final UserConverter2 INSTANCE = Mappers.getMapper( UserConverter2.class );

    @Mapping(source = "id", target = "userId")
    @Mapping(source = "name", target = "userName")
    @Mapping(source = "timeStamp", target = "date", qualifiedByName = "stamp2date1")
    @Mapping(source = "preTimeStamp", target = "preDate", qualifiedByName = "stamp2date2")
    abstract UserDo po2do(UserPo po);

    @Mappings({
        @Mapping(source = "userId", target = "id"),
        @Mapping(source = "userName", target = "name"),
        @Mapping(source = "date", target = "timeStamp", qualifiedByName = "date2stamp1"),
        @Mapping(source = "preDate", target = "preTimeStamp", qualifiedByName = "date2stamp2")
    })
    abstract UserPo do2po(UserDo ddo);

    abstract List<UserDo> poList2doList(List<UserPo> poList);

    abstract List<UserPo> doList2poList(List<UserDo> doList);

    String num2str(Long num){
        return num.toString()+" test";
    }

    Long str2num(String str){
        return Long.valueOf(str.split(" ")[0]);
    }

    @Named("stamp2date1")
    Date stamp2date(Long timeStamp) {
        return new Date(timeStamp + 1000* 60 * 60L);
    }

    @Named("date2stamp1")
    Long date2stamp(Date date) {
        return date.getTime() + 1000 * 60 * 60L;
    }

    @Named("stamp2date2")
    Date stamp2date2(Long timeStamp) {
        return new Date(timeStamp - 1000* 60 * 60L);
    }

    @Named("date2stamp2")
    Long date2stamp2(Date date) {
        return date.getTime() - 1000 * 60 * 60L;
    }
}
```

#### 查看转换器实现类

上面提到，项目的`sources/generated-classes/annotations` 目录下能够看到`mapstruct` 对转换器的实现类。
通过查看可以发现，`mapstruct` 实际上是通过实现接口或继承抽象类并重写方法的方式，实现了赋值逻辑：
类定义：

```Java
@Generated(
    value = "org.mapstruct.ap.MappingProcessor",
    date = "2022-09-25T21:13:28+0800",
    comments = "version: 1.5.2.Final, compiler: javac, environment: Java 1.8.0_202 (Oracle Corporation)"
)
public class UserConverterImpl implements UserConverter {
  
  ...
/*------------------------------*/

@Generated(
    value = "org.mapstruct.ap.MappingProcessor",
    date = "2022-09-25T21:13:28+0800",
    comments = "version: 1.5.2.Final, compiler: javac, environment: Java 1.8.0_202 (Oracle Corporation)"
)
public class UserConverter2Impl extends UserConverter2 {
```

方法实现：

```Java
@Override
public UserDo po2do(UserPo po) {
  if ( po == null ) {
    return null;
  }

  UserDo userDo = new UserDo();

  userDo.setUserId( po.getId() );
  userDo.setUserName( po.getName() );
  userDo.setDate( stamp2date( po.getTimeStamp() ) );
  userDo.setPreDate( stamp2date2( po.getPreTimeStamp() ) );
  userDo.setNumber( str2num( po.getNumber() ) );

  return userDo;
}
```

查看`mapstruct` 的实现可知，`mapstruct` 的实现是通过getter、setter方法实现的赋值逻辑，因此具有比`BeanUtils` 等使用反射的框架更好的性能。

列表赋值

```Java
@Override
public List<UserDo> poList2doList(List<UserPo> poList) {
  if ( poList == null ) {
    return null;
  }

  List<UserDo> list = new ArrayList<UserDo>( poList.size() );
  for ( UserPo userPo : poList ) {
    list.add( po2do( userPo ) );
  }

  return list;
}
```

#### 对基本数据类型赋值

对基本数据类型的赋值与对对象类型的赋值没有区别，不论是基本类型转基本类型还是基本类型转包装类，都可以完全适配上面的规则。对于基本类型与其包装类的转换，可以不额外书写转换方法。

#### 使用外部转换器

有时我们需要完成以下的转换关系：
这时可以先书写Aclass的转换器，然后在书写Bclass的转换器时引用并使用Aclass的转换器。举例如下：

实体类定义：

```Java
import lombok.Getter;
import lombok.Setter;
import lombok.ToString;

@Getter
@Setter
@ToString(callSuper = true)
public class UserPo {

    private Long id;

    private String name;

    private LoverPo loverPo;

}
/*-----------------------------------*/
import lombok.Getter;
import lombok.Setter;
import lombok.ToString;

@Getter
@Setter
@ToString(callSuper = true)
public class UserDo {

    private Long userId;

    private String userName;

    private LoverDo loverDo;

}
/*-----------------------------------*/
import lombok.Getter;
import lombok.Setter;
import lombok.ToString;

@Getter
@Setter
@ToString(callSuper = true)
public class LoverPo {

    private String loName;

    private Integer loAge;

    private String num;

}
/*-----------------------------------*/
import lombok.AllArgsConstructor;
import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.Setter;
import lombok.ToString;

@Getter
@Setter
@ToString(callSuper = true)
@AllArgsConstructor
@NoArgsConstructor
public class LoverDo {

    private String loverName;

    private int loverAge;

    private long num;

}
```

转换器定义：

```Java
import org.mapstruct.Mapper;
import org.mapstruct.Mapping;
import org.mapstruct.Mappings;
import org.mapstruct.Named;
import org.mapstruct.factory.Mappers;


@Mapper(
    uses = {
        LoverConverter.class,
    }
)
public interface UserConverter {

    public static final UserConverter INSTANCE = Mappers.getMapper( UserConverter.class );

    @Mapping(source = "id", target = "userId")
    @Mapping(source = "name", target = "userName")
    @Mapping(source = "loverPo", target = "loverDo", qualifiedByName = {"convertLover", "po2do"})
    abstract UserDo po2do(UserPo po);

    @Mappings({
        @Mapping(source = "userId", target = "id"),
        @Mapping(source = "userName", target = "name"),
        @Mapping(target = "loverPo", source = "loverDo", qualifiedByName = {"convertLover", "do2po"}),
    })
    UserPo do2po(UserDo ddo);

}
/*-----------------------------------*/
import org.`mapstruct`.Mapper;
import org.`mapstruct`.Mapping;
import org.`mapstruct`.Mappings;
import org.`mapstruct`.Named;
import org.`mapstruct`.factory.Mappers;

@Mapper
@Named("convertLover")
public interface LoverConverter {

    public static final LoverConverter INSTANCE = Mappers.getMapper( LoverConverter.class );

    @Named("po2do")
    @Mappings({
        @Mapping(source = "loName", target = "loverName"),
        @Mapping(source = "loAge", target = "loverAge")
    })
    LoverDo po2do(LoverPo po);

    @Named("do2po")
    @InheritInverseConfiguration
    LoverPo do2po(LoverDo ddo);

    default String num2str(Long num){
        return num.toString()+" test";
    }

    default Long str2num(String str){
        return Long.valueOf(str.split(" ")[0]);
    }

}
```

主函数：

```Java
public class BasicMain {
    public static void main(String[] args) {
        UserDo userDo = new UserDo();

        userDo.setUserId(1111L);
        userDo.setUserName("test");
        userDo.setLoverDo(new LoverDo("kevy", 26, 3));

        System.out.println(userDo);

        UserPo userPo = UserConverter.INSTANCE.do2po(userDo);
        System.out.println(userPo);

        UserDo userDo2 = UserConverter.INSTANCE.po2do(userPo);
        System.out.println(userDo2);
    }
}
```

控制台输出：

```text
UserDo(super=test`mapstruct`.demo3.UserDo@15db9742, userId=1111, userName=test, loverDo=LoverDo(super=test`mapstruct`.demo3.LoverDo@6d06d69c, loverName=kevy, loverAge=26, num=3))
UserPo(super=test`mapstruct`.demo3.UserPo@42a57993, id=1111, name=test, loverPo=LoverPo(super=test`mapstruct`.demo3.LoverPo@75b84c92, loName=kevy, loAge=26, num=3 test))
UserDo(super=test`mapstruct`.demo3.UserDo@6bc7c054, userId=1111, userName=test, loverDo=LoverDo(super=test`mapstruct`.demo3.LoverDo@232204a1, loverName=kevy, loverAge=26, num=3))
```

想要达成“在一个转换器中使用另一个转换器”的目的，首先需要在定义被使用的转换器时为他的类和转换方法分别使用`@Named` 注解进行命名；然后要在使用转换器的转换器的`@Mapper` 注解上使用`uses` 属性指定用到的外部转换器；最后要在转换方法的`@Mapping` 注解上通过`qualifiedByName` 属性声明使用到的方法。要注意的是，我们要同时声明被使用的类和被使用的方法，并把这两者以列表的方式组合在一起（即：以`{a, b}`的方式组合起来）。
要注意的是，被使用的外部转换器可以是接口格式的`mapstruct` 转换器，也可以是抽象类格式的`mapstruct` 转换器，甚至可以是非`mapstruct` 转换器类的普通类，只要他能够提供转换方法，且类和方法上都被`@Named` 标注。

抽象类格式的`mapstruct` 转换器：

```Java
import org.mapstruct.Mapper;
import org.mapstruct.Mapping;
import org.mapstruct.Mappings;
import org.mapstruct.Named;
import org.mapstruct.factory.Mappers;

@Mapper
@Named("convertLover2")
public abstract class LoverConverter2 {

    public static final LoverConverter2 INSTANCE = Mappers.getMapper( LoverConverter2.class );

    @Named("po2do")
    @Mappings({
        @Mapping(source = "loName", target = "loverName"),
        @Mapping(source = "loAge", target = "loverAge")
    })
    abstract LoverDo po2do(LoverPo po);

    @Named("do2po")
    @Mappings({
        @Mapping(source = "loverName", target = "loName"),
        @Mapping(source = "loverAge", target = "loAge")
    })
    abstract LoverPo do2po(LoverDo ddo);

    String num2str(Long num){
        return num.toString()+" test";
    }

    Long str2num(String str){
        return Long.valueOf(str.split(" ")[0]);
    }

}
```

对`UserConverter` 进行修改，通过将`@Mapper` 注解的`uses` 属性及`@Mapping` 注解的`qualifiedByName`属性修改为与`@Named` 对应的名称，即可使用`LoverConverter2` 转换器。

有转换器功能的普通类：

```Java
import org.mapstruct.Named;

@Named("convertLover3")
public class LoverConverter3 {

    @Named("do2po")
    public LoverPo do2po(LoverDo ddo) {
        return new LoverPo("lay", 30, "2");
    }

    @Named("po2do")
    public LoverDo do2po(LoverPo po) {
        return new LoverDo("lay", 30, 2);
    }

}
```

与刚才相同的，把对应注解的属性进行修改后，即可使用`LoverConverter3`。这里转换器方法的逻辑完全自定。上面代码片段的内容只是为了验证。

在测验上述功能时，我遇到一个问题：对实体类的属性命名时，起始的两个字母不能是小写字母+大写字母，比如"sAge"。`mapstruct` 会将这样的属性名识别为"SAge"，从而导致找不到getter方法。

### 进阶使用

#### `@Mappings`注解

刚才书写转换方法时，是把多个`@Mapping` 注解写到同一个方法上。通过使用`@Mappings` 注解，可以把这些注解聚拢到一起。这里以其中一个转换方法举例：

```Java
@Mappings({
  @Mapping(source = "userId", target = "id"),
  @Mapping(source = "userName", target = "name"),
  @Mapping(source = "date", target = "timeStamp", qualifiedByName = "date2stamp1"),
  @Mapping(source = "preDate", target = "preTimeStamp", qualifiedByName = "date2stamp2")
})
UserPo do2po(UserDo ddo);
```

#### 省略赋值

如果不书写某对字段间的关系，且这对字段在转换器内部找不到合适的转换方法，那么就不会发生赋值。
相对的，可以在书写`@Mapping` 时指定字段并使用`ignore` 属性，让`mapstruct` 不为这对字段赋值。有些时候，字段会自动赋值，如名称相同且类别相同或名称相同类别不同但存在可用的默认转换方法。这时使用`ignore` 就可以跳过对这对字段的赋值。举例如下：
`@Mapping(source = "age1", target = "age2", ignore = true)`

#### 反向赋值

上面的代码中，我们都是为`A->B`和`B->A`的转换方法分别书写`@Mapping` 映射关系。但很多场景下，在书写完一个方向的转换逻辑后，另一个方向的逻辑就显而易见了，此时可以使用`@InheritInverseConfiguration` 注解。
在书写完一个方向的转换方法`@Mapping` 后，在另一个方法之上添加这个注解，就可以让`mapstruct` 自动生成对应的映射。
但需要注意的是，反向逻辑必须是简易可推导的。也就是说，如果正向逻辑中存在复杂逻辑，比如需要在书写`@Mapping` 注解时通过`qualifiedByName` 属性声明转换方法，那么`@InheritInverseConfiguration` 不会为反向逻辑中的该字段生成映射关系，因为他找不到对应的方法。此时需要在使用`@InheritInverseConfiguration` 注解后再额外通过`@Mapping` 注解对那些特殊映射关系进行声明。
这里先以几个不需要额外使用`@Mapping` 注解的情况举例：

* 对于属性名相同类型相同的字段：`mapstruct` 可以自动映射，因此无需额外声明。

* 对于属性名不同类型相同的字段：由于正向逻辑里已经声明了字段的对应关系，因此反向逻辑里无需额外声明。

* 对于属性名相同类型不同的字段：假设在转换器中已经定义了两种类型之间双向转换的default 方法。那么在定义正向逻辑时我们不需要使用`@Mapping` 注解进行指定，因为`mapstruct` 会自动找到对应的`default` 转换方法，而反向逻辑也是一样的，所以不需要声明。

* 对于属性名不同类型不同的字段：这种场景其实是上面的第2、3种场景的组合。由于正向逻辑里声明了字段对应关系，所以不需要考虑字段名不同的问题。由于存在`default` 方法，因此不需要考虑类型转换。综上，这种场景下不需要额外声明。

上面几个场景就是刚才提到的“可推导的”场景。相对的，如果是需要通过`qualifiedByName` 指定特定的`default` 方法，甚至是指定外部转换器的方法，那么就一定要为这个赋值逻辑书写对应的`@Mapping` 注解。

`mapstruct` 的细节繁多，这里只列举一些特性，有需要请自行了解

#### 链式法则

当存在对象嵌套对象的情况时，可以使用链式法则的方式进行赋值。
`@Mapping(source = "ddo.lover.age", target = "loverAge")
UserPo do2po(UserDo ddo);`
上面的`@Mapping` 注解可以解释为如下的java语句：`po.setLoverAge(ddo.getLover().getAge())`

#### 日期格式化

在使用`@Mapping` 注解编写字段映射时，可以通过使用`dateFormat` 属性，指定字符串到Date 的转换逻辑，从而避免书写`default` 方法。

#### 数字格式化

与上面类似的，对于字符串到数字的逻辑，可以使用`@Mapping` 注解的`numberFormat` 属性。

#### 常量

通过指定`@Mapping` 注解的`constant` 属性，可以为目标字段指定固定赋值。

#### 默认值

通过指定`@Mapping` 注解的`defaultValue` 属性，可以为目标字段指定源字段为null时的赋值。

#### 多入参

```Java
@Mappings({
  @Mapping(source = "po.name", target = "userName"),
  @Mapping(source = "loverPo.name", target = "loverName")
})
abstract UserDo multi2do(UserPo po, LoverPo loverPo);
```

定义方法时可以使用多个入参，这些入参都可以作为赋值时的数据源使用。这时，书写`@Mapping` 需要指定对应的属性名。

#### 更新赋值

`void po2do(UserPo po, @MappingTarget UserDo ddo);`
定义转换方法时，可以在入参里通过@MappingTarget 指定要赋值的对象。要注意的时，这种方式下调用方法时需要传入要被赋值的对象，转换方法不会生成新对象。因此这种方式更适合用来更新对象的值。

#### @BeforeMapping    @AfterMapping

`mapstruct`提供了`@BeforeMapping` 和`@AfterMapping` 两个注解，用于在实体转换的前后做一些额外了逻辑。但是，这两个注解只能用于抽象类格式的转换器中。

#### 自动注入

可以通过在`@Mapper` 注解里配置`componentModel` 属性，让转换器实现类成为可被自动注入的`bean` 。

---

## 参考文章

<https://www.cnblogs.com/DDgougou/p/12848625.html>
<https://stackabuse.com/guide-to-mapstruct-in-java-advanced-mapping-library/>
<https://mapstruct.org/documentation/stable/reference/html/#adding-custom-methods>
