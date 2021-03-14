---
title: Axios与Asp.Net Core WebAPI参数交互
date: 2021-03-14 17:51:31
tags:
categories:
- C#基础
---

vue前後端分離開發時， NetCore WebAPI按如下寫法提交和接收參數 

參考文章地址：

https://www.cnblogs.com/yaopengfei/p/11558525.html

https://blog.csdn.net/lycz_tpself/article/details/108663443

# .NET Core WEB API 中接口參數的模型綁定方式

| **特性**       | **绑定源**                 |
| -------------- | -------------------------- |
| [FromHeader]   | 请求标头                   |
| [FromQuery]    | 请求查询字符串参数         |
| [FromForm]     | 请求正文中的表单数据       |
| [FromBody]     | 请求正文                   |
| [FromRoute]    | 当前请求中的路由           |
| [FromServices] | 作为操作参数插入的请求服务 |

<!--more-->

## Get請求

1. 通過給axios的config的屬性params賦值向後台傳值

2. 前端代碼

   ```javascript
   //axios.get(url[, config])
   //通過修改config中params參數的值，將實參傳到後台
   axios.get("http://localhost:8081/api/Demo1/GetDemo1", {
       params: {
           name: "lucky",
           age: 14,
           grade: 99,
       },
   });
   ```

   ​	後端代碼

   ```c#
       [Route("/api/[controller]/[action]")]
       [Produces("application/json")]
       [ApiController]
       public class Demo1Controller : ControllerBase
       {
   
           #region get提交的方法
           [HttpGet]
           public MessageModel<string> GetDemo1(string name, int age)
           {
           	//參數數量可以不一致
           	//還可以手工從Rquest.Query中提取到傳入的參數
               int grade = Convert.ToInt32(Request.Query["grade"]);
               return new MessageModel<string>()
               {
                   msg = "获取成功",
                   success = true,
                   response =$"{name}    {age}    {grade}"
               };
           }
   
           #endregion
       }
   ```

   

3. 可以直接在URL中加入參數實現，後端接受action不變

   ```javascript
   //2.通過URL地址後加入參數來傳值
   axios
   .get(
   "http://localhost:8081/api/Demo1/GetDemo1?name=lucky&age=14&grade=99"
   )
   .then((response) => console.log(response.data));
   ```

4. 複雜類型的傳值

   ```javascript
      	  //3.通過params傳入實體
      axios
        .get("http://localhost:8081/api/Demo1/GetDemo2", {
          params: {
            name: "lucky",
            age: 14,
            grade: 99,
          },
        })
        .then((response) => console.log(response.data));
   ```

   後端必須使用**[FromQuery]**

      ```c#
   		/// <summary>
           /// Get複雜類型的獲得
           /// </summary>
           /// <param name="userInfo"></param>
           /// <returns></returns>
           [HttpGet]
           public MessageModel<string> GetDemo2([FromQuery] UserInfo userInfo)
           {
               return new MessageModel<string>()
               {
                   msg = "获取成功",
                   success = true,
                   response = $"{userInfo.name}    {userInfo.age}    {userInfo.grade}"
               };
           }
   
    	public class UserInfo
       {
           public string name { get; set; }
           public int age { get; set; }
           public int grade { get; set; }
       }
      ```

   

5. 簡單列表類型的參數進行傳值 

   由於數組參數默認名字會加[]符號，導致無法接收，所以前端修改axios的攔截器

      ```javascript
   import axios from "axios";
   
   import qs from "qs";
   
   axios.interceptors.request.use((config) => {
     // ......其他逻辑代码
     if (config.method === "get") {
       config.paramsSerializer = function (params) {
         return qs.stringify(params, { arrayFormat: "repeat" });
       };
     }
     return config;
   });
      ```

   

      ```javascript
    	//4.通過params傳送簡單數組
         axios
           .get("http://localhost:8081/api/Demo1/GetDemo3", {
             params: {
               tmpArray1: ["aa", "bb", "cc"],
               tmpArray2: ["aaa", "bbb", "ccc"],
             },
           })
           .then((response) => console.log(response.data));
   
      ```

   後端接收，使用[FromQuery]修飾

      ```c#
   		 /// <summary>
           /// get傳列表
           /// </summary>
           /// <param name="tmpArray1"></param>
           /// <param name="tmpArray2"></param>
           /// <returns></returns>
           [HttpGet]
           public MessageModel<string> GetDemo3([FromQuery] List<string>  tmpArray1, [FromQuery] List<string> tmpArray2)
           {
               return new MessageModel<string>()
               {
                   msg = "获取成功",
                   success = true,
                   response = $"{tmpArray1[0]}    {tmpArray1[1]}    {tmpArray1[2]}  "
               };
           }
      ```

6. 複雜的實體列表（不知道best practice是什麼）

   前端代碼，把實體數組轉為字符串後，傳到後端

   ```javascript
     //4.通過params傳送實體數組
         let tmpUserInfo = [
           { name: "lucky1", age: 14, grade: 99 },
           { name: "lucky2", age: 14, grade: 99 },
         ];
         axios
           .get("http://localhost:8081/api/Demo1/GetDemo4", {
             params: {
               tmpUserInfo: JSON.stringify(tmpUserInfo),//將json數組轉為字符串
             },
           })
           .then((response) => console.log(response.data));
   ```

   後端代碼，將接收到額字符串反序列為c#對象

   ```c#
       	/// <summary>
           /// get傳列表
           /// </summary>
           /// <param name="tmpUserInfo"></param>
           /// <returns></returns>
           [HttpGet]
           public MessageModel<string> GetDemo4([FromQuery] string tmpUserInfo)
           {
   			//反序列json字符串
               List<UserInfo> list = JsonConvert.DeserializeObject<List<UserInfo>>(tmpUserInfo);
   
   
               return new MessageModel<string>()
               {
                   msg = "获取成功",
                   success = true,
                   response = $"{tmpUserInfo}   {list.Count}"
            
               };
           }
   ```

7. 文件類型，使用post提交

## POST請求

1. 簡單類型的傳送

   前端傳值

   ```javascript
    	  //axios.post(url[, data[, config]])
         // 1.简单类型参数不能通过[FromBody]获取到,通過params以Url方式傳值
         axios
           .post(
             "http://localhost:8081/api/Demo1/PostDemo1",
             {},
             {
               params: {
                 name: "lucky",
                 age: 14,
                 grade: 99,
               },
             }
           )
           .then((response) => console.log(response.data));
   ```

   後端接收

   ```c#
    		/// <summary>
           /// Post簡單類型的獲得
           /// </summary>
           /// <param name="name"></param>
           /// <param name="age"></param>
           /// <returns></returns>
           [HttpPost]
           public MessageModel<string> PostDemo1(string name, int age)
           {
               int grade = Convert.ToInt32(Request.Query["grade"]);
               return new MessageModel<string>()
               {
                   msg = "获取成功",
                   success = true,
                   response = $"{name}    {age}    {grade}"
               };
           }
   ```

2. 實體類型

   前端代碼

   ```javascript
    //3.傳入實體
         axios
           .post("http://localhost:8081/api/Demo1/PostDemo3", {
             name: "lucky",
             age: 14,
             grade: 99,
           })
           .then((response) => console.log(response.data));
   ```

   後端代碼

   ```c#
   	   /// <summary>
          /// Post接收實體類型，實體的屬性與前端json保持一致
          /// </summary>
          /// <param name="userInfo"></param>
          /// <returns></returns>
           [HttpPost]
           public MessageModel<string> PostDemo3([FromBody] UserInfo userInfo)
           {
               return new MessageModel<string>()
               {
                   msg = "获取成功",
                   success = true,
                   response = $"{userInfo.name}    {userInfo.age}    {userInfo.grade}"
               };
           }
   ```

3. 傳入實體的列表

   前端代碼

   ```javascript
    //3.傳入實體列表 
         let tmpUserInfo1 = [
           { name: "lucky1", age: 14, grade: 99 },
           { name: "lucky2", age: 14, grade: 99 },
         ];
         axios
           .post(
             "http://localhost:8081/api/Demo1/PostDemo4",
             JSON.stringify(tmpUserInfo1),
             {
               headers: {
                 "Content-Type": "application/json;charset=UTF-8",
               },
             }
           )
           .then((response) => console.log(response.data));
   ```

   後端代碼

   ```c#
   		/// <summary>
           /// Post接收列表
           /// </summary>
           /// <param name="userInfo"></param>
           /// <returns></returns>
           [HttpPost]
           public MessageModel<string> PostDemo4([FromBody] List<UserInfo> userInfo)
           {
               return new MessageModel<string>()
               {
                   msg = "获取成功",
                   success = true,
                   response = $"{userInfo.Count}"
   
               };
            }		
   ```

4. 上傳文件

   前端代碼

   ```javascript
    <el-form ref="uploadFileForm">
         <el-upload
           action="http://localhost:8081/api/Demo1/PostDemo5"
           :data="{ aaa: 'aaa', listVal: ['aaa', 'bbb'] }"
           :limit="1"
         >
           <el-button size="small" type="primary">点击上传</el-button>
         </el-upload>
       </el-form>
   ```

   後端代碼

   ```c#
   		/// <summary>
           /// 接收上傳文件
           /// </summary>
           /// <param name="formInfo"></param>
           /// <param name="listVal"></param>
           /// <param name="aaa"></param>
           /// <returns></returns>
           [HttpPost]
           public MessageModel<string> PostDemo5([FromForm] IFormCollection formInfo
               ,[FromForm] List<string> listVal, [FromForm] string aaa)
           {
               IFormFileCollection files = formInfo.Files;
               var file = files[0]; // 得到文件
             
               // 可以使用此方法获取其它参数，也可以通过[FromForm]从方法入参接收
               formInfo.TryGetValue("aaa", out StringValues aaa2);
   
               return new MessageModel<string>()
               {
                   msg = "获取成功",
                   success = true,
                   response = ""
   
               };
           }
   ```

   