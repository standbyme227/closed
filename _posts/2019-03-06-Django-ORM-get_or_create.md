---
layout: post
title: Django - get_or_create?
tags: [django, orm, get_or_create]
---

### Django get_or_create

#### 1) 개요
> get_or_create를 사용하는 구간에서 
지속적으로 duplicate_error 발생.

#### 2) get_or_create란?
> 객체를 조회하려는 시도에서   
혹시나 없는 객체라면 지정된 kwarg로 생성이 가능하도록  
예외처리를 미리 해놓는 방법.

```
try:
    obj = Person.objects.get(first_name='John', last_name='Lennon')
except Person.DoesNotExist:
    obj = Person.objects.create(
            first_name='John', last_name='Lennon',age=80
        )
    # obj = Person(first_name='John', last_name='Lennon', age=80)
    # obj.save() 
```

> 위 코드처럼 예외처리를 할 수도 있지만

```
obj, created = Person.objects.get_or_create(
    first_name='John',
    last_name='Lennon',
    defaults={'age' : 80},
)
```

> get_or_create는 tuple을 반환한다. (obj, created)
> 해당 tuple에서 obj는 말그대로 객체, created는 bool값이다.

#### 3) 문제점
> defaults로 지정하지않고, 무작정 field값으로 지정하니 duplicate이 뜬걸로 확인된다.

> defaults 처리하니 잘 돌아간다.
