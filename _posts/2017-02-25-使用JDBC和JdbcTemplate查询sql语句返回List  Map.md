---
layout:     post
title:      "使用Jdbc和JdbcTemplate查询sql语句返回List<Map>"
subtitle:   " \"\""
date:       2017-02-25 11:50:00
author:     "chao"
header-img: ""
catalog: true
tags:
    - Jdbc
    - JdbcTemplate
---

使用JDBC执行sql语句返回`List<Map>` 类型：  


    public class JdbcUtil {    
        private static Log log = LogFactory.getLog(JdbcUtil.class);
        public static Connection getConnection(String driverClassName, String url, String username, String password) {
            Connection connection = null;
            try {
                Class.forName(driverClassName);
                connection = DriverManager.getConnection(url, username, password);
                return connection;
            } catch (ClassNotFoundException e) {
                log.error("error message:" + e.getMessage());
                e.printStackTrace();
                return null;
            } catch (SQLException e) {
                log.error("error message:" + e.getMessage());
                e.printStackTrace();
                return null;
            }
        }
    
        public static List<Map<String, Object>> execQuerySql(String driverClassName, String url, String username, String password, String sql) {
            Connection connection = JdbcUtil.getConnection(driverClassName, url, username, password);
            Statement statement = null;
            ResultSet rs = null;
            List<Map<String, Object>> rsList = null;
            try {
            	//参考：http://www.sitesbay.com/jdbc/jdbc-scrollable-resultset.php
                statement = connection.createStatement(ResultSet.TYPE_SCROLL_INSENSITIVE, ResultSet.CONCUR_READ_ONLY);
                rs = statement.executeQuery(sql);
                if (rs.getType() == ResultSet.TYPE_FORWARD_ONLY) {
                    log.error("error message: ResultSet non-scrollable.");
                    return null;
                }
                rs.last();
                int rsRows = rs.getRow();
                rsList = new ArrayList<>();
                for (int i = 0; i < rsRows; i++) {
                    rs.absolute(i + 1);
                    Map<String, Object> rsMaps = new HashMap<>();
                    ResultSetMetaData resultSetMetaData = rs.getMetaData();
                    int columnCount = resultSetMetaData.getColumnCount();
                    for (int j = 0; j < columnCount; j++) {
                        String columnLable = resultSetMetaData.getColumnLabel(j + 1);
                        Object columnValue = rs.getObject(j + 1);
                        rsMaps.put(columnLable, columnValue);
                    }
                    rsList.add(rsMaps);
                }
                return rsList;
            } catch (SQLException e) {
                log.error("error message:" + e.getMessage());
                e.printStackTrace();
                return null;
            } finally {
                try {
                    if (statement != null) {
                        if (!statement.isClosed()) {
                            statement.close();
                        }
                    }
                    if (connection != null) {
                        if (!connection.isClosed()) {
                            connection.close();
                        }
                    }
                } catch (SQLException e) {
                    e.printStackTrace();
                }
            }
        }
    } 
上面的代码问题是返回的Map中的键是英文全大写（如何解决？求解），例如：

    {"USERNAME": "user","ID": 1}。



还可以使用`JdbcTemplate`中封装的方法来执行sql语句返回`List<Map>`：

```
public class JdbcUtil{  
    public static JdbcTemplate getJdbcTemplate(String driverClassName, String url, String username, String password) {
        DriverManagerDataSource driverManagerDataSource = new DriverManagerDataSource(url, username, password);
        driverManagerDataSource.setDriverClassName(driverClassName));
        JdbcTemplate jdbcTemplate = new JdbcTemplate(driverManagerDataSource);
        return jdbcTemplate;
    }
    
    public static List<Map<String, Object>> execQuerySql(String driverClassName, String url, String username, String password, String sql) {
      	JdbcTemplate jdbcTemplate = getJdbcTemplate(driverClassName, url, username, password);
      	List<Map<String, Object>> mpList = null;
      	/*JdbcTemplate封装了三个返回List<Map<String, Object>>对象的方法，可以直接调用
      	List<Map<String, Object>> queryForList(String sql, Object[] args, int[] argTypes) throws DataAccessException;
      	List<Map<String, Object>> queryForList(String sql, Object... args) throws DataAccessException;
      	List<Map<String, Object>> queryForList(String sql) throws DataAccessException;
      	*/
      	return mpList;
    }
}
```
