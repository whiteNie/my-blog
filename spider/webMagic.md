# webMagic

## 前言

>  现在提及**“网络爬虫”**基本上想到的都是 *Python* ，但是博主是一名 *Java* 开发，因为一个很小的需求去学习 *Python* 这个工作成本还是很大的，我想应该不止我一个人面临这样的问题，我就试着去搜索 ***Java***，***成熟爬虫框架***，***GitHub*** 这些关键词，果不其然 ------> [GitHub爬虫项目](https://www.zhihu.com/question/31427895)	
>
> 博主当时看到 *webMagic* 的介绍很不错，于是就是选择了这个框架去学习。

## 简介

*webMagic* 是一个 *Java* 垂直式开源爬虫框架。*webMagic* 项目分为两部分：**核心**，**拓展**。

### 核心

​	核心部分(*webmagic-core*)是一个精简的、模块化的爬虫实现。

### 组件

 *webMagic* 由四大组件构成：*Downloader*,  *PageProcessor*, *Scheduler*, *Pipeline*。并且由 *Spider* 将他们组织起来。

* Downloder

  > 负责从互联网上下载数据

* PageProcessor

  > 

### 拓展

​	扩展部分则包括一些便利的、实用性的功能。

更详细的还需要去阅读 [webMagic官网文档传送门](http://webmagic.io/docs/zh/posts/ch1-overview/thinking.html)。

## 简单使用

### 官网推荐使用 maven 

```java
<dependency>
  <groupId>us.codecraft</groupId>
  <artifactId>webmagic-core</artifactId>
  <version>0.7.3</version>
</dependency>
<dependency>
  <groupId>us.codecraft</groupId>
  <artifactId>webmagic-extension</artifactId>
  <version>0.7.3</version>
</dependency>
```

### webMagic demo1

```java
import us.codecraft.webmagic.Page;
import us.codecraft.webmagic.Site;
import us.codecraft.webmagic.Spider;
import us.codecraft.webmagic.processor.PageProcessor;

/**
 * @author niec
 * @description webMagic demo project
 * @date 2020-11-15
 */
public class GithubRepoPageProcessor implements PageProcessor {

    /**
     * 抓取网站的相关配置：
     * 编码，
     * 重试机制，
     * 重试次数等
     */
    private Site site = Site.me().setRetryTimes(3).setSleepTime(100);

    /**
     * process 是定制爬虫逻辑的核心接口，在这里编写抽取逻辑
     *
     * @param page 页面
     */
    @Override
    public void process(Page page) {
        // 定义如何抽取页面信息，并保存
        page.putField("author",
                page.getUrl().regex("https://github\\.com/(\\w+)/.*").toString()
        );
        page.putField("name",
                page.getHtml().xpath("//h1[@class='entry-title public']/strong/a/text()").toString()
        );
        if (page.getResultItems().get("name") == null) {
            //skip this page
            page.setSkip(true);
        }
        page.putField("readme", page.getHtml().xpath("//div[@id='readme']/tidyText()"));
        // 从页面发现后续的 url 地址来抓取
        page.addTargetRequests(
                page.getHtml()
                        .links()
                        .regex("(https://github\\.com/\\w+/\\w+)")
                        .all()
        );
    }


    @Override
    public Site getSite() {
        return null;
    }

    public static void main(String[] args) {
        Spider.create(new GithubRepoPageProcessor())
                .addUrl("https://github.com/code4craft")
                .thread(5)
                .run();

    }


}
```



### 编写基本的爬虫

在[webMagic demo1](#webMagic demo1)项目中可以看出，我们将一个爬虫项目分为了三部分，但是在实际的场景中我们编写一个爬虫项目可不仅仅只是这三个部分，下面我们来学习一个完整的爬虫项目需要了解并掌握的部分：

#### 实现PageProcessor

##### 爬虫配置

编码、重试机制、重试次数等。

***待完成***



##### 页面元素的抽取

页面元素的抓取是最核心的部分，对于我们下载的 HTML 页面，我们要如何操作才能得到我们想要的信息呢？

webMagic 提供了三种提取页面元素的方式：XPath, Class 选择器, 正则表达式。









