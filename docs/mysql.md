## mysql

[TOC]



### 1.mysql查询分析

- explain
- Show profile

### #TODO



### 10.mysql练习

```mysql
#查询语句练习
# 1、查询"01"课程比"02"课程成绩高的学生的信息及课程分数
select stu.*,sc1.s_score as score1,sc2.s_score as score2 from Student stu ,Score sc1,Score sc2
    where stu.s_id=sc1.s_id
      and sc1.c_id='01'
      and stu.s_id=sc2.s_id
      and sc2.c_id='02'
      and sc1.s_id=sc2.s_id
      and sc1.s_score>sc2.s_score;

select stu.* ,sc1.s_score as score1,sc2.s_score as score2
    from Student stu
        join Score sc1
            on stu.s_id=sc1.s_id and sc1.c_id='01'
        left join Score sc2
            on stu.s_id=sc2.s_id and sc2.c_id='02' or sc2.c_id = null
        where sc1.s_score>sc2.s_score;

# 2.查询"01"课程比"02"课程成绩低的学生的信息及课程分
select stu.*,sc1.s_score,sc2.s_score
    from Student stu
        join Score sc1
            on stu.s_id=sc1.s_id and sc1.c_id='01'
        left join Score sc2
            on stu.s_id=sc2.s_id and sc2.c_id='02' or sc2.c_id is null
        where sc1.s_score<sc2.s_score;

# 3.查询平均成绩大于等于60分的同学的学生编号和学生姓名和平均成绩
select stu.s_id,stu.s_name,round(avg(s_score),2) as avg_score
    from Student stu
        join Score sc
            on stu.s_id=sc.s_id
            group by stu.s_id, stu.s_name having avg_score>=60;

#  4、查询平均成绩小于60分的同学的学生编号和学生姓名和平均成绩(包括有成绩的和无成绩的)
select stu.s_id,stu.s_name,round(avg(sc.s_score),2) as avg_score
    from Student stu
         left join Score sc
                   on stu.s_id=sc.s_id
         group by stu.s_id, stu.s_name having avg_score<60
union
select stu.s_id,stu.s_name,0 as avg_score
    from Student stu  where stu.s_id not in (
        select distinct sc.s_id from Score sc
    );


# 5、查询所有同学的学生编号、学生姓名、选课总数、所有课程的总成绩
select stu.s_id,stu.s_name,count(c.c_name) as course_count, sum(sc.s_score) as scoue_count
    from Student stu,Score sc,Course c
        where stu.s_id=sc.s_id and sc.c_id=c.c_id
        group by stu.s_id, stu.s_name
union
select stu.s_id,stu.s_name,0 as course_count, 0 as scoue_count
    from Student stu
    where stu.s_id not in (
        select distinct sc.s_id from Score sc
        );

select stu.s_id,stu.s_name,count(sc.c_id) as course_count ,sum(sc.s_score) as scoue_count
    from Student stu
    left join Score sc
        on stu.s_id=sc.s_id
        group by stu.s_id, stu.s_name;


# 6、查询"李"姓老师的数量
select count(1) from Teacher where t_name like '%李%';
# 7、查询学过"张三"老师授课的同学的信息
# 这个要去重
select distinct stu.*
    from Student stu
        join Score sc on sc.s_id=stu.s_id where sc.c_id in (
            select c.c_id from Course c
                join Teacher t
                    on t.t_name='张三' and t.t_id=c.t_id
        ) ;

select a.* from
	student a
	join score b on a.s_id=b.s_id where b.c_id in(
		select c_id from course where t_id =(
			select t_id from teacher where t_name = '张三'));

# # 8、查询没学过"张三"老师授课的同学的信息
select stu.* from Student stu where stu.s_id not in (
    select  stu.s_id from Student stu join Score sc on sc.s_id=stu.s_id where sc.c_id  in(
            select  c.c_id from Course c join Teacher t on t.t_id=c.t_id where t.t_name='张三'
    ));

# 9、查询学过编号为"01"并且也学过编号为"02"的课程的同学的信息
select stu.* from Student stu
    join Score sc1 on stu.s_id=sc1.s_id and sc1.c_id='01'
    join Score sc2 on stu.s_id=sc2.s_id and sc2.c_id='02';

select a.* from
	student a,score b,score c
	where a.s_id = b.s_id  and a.s_id = c.s_id and b.c_id='01' and c.c_id='02';

#  10、查询学过编号为"01"但是没有学过编号为"02"的课程的同学的信息
select stu.* from Student stu where
    s_id in (select distinct stu.s_id from Student stu join Score sc1 on stu.s_id=sc1.s_id and sc1.c_id='01')
    and s_id not in (
    select  distinct stu.s_id from Student  stu join Score sc2 on stu.s_id=sc2.s_id where sc2.c_id ='02') ;

select a.* from
	student a
	where a.s_id in (select s_id from score where c_id='01' )
	  and a.s_id not in(select s_id from score where c_id='02');

# 11、查询没有学全所有课程的同学的信息

select stu.* from Student stu left join Score sc on sc.s_id=stu.s_id
    group by stu.s_id having count(sc.c_id) <(select count(1) from Course);

explain
select s.* from student s
left join Score s1 on s1.s_id=s.s_id
group by s_id having count(s1.c_id)<(select count(*) from course);


select * from Student where s_id in (
    select sc.s_id from Score sc group by sc.s_id having count(c_id)<(
        select count(1) from Course)
    ) union
select * from Student where Student.s_id not in (select  s_id from Score group by s_id);


select * from Student where s_id in (
    select sc.s_id from Score sc group by sc.s_id having count(c_id)<(
        select count(1) from Course)
    ) or Student.s_id not in (select  s_id from Score group by s_id);

select version();
show variables like '%pro%';
set profiling =1;
show profiles ;
show profile cpu, block io, memory,swaps,context switches,source for query 272;

set @d=now();
select stu.* from Student stu left join Score sc on sc.s_id=stu.s_id
    group by stu.s_id having count(sc.c_id) <(select count(1) from Course);
select timestampdiff(second,@d,now());

set @d=now();
select * from Student;
select timestampdiff(second,@d,now());

# 12、查询至少有一门课与学号为"01"的同学所学相同的同学的信息

select distinct st.*
from student st
          left join score sc on sc.s_id = st.s_id
where sc.c_id in (
    select sc.c_id
    from Student stu
             join Score sc
                  on stu.s_id = sc.s_id and stu.s_id = '01');

select * from student where s_id in(
	select distinct a.s_id from score a where a.c_id in(select a.c_id from score a where a.s_id='01')
	);
#TODO
```

