---
layout: post
title:  "JPA常用注解"
categories: blog
tags: ['JPA', 'spring-boot']
published: true
comments: true
script: [post.js]
---

* Do not remove this line (it will not be displayed)
{: toc}

## 注解

### @Entity

用于实体类声明之前，指出该Java类为实体类，将映射到指定的数据库表。

```java
@Entity
public class Chatbot {
  // todo
}
```

### @Table

当实体类与映射的数据库表名不同名时需要使用 `@Table`

* name 声明数据库表名

```java
@Entity
@Table(name = 'QF_CHATBOT')
public class Chatbot {
  // todo
}
```

### @Id

用于声明一个实体类的属性映射为数据库的主键列

```java
@Id
private String id;
```

### @GeneratedValue

用于标注主键的生成策略，通过strategy属性指定。默认情况下，JPA自动选择一个最适合底层数据库的主键生成策略: SqlServer 对应identity MySQL对应 auto increment

```java
@Id
@GenerateValue
private String id;
```
### @Column

当实体的属性与其映射的数据库表的列不同名时需要使用 `@Column` 标注说明。

* name 列名
* unique 是否唯一
* nullable 能否为空
* length 长度

```java
@Column(unique = true, nullable = true)
private String name;
```

## 参考资料

[JPA常见注解及使用](https://yq.aliyun.com/articles/653256)

{% endpost #9D9D9D %}

