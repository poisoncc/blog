---
layout: post
title: Django数据的聚合统计
categories: [python, django]
description: 由于工作需要用到了Django的聚合统计，觉得有必要记录一下
---

**<center>生活不易，码文不易，转载请标明<a href="http://blog.poison.cc">出处</a>，小弟在此先行谢过。</center>**

## 原数据结构

以学生献爱心捐款名单为例，从不同纬度统计总金额。

```
mysql> select * from student;
+----+-----------+-----------+--------+--------+------------+
| id | name      | grade     | class  | amount | date       |
+----+-----------+-----------+--------+--------+------------+
|  1 | 李浩浩    | 一年级    | 三班   |    100 | 2019-02-14 |
|  2 | 王丽丽    | 一年级    | 三班   |    100 | 2019-02-14 |
|  3 | 李浩浩    | 二年级    | 一班   |    500 | 2019-02-14 |
|  4 | 张三      | 三年级    | 三班   |    200 | 2019-02-14 |
|  5 | 李四      | 二年级    | 二班   |    100 | 2019-02-15 |
|  6 | 周杰伦    | 二年级    | 三班   |    400 | 2019-02-15 |
|  7 | 王力宏    | 一年级    | 一班   |    300 | 2019-02-15 |
|  8 | 姜鹏      | 三年级    | 二班   |    200 | 2019-02-15 |
|  9 | 王浩楠    | 一年级    | 二班   |    400 | 2019-02-15 |
+----+-----------+-----------+--------+--------+------------+
```

## 数据集合统计

### model

```
class Student(models.Model):
    name = models.CharField(max_length=11,default=None)
    grade = models.CharField(max_length=11,default=None)
    class_1 = models.CharField(max_length=11,default=None)
    amount = models.IntegerField(default=0)
    date = models.DateField(default=None)
    class Meta:
        managed = False
        db_table = 'student'
```

### view

```
from django.db.models import Sum,Count
from xxx.models import Student
class TestView(APIView):
    permission_classes = (permissions.AllowAny,)
    def post(self, request):
        startTime = request.data.get('startTime',None)
        endTime = request.data.get('endTime',None)

        # 求捐款总额
        total_amount = Student.objects.filter(date__gte=startTime,date__lte=endTime).aggregate(total_amount=Sum('amount'))

        # 列出各个年级捐款人数
        number_of_students_in_grade = Student.objects.filter(date__gte=startTime,date__lte=endTime).values('grade').annotate(number_of_students_in_grade=Count('name'))

        # 列出各个年级捐款金额
        grade_amount = Student.objects.filter(date__gte=startTime,date__lte=endTime).values('grade').annotate(total_number_of_cinema=Sum('amount'))

        return JsonResponse({"total_amount":total_amount['total_amount'], "number_of_students_in_grade":number_of_students_in_grade, "grade_amount":grade_amount})
```

### 结果

post请求

```
http://localhost:8000/xxx/test
{
	"startTime":"2019-02-14",
	"endTime":"2019-02-14"
}
```

输出结果

```
{
    "data": {
        "total_amount": 900,
        "number_of_students_in_grade": [
            {
                "grade": "一年级",
                "number_of_students_in_grade": 2
            },
            {
                "grade": "二年级",
                "number_of_students_in_grade": 1
            },
            {
                "grade": "三年级",
                "number_of_students_in_grade": 1
            }
        ],
        "grade_amount": [
            {
                "grade": "一年级",
                "total_number_of_cinema": 200
            },
            {
                "grade": "二年级",
                "total_number_of_cinema": 500
            },
            {
                "grade": "三年级",
                "total_number_of_cinema": 200
            }
        ]
    }
}
```

**<center>生活不易，码文不易，转载请标明<a href="http://blog.poison.cc">出处</a>，小弟在此先行谢过。</center>**
