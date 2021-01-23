---
title: Object转Json遇到的坑
tags: [fastjson]
categories: [fastjson]
date: 2021-01-23 21:30:00
---

# Object转Json遇到的坑

>  fastjson 在将Object转为Json的时候，如果Object中有相同字段，且字段为复杂对象，他们的引用指向同一个对象的时候会出现某一个对象转换异常（"$ref": "$.dataInner.list[0]"）。

测试常用JSON工具是否会有同样的坑：

```xml
<!-- 常用工具类hutool-->
<dependency>
    <groupId>cn.hutool</groupId>
    <artifactId>hutool-all</artifactId>
    <version>5.4.3</version>
</dependency>
<!-- google的gson-->
<dependency>
    <groupId>com.google.code.gson</groupId>
    <artifactId>gson</artifactId>
    <version>2.8.6</version>
</dependency>
<!-- 阿里的fastjson -->
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>fastjson</artifactId>
    <version>1.2.73</version>
</dependency>

<!-- springboot2.3.2.RELEASE 自带的jackson -->
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.11.1</version>
    <scope>compile</scope>
</dependency>
<dependency>
    <groupId>com.fasterxml.jackson.datatype</groupId>
    <artifactId>jackson-datatype-jdk8</artifactId>
    <version>2.11.1</version>
    <scope>compile</scope>
</dependency>
<dependency>
    <groupId>com.fasterxml.jackson.datatype</groupId>
    <artifactId>jackson-datatype-jsr310</artifactId>
    <version>2.11.1</version>
    <scope>compile</scope>
</dependency>
<dependency>
    <groupId>com.fasterxml.jackson.module</groupId>
    <artifactId>jackson-module-parameter-names</artifactId>
    <version>2.11.1</version>
    <scope>compile</scope>
</dependency>
```



测试对象：

```java
package com.example.way.bugTest;

import lombok.Data;

import java.util.List;

@Data
public class DataOuter {

    private String string;

    private List<DataTest> list;

    private DataInner dataInner;

}

```

DataInner：

```java
package com.example.way.bugTest;

import lombok.Data;

import java.util.List;

@Data
public class DataInner {

    private String string;

    private List<DataTest> list;

}

```

DataTest：

```java
package com.example.way.bugTest;

import lombok.Data;

import java.util.Date;

@Data
public class DataTest {

    private String string;

    private Integer integer;

    private Date date;


}
```

>  DataOuter中有DataInner中的所有字段，使用Spring的BeanUtils.copyProperties浅拷贝复现bug。

```java
package com.example.way.bugTest;

import cn.hutool.json.JSONUtil;
import com.alibaba.fastjson.JSON;
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.google.gson.Gson;
import org.springframework.beans.BeanUtils;
import java.util.ArrayList;
import java.util.Date;
import java.util.List;

public class Test {

    public static void main(String[] args) {

        DataOuter dataOuter = new DataOuter();
        dataOuter.setString("string");

        List<DataTest> dataTests = new ArrayList<>();
        DataTest dataTest = new DataTest();
        dataTest.setDate(new Date(100));
        dataTest.setInteger(100);
        dataTest.setString("str100");
        dataTests.add(dataTest);

        DataTest dataTest1 = new DataTest();
        dataTest1.setDate(new Date(200));
        dataTest1.setInteger(200);
        dataTest1.setString("str200");

        dataOuter.setList(dataTests);

        DataInner dataInner = new DataInner();
        // 浅拷贝
        BeanUtils.copyProperties(dataOuter,dataInner);
        dataOuter.setDataInner(dataInner);

        // cn.hutool.json
        System.out.println(JSONUtil.toJsonStr(dataOuter));

        // com.google.gson
        Gson gson = new Gson();
        System.out.println(gson.toJson(dataOuter));


        // com.alibaba.fastjson
        System.out.println(JSON.toJSONString(dataOuter));

        // com.fasterxml.jackson
        ObjectMapper objectMapper = new ObjectMapper();
        try {
            System.out.println(objectMapper.writeValueAsString(dataOuter));
        } catch (JsonProcessingException e) {
            e.printStackTrace();
        }
    }
}

```

结果：

```json
# cn.hutool.json
{
    "string": "string",
    "list": [
        {
            "date": 100,
            "string": "str100",
            "integer": 100
        }
    ],
    "dataInner": {
        "string": "string",
        "list": [
            {
                "date": 100,
                "string": "str100",
                "integer": 100
            }
        ]
    }
}
# com.google.gson
{
    "string": "string",
    "list": [
        {
            "string": "str100",
            "integer": 100,
            "date": "Jan 1, 1970 8:00:00 AM"
        }
    ],
    "dataInner": {
        "string": "string",
        "list": [
            {
                "string": "str100",
                "integer": 100,
                "date": "Jan 1, 1970 8:00:00 AM"
            }
        ]
    }
}
# com.alibaba.fastjson
{
    "dataInner": {
        "list": [
            {
                "date": 100,
                "integer": 100,
                "string": "str100"
            }
        ],
        "string": "string"
    },
    "list": [
        {
            "$ref": "$.dataInner.list[0]"
        }
    ],
    "string": "string"
}
# com.fasterxml.jackson
{
    "string": "string",
    "list": [
        {
            "string": "str100",
            "integer": 100,
            "date": 100
        }
    ],
    "dataInner": {
        "string": "string",
        "list": [
            {
                "string": "str100",
                "integer": 100,
                "date": 100
            }
        ]
    }
}
```

**hutool、gson、jackson测试均未发现如此问题，只有fastjson有这样的情况。**