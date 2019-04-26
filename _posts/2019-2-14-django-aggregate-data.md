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
+----+-----------+-----------+---------+--------+------------+
| id | name      | grade     | class_1 | amount | date       |
+----+-----------+-----------+---------+--------+------------+
|  1 | 李浩浩    | 一年级    | 三班    |    600 | 2019-02-14 |
|  2 | 王丽丽    | 二年级    | 三班    |    500 | 2019-02-14 |
|  3 | 张三      | 二年级    | 三班    |    400 | 2019-02-14 |
|  4 | 李四      | 三年级    | 二班    |    300 | 2019-02-14 |
|  5 | 周杰伦    | 三年级    | 二班    |    200 | 2019-02-14 |
|  6 | 姜鹏      | 三年级    | 一班    |    100 | 2019-02-14 |
+----+-----------+-----------+---------+--------+------------+
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
    def get(self, request):

        # 求捐款总额
        total_amount = Student.objects.all().aggregate(total_amount=Sum('amount'))

        # 列出各个年级捐款人数
        number_of_students_in_grade = Student.objects.all().values('grade').annotate(number_of_students_in_grade=Count('name'))

        # 列出各个年级捐款金额
        grade_amount = Student.objects.all().values('grade').annotate(total_amount=Sum('amount'))

        # 列出各个班级捐款金额
        class_amount = Student.objects.all().values('grade','class_1').annotate(total_amount=Sum('amount'))

        return JsonResponse({"total_amount":total_amount['total_amount'], "number_of_students_in_grade":number_of_students_in_grade, "grade_amount":grade_amount,"class_amount":class_amount})
```

### 结果

请求

```
http://localhost:8000/xxx/test
```

输出结果

```
{
    "data": {
        "total_amount": 2100,
        "number_of_students_in_grade": [
            {
                "grade": "一年级",
                "number_of_students_in_grade": 1
            },
            {
                "grade": "二年级",
                "number_of_students_in_grade": 2
            },
            {
                "grade": "三年级",
                "number_of_students_in_grade": 3
            }
        ],
        "grade_amount": [
            {
                "grade": "一年级",
                "total_amount": 600
            },
            {
                "grade": "二年级",
                "total_amount": 900
            },
            {
                "grade": "三年级",
                "total_amount": 600
            }
        ],
        "class_amount": [
            {
                "grade": "一年级",
                "class_1": "三班",
                "total_amount": 600
            },
            {
                "grade": "二年级",
                "class_1": "三班",
                "total_amount": 900
            },
            {
                "grade": "三年级",
                "class_1": "二班",
                "total_amount": 500
            },
            {
                "grade": "三年级",
                "class_1": "一班",
                "total_amount": 100
            }
        ]
    }
}
```

**<center>生活不易，码文不易，转载请标明<a href="http://blog.poison.cc">出处</a>，小弟在此先行谢过。</center>**
