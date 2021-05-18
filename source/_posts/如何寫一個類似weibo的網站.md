---
title: 如何寫一個類似weibo的網站
tags: springboot
category: Java
---

**爲什麼俺想寫一個類似weibo的網站呢？**

1. 想在實踐中學習，看看自己能否寫出一點好玩的東西
2. 俺很尊敬的一個人他也寫了一個，給了俺思路
3. 好玩，彷彿自己的賽博之地。沒有刪貼、沒有禁言、沒有消失 

<!--more-->

## 準備工作

創建springboot項目，[網址](https://start.spring.io/)

添加各種依賴。依賴詳情參見pom.xml文件



### 數據庫表的創建

創建數據庫表

user：

```mysql
CREATE TABLE `user` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(40) NOT NULL,
  `email` varchar(60) NOT NULL,
  `password` varchar(40) NOT NULL,
  `crt_time` datetime NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=2 DEFAULT CHARSET=utf8
```

micropost：

```mysql
CREATE TABLE `micropost` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `user_id` int(11) NOT NULL,
  `content` text NOT NULL,
  `crt_time` datetime NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=2 DEFAULT CHARSET=utf8
```



最初計劃實現關注功能，所以後期可能表會修改並且添加新的表，先留坑。



## 1、創建eneity&mapper以及Mybatis映射

首先創建實體類以及mapper類，代碼可以查看[Github](https://github.com/WangBeishan/microposts)

mybatis映射xml文件目錄必須與src文件mapper文件目錄結構相同，如圖：

![src](/home/wbs/Pictures/Screenshot from 2021-03-31 20-57-00.png)

![resources](/home/wbs/Pictures/Screenshot from 2021-03-31 20-57-15.png)



## 2、service & controller編寫

控制層這塊其實挺迷糊的，寫這篇博文的目的之一也是想屢屢知識點。

而service層，由於暫時業務上更改需求不高，暫時沒領會到service層的必要性，不過都成了「傳統」，俺也先這麼寫。



控制層這塊開始想分成幾個類來寫，分別實現不同類的功能，比如用戶控制類，博文控制類，系統控制類等。

現在所採用的方式是通過servlet接受發送請求。由於前端頁面需要判斷登錄狀態，所以在登錄成功后，後臺會創建一個新的session（假如沒有的話），通過每次請求session判斷該用戶是否已登錄來展示不同的html標籤。

登出功能就是從session里remove記錄的值。

用戶和博文控制暫時實現了基本的增刪功能。



## 3、攔截器配置

用戶未登錄前無法獲取某些請求和頁面，如獲取所有用戶等。

登錄攔截器：

```java
public class LoginInterceptor implements HandlerInterceptor {

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {

        HttpSession session = request.getSession();
        Object userId = session.getAttribute("userId");

        if(null == userId) {
            return false;
        } else {
            return true;
        }
    }
}
```

WebMvcConfig配置：

```java
@Configuration
public class WebMvcConfig implements WebMvcConfigurer {

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new LoginInterceptor())
                .addPathPatterns("/**")
                .excludePathPatterns("/login", "/", "/index", "/about",
                        "/contact", "/signupStatus",
                        "/loginStatus", "/login", "/signup",
                        "/help", "/css/**", "/images/**");
    }
}
```



## 4、分頁功能

對博文和所有用戶列表進行分頁，在pom.xml中添加以下依賴：

```xml
<dependency>
    <groupId>com.github.pagehelper</groupId>
    <artifactId>pagehelper-spring-boot-starter</artifactId>
    <version>1.3.0</version>
</dependency>
```

項目[Github](https://github.com/pagehelper/Mybatis-PageHelper)

```java
@GetMapping("/microposts")
    public void getAllPM(Model model,
                         @RequestParam(required = false, defaultValue = "1", value = "pageNum") Integer pageNum,
                         @RequestParam(defaultValue = "10", value = "pageSize") Integer pageSize,
                         HttpServletResponse response) {

        if (pageNum == null || pageNum <= 0) {
            pageNum = 1;
        }
        if (pageSize == null) {
            pageSize = 10;
        }
        PageHelper.startPage(pageNum, pageSize);
        try {
            List<Micropost> microposts = micropostService.getAllMP();
            PageInfo<Micropost> pageInfo = new PageInfo<>(microposts);
            model.addAttribute("pageInfo", pageInfo);
        } finally {
            PageHelper.clearPage();
        }
    }
```

以此方法爲例，主要有兩個參數：

pageNum：顯示的頁數（不是必須）

pageSize：每頁顯示多少條博文

將PageInfo添加到model里，前端就可以訪問了。更詳細用法參見[這裏](https://github.com/pagehelper/Mybatis-PageHelper/blob/master/wikis/en/HowToUse.md)	



## 後記

前端以model的方式從後端獲取數據，css還未寫，暫時只是簡單的界面。

後續可能會添加的東西：

註冊時郵箱的驗證、css樣式、關注與被關注功能、主頁隨機獲取未關注者的博文

暫時就這麼多，挖個坑，希望可以完成。



項目Github地址：[microposts](https://github.com/WangBeishan/microposts)
