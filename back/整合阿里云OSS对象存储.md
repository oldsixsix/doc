# 整合阿里云OSS对象存储
## 开通阿里云OSS
## 创建阿里云存储微服务
1. 在service模块下创建子模块service-oss
2. 配置pom.xml

```java
<dependencies>
    <!-- 阿里云oss依赖 -->
    <dependency>
        <groupId>com.aliyun.oss</groupId>
        <artifactId>aliyun-sdk-oss</artifactId>
    </dependency>
    <!-- 日期工具栏依赖 -->
    <dependency>
        <groupId>joda-time</groupId>
        <artifactId>joda-time</artifactId>
    </dependency>
</dependencies>
```

3. 配置阿里云服务器相关信息

```yaml
aliyun:
  oss:
    file:
      endPoint: oss-cn-beijing.aliyuncs.com
      keyId: ********
      keySecret: **********
      bucketName: *****
      fileHost: avatar
      coverHost: cover
```

4. 在加载service公共服务的时候会加载数据驱动相关依赖，需要排除
spring boot 会默认加载org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration这个类，
而DataSourceAutoConfiguration类使用了`@Configuration`注解向spring注入了dataSource bean，又因为项目（oss模块）中并没有关于dataSource相关的配置信息，所以当spring创建dataSource bean时因缺少相关的信息就会报错。
在启动类上加排除数据驱动配置
```java
@SpringBootApplication(exclude = DataSourceAutoConfiguration.class)
```

## 实现文件上传
### 创建常量读取工具类`ConstantPropertiesUtil.java`
使用@value读取applicaiton.yml配置文件中阿里云连接信息
用spring的 `InitializingBean` 的 `afterPropertiesSet` 方法来初始化配置信息，这个方法将在所有的属性被初始化后调用。
即初始化（读取配置文件信息）调用`afterPropertiesSet`将读取的信息赋值给静态变量便于后面使用
```java
package com.online.edu.eduservice.Utils;

import org.springframework.beans.factory.InitializingBean;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

/**
 * @author Double strong
 * @date 2020/3/5 17:33
 * implements InitializingBean
 * 在服务器启动的时候读取配置文件的内容
 * @value来读取yml配置文件定义的一些常量
 * @Component 把组件加入容器中才能获取配置文件的值
 */
@Component

public class ConstantPropertiesUtil implements InitializingBean {

    /*  aliyun:
      oss:
      file:
      endpoint:oss-cn-beijing.aliyuncs.com
      keyid:
      keysecret:
      bucketname:*/
    @Value("${aliyun.oss.file.endPoint}")
    private String endPoint;

    @Value("${aliyun.oss.file.keyId}")
    private String keyId;

    @Value("${aliyun.oss.file.keySecret}")
    private String keySecret;

    @Value("${aliyun.oss.file.bucketName}")
    private String bucketName;
	
	//读取图片分类
    @Value("${aliyun.oss.file.fileHost}")
    private String fileHost;
// 读取封面
    @Value("${aliyun.oss.file.coverHost}")
    private String coverHost;

    public static String END_POINT;
    public static String ACCESS_KEY_ID;
    public static String ACCESS_KEY_SECRET;
    public static String BUCKET_NAME;
    public static String FILE_HOST ;
    public static String COVER_HOST ;
    @Override
    public void afterPropertiesSet() throws Exception {
        END_POINT = endPoint;
        ACCESS_KEY_ID = keyId;
        ACCESS_KEY_SECRET = keySecret;
        BUCKET_NAME = bucketName;
        FILE_HOST = fileHost;
        COVER_HOST=coverHost;

    }
}
```

### 创建上传工具类
1. 读取OSS上传的四个变量`endpoint` 、`accessKeyId` 、`accessKeySecret`、`hostName`
2. 传入变量创建`OSSClient`实例
3. 创建你的`BucketName`
4. 自定义文件名称并`获取文件上传地址`
```java
1、构建日期路径  
    String time = new DateTime().toString("yyyy/MM/dd");
2、 构建随机码
  String uuid = UUID.randomUUID().toString();
3、 获取文件的原始文件名
    String originalFilename = file.getOriginalFilename();
4、 拼接文件名 阿里云通过/可以进行文件夹分层
      String newFile=hostName+"/"+time+"/"+uuid+"-"+originalFilename;
5、获取文件上传地址 http:// 分区名 +节点名+ 文件名
   upLoadUrl="http://"+BucketName+"."+endpoint+"/"+newFile;
阿里云通过 / 可以进行文件夹分层
```

5. 完整代码如下 

```java
package com.online.edu.eduservice.Utils;
import com.aliyun.oss.OSS;
import com.aliyun.oss.OSSClientBuilder;
import org.joda.time.DateTime;
import org.springframework.web.multipart.MultipartFile;
import java.io.InputStream;
import java.util.UUID;
/**
 * @author Double strong
 * @date 2020/3/6 12:13
 */
public class UploadUtils {

    public static String UploadFile(MultipartFile file,String host)
    {
        String upLoadUrl;

//1. 初始化OSS变量的四个常量
        try {
            InputStream inputStream = file.getInputStream();
            // Endpoint以杭州为例，其它Region请按实际情况填写。
//            String endpoint = "oss-cn-beijing.aliyuncs.com";
            String endpoint = ConstantPropertiesUtil.END_POINT;
// 云账号AccessKey有所有API访问权限，建议遵循阿里云安全最佳实践，创建并使用RAM子账号进行API访问或日常运维，请登录 https://ram.console.aliyun.com 创建。
//            String accessKeyId = "";
            String accessKeyId=ConstantPropertiesUtil.ACCESS_KEY_ID;

//            String accessKeySecret = "";
            String accessKeySecret=ConstantPropertiesUtil.ACCESS_KEY_SECRET;

//2. 创建OSSClient实例
            OSS ossClient = new OSSClientBuilder().build(endpoint, accessKeyId, accessKeySecret);
            String  BucketName="edutest11"; //分区名称
            String hostName;  //文件类型，是读取的excel还是图片
            if(host.equals("avatar"))
            {
                 hostName=ConstantPropertiesUtil.FILE_HOST;
            }
            else {
                hostName=ConstantPropertiesUtil.COVER_HOST;
            }
// 上传文件流。
//                自定义文件名称
//              1.构建日期路径
//            这里时间格式一定要确定，要不然每次输出格式都有区别
            String time = new DateTime().toString("yyyy/MM/dd");
//                2.构建随机码，防止上传文件名出现重复
            String uuid = UUID.randomUUID().toString();
//                3.获取文件的原始文件名
            String originalFilename = file.getOriginalFilename();
//                4.拼接文件名
            String newFile=hostName+"/"+time+"/"+uuid+"-"+originalFilename;

//                获取上传文件的URL地址
            upLoadUrl="http://"+BucketName+"."+endpoint+"/"+newFile;
//
            ossClient.putObject(BucketName, newFile, inputStream);
//                edutest11.oss-cn-beijing.aliyuncs.com
//                String path="http://"+BucketName+"."+"oss-cn-beijing.aliyuncs.com"+"/"+originalFilename;
// 关闭OSSClient。
            return upLoadUrl;
        } catch (Exception e) {
            e.printStackTrace();
            return "error";
        }
    }
}
```

### 完成讲师头像后端上传
上传之后返回上传图片的url方便回显
图片url是使用规则进行拼接得到的，也可以在前端进行拼接得到

```java
package com.online.edu.eduservice.controller;

import com.aliyun.oss.OSS;
import com.aliyun.oss.OSSClientBuilder;
import com.guli.common.constants.R;
import com.online.edu.eduservice.Utils.UploadUtils;
import org.apache.http.entity.ContentType;
import org.joda.time.DateTime;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.multipart.MultipartFile;

import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.IOException;
import java.io.InputStream;
import java.util.UUID;

/**
 * @author Double strong
 * @date 2020/3/5 15:31
 */
@RestController
@RequestMapping(value="/eduservice/oss")
@CrossOrigin
public class FileUploadController {


//    上传讲师头像的方法
    /**
     * 上传功能
     * 1.
     */
    @PostMapping("/upload")
    public R  UploadTeacher(@RequestParam("file") MultipartFile file,@RequestParam("host") String host) throws FileNotFoundException {
        //1.获取到上传的文件MultipartFile file
//        2.获取上传文件名称
//        3.获取文件的输入流
//
        String result = UploadUtils.UploadFile(file,host);
        if(!result.equals("error"))
        {
            return R.ok().data("urlUpload",result);
        }
        else {
            return R.error();
        }
    }
}
```





