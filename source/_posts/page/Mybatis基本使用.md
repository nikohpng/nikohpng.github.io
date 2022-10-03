---
title: Mybatis基本使用
date: 2020-12-09 09:24:16
cover: /images/mybatis.jpg
tags: mybatis
category: mybatis
---

本文主要讲解`mybatis`的`$`与`#`的使用，关联查询中各个关键字的使用

## 动态语法的使用

### #{}的使用

解析为一个JDBC预编译语句的参数标识符，把参数部分用占位符?代替

```mysql
select * from t_user from username = ?;
#处理后,会对传入数据加''
select * from t_user from username = 'Alice';
```

### ${}的使用

只做简单的字符串替换，在动态SQL解析阶段将会进行变量替换

```mysql
select * from t_user where username = 'Alice';
```

### #{}与${}对比

#{}用占位符站位后，JDBC的PreparedStatement会对传入的参数进行校验，防止SQL注入。

${}替换为具体的字符串后不会再做任何检测

### 使用场景

${}可以用于order by后的排序字段，表名，列名等。其中表名只能采用此

> 注：能使用#{}尽量用，如表名，order by的排序字段作为变量时使用${}

## 关联查询之一对多

`association`与`connection`在关联上的使用基本一致

### 单步查询

+ 实体类

```java
public class CountryPlus {
    Long id;
    String name;
    List<City> cityList;
}
public class City {
    Long id;
    String name;
    Long countryId;
}
```

+ xml配置

```xml
<select id="selectCountryPlusById" resultMap="countryPlusResultMap">
        select country.country_id as country_id,
                country,
                city_id,
                city,
                city.country_id as city_country_id,
        from country left join city on  country.country_id = city.country_id
        where country.country_id=#{id}
</select>

<!--resultMap-->
<resultMap id="countryPlusResultMap" type="canger.study.chapter04.bean.CountryPlus">
        <id column="country_id" property="id"/>
        <result column="country" property="name"/>
        <collection property="cityList" ofType="*.City">
            <id column="city_id" property="id"/>
            <result column="city" property="name"/>
            <result column="city_country_id" property="countryId"/>
        </collection>
    </resultMap>
```

### 分步查询

```xml
 <select id="selectCountryPlusByIdStep" resultMap="countryPlusResultMapStep">
        select *
        from country
        where country_id=#{id}
</select>

<!--resultMap-->
 <resultMap id="countryPlusResultMapStep" type="*.CountryPlus">
        <id column="country_id" property="id"/>
        <result column="country" property="name"/>
        <collection property="cityList"
                     select="*selectCityByCountryId"
                     column="country_id">
        </collection>
    </resultMap>
```





