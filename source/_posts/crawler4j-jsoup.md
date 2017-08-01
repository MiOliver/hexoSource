---
title: crawler4j+jsoup实现网页内容抓取
date: 2017-02-17 11:27:52
tags: 
- Crawler4j
- Jsoup
- JDBC
- 爬虫
categories: 
- tools
---

昨天花了一下午的时间，使用[Crawler4J](https://github.com/yasserg/crawler4j)+[Jsoup](https://jsoup.org/)完成了一个简单的数据抓取工作（其实楼主最近比较关注房源信息），效果见下图，废话不多说，看看怎么实现的。

{% asset_img crawler.png 抓取到的数据 %}


## 1.准备工作
- 新建一个maven项目，或者使用已有的maven项目
- pom.xml中添加Crawler4J
```
    <dependency>
        <groupId>edu.uci.ics</groupId>
        <artifactId>crawler4j</artifactId>
        <version>4.2</version>
    </dependency>
```
- pom.xml中添加Jsoup
```
    <dependency>
        <groupId>org.jsoup</groupId>
        <artifactId>jsoup</artifactId>
        <version>1.7.2</version>
    </dependency>
```
## 2.Crawler4J使用
关键两步：  
1.使用crawler4j，创建一个继承WebCrawler的爬虫类，具体的抓取逻辑在visit()方法中实现。  
2.实现控制器类指定抓取的种子（seed）、中间数据存储的文件夹、并发线程的数目：

github上给出了几个使用的例子，具体可以[参考](https://github.com/yasserg/crawler4j),此处不做赘述。

- Basic crawler：上述例子的全部源码及细节。
- Image crawler：一个简单的图片爬虫：从指定域下载图片并存在指定文件夹。这个例子演示了怎样用crawler4j抓取二进制内容。
- Collecting data from threads：这个例子演示了控制器怎样从抓取线程中收集数据/统计
- Multiple crawlers：这个例子演示了如何同时运行两个不同的爬虫。
- Shutdown crawling：这个例子演示了可以通过向控制器发送“shutdown”命令优雅的关闭抓取过程。   

## 3.JSoup使用
可以直接参考官方的[cookbook](https://jsoup.org/cookbook/).API文档讲的都很详细。

## 4.实例分享
- Controller
```
public class Controller {

    public static void main(String[] args) throws Exception {
        String crawlStorageFolder = "/Users/oliver/test/crawler";
        //设置要使用的Crawler线程数量
        int numberOfCrawlers = 1;

        CrawlConfig config = new CrawlConfig();
        config.setCrawlStorageFolder(crawlStorageFolder);

        /*
         * Instantiate the controller for this crawl.
         */
         
        PageFetcher pageFetcher = new PageFetcher(config);
        RobotstxtConfig robotstxtConfig = new RobotstxtConfig();
        //禁用robot.txt检查
        robotstxtConfig.setEnabled(false);
        RobotstxtServer robotsServer = new RobotstxtServer(robotstxtConfig, pageFetcher);
        
        CrawlController controller = new CrawlController(config, pageFetcher, robotsServer);

        //添加种子，你要扫描的网址
        controller.addSeed("http://website.com/");


        /*
         * Start the crawl. This is a blocking operation, meaning that your code
         * will reach the line after this only when crawling is finished.
         */
       controller.start(LjCrawler.class, numberOfCrawlers);
    }
}
```

- 自定义Crawler类，继承WebCrawler
```
public class LjCrawler extends WebCrawler {

    private static final Logger logger= LoggerFactory.getLogger(LjCrawler.class);

    private final static Pattern FILTERS = Pattern.compile(".*(\\.(css|js|gif|jpg"
            + "|png|mp3|mp3|zip|gz))$");

    @Autowired
    private BlogCategoryMapper blogCategoryMapper;

    private HouseRecordMapper houseRecordMapper;

    public LjCrawler() {
        houseRecordMapper =SpringBeanFactoryUtils.getApplicationContext().getBean(HouseRecordMapper.class);
    }

    @Override
    public boolean shouldVisit(Page referringPage, WebURL url) {
        String href = url.getURL().toLowerCase();
        return !FILTERS.matcher(href).matches()
                && href.startsWith("http://yourwebsite.com");
    }

    @Override
    public void visit(Page page) {
        String url = page.getWebURL().getURL();
        logger.info("URL: " + url);
        if (page.getParseData() instanceof HtmlParseData) {
            HtmlParseData htmlParseData = (HtmlParseData) page.getParseData();
            String html = htmlParseData.getHtml();
            Document document= Jsoup.parse(html);
            Elements elements= document.body().getElementsByClass("sellListContent");
            for(Element e:elements){
                HouseRecord record=new HouseRecord();
                Elements info= e.getElementsByClass("clear").get(1).getAllElements();
            
            //todo  your process 
            
            }
        }
    }
}
```