---
title:  "SpringBoot集成MinIO文件存储功能"
categories: MinIO
tags: MinIO
author: LIUREN
---

# SpringBoot集成MinIO文件存储功能

> 文件服务器是一个应用必要的组件之一。最早我搞过**FTP**，然后又用过**FastDFS**，接私活的时候我用**MongoDB**也凑合凑合。现如今时代不同了，开始流行起了**OSS**。
>
> 协议：CC BY-SA 4.0 https://creativecommons.org/licenses/by-sa/4.0/  
>
> 版权声明：本文为原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。



## 什么是OSS

全称为**Object Storage Service**，也叫**对象存储服务**，是一种解决和处理离散单元的方法，可提供基于分布式系统之上的对象形式的数据存储服务，具有可拓展、可管理、低成本等特点，支持中心和边缘存储，能够实现存储需求的弹性伸缩，主要应用于海量数据管理的各类场景。

这概念真是够难以理解的。简单说点我知道的吧，平常我们的文件地址都是 `/User/felord/video/xxx.mp4`的目录树结构，系统先要找到`User`，然后一级一级往下找一直到目标为止，这是一种结构化的存储方式。对象存储就不一样了，所有的文件都放在一个特定的池子里，只不过文件的携带有它自己的元信息，通过元信息去检索文件。这里举一个形象的例子：

```json
{"oss":[
    {"file":"xxxxx","meta":{"id":"1111"},"type":""},
    {"content":"xxxxx","meta":{"id":"1211"},"type":"","created":"","name":""}, 
]}
```

上图的`oss`就是一个对象存储，它里面存了携带信息不一样、甚至结构都不一样的东西，我们可以根据其元信息`meta`检索它们。**OSS**具有以下特点：

- 效率更高。不受复杂目录系统对性能的影响。
- 可扩展性更强。分布式架构，更便于进行水平扩展，从而容纳进任意大规模的数据。
- 可用性更强。数据一般都会有多个位于不同机器的复制，确保数据不丢失。
- 平台无关，可以通过**Restful**接口进行操作对象。

> **OSS**通常被用来存储图片、音视频等文件，以及对这些文件的处理。



## 哪些OSS可以使用

一种是**Ceph**,一个分布式存储系统，高可用，高扩展性

另一种是**Minio**，用**Golang**写的。我目前还没发现有什么坑，文档居然还有中文文档！

![](https://www.codepeople.cn/imges/00x39.png)

## Linux 下安装MinIO

windows下安装过于简单我们来研究Linux下的安装，不过同样简单

```shell
# 下载
wget https://dl.min.io/server/minio/release/linux-amd64/minio

# 赋权
chmod +x minio

# 设置用户名
export MINIO_ACCESS_KEY=minioadmin

# 设置密码
export MINIO_SECRET_KEY=minioadmin

# 启动  /home/data是自己定义的文件目录，进入minio的下载目录
./minio server /home/data

# 静默启动 
nohup ./minio server /home/data >/home/data/minio.log 2>&1 &
```

运行 ./minio server /home/data 看下一下画面即表示启动成功

![](https://www.codepeople.cn/imges/00x40.png)

访问 http://172.28.43.29:9000/minio/login接口看到页面

![](https://www.codepeople.cn/imges/00x41.png)

## SpringBoot 集成MinIO

## 引入jar包

```xml
  <dependency>
  	  <groupId>io.minio</groupId>
      <artifactId>minio</artifactId>
      <version>7.0.2</version>
  </dependency>
```

## 配置

```yml
minio:
  endpoint: http://web.codepeople.cn/
  port: 9000
  accessKey: minioadmin
  secretKey: minioadmin
  secure: false
  bucketName: "hkeducation"
  configDir: "/home/data"
```

# 

```java
package com.hope.minio.config;

import io.minio.MinioClient;
import io.minio.errors.InvalidEndpointException;
import io.minio.errors.InvalidPortException;
import io.swagger.annotations.ApiModelProperty;
import lombok.Data;
import org.checkerframework.checker.units.qual.A;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.stereotype.Component;


@Data
@Component
@ConfigurationProperties(prefix = "minio")
public class MinioConfig {

    @ApiModelProperty("endPoint是一个URL，域名，IPv4或者IPv6地址")
    private String endpoint;

    @ApiModelProperty("TCP/IP端口号")
    private int port;

    @ApiModelProperty("accessKey类似于用户ID，用于唯一标识你的账户")
    private String accessKey;

    @ApiModelProperty("secretKey是你账户的密码")
    private String secretKey;

    @ApiModelProperty("如果是true，则用的是https而不是http,默认值是true")
    private Boolean secure;

    @ApiModelProperty("默认存储桶")
    private String bucketName;

    @ApiModelProperty("配置目录")
    private String configDir;

    @Bean
    public MinioClient getMinioClient() throws InvalidEndpointException, InvalidPortException {
        MinioClient minioClient = new MinioClient(endpoint, port, accessKey, secretKey,secure);
        return minioClient;
    }
}
```

```java
package com.hope.minio.utils;

import java.io.InputStream;
import java.nio.charset.StandardCharsets;
import java.util.ArrayList;
import java.util.List;

import javax.servlet.ServletOutputStream;
import javax.servlet.http.HttpServletResponse;

import org.apache.commons.lang3.StringUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;
import org.springframework.web.multipart.MultipartFile;

import io.minio.MinioClient;
import io.minio.ObjectStat;
import io.minio.PutObjectOptions;
import io.minio.Result;
import io.minio.errors.ErrorResponseException;
import io.minio.errors.InvalidExpiresRangeException;
import io.minio.messages.Bucket;
import io.minio.messages.DeleteError;
import io.minio.messages.Item;
import lombok.SneakyThrows;

/**
 * @PackageName: com.hope.minio.utils
 * @ClassName: MinioUtil
 * @Author Hope
 * @Date 2020/7/27 11:43
 * @Description: MinioUtil
 */
@Component
public class MinioUtil {

    @Autowired
    private MinioClient minioClient;

    private static final int DEFAULT_EXPIRY_TIME = 7 * 24 * 3600;

    /**
     * 检查存储桶是否存在
     *
     * @param bucketName 存储桶名称
     * @return
     */
    @SneakyThrows
    public boolean bucketExists(String bucketName) {
        boolean flag = false;
        flag = minioClient.bucketExists(bucketName);
        if (flag) {
            return true;
        }
        return false;
    }

    /**
     * 创建存储桶
     *
     * @param bucketName 存储桶名称
     */
    @SneakyThrows
    public boolean makeBucket(String bucketName) {
        boolean flag = bucketExists(bucketName);
        if (!flag) {
            minioClient.makeBucket(bucketName);
            return true;
        } else {
            return false;
        }
    }

    /**
     * 列出所有存储桶名称
     *
     * @return
     */
    @SneakyThrows
    public List<String> listBucketNames() {
        List<Bucket> bucketList = listBuckets();
        List<String> bucketListName = new ArrayList<>();
        for (Bucket bucket : bucketList) {
            bucketListName.add(bucket.name());
        }
        return bucketListName;
    }

    /**
     * 列出所有存储桶
     *
     * @return
     */
    @SneakyThrows
    public List<Bucket> listBuckets() {
        return minioClient.listBuckets();
    }

    /**
     * 删除存储桶
     *
     * @param bucketName 存储桶名称
     * @return
     */
    @SneakyThrows
    public boolean removeBucket(String bucketName) {
        boolean flag = bucketExists(bucketName);
        if (flag) {
            Iterable<Result<Item>> myObjects = listObjects(bucketName);
            for (Result<Item> result : myObjects) {
                Item item = result.get();
                // 有对象文件，则删除失败
                if (item.size() > 0) {
                    return false;
                }
            }
            // 删除存储桶，注意，只有存储桶为空时才能删除成功。
            minioClient.removeBucket(bucketName);
            flag = bucketExists(bucketName);
            if (!flag) {
                return true;
            }

        }
        return false;
    }

    /**
     * 列出存储桶中的所有对象名称
     *
     * @param bucketName 存储桶名称
     * @return
     */
    @SneakyThrows
    public List<String> listObjectNames(String bucketName) {
        List<String> listObjectNames = new ArrayList<>();
        boolean flag = bucketExists(bucketName);
        if (flag) {
            Iterable<Result<Item>> myObjects = listObjects(bucketName);
            for (Result<Item> result : myObjects) {
                Item item = result.get();
                listObjectNames.add(item.objectName());
            }
        }
        return listObjectNames;
    }

    /**
     * 列出存储桶中的所有对象
     *
     * @param bucketName 存储桶名称
     * @return
     */
    @SneakyThrows
    public Iterable<Result<Item>> listObjects(String bucketName) {
        boolean flag = bucketExists(bucketName);
        if (flag) {
            return minioClient.listObjects(bucketName);
        }
        return null;
    }

    /**
     * 通过文件上传到对象
     *
     * @param bucketName 存储桶名称
     * @param objectName 存储桶里的对象名称
     * @param fileName   File name
     * @return
     */
    @SneakyThrows
    public boolean putObject(String bucketName, String objectName, String fileName) {
        boolean flag = bucketExists(bucketName);
        if (flag) {
            minioClient.putObject(bucketName, objectName, fileName, null);
            ObjectStat statObject = statObject(bucketName, objectName);
            if (statObject != null && statObject.length() > 0) {
                return true;
            }
        }
        return false;

    }

    /**
     * 文件上传
     *
     * @param bucketName
     * @param multipartFile
     */
    @SneakyThrows
    public void putObject(String bucketName, MultipartFile multipartFile, String filename) {
        PutObjectOptions putObjectOptions = new PutObjectOptions(multipartFile.getSize(), PutObjectOptions.MIN_MULTIPART_SIZE);
        putObjectOptions.setContentType(multipartFile.getContentType());
        minioClient.putObject(bucketName, filename, multipartFile.getInputStream(), putObjectOptions);
    }


    /**
     * 通过InputStream上传对象
     *
     * @param bucketName 存储桶名称
     * @param objectName 存储桶里的对象名称
     * @param stream     要上传的流
     * @param stream     要上传的文件类型 MimeTypeUtils.IMAGE_JPEG_VALUE
     * @return
     */
    @SneakyThrows
    public boolean putObject(String bucketName, String objectName, InputStream stream,String contentType) {
        boolean flag = bucketExists(bucketName);
        if (flag) {
        	PutObjectOptions putObjectOptions = new PutObjectOptions(stream.available(), -1);
        	/**
        	 * 开启公共类功能设置setContentType
        	 */
        	if (StringUtils.isNotBlank(contentType)) {
        		putObjectOptions.setContentType(contentType);
			}
            minioClient.putObject(bucketName, objectName, stream, putObjectOptions);
            ObjectStat statObject = statObject(bucketName, objectName);
            if (statObject != null && statObject.length() > 0) {
                return true;
            }
        }
        return false;
    }
    
	/**
     * 以流的形式获取一个文件对象
     *
     * @param bucketName 存储桶名称
     * @param objectName 存储桶里的对象名称
     * @return
     */
    @SneakyThrows
    public InputStream getObject(String bucketName, String objectName) {
        boolean flag = bucketExists(bucketName);
        if (flag) {
            ObjectStat statObject = statObject(bucketName, objectName);
            if (statObject != null && statObject.length() > 0) {
                InputStream stream = minioClient.getObject(bucketName, objectName);
                return stream;
            }
        }
        return null;
    }

    /**
     * 以流的形式获取一个文件对象（断点下载）
     *
     * @param bucketName 存储桶名称
     * @param objectName 存储桶里的对象名称
     * @param offset     起始字节的位置
     * @param length     要读取的长度 (可选，如果无值则代表读到文件结尾)
     * @return
     */
    @SneakyThrows
    public InputStream getObject(String bucketName, String objectName, long offset, Long length) {
        boolean flag = bucketExists(bucketName);
        if (flag) {
            ObjectStat statObject = statObject(bucketName, objectName);
            if (statObject != null && statObject.length() > 0) {
                InputStream stream = minioClient.getObject(bucketName, objectName, offset, length);
                return stream;
            }
        }
        return null;
    }

    /**
     * 下载并将文件保存到本地
     *
     * @param bucketName 存储桶名称
     * @param objectName 存储桶里的对象名称
     * @param fileName   File name
     * @return
     */
    @SneakyThrows
    public boolean getObject(String bucketName, String objectName, String fileName) {
        boolean flag = bucketExists(bucketName);
        if (flag) {
            ObjectStat statObject = statObject(bucketName, objectName);
            if (statObject != null && statObject.length() > 0) {
                minioClient.getObject(bucketName, objectName, fileName);
                return true;
            }
        }
        return false;
    }

    /**
     * 删除一个对象
     *
     * @param bucketName 存储桶名称
     * @param objectName 存储桶里的对象名称
     */
    @SneakyThrows
    public boolean removeObject(String bucketName, String objectName) {
        boolean flag = bucketExists(bucketName);
        if (flag) {
            minioClient.removeObject(bucketName, objectName);
            return true;
        }
        return false;
    }

    /**
     * 删除指定桶的多个文件对象,返回删除错误的对象列表，全部删除成功，返回空列表
     *
     * @param bucketName  存储桶名称
     * @param objectNames 含有要删除的多个object名称的迭代器对象
     * @return
     */
    @SneakyThrows
    public List<String> removeObject(String bucketName, List<String> objectNames) {
        List<String> deleteErrorNames = new ArrayList<>();
        boolean flag = bucketExists(bucketName);
        if (flag) {
            Iterable<Result<DeleteError>> results = minioClient.removeObjects(bucketName, objectNames);
            for (Result<DeleteError> result : results) {
                DeleteError error = result.get();
                deleteErrorNames.add(error.objectName());
            }
        }
        return deleteErrorNames;
    }

    /**
     * 生成一个给HTTP GET请求用的presigned URL。
     * 浏览器/移动端的客户端可以用这个URL进行下载，即使其所在的存储桶是私有的。这个presigned URL可以设置一个失效时间，默认值是7天。
     *
     * @param bucketName 存储桶名称
     * @param objectName 存储桶里的对象名称
     * @param expires    失效时间（以秒为单位），默认是7天，不得大于七天
     * @return
     */
    @SneakyThrows
    public String presignedGetObject(String bucketName, String objectName, Integer expires) {
        boolean flag = bucketExists(bucketName);
        String url = "";
        if (flag) {
            if (expires < 1 || expires > DEFAULT_EXPIRY_TIME) {
                throw new InvalidExpiresRangeException(expires,
                        "expires must be in range of 1 to " + DEFAULT_EXPIRY_TIME);
            }
            url = minioClient.presignedGetObject(bucketName, objectName, expires);
        }
        return url;
    }

    /**
     * 生成一个给HTTP PUT请求用的presigned URL。
     * 浏览器/移动端的客户端可以用这个URL进行上传，即使其所在的存储桶是私有的。这个presigned URL可以设置一个失效时间，默认值是7天。
     *
     * @param bucketName 存储桶名称
     * @param objectName 存储桶里的对象名称
     * @param expires    失效时间（以秒为单位），默认是7天，不得大于七天
     * @return
     */
    @SneakyThrows
    public String presignedPutObject(String bucketName, String objectName, Integer expires) {
        boolean flag = bucketExists(bucketName);
        String url = "";
        if (flag) {
            if (expires < 1 || expires > DEFAULT_EXPIRY_TIME) {
                throw new InvalidExpiresRangeException(expires,
                        "expires must be in range of 1 to " + DEFAULT_EXPIRY_TIME);
            }
            url = minioClient.presignedPutObject(bucketName, objectName, expires);
        }
        return url;
    }

    /**
     * 获取对象的元数据
     *
     * @param bucketName 存储桶名称
     * @param objectName 存储桶里的对象名称
     * @return
     */
    @SneakyThrows
    public ObjectStat statObject(String bucketName, String objectName) {
        boolean flag = bucketExists(bucketName);
        if (flag) {
            ObjectStat statObject = minioClient.statObject(bucketName, objectName);
            return statObject;
        }
        return null;
    }

    /**
     * 文件访问路径
     *
     * @param bucketName 存储桶名称
     * @param objectName 存储桶里的对象名称
     * @return
     */
    @SneakyThrows
    public String getObjectUrl(String bucketName, String objectName) {
        boolean flag = bucketExists(bucketName);
        String url = "";
        if (flag) {
            url = minioClient.getObjectUrl(bucketName, objectName);
        }
        return url;
    }



    public void downloadFile(String bucketName, String fileName, String originalName, HttpServletResponse response) {
        try {

            InputStream file = minioClient.getObject(bucketName, fileName);
            String filename = new String(fileName.getBytes("ISO8859-1"), StandardCharsets.UTF_8);
            if (StringUtils.isNotEmpty(originalName)) {
                fileName = originalName;
            }
            response.setHeader("Content-Disposition", "attachment;filename=" + filename);
            ServletOutputStream servletOutputStream = response.getOutputStream();
            int len;
            byte[] buffer = new byte[1024];
            while ((len = file.read(buffer)) > 0) {
                servletOutputStream.write(buffer, 0, len);
            }
            servletOutputStream.flush();
            file.close();
            servletOutputStream.close();
        } catch (ErrorResponseException e) {
            e.printStackTrace();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

}

```

如果此开源项目于你有帮助，麻烦点赞留言支持一下，谢谢

服务类MinioService.java和实现类MinioServiceImpl.java

```java
package com.hope.minio.service;

import org.springframework.web.multipart.MultipartFile;

import javax.servlet.http.HttpServletResponse;
import java.io.InputStream;

/**
 * @PackageName: com.hope.minio.service
 * @ClassName: MinioService
 * @Author Hope
 * @Date 2020/7/27 9:58
 * @Description: MinioService
 */
public interface MinioService {


    /**
     * 判断 bucket是否存在
     *
     * @param bucketName
     * @return
     */
    boolean bucketExists(String bucketName);

    /**
     * 创建 bucket
     *
     * @param bucketName
     */
    void makeBucket(String bucketName);

    /**
     * 文件上传
     *
     * @param bucketName
     * @param objectName
     * @param filename
     */
    void putObject(String bucketName, String objectName, String filename);

    /**
     * 文件上传
     *
     * @param bucketName
     * @param objectName
     * @param stream
     */
    void putObject(String bucketName, String objectName, InputStream stream, String contentType);

    /**
     * 文件上传
     *
     * @param bucketName
     * @param multipartFile
     */
    void putObject(String bucketName, MultipartFile multipartFile, String filename);

    /**
     * 删除文件
     * @param bucketName
     * @param objectName
     */
    boolean removeObject(String bucketName,String objectName);

    /**
     * 下载文件
     *
     * @param fileName
     * @param originalName
     * @param response
     */
    void downloadFile(String bucketName, String fileName, String originalName, HttpServletResponse response);

    /**
     * 获取文件路径
     * @param bucketName
     * @param objectName
     * @return
     */
    String getObjectUrl(String bucketName,String objectName);
}
```

```java
package com.hope.minio.service.impl;

import com.hope.minio.service.MinioService;
import com.hope.minio.utils.MinioUtil;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.web.multipart.MultipartFile;

import javax.servlet.http.HttpServletResponse;
import java.io.InputStream;

/**
 * @PackageName: com.hope.minio.service.impl
 * @ClassName: MinioServiceImpl
 * @Author Hope
 * @Date 2020/7/27 9:59
 * @Description: MinioServiceImpl
 */
@Service
public class MinioServiceImpl implements MinioService {

    @Autowired
    private MinioUtil minioUtil;

    /**
     * 判断 bucket是否存在
     *
     * @param bucketName
     * @return
     */
    @Override
    public boolean bucketExists(String bucketName) {
        return minioUtil.bucketExists(bucketName);
    }

    /**
     * 创建 bucket
     *
     * @param bucketName
     */
    @Override
    public void makeBucket(String bucketName) {
        minioUtil.makeBucket(bucketName);
    }

    /**
     * 文件上传
     *
     * @param bucketName
     * @param objectName
     * @param filename
     */
    @Override
    public void putObject(String bucketName, String objectName, String filename) {
        minioUtil.putObject(bucketName, objectName, filename);
    }


    @Override
    public void putObject(String bucketName, String objectName, InputStream stream, String contentType) {
        minioUtil.putObject(bucketName, objectName, stream, contentType);
    }

    /**
     * 文件上传
     *
     * @param bucketName
     * @param multipartFile
     */
    @Override
    public void putObject(String bucketName, MultipartFile multipartFile, String filename) {
        minioUtil.putObject(bucketName, multipartFile, filename);
    }

    /**
     * 删除文件
     * @param bucketName
     * @param objectName
     */
    @Override
    public boolean removeObject(String bucketName,String objectName) {
        return minioUtil.removeObject(bucketName,objectName);
    }

    /**
     * 下载文件
     *
     * @param fileName
     * @param originalName
     * @param response
     */
    @Override
    public void downloadFile(String bucketName, String fileName, String originalName, HttpServletResponse response) {
        minioUtil.downloadFile(bucketName,fileName, originalName, response);
    }

    /**
     * 获取文件路径
     * @param bucketName
     * @param objectName
     * @return
     */
    @Override
    public String getObjectUrl(String bucketName,String objectName) {
        return minioUtil.getObjectUrl(bucketName,objectName);
    }



}
```

文件实现方法调用如下：

MinioController.java

```shell
package com.hope.minio.controller;

import com.hope.minio.common.result.Result;
import com.hope.minio.common.result.ResultUtil;
import com.hope.minio.config.MinioConfig;
import com.hope.minio.service.MinioService;
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiImplicitParam;
import io.swagger.annotations.ApiImplicitParams;
import io.swagger.annotations.ApiOperation;
import org.apache.commons.lang3.StringUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.util.MimeTypeUtils;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.multipart.MultipartFile;

import java.io.InputStream;
import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.UUID;

@RequestMapping("/minio")
@RestController
@Api(tags = "文件上传接口")
public class MinioController {

    @Autowired
    private MinioService minioService;

    @Autowired
    private MinioConfig minioConfig;

    @ApiOperation(value = "使用minio文件上传")
    @PostMapping("/uploadFile")
    @ApiImplicitParams({
            @ApiImplicitParam(dataType = "MultipartFile", name = "file", value = "上传的文件", required = true),
            @ApiImplicitParam(dataType = "String", name = "bucketName", value = "对象存储桶名称", required = false)
    })
    public Result uploadFile(MultipartFile file, String bucketName) {
        try {
            bucketName = StringUtils.isNotBlank(bucketName) ? bucketName : minioConfig.getBucketName();
            if (!minioService.bucketExists(bucketName)) {
                minioService.makeBucket(bucketName);
            }
            String fileName = file.getOriginalFilename();
            String objectName = new SimpleDateFormat("yyyy/MM/dd/").format(new Date()) + UUID.randomUUID().toString().replaceAll("-", "")
                    + fileName.substring(fileName.lastIndexOf("."));
            minioService.putObject(bucketName, file, objectName);
            return ResultUtil.success(minioService.getObjectUrl(bucketName, objectName));
        } catch (Exception e) {
            e.printStackTrace();
            return ResultUtil.sendErrorMessage("上传失败");
        }
    }
    
    public Result uploadIputSreamFile(MultipartFile file, String bucketName) {
        try {
            bucketName = StringUtils.isNotBlank(bucketName) ? bucketName : minioConfig.getBucketName();
            if (!minioService.bucketExists(bucketName)) {
                minioService.makeBucket(bucketName);
            }
            String fileName = file.getOriginalFilename();
            String objectName = new SimpleDateFormat("yyyy/MM/dd/").format(new Date()) + UUID.randomUUID().toString().replaceAll("-", "")
                    + fileName.substring(fileName.lastIndexOf("."));

            InputStream inputStream = file.getInputStream();
            minioService.putObject(bucketName, objectName, inputStream, MimeTypeUtils.IMAGE_JPEG_VALUE);
            inputStream.close();
            return ResultUtil.success(minioService.getObjectUrl(bucketName, objectName));
        } catch (Exception e) {
            e.printStackTrace();
            return ResultUtil.sendErrorMessage("上传失败");
        }
    }
}
```

然后接口调用

![](https://www.codepeople.cn/imges/00x42.png)

上传成功后返回的链接地址如果想要在浏览器中直接访问则需要配置安全策略

![](https://www.codepeople.cn/imges/00x43.png)

![](https://www.codepeople.cn/imges/00x44.png)

这样允许匿名访问了。**对于有些不能识别的文件类型，浏览器访问会直接下载**如果不想直接下载可以设置上传的contentType。或者放到html中就可以直接显示不影响使用。代码里已经封装了可以自动设置文件类型(上传流文件时)

源代码仓库地址：https://gitee.com/VCS/min-io-springboot2.x



博客地址：<https://www.codepeople.cn>

=====================================================================

微信公众号：

![](https://www.codepeople.cn/imges/weixin_icon/weixin.jpg)