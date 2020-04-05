---
title:  "SpringBoot2.x使用EasyPOI导入Excel浅谈"
categories: SpringBoot2.0 EasyPOI
tags: SpringBoot2.0 EasyPOI
author: LIUREN
---

# SpringBoot2.x使用EasyPOI导入Excel浅谈

> 平时经常遇到客户要帮忙导入一些数据到数据库中，有些数据比较多有时候手动录入就会很耗时间，所以就自己写一个Excel导入的demo记录一下我对EasyPOI的误区；本文使用`SpringBoot2.0`,`EasyPOI`



## 开发框架

```shell
框架：SpringBoot2.0
java jdk 1.8
开发工具：Eclipse
数据库：Orcal
```



## 一.首先在pom.xml中导入EasyPOI的架包

pom.xml的主要文件信息如下：

```xml
<!-- easypoi -->
		<dependency>
            <groupId>cn.afterturn</groupId>
            <artifactId>easypoi-base</artifactId>
            <version>3.0.3</version>
        </dependency>
        <dependency>
            <groupId>cn.afterturn</groupId>
            <artifactId>easypoi-web</artifactId>
            <version>3.0.3</version>
        </dependency>
        <dependency>
            <groupId>cn.afterturn</groupId>
            <artifactId>easypoi-annotation</artifactId>
            <version>3.0.3</version>
        </dependency>
```



## 二.然后编写Controller

Controller：

```
/**   
 * @author lr
 * @date 2018年12月26日 下午6:51:43 
 * @version V1.0.0   
 */
package com.louis.sql.tools.controller;

import java.util.List;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.multipart.MultipartFile;

import com.louis.sql.tools.model.LongHuaAreaDO;
import com.louis.sql.tools.result.HttpResult;
import com.louis.sql.tools.service.LongHuaAreaService;

@RestController
@RequestMapping("longhua")
public class LongHuaController {

    @Autowired
    private LongHuaAreaService longHuaAreaService;
    /**
     * @Title: listAllData 
     * @Description: 查询处所有的数据并展示
     * @param @return  参数说明 
     * @return HttpResult    返回类型 
     * @throws
     */
    @RequestMapping(value = "/list", method = RequestMethod.GET)
    public HttpResult listAllData(){
        return HttpResult.ok(longHuaAreaService.listAll());
    }
    
    @RequestMapping(value = "/deleteByName", method = RequestMethod.DELETE)
    public HttpResult deleteByName(@RequestBody List<LongHuaAreaDO> records) {
        
        return HttpResult.ok(longHuaAreaService.deleteByName(records));
    }
    
    /**
     * @Title: importExcel 
     * @Description: 导入Excel方法
     * @param @param file  参数说明 
     * @return void    返回类型 
     * @throws
     */
    @PostMapping("/importExcel")
    public void importExcel(@RequestParam("file") MultipartFile file) {
        String fileName = file.getOriginalFilename();
        try {
            longHuaAreaService.batchImport(fileName, file);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```



## 三.接口LongHuaAreaService

```java
/**   
 * @author lr
 * @date 2018年12月26日 下午9:26:02 
 * @version V1.0.0   
 */
package com.louis.sql.tools.service;

import java.io.IOException;
import java.util.List;

import org.springframework.web.multipart.MultipartFile;

import com.louis.sql.tools.model.LongHuaAreaDO;

public interface LongHuaAreaService {

    /** 
     * @Title: listAll 
     * @Description: 查询所有的龙华区监管人员
     * @param @return  参数说明 
     * @return String    返回类型 
     * @throws 
     */
    List<LongHuaAreaDO> listAll();

    /** 
     * @Title: deleteByName 
     * @Description: TODO(这里用一句话描述这个方法的作用) 
     * @param @param records
     * @param @return  参数说明 
     * @return String    返回类型 
     * @throws 
     */
    int deleteByName(List<LongHuaAreaDO> records);

    /** 
     * @Title: delete 
     * @Description: TODO(这里用一句话描述这个方法的作用) 
     * @param @param record
     * @param @return  参数说明 
     * @return int    返回类型 
     * @throws 
     */
    int delete(LongHuaAreaDO record);

    /** 
     * @Title: batchImport 
     * @Description: 导入Excel中的数据 
     * @param @param fileName
     * @param @param file  参数说明 
     * @return void    返回类型 
     * @throws 
     */
    void batchImport(String fileName, MultipartFile file) throws IOException, Exception;

}
```



## 四.实现方法LongHuaAreaServiceImpl

```java
/**   
 * @author lr
 * @date 2018年12月26日 下午9:26:50 
 * @version V1.0.0   
 */
package com.louis.sql.tools.service.impl;

import java.io.IOException;
import java.util.ArrayList;
import java.util.List;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;
import org.springframework.web.multipart.MultipartFile;

import com.louis.sql.tools.dao.LongHuaAreaMapper;
import com.louis.sql.tools.model.LongHuaAreaDO;
import com.louis.sql.tools.service.LongHuaAreaService;

import cn.afterturn.easypoi.excel.ExcelImportUtil;
import cn.afterturn.easypoi.excel.entity.ImportParams;

@Service
public class LongHuaAreaServiceImpl implements LongHuaAreaService{

    @Autowired
    private LongHuaAreaMapper longHuaAreaMapper;
    
    @Override
    public List<LongHuaAreaDO> listAll() {
        
        return longHuaAreaMapper.listAll();
    }

    @Override
    public int delete(LongHuaAreaDO record) {
            return longHuaAreaMapper.deleteByName(record.getName());
    }
    
    @Override
    public int deleteByName(List<LongHuaAreaDO> records) {
        for (LongHuaAreaDO record : records) {
            delete(record);
        }
        return 1;
    }

    /*
     * Title: batchImport
     *Description: 
     * @param fileName
     * @param file 
     * @see com.louis.sql.tools.service.LongHuaAreaService#batchImport(java.lang.String, org.springframework.web.multipart.MultipartFile) 
     */
    @Transactional(readOnly = false,rollbackFor = Exception.class)
    @Override
    public void batchImport(String fileName, MultipartFile file) throws IOException, Exception {
        ImportParams params = new ImportParams();
        params.setTitleRows(0);
        params.setHeadRows(1);
        List<LongHuaAreaDO> list = ExcelImportUtil.importExcel(file.getInputStream(), LongHuaAreaDO.class, params);
        for (int i = 0; i <list.size() ; i++) {
            System.out.println(list.get(i));
            System.out.println(list.get(i).getId());
        }
        
    }


}
```

## 五.实体类方法LongHuaAreaDO

```java
/**   
 * @author lr
 * @date 2018年12月26日 下午6:32:48 
 * @version V1.0.0   
 */
package com.louis.sql.tools.model;

import java.io.Serializable;
import java.util.Date;

import cn.afterturn.easypoi.excel.annotation.Excel;
import cn.afterturn.easypoi.excel.annotation.ExcelTarget;

@ExcelTarget("BZ_JCRY")
public class LongHuaAreaDO implements Serializable{

    private static final long serialVersionUID = -4277023292843262708L;

    @Excel(name = "ID", orderNum = "0")
    private String id;
    @Excel(name = "编号", orderNum = "1")
    private String cerNo;
    @Excel(name = "姓名", orderNum = "2")
    private String legalNo;
    @Excel(name = "姓名", orderNum = "3")
    private String name;
    @Excel(name = "性别", replace = {"男_0", "女_1"}, orderNum = "4")
    private String sex;
    @Excel(name = "手机号", orderNum = "5")
    private String mobTel;
    @Excel(name = "单位名称", orderNum = "6")
    private String corpName;
    @Excel(name = "部门名称", orderNum = "7")
    private String deptName;
    @Excel(name = "职位", orderNum = "8")
    private String post;
    @Excel(name = "专门", orderNum = "9")
    private String expertise;
    @Excel(name = "组织机构", orderNum = "10")
    private String regOrg;
    @Excel(name = "组织机构简称", orderNum = "11")
    private String bizOrg;
    @Excel(name = "更新时间", exportFormat = "yyyy-MM-dd HH:mm:ss", orderNum = "12")
    private Date updateTime;
    @Excel(name = "办公室名称", orderNum = "13")
    private String officeName;
    @Excel(name = "单位编码", orderNum = "14")
    private String corpCode;
    @Excel(name = "部门编码", orderNum = "15")
    private String deptCode;
    @Excel(name = "创建时间", exportFormat = "yyyy-MM-dd HH:mm:ss", orderNum = "16")
    private Date createTime;
    @Excel(name = "雇佣编号", orderNum = "17")
    private String employeeCode;
    @Excel(name = "是否在职", orderNum = "18")
    private String isextract;

    
    @Override
    public String toString() {
        return "LongHuaAreaDO [id=" + id + ", cerNo=" + cerNo + ", legalNo=" + legalNo + ", name=" + name
                + ", sex=" + sex + ", mobTel=" + mobTel + ", corpName=" + corpName + ", deptName=" + deptName
                + ", post=" + post + ", expertise=" + expertise + ", regOrg=" + regOrg + ", bizOrg=" + bizOrg
                + ", updateTime=" + updateTime + ", officeName=" + officeName + ", corpCode=" + corpCode
                + ", deptCode=" + deptCode + ", createTime=" + createTime + ", employeeCode=" + employeeCode
                + ", isextract=" + isextract + "]";
    }

    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id == null ? null : id.trim();
    }

    public String getCerNo() {
        return cerNo;
    }

    public void setCerNo(String cerNo) {
        this.cerNo = cerNo == null ? null : cerNo.trim();
    }

    public String getLegalNo() {
        return legalNo;
    }

    public void setLegalNo(String legalNo) {
        this.legalNo = legalNo == null ? null : legalNo.trim();
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name == null ? null : name.trim();
    }

    public String getSex() {
        return sex;
    }

    public void setSex(String sex) {
        this.sex = sex == null ? null : sex.trim();
    }

    public String getMobTel() {
        return mobTel;
    }

    public void setMobTel(String mobTel) {
        this.mobTel = mobTel == null ? null : mobTel.trim();
    }

    public String getCorpName() {
        return corpName;
    }

    public void setCorpName(String corpName) {
        this.corpName = corpName == null ? null : corpName.trim();
    }

    public String getDeptName() {
        return deptName;
    }

    public void setDeptName(String deptName) {
        this.deptName = deptName == null ? null : deptName.trim();
    }

    public String getPost() {
        return post;
    }

    public void setPost(String post) {
        this.post = post == null ? null : post.trim();
    }

    public String getExpertise() {
        return expertise;
    }

    public void setExpertise(String expertise) {
        this.expertise = expertise == null ? null : expertise.trim();
    }

    public String getRegOrg() {
        return regOrg;
    }

    public void setRegOrg(String regOrg) {
        this.regOrg = regOrg == null ? null : regOrg.trim();
    }

    public String getBizOrg() {
        return bizOrg;
    }

    public void setBizOrg(String bizOrg) {
        this.bizOrg = bizOrg == null ? null : bizOrg.trim();
    }

    public Date getUpdateTime() {
        return updateTime;
    }

    public void setUpdateTime(Date updateTime) {
        this.updateTime = updateTime;
    }

    public String getOfficeName() {
        return officeName;
    }

    public void setOfficeName(String officeName) {
        this.officeName = officeName == null ? null : officeName.trim();
    }

    public String getCorpCode() {
        return corpCode;
    }

    public void setCorpCode(String corpCode) {
        this.corpCode = corpCode == null ? null : corpCode.trim();
    }

    public String getDeptCode() {
        return deptCode;
    }

    public void setDeptCode(String deptCode) {
        this.deptCode = deptCode == null ? null : deptCode.trim();
    }

    public Date getCreateTime() {
        return createTime;
    }

    public void setCreateTime(Date createTime) {
        this.createTime = createTime;
    }

    public String getEmployeeCode() {
        return employeeCode;
    }

    public void setEmployeeCode(String employeeCode) {
        this.employeeCode = employeeCode == null ? null : employeeCode.trim();
    }

    public String getIsextract() {
        return isextract;
    }

    public void setIsextract(String isextract) {
        this.isextract = isextract == null ? null : isextract.trim();
    }
    
    
    
}
```

注意：一定要有toString构造方法

## 六.要导入的Excel中的内容格式如下：

![Excel内容格式](https://www.codepeople.cn/imges/EasyPOI/001.png)

## 七.使用Swagger-ui的的接口方法进行导入

![导入方法](https://www.codepeople.cn/imges/EasyPOI/002.png)

## 八.导入后的效果如下

![导入后台打印效果](https://www.codepeople.cn/imges/EasyPOI/003.png)

## 九.Demo下载

下载地址：https://github.com/PlayTaoist/SpringBoot2-FastDFS



**目前只写到从Excel中读取到了数据，后续如何插入到数据库中大家可以自己完善一下**



=======================================================================================

博客地址：<https://www.codepeople.cn>

=======================================================================================

微信公众号：

![](https://www.codepeople.cn/imges/weixin_icon/weixin.jpg)