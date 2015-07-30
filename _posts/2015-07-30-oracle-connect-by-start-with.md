---
layout: post
title:  "使用 connect by 遍历树状结构的表"
keywords: "oracle connectby"
description: "通过START WITH . . . CONNECT BY . . .子句来实现SQL的层次查询."
category: oracle
tags: oracle
---
###基本语法
connect by结构化查询的基本语法：

```sql
	select [level], column, expr... from tablename
	start with cond1
	connect by  prior cond2
	where cond3
```

用上述语法的查询可以取得这棵树的所有记录。
其中：
`level`关键字是可选的，表示等级，1表示`root`,2表示`root`的`child`,其他相同的规则。
`cond1`是根结点的限定语句，当然可以放宽限定条件，以取得多个根结点，实际就是多棵树。
`cond2`是连接条件，其中用prior表示上一条记录，比如`connect by prior id=praentid`就是说上一条记录的id是本条记录的`praentid`，即本记录的父亲是上一条记录。
`cond3`是过滤条件，用于对返回的所有记录进行过滤。
`prior`和`start with`关键字是可选项
`connect by prior`是指定父子关系，其中`prior`的位置不一定要在`connect by`之后，对于一个真实的层次关系，这也是必须的。
`prior`运算符必须放置在连接关系的两列中某一个的前面。对于节点间的父子关系，`prior`运算符在一侧表示父节点，在另一侧表示子节点，从而确定查找树结构是的顺序是自顶向下还是自底向上。在连接关系中，除了可以使用列名外，还允许使用列表达式。
`start with`子句为可选项，用来标识哪个节点作为查找树型结构的根节点。若该子句被省略，则表示所有满足查询条件的行作为根节点

###举个栗子

现在有一张`Menu`表,唯一主键`Id`，还有一个`parentId`属性，表示它的父节点的`Id`

表结构如下：

```sql
create table SYS_MENU
(
   ID                   NUMBER(10)  primary key ,
   MENU_CODE,			VARCHAR2(20),
   MENU_NAME            VARCHAR2(20),
   PARENT_ID 			NUMBER(10)
)
```

查询语句

```sql
SELECT
	ID,
	MENU_CODE,
	MENU_NAME,
	PARENT_CODE,
	LEVEL
FROM
	SYS_MENU T
WHERE
	MENU_STATUS = 1 CONNECT BY PRIOR MENU_CODE = PARENT_CODE START WITH PARENT_CODE = '1' ORDER SIBLINGS BY MENU_ORDER ASC,
	MENU_CODE ASC;
select * from SYS_MENU;
```

结果如下：

```
	ID  MENU_CODE 	MENU_NAME   PARENTCODE LEVEL
	1	1001		系统管理		1			1
	2	1001001		系统管理		1001		2
	3	1001001001	菜单管理		1001001		3
	5	1001001003	操作管理		1001001		3
	4	1001001002	角色管理		1001001		3
	6	1001001006	用户管理		1001001		3
	7	1001001009	部门管理		1001001		3
	59	1001001010	常量管理		1001001		3
	60	1002		公众号管理	1			1
	61	1002001		账号管理		1002		2
	62	1002001001	账号列表		1002001		3
	63	1003		微信菜单		1			1
	64	1003001		微信菜单		1003		2
	65	1003001001	菜单列表		1003001		3
	200	1004		消息管理		1			1
	201	1004001		消息管理		1004		2
	8	1004001004	关注消息管理	1004001		3
	223	1004001005	群发消息管理	1004001		3
	250	1004001006	用户消息管理	1004001		3
	202	1005		微信机器人	1			1
	203	1005001		自动回答		1005		2
	204	1005001001	内容管理		1005001		3
	205	1006		公众号素材	1			1
	206	1006001		素材管理		1006		2
	207	1006001001	素材列表		1006001		3
	208	1007		粉丝关注数	1			1
	209	1007001		分组管理		1007		2
	210	1007001001	分组管理		1007001		3
	211	1007001002	关注者列表	1007001		3
	220	1008		生成二维码	1			1
	221	1008001		二维码管理	1008		2
	222	1008001001	二维码管理	1008001		3
	260	1008001002	创建二维码	1008001		3
	230	1009		支付管理		1			1
	231	1009001		支付管理		1009		2
	240	1009001001	订单管理		1009001		3
	290	1009001003	活动配置		1009001		3
	280	1009001002	自动充值导出	1009001		3
```

###使用SIBLINGS关键字排序

对于层次查询如果用`order by`排序，比如`order by MENU_CODE`则是先做完层次获得`level`,然后按`MENU_CODE`排序，这样破坏了层次，比如特别关注某行的深度，按level排序，也是会破坏层次的。
在oracle10g中，增加了`siblings`关键字的排序。
语法：`order  siblings  by <expre>`
它会保护层次，并且在每个等级中按`expre`排序。

