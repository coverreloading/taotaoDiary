#day03



##1.后台管理商品的添加功能

1. 商品分类选择

2. 上传图片

3. 富文本编辑器（kindEditor）

4. 实现商品的添加



##2.类目选择

###2.1 easyui Tree 异步获取

1. 初始化tree的url：

/item/cat/list

2. 请求的参数

Id（当前节点的id，根据此id查询子节点）

3. 返回数据的格式json数据：



```javascript

[{    

    "id": 1,    //当前节点的id

    "text": "Node 1",    //节点显示的名称

    "state": "closed"    //节点的状态，如果是closed就是一个文件夹形式，

                 		// 当打开时还会 做一次请求。如果是open就显示为叶子节点。

},{    

    "id": 2,    

    "text": "Node 2",    

    "state": "closed"   

}] 

```



###2.2 sql



```sql

SELECT * FROM `tb_item_cat` where parent_id=父节点id;

```

###2.3 Service层



1. 功能：根据parentId查询商品分类列表。

2. 参数：parentId

3. 返回值：返回tree所需要的数据结构，是一个节点列表。

可以创建一个tree node的pojo表示节点的数据，也可以使用map。

List<TreeNode>



####2.3.1 pojo: treenode

**通用pojo，放入common**



```java

public class TreeNode {

	

	private long id;

	private String text;

	private String state;

	

	public TreeNode(long id, String text, String state) {

		this.id = id;

		this.text = text;

		this.state = state;

	}

********get、set方法省略********

}

```



####2.3.2 @service



```java

@Service

public class ItemCatServiceImpl implements ItemCatService {

	@Autowired

	private TbItemCatMapper itemCatMapper;

	@Override

	public List<TreeNode> getItemCatList(long parentId) {

		//根据parentId查询分类列表

		TbItemCatExample example = new TbItemCatExample();

		//设置查询条件

		Criteria criteria = example.createCriteria();

		criteria.andParentIdEqualTo(parentId);

		//执行查询

		List<TbItemCat> list = itemCatMapper.selectByExample(example);

		//分类列表转换成TreeNode的列表

		List<TreeNode> resultList = new ArrayList<>();

		for (TbItemCat tbItemCat : list) {

			//创建一个TreeNode对象

			TreeNode node = new TreeNode(tbItemCat.getId(), tbItemCat.getName(), 

					tbItemCat.getIsParent()?"closed":"open");

			resultList.add(node);

		}

		

		return resultList;

	}

}

```

#### 2.3.3 @control



1. 功能：接收页面传递过来的id，作为parentId查询子节点。

2. 参数：Long id

3. 返回值：要返回json数据要使用@ResponseBody。List<TreeNode>



```java

@Controller

@RequestMapping("/item/cat")

public class ItemCatController {



	@Autowired

	private ItemCatService itemCatService;

	

	@RequestMapping("/list")

	@ResponseBody

	public List<TreeNode> getItemCatList(@RequestParam(value="id", defaultValue="0")Long parentId) {

		List<TreeNode> list = itemCatService.getItemCatList(parentId);

		return list;

	}

	

}

```

## 3 图片上传

### 3.1 传统与集群环境上传图片



1. 传统



2. 集群



### 3.2 nginx



**Nginx：http服务，反向代理，负载均衡**



centos7 安装nginx （笔记linux）



### 3.3 图片服务器



#### 3.3.1 搭建



**centos + nginx + vsftpd**



### 3.4 图片服务器配置

#### 3.4.1 测试ftp：

1. 客户端测试

2. 使用java代码访问ftp服务器，使用apache的FTPClient工具访问ftp服务器。需要在pom文件中添加依赖：



```xml

// common中

<dependency>

	<groupId>commons-net</groupId>

	<artifactId>commons-net</artifactId>

	<version>${commons-net.version}</version>

</dependency>

```

```java

public class FTPClientTest {

	@Test

	public void testFtp() throws Exception {

		// 1、连接ftp服务器

		FTPClient ftpClient = new FTPClient();

		ftpClient.connect("192.168.25.133", 21);

		// 2、登录ftp服务器

		ftpClient.login("ftpuser", "ftpuser");

		// 3、读取本地文件

		FileInputStream inputStream = new FileInputStream(new File("D:\\Documents\\Pictures\\images\\2010062119283578.jpg"));

		// 4、上传文件

		// 1）指定上传目录

		ftpClient.changeWorkingDirectory("/home/ftpuser/www/images");

		// 2）指定文件类型

		ftpClient.setFileType(FTPClient.BINARY_FILE_TYPE);

		// 3)第一个参数：文件在远程服务器的名称

		// 4)第二个参数：文件流

		ftpClient.storeFile("hello.jpg", inputStream);

		// 5、退出登录

		ftpClient.logout();

	}

}

```



### 3.5 图片上传实现

#### 3.5.1 需求分析

**Common.js**



1. 绑定事件

2. 初始化参数

3. url ： /pic/upload

4. 参数 ：uploadFile

5. 返回参数 ：json 

（富文本编辑器文档： http://kindeditor.net/docs/upload.html）



``` json

//成功时

{

        "error" : 0,

        "url" : "http://www.example.com/path/to/file.ext"

}

//失败时

{

        "error" : 1,

        "message" : "错误信息"

}

```

#### 3.5.2 Service



1. 功能：接收controller层传递过来的图片对象，把图片上传到ftp服务器。给图片生成一个新的名字。

2. 参数：MultiPartFile uploadFile

3. 返回值：返回一个Map，包含error与message。

4. ftp配置文件(resource.properties)



```resource.properties

#FTP相关配置

#FTP的ip地址

FTP_ADDRESS=192.168.1.134

FTP_PORT=21

FTP_USERNAME=ftpuser

FTP_PASSWORD=ftpuser

FTP_BASE_PATH=/home/ftpuser/www/images

#图片服务器的相关配置

#图片服务器的基础url

IMAGE_BASE_URL=http://192.168.1.134/images

#服务层基础url

REST_BASE_URL=http://192.168.1.134:8080/rest

REST_CONTENT_SYNC_URL=/cache/sync/content/

```



5. **修改applicationContext-dao.xml扫描加载文件改为location="classpath:resource/*.properties"以加载文件**



```java

// 通过@Value("${}")加载

@Value("${FTP_ADDRESS}")

private String FTP_ADDRESS;

```

6. 使用org.joda.time.DataeTime包简化日期:`new DateTime().toString("/yyyy/MM/dd");`

7. 对FtpUtil.uploadFile方法返回result,判断成功失败，并在resultMap插入error与message的值，作为返回前端json数据的map。



#### 3.5.3 Controller

1. code

```java

@Controller

public class PictureController {

	@Autowired

	private PictureService pictureService;

	@RequestMapping("/pic/upload")

	@ResponseBody

	public String pictureUpload(MultipartFile uploadFile) {

		Map result = pictureService.uploadPicture(uploadFile);

		//为了保证功能的兼容性，需要把Result转换成json格式的字符串。

		String json = JsonUtils.objectToJson(result);

		return json;

	}

}

```

2. springmvc.xml配置多部件解析器（引入 org.springframework.web.multipart.commons.CommonsMultipartResolver）



```xml

<!-- 定义文件上传解析器 -->	

<bean id="multipartResolver"class="org.springframework.web.multipart.commons.CommonsMultipartResolver">

	<!-- 设定默认编码 -->

	<property name="defaultEncoding" value="UTF-8"></property>

	<!-- 设定文件上传的最大值5MB，5*1024*1024 -->

	<property name="maxUploadSize" value="5242880"></property>

</bean>

```


## 4 富文本编辑器

### 4.1 使用方法

1. 引入js

2. 添加类型为textarea的input

 `<textarea style="width:800px;height:300px;visibility:hidden;" name="desc"></textarea>`

3. 调用js初始化

`itemAddEditor = KindEditor.create("#itemAddForm [name=desc]", TT.kingEditorParams);`

4. 同步方法sync();

5. 提交表单



```

$.post("/item/save",$("#itemAddForm").serialize(), function(data){

	if(data.status == 200){

    	$.messager.alert('提示','新增商品成功!');

	}

});

```

## 5 商品添加

1. pojo ( tb_item , tb_item_desc )

2. service

3. controller





















