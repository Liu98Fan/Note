## springBoot实现文件上传功能（结合Editor.m工具）

​	一直将editor.m的markdown编辑工具的图片上传坑留到今天，实在忍不住想要把它解决了。

嗯首先我们得页面是这样得：

![](C:\Users\A\Desktop\笔记\我的笔记_files\2018-10-07_222405.png)

JS中的声明是这样的：

```javascript
    var testEditor;
    $(function() {
        testEditor = editormd("test-editormd",{
            width   : "100%",
            height  : 640,
            syncScrolling : "single",
            path    : "/pageResources/mark/",
            imageUpload : true,
            imageFormats : ["jpg", "jpeg", "gif", "png", "bmp"],
            imageUploadURL : "/tools/uploadImages",
        });
    });
```

根据官方文档（官网莫名访问不了，各路博客搜集来的信息），这个图片上传的文件参数名是editormd-image-file,要求返回一组格式统一的json数据，下面给出后台处理代码:

```java
package cn.bestrivenlf.myWeb.controller;

import net.sf.json.JSONObject;
import org.apache.commons.lang.RandomStringUtils;
import org.apache.commons.lang.math.RandomUtils;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.multipart.MultipartFile;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.File;
import java.io.IOException;
import java.util.Random;

/**
 * @Author: liufan
 * @Date: 2018/10/7 21:13
 * @Description:
 */
@Controller
@RequestMapping("/tools")
public class ToolsController {

    @Value("${file.uploadPath}")
    String uploadPath;
    @Value("${file.accessPath}")
    String accessPath;
    
    @RequestMapping("/uploadImages")
    @ResponseBody
    public JSONObject uploadImages(@RequestParam(value = "editormd-image-file", required = true) MultipartFile file,
                                   HttpServletRequest request, HttpServletResponse response){
        String trueFileName = file.getOriginalFilename();
        String suffix = trueFileName.substring(trueFileName.lastIndexOf("."));
        String fileName = System.currentTimeMillis()+"_"+RandomStringUtils.randomNumeric(5) +suffix;
        String path = uploadPath;
        System.out.println(path);
        File targetFile = new File(path, fileName);
        if(!targetFile.exists()){
            try {
                targetFile.createNewFile();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        try {
            file.transferTo(targetFile);
        } catch (Exception e) {
            e.printStackTrace();
        }
        JSONObject res = new JSONObject();
        res.put("url", accessPath+fileName);
        res.put("success", 1);
        res.put("message", "upload success!");
        return res;

    }
}

```

这个代码实现了将文件保存到uploadPath路径，同时给前台返回图片的访问路径accessPath+文件名。

接下来就是将uploadPath和accessPath进行绑定，实现在网站中访问accessPath这个虚拟路径的时候可以映射到磁盘上的uploadPath的真实路径。相信写过其他项目的童鞋都应该知道，这个功能以前都是在Tomcat的配置文件中配置的，但是SpringBoot的Tomcat是内置的，如果修改配置需要重写Tomcat的配置类，比较麻烦，所以SpringBoot提供了重写MVCConfigur的方法来实现虚拟目录的映射，下面贴出配置代码：

```java
@Configuration
public class MyMvcConfig implements WebMvcConfigurer {
 
    @Value("${file.accessPath}")
    String accessPath;
    @Value("${file.uploadPath}")
    String uploadPath;

    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        						
 //据说是Springboot版本的原因，反正我用的是2.0.5，在绝对路径映射上需要加上file:，否则映射失败！
        registry.addResourceHandler(accessPath+"**").addResourceLocations("file:"+uploadPath);
    }
}
```

然后accessPath和uploadPath的参数直接写在配置文件里即可：

```yaml
file:
  uploadPath: E:\\images\\
  accessPath: /upload/markPic/
```

最后结果如图:

![](C:\Users\A\Desktop\笔记\我的笔记_files\2018-10-07_223445.png)

当然这个实现还不是很完美，因为当我图片上传错误后，需要将其删除，而此时之前的错误图片已经存储在本地或者服务器上了，不能只能删除，关于这方面的处理暂时还没有想出一个好的方法，如果有思路的同学欢迎告知！





