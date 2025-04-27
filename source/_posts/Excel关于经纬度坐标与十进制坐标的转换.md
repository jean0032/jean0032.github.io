---
title: Excel关于经纬度坐标与十进制坐标的转换
date: 2025-04-16 19:59:12
tags: [excel, GIS]
categories: GIS
---

## 十进制转度分秒

```excel
=TEXT(INT(A1),"0")&"°"&TEXT(INT((A1-INT(A1))*60),"00")&"′"&TEXT(((A1-INT(A1))*60-INT((A1-INT(A1))*60))*60,"00.00")&"″"
```

## 度分秒转十进制

```excel
=LEFT(A1,FIND("°",A1)-1)+MID(A1,FIND("°",A1)+1,FIND("′",A1)-FIND("°",A1)-1)/60+MID(A1,FIND("′",A1)+1,FIND("″",A1)-FIND("′",A1)-1)/3600
```

注意：度分秒的要完整，例如要用23°1′0″的形式；23°1′则会出错。
