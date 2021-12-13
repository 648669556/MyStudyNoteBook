查询书库列表

```sql
SELECT * FROM `stackroom`;
```

新增书库

```sql
INSERT INTO stackroom(type) VALUE(#{type});
```

修改书库类型

```sql
UPDATE stackroom SET type=#{type} where stack_id=#{id};
```

删除书库

```sql
DELETE FROM stackroom WHERE stack_id=#{id};
```

查询书库数量

```sql
SELECt count(1) FROM stackroom;
```



------

查询书本类型数量

```sql
SELECT count(0) FROM books;
```

查询全部书本类型 （分页）(分类)（动态SQL实现）

```sql
--从index(从0开始)开始向后lenth个
SELECT b_id,`name`,publisher,author,price,number,type FROM books JOIN stackroom ON books.stackid=stackroom.stack_id 
<if test="stackid!=null">	
 WHERE stackid = #{stackid}
</if>
<if test="index != null and length!=null">
  limit #{index},#{length};
</if>
```

查询某一本书通过书名

```sql
SELECT b_id,`name`,publisher,author,price,number,type FROM books JOIN stackroom ON books.stackid=stackroom.stack_id WHERE `name` like %#{bookname}% limit #{index},#{length}
```

新增一本书类型

```sql
INSERT INTO books(`name`,publisher,author,price,number,stackid)
VALUES(#{bookname},#{publisher},#{author},#{price},#{number},#{stackid});
```

修改一本书类型

```sql
UPDATE books set 
`name` = #{bookname},
publisher = #{publisher},
author = #{author},
price = #{price}
WHERE
b_id = #{bookid};
```

删除书本类型

```sql
Delete From books where b_id=#{bookid}
```



------

实体书与书类别对应 新增书本

```sql
INSERT INTO san(san,bid)VALUES(#{uuid},#{bid});
```

```sql
UPdate books Set number=number+1 where bid=#{bid}
```

-------

查询一本未借出的实体书

```sql
SELECT san FROM san WHERE isborrow=1 AND bid=#{bookid} limit  1;
```

------

#### 借书

borrow表的更新

```sql
INSERT INTO borrow(userid,sanid,borrow_time)
VALUES(#{userid},#{sanid},#{borrow_time});
```

san表的更新

```sql
UPDATE san SET
isborrow = 0;
WHERE san =#{sanid};
```

books表的更新

```sql
UPDATE books SET number=number-1 WHERE b_id=#{bookid};
```

#### 还书

查询用户要还的书

```sql
SELECT b_id,`name`,publisher,author,price,isborrow FROM books
JOIN san ON books.b_id=san.bid
WHERE san.san IN
(
SELECT san FROM borrow 
WHERE borrow.userid=#{userid}
)

```

更新books表

```sql
UPDATE books SET number=number+1 WHERE b_id=#{bookid};
```

更新borrow表

```sql
UPDATE borrow SET return_time=#{time} where sanid=#{sanid}
```

更新san表

```sql
UPDATE san SET isborrow=1 where san.san=#{sanid}
```

#### 查询书库

查询

-------

## 接口查询

1. 搜索书籍通过书本名称

   - 请求方式：post

   - 请求的url：

     project/customer

   - 请求参数

     ```json
     {
         type:"searchBybook"//查询的类型
         data:{
         stackbook:1,//要查询的书库的id
         name:"xxxxx"//要查询的书本名字
         page:1,
         length:10
     }
     }
     ```

     

   - 返回参数

     - ```json
       {code:0,//返回的状态码，是否查询成功 0：成功 1：未登录 2：服务器错误
        msg:"",//如果查询失败返回查询失败的原因
        data{//查询成功的数组
        {name:"",price:5,xx},
       {name:"",price:5,xx},
       {name:"",price:5,xx}
       }
       }
       ```

     - 

2. 用户信息获取

   - 请求方式：post
   - 请求的url
     - project/customer
   - 请求参数

   ```JSON
   {
       type:"getuserinfo",
       data{
   }
   }
   ```

   - 返回参数

   ```json
   {
       code:0,
       msg:"",
       data{
       headimg:"http://192.168.1.1/index/file?fdsaf",
       username:"小帅哥"
   }
   }
   ```

3. 书库类别获取

   - 请求方式 post

   - 请求地址：project/customer

   - 请求参数

     ```json
     {
         type:"getBookStack",
         data{
         page:1,
         length:2
     }
     }
     ```

   - 返回参数

     ```json
     {
         code:0,
         msg:"",
         data{
         {stackid:1,stackname:"科学"},
         {stackid:2,stackname:"科学"},
         {stackid:3,stackname:"科学"}
     }
     }
     ```

   4.获取图书列表

   - ​	请求方式：post

   - 请求地址：project/customer

   - 请求参数

     ```json
     {
         type:"searchBybook"//查询的类型
         data:{
         stackbook:1,//要查询的书库的id stackbook 也为空则获取全部的
         page:1,
         length:10
     }
     }
     ```

   - 返回参数

     ```json
     {code:0,//返回的状态码，是否查询成功 0：成功 1：未登录 2：服务器错误
      msg:"",//如果查询失败返回查询失败的原因
      count:10,
      data{//查询成功的数组
      {name:"",price:5,stackroom:{stackid:1,type:"国学"}},
     {name:"",price:5,stackroom:{stackid:1,type:"国学"}},
     {name:"",price:5,stackroom:{stackid:1,type:"国学"}}
     }
     }
     ```

     

