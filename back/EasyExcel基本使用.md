# EasyExcel
## 使用场景
 - 数据导入，减轻录入工作量
 - 数据导出，统计信息归档
 - 数据传输，异构系统之间数据传输
## EasyExcel介绍
**节省内存，逐行读取** 
 - Java领域解析、生成Excel比较有名的框架有Apache poi、jxl等。但他们都存在一个严重的问题就是非常的耗内存。如果你的系统并发量不大的话可能还行，但是一旦并发上来后一定会OOM或者JVM频繁的full gc。
 - EasyExcel是阿里巴巴开源的一个excel处理框架，以使用简单、节省内存著称。EasyExcel能大大减少占用内存的主要原因是在解析Excel时没有将文件数据一次性全部加载到内存中，而是从磁盘上一行行读取数据，逐个解析。
 - EasyExcel采用一行一行的解析模式，并将一行的解析结果以观察者的模式通知处理（AnalysisEventListener）。
## EasyExcel读写操作
导入依赖

```java
  		<dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>easyexcel</artifactId>
            <version>2.1.1</version>
        </dependency>
```
### 创建实体类
```java
package com.online.edu.eduservice.entity;

import com.alibaba.excel.annotation.ExcelProperty;
import lombok.Data;
import lombok.ToString;

/**
 * @author Double strong
 * @date 2020/4/20 9:33
 */
@Data
@ToString
public class ExcelDemo {
//    设置表头
    @ExcelProperty("学生编号")
    private int sno;

    @ExcelProperty("学生姓名")
    private String sname;
}

```
### 测试写操作
```java
//    easyExcel测试写数据
    @Test
    public void testWrite(){
        List<ExcelDemo> list=new ArrayList<>();
        for (int i=0;i<10;i++)
        {
            ExcelDemo excelDemo=new ExcelDemo();
            excelDemo.setSno(i);
            excelDemo.setSname("张三"+i);
            list.add(excelDemo);
        }
        String fileName="D:\\111.xlsx";
 //传入 文件地址，写操作类对象，表单名，数据集合
        EasyExcel.write(fileName,ExcelDemo.class).sheet("表单1").doWrite(list);
    }
```
### 测试读操作
#### 创建读取操作监听器`ExcelListener` 
1. 实现 `AnalysisEventListener<ExcelDemo`>接口方法
2. `invoke`一行行的读取数据
3. `doAfterAllAnalysed`整个过程结束后执行该方法
4. `invokeHeadMap`读取表头信息
```java
package com.online.edu.eduservice.Utils;

import com.alibaba.excel.context.AnalysisContext;
import com.alibaba.excel.event.AnalysisEventListener;
import com.online.edu.eduservice.entity.ExcelDemo;
import lombok.Data;

import java.util.ArrayList;
import java.util.List;
import java.util.Map;

/**
 * @author Double strong
 * @date 2020/4/20 10:02
 * 创建EasyExcel读取监听器
 */
@Data
public class ExcelListener extends AnalysisEventListener<ExcelDemo> {

    List<ExcelDemo> list=new ArrayList<>();

//    一行行的读取excel的内容
    @Override
    public void invoke(ExcelDemo excelDemo, AnalysisContext analysisContext) {
        System.out.println("*****"+excelDemo);
        list.add(excelDemo);
    }


    @Override
    public void doAfterAllAnalysed(AnalysisContext analysisContext) {

    }
//    读取excel表头信息
    @Override
    public void invokeHeadMap(Map<Integer, String> headMap, AnalysisContext context) {
        System.out.println("表头信息"+headMap);
    }
}
```
#### 实现读操作
1. 创建已实现的读操作监听器
2. 创建excel工作蒲

```java
@Test
    public void testEasyExcelRead(){
        String fileName="D:\\111.xlsx";
        ExcelListener excelListener=new ExcelListener();
        // 这里 需要指定读用哪个class去读，然后读取第一个sheet 文件流会自动关闭
        EasyExcel.read(fileName,ExcelDemo.class,excelListener).sheet().doRead();
        List<ExcelDemo> list = excelListener.getList();
//        得到excel表中的数据
        System.out.println(list);

    }
```
## 分类业务实现
利用excel导入分类，只用传入一级分类和二级分类名称。
第一列：一级分类
第二列：二级分类
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200520115222907.png)
### 创建Excel对应的实体类
 `@ExcelProperty`("oneSubjectName")用来标记列名
```java
package com.online.edu.eduservice.entity;

import com.alibaba.excel.annotation.ExcelProperty;
import lombok.Data;

/**
 * @author Double strong
 * @date 2020/4/20 10:34
 */
@Data
public class ExcelSubjectData {

    @ExcelProperty("oneSubjectName")
    private String oneSubjectName;

    @ExcelProperty("twoSubjectName")
    private String twoSubjectName;
}
```
### 创建读取Excel监听器
在监听器`invoke`函数中写读的逻辑业务规则
1. 在监听器中创建有参构造函数用于传递`EduSubjectService`执行数据库写操作
2.  `List<ExcelSubjectData>`用于存储excel中数据
3. `invoke`用于一行行读取excel中的数据

```java
1、 判断第一行数据是否为空，为空抛出异常
2、 如果不为空，则获取第一列和第二列的数据判断是否为空或“”
3、 如果两列都不为空或“”，则进行下一步
4、 判断一级分类是否存在数据库，如果不存在则插入数据库中
5、 判断二级分类是否存在数据库，如果不存在则插入数据库中
6、 最后返回list--excel中的数据
```

```java
package com.online.edu.eduservice.Utils;
import com.alibaba.excel.context.AnalysisContext;
import com.alibaba.excel.event.AnalysisEventListener;
import com.guli.common.exceptionhandler.GuliException;
import com.online.edu.eduservice.entity.EduSubject;
import com.online.edu.eduservice.entity.ExcelSubjectData;
import com.online.edu.eduservice.service.EduSubjectService;
import lombok.Data;
import java.util.ArrayList;
import java.util.List;

/**
 * @author Double strong
 * @date 2020/4/20 10:39
 */
@Data
public class SubjectExcelListener extends AnalysisEventListener<ExcelSubjectData> {

//    创建有参构造，用于传递subjectService，操作数据库
    public EduSubjectService subjectService;
    List<ExcelSubjectData> list=new ArrayList<>();

    public SubjectExcelListener() {
    }
    public SubjectExcelListener(EduSubjectService eduSubjectService) {
        this.subjectService=eduSubjectService;
    }

    /***
     *
     * 一行行读取excel中的内容
     * @param excelSubjectData
     * @param analysisContext
     */
    @Override
    public void invoke(ExcelSubjectData excelSubjectData, AnalysisContext analysisContext) {
        if(excelSubjectData==null)
        {
            throw new GuliException(20001,"添加失败");
        }
        ExcelSubjectData listSubject=new ExcelSubjectData();
//        不为空，读取每一行来操作
//        判断一级分类是否存在
        System.out.println("0000000000000");
        if(excelSubjectData.getOneSubjectName()!=null&&!excelSubjectData.getOneSubjectName().equals(""))
        {
            System.out.println("11111111111111");
            String oneSubjectName = excelSubjectData.getOneSubjectName();
            System.out.println(oneSubjectName);
            Integer integer1 = subjectService.judgeTitle(oneSubjectName);
            System.out.println(integer1);
            if (integer1 == 0) {
//            该一级分类数据库不存在，则插入
                EduSubject eduSubject = new EduSubject();
                eduSubject.setTitle(excelSubjectData.getOneSubjectName());
                eduSubject.setParentId("0");
                eduSubject.setSort(0);
                subjectService.save(eduSubject);
            }
            listSubject.setOneSubjectName(excelSubjectData.getOneSubjectName());
        }
        if(excelSubjectData.getTwoSubjectName()!=null&&!excelSubjectData.getTwoSubjectName().equals(""))
        {
//            判断二级分类是否 存在
            Integer integer2 = subjectService.judgeTitle(excelSubjectData.getTwoSubjectName());
            if(integer2==0)
            {
//                该二级标签不存在，则插入到数据库中
//                获取一级标签对应的id，即为二级标签的父id
                String parent_id = subjectService.getIdByTitle(excelSubjectData.getOneSubjectName());
                EduSubject eduSubject = new EduSubject();
                eduSubject.setTitle(excelSubjectData.getTwoSubjectName());
                eduSubject.setParentId(parent_id);
                eduSubject.setSort(0);
                subjectService.save(eduSubject);
            }
            listSubject.setTwoSubjectName(excelSubjectData.getTwoSubjectName());
        }
        list.add(listSubject);

    }

    @Override
    public void doAfterAllAnalysed(AnalysisContext analysisContext) {

    }
}
```
### 编写业务逻辑
过程详解

```java
1、 获取文件输入流
2、 创建监听器传入service用于执行数据库写操作
3、 执行读操作 EasyExcel.read传入输入流、实体类、监听器等信息
4、 返回读取文件的list集合
```
对应`ServiceImpl`
```java
@Override
    public List<ExcelSubjectData> importSubjectByEasyExcel(MultipartFile file, EduSubjectService eduSubjectService) {

        try {
            //        1.获取文件输入流
            InputStream inputStream = file.getInputStream();
//            创建监听器
            SubjectExcelListener subjectExcelListener=new SubjectExcelListener(eduSubjectService);
//            执行读操作
            EasyExcel.read(inputStream, ExcelSubjectData.class,subjectExcelListener).sheet("Sheet1").doRead();
            List<ExcelSubjectData> list = subjectExcelListener.getList();
            return list;
        } catch (IOException e) {
            e.printStackTrace();
            throw new GuliException(20002,"添加课程分类失败");
        }
    }
```
对应`Controller`

```java
 /**
     * 根据EasyExcel来上传excel
     * @param file
     * @return
     */
    @PostMapping("/importExcelSubjectByEasyExcel")
    public R importExcelSubjectByEasyExcel(@RequestParam("file")MultipartFile file){
        List<ExcelSubjectData> strings = eduSubjectService.importSubjectByEasyExcel(file,eduSubjectService);
        return R.ok().data("message",strings);
    }
```


