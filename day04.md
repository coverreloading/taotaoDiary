# 规格参数



## 1. 商品规格

### 1.1实现方法1，建表

#### 1.1.1 sql代码



```sql

// 1

SELECT

*

FROM tb_item_param_key pk ,tb_item_param_value pv ,tb_item_param_group pg  

WHERE 

item_id=855739 and pv.param_id=pk.id and pk.group_id = pg.id;

// 2

SELECT   

pg.group_name ,pk.param_name , pv.param_value

FROM 

tb_item_param_value pv LEFT JOIN tb_item_param_key pk ON pv.param_id=pk.id LEFT JOIN  tb_item_param_group pg ON pk.group_id=pg.id

WHERE 

item_id = 855739;

```

#### 1.1.2 存在问题

1. 多张表

2. 复杂sql

3. 关联查询

4. 新添加商品规格不可变



### 1.2 实现方法2，模板

1. 每一个商品分类对一个规格参数模板。



```json

[

    {

        "group": "主体",  //组名称

        "params": [ // 记录规格成员

            "品牌",

            "型号",

            "颜色",

            "上市年份",

            "上市月份"

        ]

}，

{

        "group": "网络",  //组名称

        "params": [ // 记录规格成员

            "4G",

            "3G,

            "2G"

        ]

}

]

```



2. 使用模板

每个商品对应一唯一的规格参数。在添加商品时，可以根据规格参数的模板。生成一个表单。保存规格参数时。还可以生成规格参数的json数据。保存到数据库中。



```json

[

    {

        "group": "主体",

        "params": [

            {

                "k": "品牌",

                "v": "苹果（Apple）"

            },

            {

                "k": "型号",

                "v": "iPhone 6 A1589"

            },

{

                "k": "智能机",

                "v": "是 "

            }



        ]

}

]

```

## 2.创建规格参数

### 2.1 选择商品分类

#### 2.1.1 功能分析

1. 请求url：/item/param/query/itemcatid/{itemCatId}

2. 参数:itemCarId,从url中获取

3. 返回值:TaotaoResult

#### 2.1.2 Dao

1. 从tb_item_param表中根据商品分类id查询内容。

单表操作。可以使用逆向工程的代码。





#### 2.1.3 Service

1. 功能:接受商品分类ID,调用mapper查询tb_item_param 表,返回结果TaotaoResult

```java

@Autowired

	private TbItemParamMapper itemParamMapper;

	

	@Override

	public TaotaoResult getItemParamByCid(long cid) {

		TbItemParamExample example = new TbItemParamExample();

		Criteria criteria = example.createCriteria();

		criteria.andItemCatIdEqualTo(cid);

		List<TbItemParam> list = itemParamMapper.selectByExample(example);		//判断是否查询到结果

		if (list != null && list.size() > 0) {

			return TaotaoResult.ok(list.get(0));

		}

		

		return TaotaoResult.ok();

	}

```

#### 2.1.4 Controller



```java

@Autowired

	private ItemParamService itemParamService;

	

	@RequestMapping("/query/itemcatid/{itemCatId}")

	@ResponseBody

	public TaotaoResult getItemParamByCid(@PathVariable Long itemCatId) {

		TaotaoResult result = itemParamService.getItemParamByCid(itemCatId);

		return result;

	}

```

### 2.2提交规格参数

#### 2.2.1 需求分析

1. 页面文本框内容转换为json,提交后台,保存到param规格表



2. 请求的url:/item/param/save/{cid}



3. 参数: String paramData



返回值:TaotaoResult

#### 2.2.2 Dao

**保存规格模板**

#### 2.2.3 Service

**接受TbItemParam,调用mapper插入param表,放回TaotaoResult**



```java

@Override

public TaotaoResult insertItemParam(TbItemParam itemParam) {

   //补全pojo

   itemParam.setCreated(new Date());

   itemParam.setUpdated(new Date());

   //插入到规格参数模板表

   itemParamMapper.insert(itemParam);

   return TaotaoResult.ok();

}

```

#### 2.2.4 Controller

**功能：接收cid、规格参数模板。创建一TbItemParam对象。调用Service返回TaotaoResult。返回json数据**



```java

@RequestMapping("/save/{cid}")

	@ResponseBody

	public TaotaoResult insertItemParam(@PathVariable Long cid, String paramData) {

		//创建pojo对象

		TbItemParam itemParam = new TbItemParam();

		itemParam.setItemCatId(cid);

		itemParam.setParamData(paramData);

		TaotaoResult result = itemParamService.insertItemParam(itemParam);

		return result;

	}

```

## 3 根据商品规格生成表单

1. 





2. service修改

selectByExampleWithBLOBs方法,包含大文本




## 4 保存商品规格参数

### 4.1 需求分析

1. 将规格参数表单内容转化为json后跟商品基本信息描述同时提交后台.

### 4.2 Dao

1. 向tb_item_param_item添加数据

### 4.3 Service

1. 接收规格参数内容,和商品id,以pojo形式调用mapper插入表,返回TaotaoResult.ok()

2. code



```java

private TaotaoResult insertItemParamItem(Long itemId, String itemParam) {

		//创建一个pojo

		TbItemParamItem itemParamItem = new TbItemParamItem();

		itemParamItem.setItemId(itemId);

		itemParamItem.setParamData(itemParam);

		itemParamItem.setCreated(new Date());

		itemParamItem.setUpdated(new Date());

		//向表中插入数据

		itemParamItemMapper.insert(itemParamItem);

		

		return TaotaoResult.ok();

		

	}

```

### 4.4 Controller



```java

	@RequestMapping(value="/item/save", method=RequestMethod.POST)

	@ResponseBody

	private TaotaoResult createItem(TbItem item, String desc, String itemParams) throws Exception {

		TaotaoResult result = itemService.createItem(item, desc, itemParams);

		return result;

	}

```



































































