# KingBase Writer
## 一、插件名称
名称：**kingbasewriter**
<br/>
## 二、支持的数据源版本
**KingBase 8.2及8.3**
<br/>
## 三、参数说明

- **connection**
  - 描述：数据库连接参数，包含jdbcUrl、schema、table等参数
    - 必选：是
    - 字段类型：List
      - 示例：指定jdbcUrl、schema、table
    ```json
    "connection": [{
         "jdbcUrl": "jdbc:kingbase8://localhost:54321/database",
         "table": ["table"],
         "schema":"public"
        }] 
     ```
  - 默认值：无

<br/>

- **jdbcUrl**
    - 描述：针对KingBase数据库的jdbc连接字符串
    - 必选：是
    - 字段类型：String
    - 默认值：无
    
<br/>
 
 - **schema**
   - 描述：数据库schema名
   - 必选：否
   - 字段类型：String
   - 默认值：无
 
 <br/>  
 
- **table**
     - 描述：目的表的表名称。目前只支持配置单个表，后续会支持多表
     - 必选：是
     - 字段类型：List
     - 默认值：无
     
 <br/>
    
- **username**
    - 描述：数据源的用户名
    - 必选：是
    - 字段类型：String
    - 默认值：无
    
<br/>
    
- **password**
    - 描述：数据源指定用户名的密码
    - 必选：是
    - 字段类型：String
    - 默认值：无
    
<br/>
      
- **column**
    - 描述：目的表需要写入数据的字段,字段之间用英文逗号分隔。例如: "column": ["id","name","age"]
    - 必选：是
    - 默认值：否
    - 字段类型：List
    - 默认值：无
    
<br/>
 
 - **fullcolumn**
   - 描述：目的表中的所有字段，字段之间用英文逗号分隔。例如: "column": ["id","name","age","hobby"]，如果不配置，将在系统表中获取
   - 必选：否
   - 字段类型：List
   - 默认值：无
 
 <br/>
    
- **preSql**
    - 描述：写入数据到目的表前，会先执行这里的一组标准语句
    - 必选：否
    - 字段类型：String
    - 默认值：无
    
<br/>
    
- **postSql**
    - 描述：写入数据到目的表后，会执行这里的一组标准语句
    - 必选：否
    - 字段类型：String
    - 默认值：无
    
<br/>
    
- **writeMode**
    - 描述：仅支持insert、update操作，可以搭配insertSqlMode使用
    - 必选：是
    - 字段类型：String
    - 默认值：无
    
<br/>
    
- **insertSqlMode**
    - 描述：控制写入数据到目标表采用  COPY table_name [ ( column_name [, ...] ) ] FROM STDIN DELIMITER 'delimiter_character'语句，提高数据的插入效率
    - 注意: 目前该参数值固定传入 copy，否则抛出提示为not support insertSqlMode的RuntimeException。当指定此参数时，writeMode的值必须为 insert，否则设置无效
    - 必选：否
    - 字段类型：String
    - 默认值：copy
    
<br/>
    
- **batchSize**
    - 描述：一次性批量提交的记录数大小，该值可以极大减少FlinkX与数据库的网络交互次数，并提升整体吞吐量。但是该值设置过大可能会造成FlinkX运行进程OOM情况
    - 必选：否
    - 字段类型：int
    - 默认值：1024
    
<br/>
    
- **updateKey**
  - 描述：当写入模式为update时，需要指定此参数的值为唯一索引字段
  - 注意：
    - 采用`merge into`语法，对目标表进行匹配查询，匹配成功时更新，不成功时插入；
  - 必选：否
  - 字段类型：Map<String,List>
    - 示例："updateKey": {"key": ["id"]}
  - 默认值：无    
    
## 四、配置示例
1、insert
```json
{
  "job": {
    "content": [{
      "reader": {
        "parameter": {
          "sliceRecordCount": ["100"],
          "column": [
            {
              "name": "id",
              "type": "int"
            },
            {
              "name": "user_id",
              "type": "int"
            },
            {
              "name":"name",
              "type":"string"
            }
          ]
        },
        "name": "streamreader"
      },
      "writer": {
        "name": "kingbasewriter",
        "parameter": {
          "connection": [{
            "jdbcUrl": "jdbc:kingbase8://localhost:54321/test",
            "table": [
              "tableTest"
            ],
            "schema":"test"
          }],
          "username": "username",
          "password": "password",
          "column": ["id","user_id","name"],
          "writeMode": "insert",
          "batchSize": 1024,
          "preSql": [],
          "postSql": []
        }
      }
    }],
    "setting": {
      "speed": {
        "channel": 1,
        "bytes": 0
      },
      "errorLimit": {
        "record": 100
      }
    }
  }
}
```
2、 insert with copy mode
```json
{
  "job": {
    "content": [{
      "reader": {
        "parameter": {
          "sliceRecordCount": ["100"],
          "column": [
            {
              "name": "id",
              "type": "int"
            },
            {
              "name": "user_id",
              "type": "int"
            },
            {
              "name":"name",
              "type":"string"
            }
          ]
        },
        "name": "streamreader"
      },
      "writer": {
        "name": "kingbasewriter",
        "parameter": {
          "connection": [{
            "jdbcUrl": "jdbc:kingbase8://localhost:54321/ide",
            "table": [
              "tableTest"
            ],
            "schema":"test"
          }],
          "username": "username",
          "password": "password",
          "column": ["id","user_id","name"],
          "writeMode": "insert",
          "batchSize": 1024,
          "preSql": [],
          "postSql": [],
          "insertSqlMode": "copy"
        }
      }
    }],
    "setting": {
      "speed": {
        "channel": 1,
        "bytes": 0
      },
      "errorLimit": {
        "record": 100
      }
    }
  }
}
```
3、 update
```json
{
  "job": {
    "content": [{
      "reader": {
        "parameter": {
          "sliceRecordCount": ["100"],
          "column": [
            {
              "name": "id",
              "type": "int"
            },
            {
              "name": "user_id",
              "type": "int"
            },
            {
              "name":"name",
              "type":"string"
            }
          ]
        },
        "name": "streamreader"
      },
      "writer": {
        "name": "kingbasewriter",
        "parameter": {
          "connection": [{
            "jdbcUrl": "jdbc:kingbase8://localhost:54321/database",
            "table": ["tableTest"],
            "schema":"test"
          }],
          "username": "username",
          "password": "password",
          "column": ["id","user_id","name"],
          "writeMode": "update",
          "updateKey": {"key": ["id"]},
          "batchSize": 1024,
          "preSql": [],
          "postSql": []
        }
      }
    }],
    "setting": {
      "speed": {
        "channel": 1,
        "bytes": 0
      },
      "errorLimit": {
        "record": 100
      }
    }
  }
}
```
