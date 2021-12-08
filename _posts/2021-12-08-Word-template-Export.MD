---
title:  "使用Word模板导出标准表Word样式文件"
categories:  Word poi-tl Excel
tags: Word poi-tl Excel
author: LIUREN
---

# 使用Word模板导出标准表Word样式文件

> 使用Word模板导出标准表Word样式文件
>
> 协议：CC BY-SA 4.0 https://creativecommons.org/licenses/by-sa/4.0/  
>
> 版权声明：本文为原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。



[TOC]

## 接了个需求要做Word文件导出，导出的Word中有样式还有表格。表格还要合并；最离谱的是还要有附件。

表头样式和内容是这样：

![](https://www.codepeople.cn/imges/00x125.png)

表的合并这样：

![](https://www.codepeople.cn/imges/00x126.png)

竖向表格

![](https://www.codepeople.cn/imges/00x127.png)

附件和附件内容是这样：

![](https://www.codepeople.cn/imges/00x128.png)

具体这种样式怎么实现。下面讲解下

## 第一步：首先我们要引用一个工具包poi-tl

```xml
<dependency>
            <groupId>com.deepoove</groupId>
            <artifactId>poi-tl</artifactId>
            <version>1.10.2</version>
            <exclusions>
                <exclusion>
                    <artifactId>xalan</artifactId>
                    <groupId>xalan</groupId>
                </exclusion>
            </exclusions>
        </dependency>
```

至于为啥是1.10.2 不是 11版本暂时跟项目架包冲突，所以改为10的版本

## 第二步：做一个Word模板

模板怎么做。如果是文本，我们直接使用 {{变量名}}

表格也可以这么使用

这个是我的模板：

![](https://www.codepeople.cn/imges/00x129.png)

## 第三步：开始编写代码

1.先建一个实体类：

```java
package com.example.test.controller;

import java.util.List;

import com.deepoove.poi.data.AttachmentRenderData;
import com.deepoove.poi.data.TableRenderData;
import com.deepoove.poi.expression.Name;

import lombok.Data;
@Data
public class WordDO {

	private TableRenderData order;
	
	/**
	 * 一、用户总体应用情况-表
	 */
	@Name("detail_table")
	private UserApplicationTableDO userApplicationTableDO;
	
	/**
	 * 三、用户分析-表
	 */
	@Name("table1")
	private UserApplicationTableDO table1;
	
	/**
	 * 三、用户分析-表
	 */
	@Name("table2")
	private UserApplicationTableDO table2;
	
	/**
	 * 四、用户行为分析-表
	 */
	@Name("table3")
	private UserApplicationTableDO table3;
	
	@Name("table5")
	private List<Table5Entity> table5;
	/**
	 * 六、产品生产力
	 */
	@Name("table6")
	private UserApplicationTableDO table6;
	
	/**
	 * 六、产品生产力
	 */
	@Name("table7")
	private UserApplicationTableDO table7;
	
	@Name("table8")
	private UserApplicationTableDO table8;
	
	private String header;
	private String secondHeader;
	private String monthReportName;
	private String monthReportContent;
	
	@Name("xlsx")
	private AttachmentRenderData xlsx;
}

```

2.然后是Controller 实现方法我直接写在Controller里了

```java
package com.example.test.controller;

import java.io.File;
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

import org.springframework.stereotype.Controller;
import org.springframework.util.ResourceUtils;
import org.springframework.web.bind.annotation.RequestMapping;

import com.deepoove.poi.XWPFTemplate;
import com.deepoove.poi.config.Configure;
import com.deepoove.poi.data.AttachmentType;
import com.deepoove.poi.data.Attachments;
import com.deepoove.poi.data.RowRenderData;
import com.deepoove.poi.data.Rows;
import com.deepoove.poi.policy.AttachmentRenderPolicy;
import com.deepoove.poi.policy.RenderPolicy;


@Controller
@RequestMapping("word")
public class PaymentExampleController {

	@RequestMapping("word")
	public void getWord() throws Exception {
		File file = ResourceUtils.getFile("classpath:wordtemplate/month_report_template.docx");
		FileInputStream inputStream = new FileInputStream(file);
		WordDO worDo = new WordDO();    
		worDo.setHeader("中国集团内部部文件");
		worDo.setSecondHeader("中国集团〔2021〕3 号");
		worDo.setMonthReportName("中国集团服务运营通报（2021年第1期）");
		worDo.setMonthReportContent("运营管理部、安全管理中心、大数据应用部、管理信息系统部、研发创新中心、智慧中台运营中心、计费清结算中心、建筑部门：\\r\\n\"\r\n"
				+ "					+ \"引言部分（待补充）。\\r\\n");
		
		/**
		 * 第一个表格：用户总体应用情况-表格
		 */
		UserApplicationTableDO detailTable = new UserApplicationTableDO();
		RowRenderData good001 = Rows.of("001", "墙纸-001", "1600").center().create();
		RowRenderData good002 = Rows.of("002", "墙纸-002", "100").center().create();
		List<RowRenderData> goods = Arrays.asList(good001, good002);
		detailTable.setLabors(goods);
		worDo.setUserApplicationTableDO(detailTable);
		
		/**
		 * 第一个表格：产品使用情况-表格
		 */
		UserApplicationTableDO product_table = new UserApplicationTableDO();
		RowRenderData product_z_001 = Rows.of("001", "智慧中台运营中心", "2","联调测试平台", "XX项目、XX业务").center().create();
		RowRenderData product_z_002 = Rows.of("002", "智慧中台运营中心", "2","用户模拟平台", "XX项目").center().create();
		List<RowRenderData> product_z = Arrays.asList(product_z_001, product_z_002);
		product_table.setLabors(product_z);
		worDo.setTable1(product_table);
		
		
		/**
		 * 第二个表格：用户总体应用情况-表格
		 */
		UserApplicationTableDO detailTable2 = new UserApplicationTableDO();
		RowRenderData table1 = Rows.of("001", "墙纸-001", "1600","001", "墙纸-001", "1600","4").center().create();
		RowRenderData table2 = Rows.of("002", "墙纸-002", "100","001", "墙纸-001", "1600","4").center().create();
		List<RowRenderData> tables = Arrays.asList(table1, table2);
		detailTable2.setLabors(tables);
		worDo.setTable2(detailTable2);
		
		/**
		 * 第三个表格：用户总体应用情况-表格
		 */
		UserApplicationTableDO detailTable3 = new UserApplicationTableDO();
		RowRenderData table3 = Rows.of("001", "基础建设", "以桑基图形式展现").center().create();
		RowRenderData table4 = Rows.of("002", "基础建设", "以桑基图形式展现").center().create();
		List<RowRenderData> tables_x = Arrays.asList(table3, table4);
		detailTable3.setLabors(tables_x);
		worDo.setTable3(detailTable3);
		
		/**
		 * 第三个表格：用户总体应用情况-表格
		 */
		UserApplicationTableDO detailTable6 = new UserApplicationTableDO();
		RowRenderData table_y_001 = Rows.of("001", "基础建设平台", "100", "15", "1500", "5").center().create();
		RowRenderData table_y_002 = Rows.of("002", "基础建设平台", "100", "15", "1500", "5").center().create();
		List<RowRenderData> tables_y = Arrays.asList(table_y_001, table_y_002);
		detailTable6.setLabors(tables_y);
		worDo.setTable6(detailTable6);
		
		/**
		 * 第三个表格：七、本月新增需求情况
		 */
		UserApplicationTableDO detailTable7 = new UserApplicationTableDO();
		RowRenderData table_7_001 = Rows.of("001", "售楼部中心", "XXX业务/项目", "试点类", "售楼部平台").center().create();
		RowRenderData table_7_002 = Rows.of("002", "售楼部支撑中心", "XXX业务/项目", "试点类", "售楼部平台").center().create();
		List<RowRenderData> tables_7 = Arrays.asList(table_7_001, table_7_002);
		detailTable7.setLabors(tables_7);
		worDo.setTable7(detailTable7);
		
		/**
		 * 第三个表格：业务测试产品联系人
		 */
		UserApplicationTableDO detailTable8 = new UserApplicationTableDO();
		RowRenderData table_8_001 = Rows.of("数据库测试产品服务", "李二林", "", "").center().create();
		RowRenderData table_8_002 = Rows.of("性能测试产品服务", "李二林", "", "").center().create();
		RowRenderData table_8_003 = Rows.of("性能测试产品服务", "李二林", "", "").center().create();
		RowRenderData table_8_004 = Rows.of("性能测试产品服务", "李二林", "", "").center().create();
		RowRenderData table_8_005 = Rows.of("性能测试产品服务", "李二林", "", "").center().create();
		List<RowRenderData> table_8 = Arrays.asList(table_8_001, table_8_002,table_8_003, table_8_004, table_8_005);
		detailTable8.setLabors(table_8);
		worDo.setTable8(detailTable8);
		
		RenderPolicy hackLoopSameLineTableRenderPolicy = new LoopColumnTableRenderPolicyNew(true);
		List<Table5Entity> table5 = new ArrayList<>();
		Table5Entity table5Entity = new Table5Entity();
		table5Entity.setNum(1);
		table5Entity.setProductName("售楼部平台");
		table5Entity.setUserRegisterCount(1000);
		table5Entity.setUserAddCount(100);
		table5Entity.setActiveCount(101);
		table5Entity.setActivePerenct("100%");
		table5Entity.setVisitDepth("2");
		table5Entity.setSilenceUserCount("10");
		table5Entity.setSilencePercent("20%");
		table5Entity.setLoseUserCount("123");
		table5Entity.setLosePercent("12%");
		table5Entity.setBackUserCount("21");
		table5Entity.setBackPercent("2%");
		table5.add(table5Entity);
		table5.add(table5Entity);
		table5.add(table5Entity);
		worDo.setTable5(table5);
		
		
		/**
		 * 附件 -- 接口获取一个Excel文件流
		 */
		//worDo.setXlsx(Attachments.ofLocal("D:\\sts4.10.0_workspace\\test\\test\\src\\main\\resources\\wordtemplate\\a.xlsx", AttachmentType.XLSX).create());
		File fileExcel = ResourceUtils.getFile("classpath:wordtemplate/a.xlsx");
		FileInputStream excelinputStream = new FileInputStream(fileExcel);
		worDo.setXlsx(Attachments.ofStream(excelinputStream, AttachmentType.XLSX).create());
		/**
		 * 读取Excel内容并 写入word
		 */
		List<Integer> list = new ArrayList<>();
		list.add(2);
		
		Configure config = Configure.builder()
							.bind("detail_table", new DetailTablePolicy(3))
							.bind("table1", new DetailTablePolicyMerge(5,list))
							.bind("table2", new DetailTablePolicy(7))
							.bind("table3", new DetailTablePolicy(3))
							.bind("table6", new DetailTablePolicy(6))
							.bind("table7", new DetailTablePolicy(5))
							.bind("table8", new DetailTablePolicy(4))
							.bind("table5", hackLoopSameLineTableRenderPolicy)
							.bind("xlsx", new AttachmentRenderPolicy())
							.build();
        
	    XWPFTemplate template = XWPFTemplate.compile(inputStream, config).render(worDo);
	    template.writeAndClose(new FileOutputStream("D:\\sts4.10.0_workspace\\test\\test\\src\\main\\resources\\wordtemplate\\templateAAA.docx"));
	    System.out.println("========================>>>>");
	}
	

}

```

3.UserApplicationTableDO类

```java
package com.example.test.controller;

import java.util.List;

import com.deepoove.poi.data.RowRenderData;

import lombok.Data;

@Data
public class UserApplicationTableDO {

	private List<RowRenderData> labors ;
	
}
```

4.表格渲染方法实现 DetailTablePolicy

```java
package com.example.test.controller;

import java.util.List;

import org.apache.poi.xwpf.usermodel.XWPFTable;
import org.apache.poi.xwpf.usermodel.XWPFTableRow;

import com.deepoove.poi.data.RowRenderData;
import com.deepoove.poi.policy.DynamicTableRenderPolicy;
import com.deepoove.poi.policy.TableRenderPolicy;

public class DetailTablePolicy extends DynamicTableRenderPolicy{

	// 表格有几列
    int column = 0;

    
	public DetailTablePolicy(int i) {
		column = i;
	}


	@Override
	public void render(XWPFTable table, Object data) throws Exception {
		if (null == data) return;
		TableStyle.setTableStyle(table);
		
		UserApplicationTableDO detailData = (UserApplicationTableDO) data;

        List<RowRenderData> goods = detailData.getLabors();
        if (null != goods) {
            table.removeRow(Contanst.START_ROW);
            for (int i = 0; i < goods.size(); i++) {
                XWPFTableRow insertNewTableRow = table.insertNewTableRow(Contanst.START_ROW);
                for (int j = 0; j < column; j++) insertNewTableRow.createCell();
                //TableStyle.setFontStyle();
                TableRenderPolicy.Helper.renderRow(table.getRow(Contanst.START_ROW), goods.get(i), TableStyle.setFontStyle());
            }
        }
	}

}

```

5.单元格合并实现类

```java
package com.example.test.controller;

import java.util.ArrayList;
import java.util.List;

import org.apache.poi.xwpf.usermodel.XWPFTable;
import org.apache.poi.xwpf.usermodel.XWPFTableRow;

import com.deepoove.poi.data.CellRenderData;
import com.deepoove.poi.data.RowRenderData;
import com.deepoove.poi.policy.DynamicTableRenderPolicy;
import com.deepoove.poi.policy.TableRenderPolicy;
import com.deepoove.poi.util.TableTools;

import cn.hutool.core.collection.CollUtil;

public class DetailTablePolicyMerge extends DynamicTableRenderPolicy{

	// 表格有几列
    int column = 0;
    List<Integer> listMerge = new ArrayList<>();
    
	public DetailTablePolicyMerge(int i, List<Integer> list) {
		column = i;
		list.forEach(e->{listMerge.add(e);});
	}


	@Override
	public void render(XWPFTable table, Object data) throws Exception {
		if (null == data) return;
		TableStyle.setTableStyle(table);
		
		UserApplicationTableDO detailData = (UserApplicationTableDO) data;

        List<RowRenderData> goods = detailData.getLabors();
        if (null != goods) {
            table.removeRow(Contanst.START_ROW);
            for (int i = 0; i < goods.size(); i++) {
                XWPFTableRow insertNewTableRow = table.insertNewTableRow(Contanst.START_ROW);
                for (int j = 0; j < column; j++) insertNewTableRow.createCell();
                //合并单元格
                TableRenderPolicy.Helper.renderRow(table.getRow(Contanst.START_ROW), goods.get(i), TableStyle.setFontStyle());
            }
        }
        
        // 合并第0列的第1行到第2行的单元格 [2,3,4]
        if(CollUtil.isNotEmpty(listMerge)) {
        	
        	for (int i = 0; i < listMerge.size(); i++) {
        		if(i==0) {
        			TableTools.mergeCellsVertically(table, 1, 1, listMerge.get(i));
        			TableTools.mergeCellsVertically(table, 2, 1, listMerge.get(i));
        		}else {
        			TableTools.mergeCellsVertically(table, 1, i + listMerge.get(i-1), listMerge.get(i));
            		TableTools.mergeCellsVertically(table, 2, i + listMerge.get(i-1), listMerge.get(i));
        		}
				
			}
        	
        }
        
	}

	

	
}

```

6.用到的常量Contanst

```java
package com.example.test.controller;

public class Contanst {

	public static int START_ROW = 1;
}
```

7.表格样式TableStyle

```java
package com.example.test.controller;

import org.apache.poi.xwpf.usermodel.XWPFTable;

import com.deepoove.poi.data.style.Style;

public class TableStyle {
	
	/**
	 * @Title: setTableStyle
	 * @Description: 设置表格边框 为黑色横线
	 * @Author liuren
	 * @DateTime 2021年12月2日 下午9:58:43
	 * @param table
	 */
	public static void setTableStyle(XWPFTable table) {
		table.setInsideHBorder(XWPFTable.XWPFBorderType.SINGLE,1,1,"010101"); //设置table的内部横向边框
        table.setInsideVBorder(XWPFTable.XWPFBorderType.SINGLE,1,1,"010101");//设置table的内部纵向边框
        table.setBottomBorder(XWPFTable.XWPFBorderType.SINGLE,1,1,"010101"); //设置table的顶部边框
        table.setTopBorder(XWPFTable.XWPFBorderType.SINGLE,1,1,"010101");//设置table的顶部边框
        table.setLeftBorder(XWPFTable.XWPFBorderType.SINGLE,1,1,"010101");//设置table的顶左边框
        table.setRightBorder(XWPFTable.XWPFBorderType.SINGLE,1,1,"010101");//设置table的右部边框
	}

	/**
	 * @Title: setFontStyle
	 * @Description: 设置动态表格中的字体样式和字体大小
	 * @Author liuren
	 * @DateTime 2021年12月2日 下午10:02:10
	 * @return
	 */
	public static Style setFontStyle() {
		Style style = new Style();
        style.setFontFamily("仿宋");
        style.setFontSize(14);
        return style;
	}
}

```



最后我的整体的pom.xml

```java
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.example</groupId>
    <artifactId>test</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>test</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <spring-boot.version>2.3.7.RELEASE</spring-boot.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-validation</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
		<dependency>
			<groupId>org.projectlombok</groupId>
			<artifactId>lombok</artifactId>
		</dependency>
		<dependency>
		    <groupId>cn.hutool</groupId>
		    <artifactId>hutool-all</artifactId>
		    <version>5.7.11</version>
		</dependency>
		<dependency>
		    <groupId>com.sun.mail</groupId>
		    <artifactId>javax.mail</artifactId>
		    <version>1.6.2</version>
		</dependency>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-openfeign</artifactId>
			<version>2.2.9.RELEASE</version>
		</dependency>
		<dependency>
		  <groupId>org.apache.commons</groupId>
		        <artifactId>commons-collections4</artifactId>
		        <version>4.1</version>
		</dependency>
		
		<dependency>
			<groupId>com.google.guava</groupId>
			<artifactId>guava</artifactId>
			<version>27.0-jre</version>
		</dependency>
		<dependency>
		  <groupId>com.deepoove</groupId>
		  <artifactId>poi-tl</artifactId>
		  <version>1.11.0</version>
		  <exclusions>
		        <exclusion>
		            <artifactId>xalan</artifactId>
		            <groupId>xalan</groupId>
		        </exclusion>
		    </exclusions>
		</dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
            <exclusions>
                <exclusion>
                    <groupId>org.junit.vintage</groupId>
                    <artifactId>junit-vintage-engine</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        
	    <dependency>
	        <groupId> e-iceblue </groupId>
	        <artifactId>spire.office.free</artifactId>
	        <version>3.9.0</version>
	    </dependency>
    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>${spring-boot.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.8.1</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                    <encoding>UTF-8</encoding>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <version>2.3.7.RELEASE</version>
                <configuration>
                    <mainClass>com.example.test.TestApplication</mainClass>
                </configuration>
                <executions>
                    <execution>
                        <id>repackage</id>
                        <goals>
                            <goal>repackage</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
	 
</project>
```



最后附上我的Word模板，供大家下载：https://www.codepeople.cn/imges/month_report_template.docx



这就完成了，然后调用下接口，就能生成Word了。还带附件哦。

下次抽时间 写下 Word生成的PDF的方法

=====================================================================

微信公众号：

![](https://www.codepeople.cn/imges/weixin_icon/weixin.jpg)