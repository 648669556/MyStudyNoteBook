#JDBC连接池与dbUtils学习
--
##JDBC连接池的使用

```java
    private static String url;
    private static String user;
    private static String password;
//连接数据库需要的url和账号密码
    private static int initCount = 20; //初始化时候的数量
    private static int maxCount = 50;   //连接池的最大数量
    private static int currentCount = 0; //当前的连接池数量
    //使用链表形式的话更加的高效，大部分都是增加和删除没有查找
    private static LinkedList<Connection> ConnectionPool = new LinkedList<>(); //数据库连接池

    
```
1. 在静态代码块中初始化连接和参数
```java
static {
        try {
            //
            Properties properties = new Properties();
            try {
//用DaoUtil.class.getClassLoader().getResourceAsStream("")获取的地址是Path目录
// 下的所以我们的配置文件需要放在src下
                properties.load(DaoUtil.class.getClassLoader().getResourceAsStream("Database.properties"));
            } catch (IOException e) {
                e.printStackTrace();
            }
            url = properties.getProperty("url");//获取账号密码什么的常规操作
            user = properties.getProperty("user");
            password = properties.getProperty("password");
            Class.forName(properties.getProperty("driverClassname"));//注册驱动

            for (int i = 0; i < initCount; i++) {//初始化连接池
                try {
                    ConnectionPool.add(getConnect());
                    currentCount++;
                } catch (SQLException throwables) {
                    throwables.printStackTrace();
                }
            }
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
    }
```
2.  编写创建新连接的代码
```java
 private static Connection createConnection() {
        Connection conn = null;
        try {
            conn = DriverManager.getConnection(url, user, password);
        } catch (SQLException throwables) {
            throwables.printStackTrace();
        }
        return conn;
    }
```
3. 编写获取连接的代码
```java
public static Connection getConnect() throws SQLException {
        Connection conn = null;
        if (ConnectionPool.size() > 0) {
            synchronized (ConnectionPool) {
                conn = ConnectionPool.getFirst();
                ConnectionPool.removeFirst();
            }
        } else {
            if (currentCount < maxCount) {
                conn = createConnection();
                currentCount++;
            } else {
                //maxCount+=50;
                //conn=createConnection();
                //currentCount++;
                throw new SQLException("已经没有数据库连接");
            }
        }
        return conn;
    }
```
4. 关闭连接 

   ```java
    public static void close(Connection conn, PreparedStatement ps, ResultSet rs) {
           if (rs != null) {
               try {
                   rs.close();
               } catch (SQLException throwables) {
                   throwables.printStackTrace();
               }
           }
           if (ps != null) {
               try {
                   ps.close();
               } catch (SQLException throwables) {
                   throwables.printStackTrace();
               }
           }
           if (conn != null) {
               synchronized (ConnectionPool) {
                   //这里关闭连接其实是将连接放回连接池
                   ConnectionPool.add(conn);
               }
           }
       }
   ```

   这样基本上就算是完成了一个简单的连接池了

   但是还是存在很多的问题，比如高并发的数量一多就会存在连接池数量不够的情况。**可以考虑用队列排队获取连接来解决**

**在最后放上源码**

```java
package com.chen.util;

import java.io.IOException;
import java.sql.*;
import java.util.LinkedList;
import java.util.Properties;

public class DaoUtil {
    private static String url;
    private static String user;
    private static String password;
    private static int initCount = 20;
    private static int maxCount = 50;
    private static int currentCount = 0;
    private static LinkedList<Connection> ConnectionPool = new LinkedList<>();

    static {
        try {
            Properties properties = new Properties();
            try {
 properties.load(DaoUtil.class.getClassLoader().getResourceAsStream("Database.properties"));
            } catch (IOException e) {
                e.printStackTrace();
            }
            url = properties.getProperty("url");
            user = properties.getProperty("user");
            password = properties.getProperty("password");
            Class.forName(properties.getProperty("driverClassname"));

            for (int i = 0; i < initCount; i++) {
                try {
                    ConnectionPool.add(getConnect());
                    currentCount++;
                } catch (SQLException throwables) {
                    throwables.printStackTrace();
                }
            }
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
    }

    private static Connection createConnection() {
        Connection conn = null;
        try {
            conn = DriverManager.getConnection(url, user, password);
        } catch (SQLException throwables) {
            throwables.printStackTrace();
        }
        return conn;
    }

    public static Connection getConnect() throws SQLException {
        Connection conn = null;
        if (ConnectionPool.size() > 0) {
            synchronized (ConnectionPool) {
                conn = ConnectionPool.getFirst();
                ConnectionPool.removeFirst();
            }
        } else {
            if (currentCount < maxCount) {
                conn = createConnection();
                currentCount++;
            } else {
                maxCount+=50;
                conn=createConnection();
                currentCount++;
                throw new SQLException("已经没有数据库连接,扩容中...");
            }
        }
        return conn;
    }

    public static void close(Connection conn, PreparedStatement ps, ResultSet rs) {
        if (rs != null) {
            try {
                rs.close();
            } catch (SQLException throwables) {
                throwables.printStackTrace();
            }
        }
        if (ps != null) {
            try {
                ps.close();
            } catch (SQLException throwables) {
                throwables.printStackTrace();
            }
        }
        if (conn != null) {
            synchronized (ConnectionPool) {
                ConnectionPool.add(conn);
            }
        }
    }
}

```

## 通过DBUtils学习

知道了长参数的使用

例如

```java
public int update(String sql, Object... params) throws SQLException {
        PreparedStatement preparedStatement = null;
        preparedStatement = conn.prepareStatement(sql);
        int sqlcount = preparedStatement.getParameterMetaData().getParameterCount();
        if (params != null) {
            int Count = params.length;
            if (sqlcount != Count) {
                throw new SQLException("sql内的参数和传入的参数数量不匹配！");
            } else {
                for (int i = 0; i < params.length; ++i) {
                    Object temp = params[i];
                    preparedStatement.setObject(i+1,temp);
                }
            }
        }
        int resultSet = preparedStatement.executeUpdate();
        return resultSet;
    }
```

其中params参数是可以写可以不写 通过判断参数数量是否一样来去除第一个参数不匹配的问题。

然后就是 ``preparedStatement.setobject(i+1,temp)``方法可以不判断object的类型就填入preparedstatement中。