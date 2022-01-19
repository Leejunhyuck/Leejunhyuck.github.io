---
layout: post
title: Pageable을 파라미터 Sort(정렬)정보도 같이  받는 방법
date: 2021-01-19 10:09:23 +0900
category: Spring
---
# Pageable을 파라미터 Sort(정렬)정보도 같이  받는 방법

* 정렬 기준이 하나로 동일한 경우
```
@GetMapping("/item")
fun getItem(@PageableDefault(page = 0, size = 20, sort = ["regDate"],
              direction = Sort.Direction.DESC) pageable: Pageable) : Result {
    return itemService.getPaging(pageable)
}

itemRepository.findAll(pageable)

```

* 정렬 기준이 둘 이상인 경우
```
@GetMapping("/item")
fun getItem(@PageableDefault(page = 0, size = 20)
                @SortDefault.SortDefaults({
                    SortDefault(sort = ["regDate"], direction = Sort.Direction.DESC),
                    SortDefault(sort = ["upDate"], direction = Sort.Direction.DESC),
                })
                pageable: Pageable) : Result {
    return itemService.getPaging(pageable)
}

```