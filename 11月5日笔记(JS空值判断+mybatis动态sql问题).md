## JS中对于一群输入框空值的判断

以前都是使用表单的required属性，或者使用框架的属性，今天尝试了一下，可以从父元素找寻一类input获取其value进行验证：

```javascript
$("#contactForm").find(" input").each(function (index,item) {
            if($(item).val()==''||typeof ($(item).val())=='undefined'){
                mark=1;
            }
        })
```

有时候获取val()的时候会出现undefined，打印也是undefined，可是它的字符串与undefined比对结果却是false。这时候可以使用typeof进行验证。

## myBatis的动态sql问题（multi-statement not allow）

之前一直在思考如何对于一大串需要存入数据库的json数据的存储问题，想要尽可能的降低访问数据库的频率。可是后来发现暂时没这个水平，本来打算使用mybatis的动态sql拼装语句，然而动态sql大多数时候还是用于拼装一条sql语句，对于有级联关系的多条sql语句还不如在service层中循环存储。

当然这里还是记录一下多条sql语句处理在Druid数据源下的一个小问题：

会报错multi-statement not allow问题，只需要配置一下Druid即可：

```java
@Autowired
    WallFilter wallFilter;

    @Bean(name = "dataSource", destroyMethod = "close")    //声明其为Bean实例
    @Primary  //在同样的DataSource中，首先使用被标注的DataSource
    @ConfigurationProperties(prefix = "spring.datasource")
    public DataSource dataSource(){
        DruidDataSource datasource = new DruidDataSource();

        // filter
        List<Filter> filters = new ArrayList<>();
        filters.add(wallFilter);
        datasource.setProxyFilters(filters);

        return datasource;
    }

    @Bean(name = "wallFilter")
    @DependsOn("wallConfig")
    public WallFilter wallFilter(WallConfig wallConfig){
        WallFilter wallFilter = new WallFilter();
        wallFilter.setConfig(wallConfig);
        return wallFilter;
    }

    @Bean(name = "wallConfig")
    public WallConfig wallConfig(){
        WallConfig wallConfig = new WallConfig();
        wallConfig.setMultiStatementAllow(true);//允许一次执行多条语句
        wallConfig.setNoneBaseStatementAllow(true);//允许一次执行多条语句
        return wallConfig;
    }

```

