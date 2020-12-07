---
layout:     post
title:      java8 stream best practice
subtitle:   all contains
date:       2020-07-02
author:     laosuan
header-img: img/java-8-stream.jpg
catalog: true
tags:
    - java
---

## 1 拼接集合中的字符串

```java
String allStr = courseList.stream().map(e -> e.getTitle()).collect(Collectors.joining());
```

## 2 集合中数字求和

```java
double totalDuration = courseList.stream().mapToDouble(d -> Double.valueOf(d.getDuration())).sum();
```

## 3 集合中的字段构造成数组

```java
List<Integer> questionIdList = questionList.stream().map(Question::getId).collect(Collectors.toList());
```

## 4 filter过滤&集合中的字段构造成map数组

```java
List<Map<String, Object> remoteMapData = new ArrayList<>();

List<Map<String, Object>> finalRemoteData = remoteMapData.stream().filter(r -> !r.get("user_id").equals(CategoryType.ADMIN_DOWN_ID)).map(r -> {
                Map<String, Object> mapR = new HashMap<>();
                mapR.put("portId", r.get("_id"));
                mapR.put("portAuthor", r.get("user"));
                return mapR;
            }).collect(Collectors.toList());
```

## 5 构造指定值的数组并遍历

```java
Stream.of("1", "2", "3").forEach(System.out::println);
```







