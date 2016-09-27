## 跨域:

1. 不同域名

2. 不同端口

**ajax不能跨域请求,使用jsonp解决跨域问题**

>jsonp:ajax不能跨域访问json,但能跨域访问js

`$.getJSONP(this.URL_Serv, category.getDataService);`

将json数据保存在js中,jsonp调用js中方法eg:category.getDataService(),获取到json数据







## 商品列表

### service

`public CatResult getItemCatList() ;`

`private List<?> getCatList(long parentId) `

### controller

```java

@RequestMapping("/itemcat/list")

     @ResponseBody

     public Object getItemCatList(String callback) {

          CatResult catResult = itemCatService.getItemCatList();

          MappingJacksonValue mappingJacksonValue = new MappingJacksonValue(catResult);

          mappingJacksonValue.setJsonpFunction(callback);

          return mappingJacksonValue;

     } 

```