---
layout: post
title:  "从零开始搭建一个图片服务器"
date:   2018-07-21 18:00:00 
categories: blog
tags: ['nginx','spring-boot']
published: true
comments: true
script: [post.js]
---

* Do not remove this line (it will not be displayed)
{: toc}

我做笔记经常用cmd markdown这个应用，免费的版本没有图片上传的功能，所以遇到图片只能使用在线的链接。我自己有个阿里云主机，所以就想搭个图片服务器，供写笔记使用。

这里我使用spring-boot2.0开发了图片上传的功能，然后使用nginx做图片服务器。

## 图片上传功能

因为我要做个web应用，所以需要一个文件上传接口、对文件的I/O处理、将图片地址信息的持久化。

下面将 `contorller` 代码贴出来。

```java
@RestController
@RequestMapping("/rest")
public class FileUploadController {

    private static String UPLOADED_FOLDER = "/Users/upload/";

    @Autowired
    private PictureRepository pictureRepository;

    @PostMapping("/upload")
    public Result SimpleFileUpload (@RequestParam("file") MultipartFile file, RedirectAttributes redirectAttributes) {
        Result res = new Result();
        if (file.isEmpty()) {
            redirectAttributes.addFlashAttribute("message", "Please select a file to upload");
            return res.fail();
        }

        String datePath = generateDateDir();
        String uploadPath = UPLOADED_FOLDER + generateDateDir();
        String name = file.getName();
        String oldName = file.getOriginalFilename();
        String picNewName = generateRandomFileName(name, oldName);
        try {
            byte[] bytes = file.getBytes();
            Path dir = Paths.get(uploadPath);
            try {
                Path newDir = Files.createDirectories(dir);
            } catch (FileAlreadyExistsException e) {

            } catch (IOException e) {
                e.printStackTrace();
            }
            Path path = Paths.get(uploadPath +  picNewName);
            Files.write(path, bytes);
            
            // 文件权限处理
            Set<PosixFilePermission> perms = new HashSet<PosixFilePermission>();
            perms.add(PosixFilePermission.OWNER_WRITE);
            perms.add(PosixFilePermission.OWNER_READ);
            perms.add(PosixFilePermission.OWNER_EXECUTE);

            perms.add(PosixFilePermission.GROUP_READ);
            perms.add(PosixFilePermission.GROUP_EXECUTE);

            perms.add(PosixFilePermission.OTHERS_READ);
            perms.add(PosixFilePermission.OTHERS_EXECUTE);

            Files.setPosixFilePermissions(dir, perms);
            Files.setPosixFilePermissions(path, perms);
            redirectAttributes.addFlashAttribute("message", "You successfully uploaded '" + file.getOriginalFilename() + "'");

        } catch (IOException o) {
            o.printStackTrace();
        }
        Picture p = new Picture("/upload/" + datePath + picNewName);
        // 将图片路径信息保存进数据库
        Picture picture = pictureRepository.save(p);
        return res.success().add("data", picture);
    }
    
    // 生成唯一的文件名称
    public static String generateRandomFileName (String filename, String ext) {
        // System.out.printf(filename);
        // String ext = filename.substring(filename.indexOf("."));
        return UUID.randomUUID().toString() + "." + "png";
    }
    
    // 生成日期维度的路径
    public static String generateDateDir () {
        Date date = new Date();
        return new SimpleDateFormat("yyyy/MM/dd").format(date) + "/";
    }
}

```
通过代码我们可以看出来，我们发送 `rest/upload` 请求后，应用会对接收到的 `file` 字段的值(文件数据)进行判断，为空返回失败信息。

如果文件信息存在，对文件进行I/O处理，写入到 `/upload/2018/09/11/` 目录下面，如果这个目录不存在，创建这个目录，之后将文件写入。在这个步骤，我们会给文件重写一个唯一的名字，并对文件的权限进行控制。

将文件信息写入到磁盘之上后，我们需要将图片信息保存下来，也就是持久化到数据库中。

最后返回成功信息，将图片路径信息一并返回，供用户使用。

![](http://47.93.235.47:5001/upload/2018/07/21/d3bfa6ba-4142-46ba-a177-d1ca6155bf6f.png)

至此，一个图片上传功能开发完毕。

## nginx图片服务器

图片保存了下来，路径用户也知晓，如果现在用户访问这个路径就可以访问到图片，那该怎么做？其实就是用户通过这个路径可以访问到主机上的资源。

这里用nginx实现，尽管spring-boot也集成了图片服务器的功能(下次吧，嘻嘻)。

nginx实现很简单。

```
server{
  listen 5001;
  server_name localhost;
  root /Users/image_server/;
  autoindex on;
}
```

这样就可以满足一个简单图片服务器的需求了。

## 遇到的问题

### 将war包部署到主机的tomcat后，接口访问404

后来经过搜索和咨询同事是因为spring-boot内置了tomcat，打war包时需要将tomat移除，并添加另一个依赖。

```
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-web</artifactId>
  <exclusions>
      <exclusion>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-tomcat</artifactId>
    </exclusion>
  </exclusions>
</dependency>
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-tomcat</artifactId>
  <scope>provided</scope>
</dependency>
```

### 访问图片资源时，403 forbidden

禁止访问，权限问题。刚开始是直接在服务器上执行赋权命令 `chmod` ，已经上传的图片可以访问，但是新上传的图片还是403。

思来想去，如果能在写文件的时候赋权应该就没问题了。查了查果然有这个api。

```
Set<PosixFilePermission> perms = new HashSet<PosixFilePermission>();
perms.add(PosixFilePermission.OWNER_WRITE);
perms.add(PosixFilePermission.OWNER_READ); 
perms.add(PosixFilePermission.OWNER_EXECUTE);

perms.add(PosixFilePermission.GROUP_READ);
perms.add(PosixFilePermission.GROUP_EXECUTE);

perms.add(PosixFilePermission.OTHERS_READ);
perms.add(PosixFilePermission.OTHERS_EXECUTE);

Files.setPosixFilePermissions(dir, perms);
Files.setPosixFilePermissions(path, perms);
```

对新建的文件夹和文件统一赋权，这下就没问题了。

### mysql 权限问题，对某个库没有操作权限

部署时报出权限的错误

```
Access denied for user 'root'@'127.0.0.1' to database
```

```
SELECT host,user,Grant_priv,Super_priv FROM mysql.user;
// 如果看到授权的权限没有打开
UPDATE mysql.user SET Grant_priv='Y', Super_priv='Y' WHERE User='root';
FLUSH PRIVILEGES;
```





## 思考

对java的使用，还停留在大四那年的记忆里，工作之后从事了前端开发，离服务端渐行渐远。

而现在的前端开发在工程化上越来越向服务端靠拢，譬如npm与maven的包管理功能，webpack与maven的打包功能等。现在做前端开发，就不能不学习webpack，学习webpack就不能不了解node，学习node就会了解一些服务端知识，好像绕了一圈又绕回来了。

所以在前两年的时候，我不再把自己定位是一个前端开发工程师，而是一个工程师，一个Coder。

编码不分前后端，设计模式，算法，这些都是通用的。

而我要持续学习计算机知识。

放张图自勉。
![](http://47.93.235.47:5001/upload/2018/07/21/853665d6-17ed-4a11-b4a8-076ffad35d42.png)


## 参考资料

[springboot(十七)：使用Spring Boot上传文件](http://www.ityouknow.com/springboot/2018/01/12/spring-boot-upload-file.html)

[Spring Boot初学者要知道的干货](https://www.jianshu.com/p/2b55b589048a)

[[解决] Error Code: 1044. Access denied for user 'root'@'%' to database](https://blog.csdn.net/odailidong/article/details/50770988)

[Java Code Examples for java.nio.file.Files.setPosixFilePermissions()](https://www.programcreek.com/java-api-examples/?class=java.nio.file.Files&method=setPosixFilePermissions)


{% endpost #9D9D9D %}