---
published: true
title: "Spark - Nice to know"
excerpt: "Nice to know about spark"
toc: true
toc_sticky: true
toc_label: "Nice to know"
toc_icon: "terminal"
author_profile: false
comments: true
header:
  overlay_image: "assets/images/covers/cover-cloud.jpg"
  overlay_filter: 0.2
  teaser: "assets/images/covers/cover-cloud.jpg"
categories: [spark, sql, scala]
---

# Spark session

```java
val newSession = spark.newSession
```
A copy of the current session state is created containing temp views and the like and provide that for further use. Any change you make in either of the sessions moving forward will not be reflected in the other. In fact, the best way to think of it is like spark session version control branching. This allows you testing somecode without affecting your primary session. You can get back and forth between sessions using:

```java
SparkSession.setActiveSession(copiedSession)
```
