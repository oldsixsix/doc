# Mybatis-plus
@[toc]
## 快速开始
 1. 插入依赖
 2. 安装lombok插件
 3. 配置
	1. 数据库相关配置
	2. 启动类上配置`@MapperScan`注解
	3. yml文件上配置`mapper-locations: classpath:xml/*Mapper.xml`
	4. `typeAliasesPackage`：MyBaits 别名包扫描路径，通过该属性可以给包中的类注册别名，注册后在 Mapper 对应的 XML 文件中可以直接使用类名，而不用使用全限定的类名(即 XML 中调用的时候不用包含包名)
	5. `typeHandlersPackage`通常用于自定义类型转换。
## 代码生成器
1. 添加代码生成器依赖
```
代码生成器
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-generator</artifactId>
    <version>latest-version</version>
</dependency>
模板引擎
<dependency>
    <groupId>org.apache.velocity</groupId>
    <artifactId>velocity-engine-core</artifactId>
    <version>latest-velocity-version</version>
</dependency>
```
2. 代码生成器模板
````
import com.baomidou.mybatisplus.annotation.DbType;
import com.baomidou.mybatisplus.annotation.IdType;
import com.baomidou.mybatisplus.generator.AutoGenerator;
import com.baomidou.mybatisplus.generator.config.DataSourceConfig;
import com.baomidou.mybatisplus.generator.config.GlobalConfig;
import com.baomidou.mybatisplus.generator.config.PackageConfig;
import com.baomidou.mybatisplus.generator.config.StrategyConfig;
import com.baomidou.mybatisplus.generator.config.rules.DateType;
import com.baomidou.mybatisplus.generator.config.rules.NamingStrategy;
import org.junit.Test;

/**
 * @author doubleStrong
 * @since 2020/05/13
 */
public class CodeGenerator {

    @Test
    public void run() {

        // 1、创建代码生成器
        AutoGenerator mpg = new AutoGenerator();

        // 2、全局配置
        GlobalConfig gc = new GlobalConfig();
        String projectPath = System.getProperty("user.dir");
        gc.setOutputDir(projectPath + "/src/main/java");

        gc.setAuthor("doublestrong");
        gc.setOpen(false); //生成后是否打开资源管理器
        gc.setFileOverride(false); //重新生成时文件是否覆盖
        gc.setServiceName("%sService");	//去掉Service接口的首字母I
        gc.setIdType(IdType.ID_WORKER_STR); //主键策略  全局唯一ID
        gc.setDateType(DateType.ONLY_DATE);//定义生成的实体类中日期类型
        gc.setSwagger2(true);//开启Swagger2模式

        mpg.setGlobalConfig(gc);

        // 3、数据源配置
        DataSourceConfig dsc = new DataSourceConfig();
        dsc.setUrl("jdbc:mysql://localhost:3306/mybatisdemo");
        dsc.setDriverName("com.mysql.jdbc.Driver");
        dsc.setUsername("root");
        dsc.setPassword("123456");
        dsc.setDbType(DbType.MYSQL);
        mpg.setDataSource(dsc);

        // 4、包配置
        PackageConfig pc = new PackageConfig();
        pc.setModuleName("eduservice"); //模块名
        pc.setParent("com.online.edu");

        pc.setController("controller");
        pc.setEntity("entity");
        pc.setService("service");
        pc.setMapper("mapper");
        mpg.setPackageInfo(pc);

        // 5、策略配置
        StrategyConfig strategy = new StrategyConfig();
//        其他都不需要变，只需要修改表名，
        strategy.setInclude("edu_video","edu_course_description");
        strategy.setNaming(NamingStrategy.underline_to_camel);//数据库表映射到实体的命名策略 驼峰命令映射方式
        strategy.setTablePrefix(pc.getModuleName() + "_"); //生成实体时去掉表前缀
        strategy.setColumnNaming(NamingStrategy.underline_to_camel);//数据库表字段映射到实体的命名策略  驼峰命令映射方式
        strategy.setEntityLombokModel(true); // lombok 模型 @Accessors(chain = true) setter链式操作
        strategy.setRestControllerStyle(true); //restful api风格控制器
        strategy.setControllerMappingHyphenStyle(true); //url中驼峰转连字符

        mpg.setStrategy(strategy);

        // 6、执行
        mpg.execute();
    }
}

````

## 逻辑删除
效果: 使用mp自带方法删除和查找都会附带逻辑删除功能 (自己写的xml不会)
````
example
删除 update user set deleted=1 where id =1 and deleted=0
查找 select * from user where deleted=0
````
1. 添加配置类
2. yml配置逻辑删除默认值
3. 实体类逻辑删除字段上添加`@TableLogic`
```
mybatis-plus:
  global-config:
    db-config:
      logic-delete-field: flag  #全局逻辑删除字段值 3.3.0开始支持，详情看下面。
      logic-delete-value: 1 # 逻辑已删除值(默认为 1)
      logic-not-delete-value: 0 # 逻辑未删除值(默认为 0)
 全局逻辑删除字段采用就近原则，先匹配注解，如果没有再找全局
但如果实体类上有 @TableLogic 则以实体上的为准，忽略全局。 即先查找注解再查找全局，都没有则此表没有逻辑删除。
```

## 字段类型处理器
用于 JavaType 与 JdbcType 之间的转换，用于 PreparedStatement 设置参数值和从 ResultSet 或 CallableStatement 中取出一个值，
PreparedStatement从JavaType 获取值并设置到数据库中
ResultSet 从数据库中获取值并设置到JavaType 中
通过这两种方式实现 JavaType 与 JdbcType 之间的转换，通常用于自定义类型处理器
我们在写mapper映射器的配置文件时，不经意间已经用到类型转换，不过是mybatis帮我们完成的。

```
<update id="update" parameterType="twm.mybatisdemo.pojo.User">
    update user set
    username=#{username},password=#{password},address=#{address}  where id=#{id}
</update>
```
像上面例子，只需要向update方法传入一个user对象，mybatis利用反射拆开user对象，然后根据对象中的字段在预处理语句（PreparedStatement）中设置参数，并且根据字段的类型，使用setXXX()方式设置相应的值。XXX可以是Integer，String，Date等Java类型。
同理，在从结果集（ResultSet）中取出一个值时，将使用rs.getInt、rs.getString、rs.getTimeStamp等方法将数据转换为Java对象。

那么问题来了，javaType和jdbcType的转换关系由谁来定呢？这就是类型处理器（type handlers）的功能所在。
比如java.lang.String转成JDBC.Varchar，java.lang.Integer转成JDBC.int。MyBatis使用内建的类型处理器能转换所有的基本数据类型、基本类型的包装类型、byte[] 、java.util.Date、java.sql.Date、java,sql.Time、java.sql.Timestamp、java枚举类型等。
不过对于自定义的类型怎么办呢？
需要创建一个自定义的类型处理器(TypeHandler)了
1、创建类型处理器：
```
class AddressTypeHandler extends BaseTypeHandler<Address>{
```
通过ide自动生成代码，可以看到父类BaseTypeHandler有四个方法需要我们实现，包括三个get方法，一个set方法。

2、实现get方法
三个get方法都是用于将数据库获得的记录集里的address字段转成java Address类型的对象。
所以我们首先给Address类增加一个构造方法，用途：根据字符串生成一个实例对象。
//假设我们存储在db中的字符串是以","号分隔省市关系的
```
public Address(String address) {  
    if (address != null) {  
        String[] segments = address.split(",");  
        if (segments.length > 1) {  
            this.province = segments[0];
            this.city = segments[1];  
        } 
        else if (segments.length > 0) {  
            this.city = segments[0];  
        }  
    }  
}
```
然后实现AddressTypeHandler类中的三个get方法：（有强迫症，不喜欢看arg0，arg1。所以顺便也改一下）
````
@Override
public Address getNullableResult(ResultSet rSet, String columnName)
        throws SQLException {
    return new Address(rSet.getString(columnName));
}

@Override
public Address getNullableResult(ResultSet rSet, int columnIndex)
        throws SQLException {
    return new Address(rSet.getString(columnIndex));
}

@Override
public Address getNullableResult(CallableStatement cStatement, int columnIndex)
        throws SQLException {
    return new Address(cStatement.getString(columnIndex));
}
````

3、实现set方法

set方法是用来将java类型转成数据库存储的类型。
这里我们先实现一下Address类的toString()方法（如果toString另有它用，那么就另外用一个方法名）
```
@Override
public String toString() {
    return this.province + "," + this.city;
}
```
然后实现AddressTypeHandler类中的setNonNullParameter
```
@Override
public void setNonNullParameter(PreparedStatement pStatement, int index,
        Address address, JdbcType jdbcType) throws SQLException {
    pStatement.setString(index, address.toString());
}
```
4、在myBatis配置文件中注册该类
```
<!-- 注册自定义类型处理器 -->
<typeHandlers>
    <typeHandler handler="twm.mybatisdemo.type.AddressTypeHandler" />
</typeHandlers>
```
这里有个小插曲，我当时在配置文件最后插入了这段，结果提示错误：
````
The content of element type “configuration” must match
“(properties?,settings?,typeAliases?,typeHandlers?,objectFactory?,objectWrapperFactory?,reflectorFactory?,plugins?,environments?,databaseIdProvider?,mappers?)”.
````
原来在configuration里的标签，必须按照提示的这个顺序写，不然就报错。
因此typeHandlers必须放在environments前面，typeAliases后面。
## 自动填充功能
实现元对象处理器接口：`com.baomidou.mybatisplus.core.handlers.MetaObjectHandler`
注解填充字段 `@TableField(.. fill = FieldFill.INSERT)` 生成器策略部分也可以配置！
例如用于自动填充课程的创建时间`gmtCreate`和更新时间`gmtModified`字段
1. 实现自定义实现类`MyMetaObjectHandler`实现`MetaObjectHandler`接口
2. 自定义填充规则
3. 添加填充注解到字段上
```
import com.baomidou.mybatisplus.core.handlers.MetaObjectHandler;
import org.apache.ibatis.reflection.MetaObject;
import org.springframework.stereotype.Component;
import java.util.Date;

/**
 * @author Double strong
 * @date 2020/3/4 12:35
 * 自动填充功能
 */
@Component
public class MyObjectHandler implements MetaObjectHandler {
// 插入字段时的填充规则
    @Override
    public void insertFill(MetaObject metaObject) {
//        在这里添加填充属性
       this.setFieldValByName("gmtCreate",new Date(),metaObject);
       this.setFieldValByName("gmtModified",new Date(),metaObject);
    }
// 更新字段的时候填充规则
    @Override
    public void updateFill(MetaObject metaObject) {
        
        this.setFieldValByName("gmtModified",new Date(),metaObject);
    }
}
//字段上添加注解
  @ApiModelProperty(value = "创建时间")
    @TableField(fill = FieldFill.INSERT)
    private Date gmtCreate;

    @ApiModelProperty(value = "更新时间")
    @TableField(fill = FieldFill.INSERT_UPDATE)
    private Date gmtModified;
```
## 分页插件
1. 添加配置类
2. 使用分页功能
```
//1. 添加配置类
//Spring boot方式
@EnableTransactionManagement
@Configuration
@MapperScan("com.baomidou.cloud.service.*.mapper*")
public class MybatisPlusConfig {

    @Bean
    public PaginationInterceptor paginationInterceptor() {
        PaginationInterceptor paginationInterceptor = new PaginationInterceptor();
        // 设置请求的页面大于最大页后操作， true调回到首页，false 继续请求  默认false
        // paginationInterceptor.setOverflow(false);
        // 设置最大单页限制数量，默认 500 条，-1 不受限制
        // paginationInterceptor.setLimit(500);
        // 开启 count 的 join 优化,只针对部分 left join
        paginationInterceptor.setCountSqlParser(new JsqlParserCountOptimize(true));
        return paginationInterceptor;
    }
}
```
```
//2.使用分页插件细节
   @PostMapping("/getCourseInfoByCondition/{page}/{limit}")
    public R getCourseByCondition(@PathVariable("page") long page, @PathVariable("limit") long limit, @RequestBody QueryCourse queryCourse){

//        创建一个分页对象 (page,limit) 传入当前页以及每页最大数据数
        Page<EduCourse> pageConditon=new Page<>(page,limit);
        eduCourseService.getCourseCondition(pageConditon,queryCourse);
//        pageConditon位分页查询后数据封装的对象
        long total = pageConditon.getTotal();
//        records为分页数据
        List<EduCourse> records = pageConditon.getRecords();
//        根据EduCourse查出CourseListTable 填写表单
        List<CourseListTable> courseListTables= eduCourseService.getCourseListTable(records);
        return R.ok().data("total",total).data("records",courseListTables);
    }
  /**
     * 分页查询课程列表
     * @param pageConditon
     * @param queryCourse
     */
    @Override
    public void getCourseCondition(Page<EduCourse> pageConditon, QueryCourse queryCourse) {
//        查询条件为空
        if(queryCourse==null)
        {
            baseMapper.selectPage(pageConditon,null);
            return;
        }
//        查询条件不为空，则进行动态拼接
        QueryWrapper<EduCourse> eduCourseQueryWrapper=new QueryWrapper<>();

         String courseName=queryCourse.getCourseName();
         String teacherName=queryCourse.getTeacherName();
         String subjectId=queryCourse.getSubjectId();

         if(!StringUtils.isEmpty(courseName))
         {
             eduCourseQueryWrapper.like("title",courseName);
         }
         if(!StringUtils.isEmpty(teacherName))
         {

//             如果teachername不为空，则需要在teacherSevice中查找
             List<String> teacherId = eduTeacherService.getTeacherId(teacherName);
             eduCourseQueryWrapper.in("teacher_id",teacherId);
         }
         if(!StringUtils.isEmpty(subjectId))
         {
             eduCourseQueryWrapper.eq("subject_id",subjectId);
         }
         baseMapper.selectPage(pageConditon,eduCourseQueryWrapper);

    }
```
##
