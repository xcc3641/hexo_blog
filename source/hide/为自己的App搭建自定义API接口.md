title: 为自己的App搭建自定义API接口
date: 2015-11-21 00:02:52
categories: 技术
tags:
	- 云服务器
	- Python
	- Java
---

做过爬虫，爬取过自己喜欢的图片（咳咳），当初只会手动执行并且下载到本地，这样的话比较无聊，不好玩。后来跟着高架构学了点服务器和简单的API接口搭建。用服务器可以自动化很多操作（配合Python），API接口搭建自己用的是（Java + Tomcat），数据储存是Mysql。

<!-- more -->

### 实践
#### 服务器
##### Tomcat安装和启动
1. 首先同样我们需要将Tomcat 7下载下来。打开Tomcat的官网。
我们选择左边的Tomcat 7下载
2. 我的是Ubuntu系统的服务器，选择tar.gz下载方式，用本地化工具将该文件上传给服务器
3. 我是讲文件放在`/usr/local/software/`下，然后用`tar`命令进行解压
4. 当解压成功后，进入`tomcat7/bin`目录下，输入`sh startup.sh`启动tomcat
5. 在浏览器输入`服务器ip:8080`，出现tomcat默认界面即启动成功

##### 将程序部署到服务器
1. 对Eclipse项目右键 --> Export --> WAR file
2. 将打包好的war项目放在服务器tomcat目录下的webapps文件夹中
3. 重启tomcat服务器即可`tomcat/bin`目录下 `sh shutup.sh`后`sh startup.sh`

#### Java代码
##### 数据库（DAO）
###### 数据库连接
```
	String url = "jdbc:mysql://localhost:3306/数据库名";
	String usrname = "root";
	String passwd = "***";
	Class.forName("com.mysql.jdbc.Driver");
	connection = DriverManager.getConnection(url, usrname, passwd);
```
###### 查询语句
```
		public ResultSet selectSQL(String sql,Connection connection) {
        ResultSet rs = null;
        try {
            statement = connection.prepareStatement(sql);
            rs = statement.executeQuery(sql);
        } catch (SQLException e) {
            e.printStackTrace();
        }
        return rs;
    }
```

##### 主类（extends HttpServlet）
在`doGet`方法里写（贴下关键代码）
```
PrintWriter pw=response.getWriter();
Dao d   = new Dao();
Connection connection =   d.connectSQL();
int index = 0;
String  url = "";
String s = "select url from img where date="+date;
JSONObject jsonObject = new JSONObject();
JSONArray jsonArray = new JSONArray();
ResultSet rs = d.selectSQL(s,connection);
```
数据用JSON包装：
```java
while(rs.next()){
	url = rs.getString("url");
	jsonArray.add(index, url);
	jsonObject.put("date", today);
	jsonObject.put("urls", jsonArray);
	}
```
显示：
```java
pw.println(jsonObject);
pw.flush();
pw.close();
```
接下来访问地址`主机ip:8080/项目名/主类名?date=`改程序即可根据用户输入的日期（YYMMDD格式）进行查询数据库，根据日期输出满足条件的

所有数据。
该数据为JSON。

#### Python
新学了如何用python操作mysql，其他的都是以前的知识，比如网络请求（requests），正则匹配（re）。
##### 数据库操作核心代码
```python
db = MySQLdb.connect("localhost", usrname, passwd, "db_girlimg")
cursor = db.cursor()
first_sql = "select url from img WHERE id=%d" % (1)
cursor.execute(first_sql)
first_sql_data = cursor.fetchall();
db.commit()
if len(first_sql_data) != 0:
	mark_url = first_sql_data[0]
	mark_url = mark_url[0]
	print(mark_url)
else:
	mark_url = "xxx" # 目标数据标志url
```

根据mark_url判断进行循环插入还是更新mark_url然后break

```python
for url in urls:
	if mark_url != url:
		insert_sql = "INSERT INTO img(url,date) VALUES ('%s','%s')" % \
	(url,  today)
		cursor.execute(insert_sql)
		db.commit()
	else:
		update_sql = "update img set url='%s' , date='%s' where id ='%d'" % (urls[0], today, 1)
		cursor.execute(update_sql)
		db.commit()
		break;

	db.close()

```
##### 爬取信息

代码简单不用贴，简单思路为：

http：`requests.get`进行访问url,获取该网页的内容`content = http.text`

然后对目标地址进行正则匹配`re.findall（正则表达式,content,re.S）`

#### Android
这方面就可以访问取出自己要的数据，然后进行使用。
解析自己包装好的数据，比用其他API里的数据要有成就感的多吧？


### 收获
服务器基本常识： Get
简单搭建接口技巧：Get
成就感与幸福感：ヾ(⌒(ノ'ω')ノ




