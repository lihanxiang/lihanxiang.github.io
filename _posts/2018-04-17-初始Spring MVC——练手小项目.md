---
layout:     post
title:      初始Spring MVC——练手小项目
subtitle:   练手项目
date:	    2018-04-17
author:	    L
header-img: "img/post/2018-04/17s.jpg"
tags:
    - Spring MVC
    - Java Web
---

### 初始Spring MVC

> 前几天开始了我的spring学习之旅，由于之前使用过MVC模式来做项目，所以我先下手的是 Spring MVC，做个练手项目，非常简单

##### 项目介绍：

 用户输入信息 -> 后台处理 -> 输出信息

##### 开始

1. 创建Spring MVC 项目（创建时下载所需文件）

![](https://upload-images.jianshu.io/upload_images/3426615-d084130830d4db65.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
<br>
2.创建完的项目目录是这样的

![](https://upload-images.jianshu.io/upload_images/3426615-78e04adbbf1a2c54.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
<br>
3. 配置Web项目结构

参考另一篇文章[IDEA如何创建及配置Web项目（多图）](https://www.jianshu.com/p/8d49d36a3c7e)

有变化的是： 不需要自己创建 **lib** 文件夹，在创建项目时已经建立，另外在 **WEB-INF** 建立 **jsp** 用于存放 **.jsp** 文件，其他的都没什么区别

配置完的样子（文件夹旁有箭头是因为我在做完项目才截图，具体的类以及其他文件都已在内）：

![](https://upload-images.jianshu.io/upload_images/3426615-e4e64f432bfe4d12.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
<br>

4. 配置Spring MVC的核心 —— **dispatcher-servlet.xml**
>Spring MVC provides an annotation-based programming model where **@Controller** and **@RestController** components use annotations to express request mappings, request input, exception handling, and more. Annotated controllers have flexible method signatures and do not have to extend base classes nor implement specific interfaces.

如官方文档所说，我也使用了注解来定义控制器（Controller）和服务（Service）.

首先需要添加的是
 ```
<context:component-scan base-package="controller"/>
<context:component-scan base-package="service"/>
```
这样我们就能通过注解来定义**Controller**以及**Service**.

然后，我们需要配置视图解析器，在具体操作时只要写出视图的名称(xxx)，就可以采用URL拼接的方式，达到这种效果：/WEB-INF/jsp/xxx.jsp

```
<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
    <property name="prefix" value="/WEB-INF/jsp/"/>
    <property name="suffix" value=".jsp"/>
</bean>
```

5.  开始进行具体的操作

首先是基本数据类：
```
public class Product {
    private long id;
    private String name;
    private String price;
    private String inventory;

    public void setId(long id) {
        this.id = id;
    }
    public long getId() {
        return id;
    }

    public void setName(String name) {
        this.name = name;
    }
    public String getName() { return name; }
    public void setPrice(String price) {
        this.price = price;
    }
    public String getPrice() { return price; }
    public void setInventory(String inventory) {
        this.inventory = inventory;
    }
    public String getInventory() { return inventory; }
}
```
<br>

接下来是收集用户输入所需要的表单类：
```
public class ProductForm {
    private String name;
    private String price;
    private String inventory;
    public void setName(String name) {
        this.name = name;
    }

    public void setPrice(String price) {
        this.price = price;
    }

    public void setInventory(String inventory) {
        this.inventory = inventory;
    }

    public String getName() {
        return name;
    }
    public String getPrice() {
        return price;
    }
    public String getInventory() {
        return inventory;
    }
}
```
<br>

然后编写服务接口，以及它的实现类

```
import domain.Product;

public interface ProductService {
    Product add(Product product);
    Product get(long id);
}

```

**ProductServiceImpl**就是刚才配置的服务（Service）

```
import domain.Product;
import org.springframework.stereotype.Service;
import java.util.HashMap;
import java.util.Map;
import java.util.concurrent.atomic.AtomicLong;

@Service
public class ProductServiceImpl implements ProductService{
    private Map<Long, Product> productMap = new HashMap<>();
    private AtomicLong atomicLong = new AtomicLong();

    @Override
    public Product add(Product product){
        long id = atomicLong.incrementAndGet();
        product.setId(id);
        productMap.put(id, product);
        return product;
    }

    @Override
    public Product get(long id){
        return productMap.get(id);
    }
}
```
<br>

最重要的是**控制器**：

采用**@Autowired**注解的方式自动装配Service，
使用**@RequestMapping**注解的方式，将注解中的路径映射到控制器：
```
import domain.Product;
import form.ProductForm;
import org.apache.commons.logging.Log;
import org.apache.commons.logging.LogFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.servlet.mvc.support.RedirectAttributes;
import service.ProductService;

@Controller
public class ProductController {
    //日志记录具体操作
    private static final Log logger = LogFactory.getLog(ProductController.class);

    @Autowired
    private  ProductService productService;

    //method = RequestMethod.POST表示只接受POST形式的请求
    @RequestMapping(value = "/product_save", method = RequestMethod.POST)
    //采用POST的方式发送请求
    public String saveProduct(ProductForm productForm, RedirectAttributes redirectAttributes){
        logger.info("saveProduct called");

        //获得用户输入
        Product product = new Product();
        product.setName(productForm.getName());
        product.setPrice(productForm.getPrice());
        product.setInventory(productForm.getInventory());

        //添加有记录的产品，并且根据ID进行重定向
        Product savedProduct = productService.add(product);
        redirectAttributes.addFlashAttribute("message", "Add product Successfully");
        return "redirect:/product_view/" + savedProduct.getId();
    }

    //根据ID，将用户输入展示在ProductView中
    @RequestMapping(value = "product_view/{id}")
    public String viewProduct(@PathVariable Long id, Model model){
        //根据ID得到信息
        Product product = productService.get(id);
        model.addAttribute("product", product);
        //ProductView.jsp
        return "ProductView";
    }
}
```
<br>
需要注意的是，在现在的Spring版本中，如果直接对Service进行注解，将会有产生警告：
![](https://upload-images.jianshu.io/upload_images/3426615-5c63d619a6d6712e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

按住Alt + Enter 修正错误，会看见提示：

![](https://upload-images.jianshu.io/upload_images/3426615-4b63db8e4d3d88fe.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

官方推荐使用构造器注入

到此为止，后台工作就结束了

6. 页面

接下来是JSP的编写：

首先是用户输入的页面 (ProductForm.jsp)

```
<body>
    <div>
        <form action="product_save" method="post">
            <fieldset>
                <legend>Add a product</legend>
                <p>
                    <label for="name">Product Name: </label>
                    <input type="text" id="name" name="name">
                </p>
                <p>
                    <label for="price">Price: </label>
                    <input type="text" id="price" name="price">
                </p>
                <p>
                    <label for="inventory">Inventory: </label>
                    <input type="text" id="inventory" name="inventory">
                </p>
                <p>
                    <input id="reset" type="reset" tabindex="4">
                    <input id="submit" type="submit" tabindex="5" value="Add Product">
                </p>
            </fieldset>
        </form>
    </div>
</body>
```
比较表单中的**<form action="product_save">**
和控制器中的**@RequestMapping(value = "/product_save")**就能够知道，通过@RequestMapping注解，任何**"/product_save"**开头的路径都会被映射到控制器中，并采用saveProduct()方法

然后是显示用户输入的页面 (ProductView.jsp)

```
<body>
    <div>
        <h3>${message}</h3>
        <h4>Details:</h4>
        Product Name: ${product.name}<br>
        Price: ${product.price}<br>
        Inventory: ${product.inventory}<br>
    </div>
</body>
```
${message}就是刚才在控制器中重定向页面的属性，它会在页面头部输出"Add product Successfully"

##### 项目构建完毕

最终的目录结构是这样：

![](https://upload-images.jianshu.io/upload_images/3426615-3c9d0a81f74d2c04.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

然后我们运行， 输入：

![](https://upload-images.jianshu.io/upload_images/3426615-70edad81b60fb0a7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


页面难看，主要是用Spring MVC做出来的就可以了

结果是这样的：

![](https://upload-images.jianshu.io/upload_images/3426615-97cced76b8bcbbe2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

目光移至URL，填写信息提交后，页面重定向至 **product_view/{id}** 处，当前id = 1

到处，我们的这个练手的小项目就结束了，有什么问题都可以私信我

