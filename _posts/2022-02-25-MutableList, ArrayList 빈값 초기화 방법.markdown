---
layout: post
title: is Prefix로 변수 선언시 직렬화 과정에서 누락되는 것 해결 방안
date: 2021-02-07 03:50:00 +0900
category: Grammar
---
# MutableList, ArrayList 빈 값 초기화 방법

* MutableList 빈값 초기화 방법을 자꾸 까먹어서 기록용으로 남깁니다.

  * val bookList: MutableList<Book> = mutableListOf()
  * val bookList: List<Book> = ArrayList()
  * val mutableList : MutableList<Kolory> = ArrayList()