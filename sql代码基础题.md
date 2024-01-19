![image](https://github.com/will-lv/data-analysis-foundation-map/assets/97819830/72145802-be97-4924-85f3-a2326f4e4cce)

1.根据上图写出创建数据库’sql50'的sql语句

```sql
drop database IF EXISTS sql50;
create database sql50 default charset utf8 collate utf8_general_ci; --创建数据库
use sql50;
```

创建数据表

```sql
--建表语句，建表函数create table，class为表名
--cid为字段名，int为数据类型
create table calss(
	cid int not null auto_increment primary key,
	caption varchar(16) not null
)default charset=utf8;

create table student(
	sid int not null auto_increment primary key,
	sname varchar(16) not null,
    class_id int not null,
    gender char(1) not null,
    s_brith varchar(20) not null,
    constraint fk_student_class foreign key (class_id) references class(cid)
)default charset=utf8;

create table teacher(
	tid int not null auto_increment primary key,
	tname varchar(16) not null
)default charset=utf8;

create table course(
	course int not null auto_increment primary key,
	cname varchar(16) not null,
    teacher_id int not null,
    constraint fk_course_teacher foreign key (teacher_id) references teacher(tid)
)default charset=utf8;

create table score(
	student_id int not null,
	sourse_id int not null,
    num int not null,
    constraint fk_score_course foreign key (course_id) references course(course_id),
    constraint fk_score_student foreign key (student_id) references student(student_id)
)default charset=utf8;
```



#### 查询下个季度过生日的学生姓名和年龄（过了1月1日加1岁）

```sql
--时间是能加减乘除的
select sname,year(now())-year(s_brith) as age
from student
where quarter(s_brith) = quarter(date_add(now(),interval 1 quarter));
--date_add(now(),interval 1 quarter)再当前日期的基础上增加一个季度
--quarter(date_add(now(),interval 1 quarter))获取下一季度的季度数

```



#### 查询任意找一个学生，这名学生是在1月过生日的概率，结果保留三位小数

```sql
select round(avg(if(month(s_birth) = 1,1,0)), 3) '概率' from student;
--if(month(s_birth) = 1,1,0)：判断学生出生月份是否为1月份。如果是，则返回1；否则，返回0
select round(avg(month(s_birth) = 1), 3) '概率' from student;
--month(s_birth) = 1：判断出生月份是否为1月，返回布尔值（true或false）,avg()，将true视为1，false视为0，得到1月份出生的比例。
```



#### 查询课程表中所有不重复的课程名称

```sql
select distinct cname from course;
```



#### 查询姓“孙”和“马”的老师的个数

```sql
select count(1)
from teacher
where tname regexp '^[孙马]';
--正则表达式^[孙马] 表示以“孙”或“马”开头的姓氏
```



#### 查询没有教授课程的老师姓名

```sql
select tname
from teacher
left join course on teacher.tid = course.tid
where course.cid is null;
--is null是一种=null的含义，但是由于null本来就是空，没有等于的说法
```



#### 查询除了’1‘班的学生的'sid'，班级名称和年龄（按出生日期来算，过了生日加一岁）

```sql
select sid,caption,timestampdiff(year,s_birth,now()) as age
from student 
join class on class_id = cid and calss_id != 1;
--timestampdiff是mysql数据库中一个用于计算两个日期或时间值之间差值的函数。其函数语法如下：
--timestampdiff(unit,datetime1,datetime2)
--unit是需要计算的时间单位，可以是second,minute,hour,day,month,week,year等等。
--datetime1和datetime2是两个日期或时间值，可以是date、datetime、timestamp或字符串类型
```



#### 查询所有学生和老师的姓名和年龄，其中学生年龄为实际年龄（按出生日期来算，过了生日加一岁），教师年龄默认为30岁。如果有相同的姓名，都要显示，结果中先显示学生，再显示教师

```sql
select sname,timestampdiff(year,s_birth,now()) as age
from student
union all
select tname,30 as age
from teacher;
```



#### 查询任意一个月’数据分析-1‘班有学生过生日的概率。结果：班级id和概率(xx.x%)

```sql
select caption,concat(round(count(distinct month(s_birth))/12*100,1),'%') '概率'
from student 
join class on class_id = cid and captionn = "数据分析-1";
```

- round(count(distinct month(s_birth))/12*100, 1)：计算概率值。
  - `count(distinct month(s_birth))`：统计不同出生月份的学生个数。
  - `/12`：除以12得到占比。
  - `*100`：乘以100转换为百分数。
  - `round(..., 1)`：保留1位小数。



#### 计算每个班级的班级名称，男生占比（升序排列）

```sql
select caption,count(case when gender = '男' then 1 end)/count(1) as '男生占比'
from student s join class c on c.cid = s.class_id
group by caption
order by 2;
--case when 条件 then 结果1 else 结果2 end
--条件：判断条件。
--结果1：条件为真时返回的值。
--结果2：条件为假时返回的值
```



#### 查询课程‘1’成绩分级人数。E<60，D<70，C<80，B<90，A<=100

```sql
select
	  case when score<60 then 'E'
	  	   when score<70 then 'D'
	  	   when score<80 then 'C'
	  	   when score<90 then 'B'
	  	   else 'A' end '分级',
	  count(1) '人数',
from score
where course_id = 1
group by 1;
```



#### 计算每个班级的男女比例（逆序排列）

```sql
select caption,sum(if(gender='男',1,0))/sum(gender='女') '男女比例'
from student s join class c on c.cid = s.class_id
group by cid
order by 2 desc;
```



#### 查询今年到现在每月过生日的学生人数

```sql
select month(s_birth) as mth,count(1) as num
from student
where date_format(s_birth,"%m%d") <= date_format(now(),"%m%d")
group by mth;
```



查询今年到现在按照月累计过生日的学生人数，结果按照月份升序

```sql
select month(s_birth) as mth,sum(count(distinct sname)) over(order by month(s_birth)) as num
from student
where date_format(s_birth,"%m%d") <= date_format(now(),"%m%d")
group by mth
order by mth;
--累计计算：窗口函数 sum(...) over(...) 在每个分组内进行累计计算：
--从第一个月开始，累加每个月之前（含）的不同学生人数。
--累加顺序根据 over(order by month(s_birth)) 指定的排序规则，即按出生月份升序排列。
```



#### 查询每门课程的平均成绩‘AVG’和成绩标准差‘SD’，结果按平均成绩降序排序，标准差升序排

```sql
select cname,round(avg(score),2) 'AVG', round(stddev(score),2) 'SD'
from score s join course c on c.course_id = s.course_id
group by cname
order by AVG desc,SD asc;
```



#### 查询平均分（去除最高分和最低分）

要记得count(a)-2



#### 查询同名同姓学生名单，并统计人数

```sql
select sname,count(1)
from student
group by sname
having count(1)>1;
```



#### 查询未选修所有课程的学生的学号、姓名。

```sql
select student.sid,student.sname
from score
left join student on score.student_id = student.sid
left join course using(course_id)
group by student_id
having count(distinct cname) != (select count(distinct cname) from course);
```



#### 查询“数据分析1班”每个学生的学号、姓名、总成绩、平均成绩

有”每个“词，一般都要使用group by



#### 查询选修了“统计学”课程的男生及总体（男生和女生的总和）的人数和平均分（使用with roolup函数）

``` sql
select ifnull(gender,"总体") as gender,
	   count(sid) "人数",avg(score) "平均分"
from score sc join student s on s.sid = sc.student_id
	 join course c on sc.course_id = c.course_id
where c.cname = "统计学"
group by gender
with rollup
having s.gender = "男" or s.gender is null;
--ifnull,接受两个参数，如果不是null,则返回第一个参数，否则，第二个
--with rollup作用添加总计行，以显示分组结果的汇总情况
```

![image](https://github.com/will-lv/data-analysis-foundation-map/assets/97819830/99fbc249-0a5e-45f3-b3eb-20aaf82da0d1)



#### 查询“统计学”课程所有学生成绩的美式排名，并进行分页展示，每页显示3人的成绩和排名，显示第3页的

```sql
select student_id,score rank() over(order by score desc) rk
from score join course using(course_id)
where cname = "统计学"
limit 6,3;
--从结果集中的第六条记录开始，返回3条记录
--美式排名（American Ranking）是一种排名方法，它也被称为“高尔夫排名”或“连续排名”。在美式排名中，如果两个或更多的项目有相同的值，那么它们将得到相同的排名。但是，和中式排名不同，美式排名的下一个项目不会跳过任何名次。
```



#### 查询课程“统计学”成绩第7名的学生成绩单（不考虑成绩并列）

```sql
select student_id,score
from score join course using(course_id)
where cname = "统计学"
order by score desc
limit 1 offset 6;
--limit 1 offset 6:返回一条，从第七条开始（第一条索引为0，所以偏移6条）
```



#### 查询两门及以上不及格的学生的学号、姓名、选修课程数量

```sql
select sid,sname,total_score
from
	(select student.sid,student.sname,count(1) 			total_score,sum(score<60) num
    from score left join student on student_id = sid
    group by student_id) t
where num > 1;
```



#### 查询选修“渭河”老师所授课程中，每门课程成绩最高的学生名字以及成绩（考虑并列）

```sql
select cname,sname,score
from (select course.cname,
     		 student.sname,
     		 score.score,
     		 rank() over (partition by course.cname order by score.num desc) rk
      from score
      join student on score.student_id = student.sid
      join course using(course_id)
      join teacher on course.teacher_id = teacher.tid
      where teacher.tname = "渭河") t
where rk = 1;
```

```sql
rank() over (
  partition by [表达式]
  order by [表达式] [asc | desc]
)
-- partition by [表达式]:分区表达式，用于将数据分成多个分区
```



#### 查询每门课程获得最高三个分数的学生id，（考虑同一个成绩由多个学生获得）。按照课程ID和学生ID升序

```sql
select cname,student_id
from (
	select cname,student_id,course_id,
			dense_rank() over(partition by cname order by score desc) rk
	from score join course using(course_id) t)
where rk < 4
order by course_id,student_id asc;
```

`rank()` 函数和 `dense_rank()` 函数都是用于计算分区内记录的排名，但 `rank()` 函数会将并列的排名视为不同排名，而 `dense_rank()` 函数会将并列的排名合并为连续的数字



#### 按照出生年月对学生及逆行分组，并进行每组中年龄最小学生的学号、姓名和“统计学”成绩

```sql
select sid,sname,s.name
from score s
join student on student.sid = s.student_id
join course using(course_id)
where cname = "统计学" and s_birth in(select max(s_birth) from student group by date_format(s_birth,"%y%m"));
```



#### 查询成绩排名第二和倒数第二的差值不小于30分的课程id（成绩排名-中式排名）

```sql
select course_id
from (
    select course_id,score,
    	   row_number() over(partition by course_id order by score asc) asc_rk,
    	   row_number() over(partition by course_id order by score desc) desc_rk
    from score
    ) t
where
	asc_rk = 2 or desc_rk = 2
group by course_id
having (max(score)-min(score)) >= 30;
--row_number() 函数会将每个分区内的记录从 1 开始编号
```



#### 查询与学号为2的学生选修的课程完全相同的其他学生的学号和姓名

```sql
select student.sid,student.sname
from score
	join course using(course_id)
	join student on score.student_id = student.sid
	--只选择那些在score表中sid为2的course_id，第一次筛选
	and score.course_id in (select course_id from score where student_id = 2)
	--第二次筛选，选修数量等于sid=2的选修数量
where score.student_id in(
	select student_id from score where student_id != 2
	group by student_id
	having count(1) = (select count(1) from score where student_id = 2)
)
group by student_id
having cuont(1) = (select count(1) from score where student_id = 2);
```

## 增删改

**一般来说自己建数据库，或者数据仓库的同学需要深入学习，对于数据分析师来说，学会如何建表，修改字段等已经足够用，剩下的可以靠chatgpt帮忙补齐**

#### 1.创建一个表’info'

![image](https://github.com/will-lv/data-analysis-foundation-map/assets/97819830/eb24f3f6-73bf-4213-94a9-1aa1bef7d708)


```sql
create table 'info' (
	'sid' int(11) not null auto_increment primary key comment '自增ID',
    'student_id' int(11) not null comment '学生ID',
    'course_id' int(11) not null comment '课程ID',
    'score' int(11) not null default 0 comment '成绩',
    sname varchar(16) comment '学生姓名',
    s_birth datetime comment '生日',
    constraint 'fk_sc_student' foreign key ('student_id') references 'student' ('sid'),
    constraint 'fk_sc_course' foreign key ('course_id') references 'course' ('course_id'),
) default charset=utf8;
```

#### 2.然后把当前数据库中所有相关数据导入到info表中

```sql
insert into info(student_id,course_id,score,sname,s_birth)
	select student_id,course_id,score,sname,s_birth
	from score join student s on s.sid = score.student_id; 
```

#### 3.把info表中“渭河”的统计学成绩修改为85分

```sql
update info
set score = 85
where sname = '渭河'
and course_id in (select course_id from course where cname = '统计学');
```

#### 4.向info表中插入一些记录，这些记录要求符合以下条件：

1）学生ID为：没上过课程“2”

2）课程ID为：3

3）成绩为：课程“3”的最高分

```sql
insert into info(student_id,course_id,score)
select distinct student_id,3 as crouse_id,max
from
score join student s on score.student_id = s.sid and score.course_id != 2,
(select max(score) 'max' from score where course_id = 3) t;
```



#### 5.删除info表中学生“2”的课程“1”成绩

```sql
delete from info where course_id =1 and student_id = 2;
```



#### 6.删除teacher表中没有任何教授任何课程的老师

```sql
delete from teacher where tid not in (select distinct teacher_id from course);
```













