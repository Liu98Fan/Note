<center><h1>IDEA快速搭建SpringBoot项目</h1></center>

+ 打开IDEA，File->New->project,选择spring initializer,next下去即可

  ![pic1](C:\Users\A\Desktop\笔记\我的笔记_files\pic1.png)

+ 创建完毕后得到如下所示目录:

  ​	![pic2](C:\Users\A\Desktop\笔记\我的笔记_files\pic2.png)

+ 编写案例测试spring boot 

  ```java
  package cn.bestrivenlf.mydemo01.controller;
  
  import org.springframework.stereotype.Controller;
  import org.springframework.web.bind.annotation.RequestMapping;
  import org.springframework.web.bind.annotation.ResponseBody;
  
  /**
   * @Author: liufan
   * @Date: 2018/9/30 11:03
   * @Description:
   */
  @Controller
  public class HelloClass {
      @ResponseBody
      @RequestMapping("/hello")
      public String hello(){
          return "hello";
      }
  }
  
  ```

  ![pic3](C:\Users\A\Desktop\笔记\我的笔记_files\pic3.png)

+ 运行main函数测试即可



### 注：

==Mydemo01Application 这个启动类必须与controller这些package同级，这样springBoot的自动扫描才可以识别出里面的controller等组件，且启动类不能直接在根目录java下，否则会报错。==







