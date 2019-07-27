## mysql

[TOC]



### 1.mysql查询分析

- explain
- Show profile

####    时间分析

```
show processlist ; 查看系统状态
select @@profiling;  &&  show variables like '%pro%'; 查看是否开启 off/on
set profiling =1; 开启
show profiles;查看运行时间
query_id -- duration -- query
show profile all for query query_id  ##不起作用啊
#TODO
```



####    执行分析

1. ```
   explain select ****
   输出
   id--select_type--table--possible_keys--key--key_len--ref--rows--Extra
   ```

   ```
   1.id ：查询序号
   2.select_type ：select 类型
   	simple 简单查询 不使用union或子查询
   	primary 最外面的select
   	union UNION中的第二个或后面的SELECT语句
   	dependent union union中的第二个或后面的SELECT语句或后面的语句，取决于外面的查询
   	union result union的结果
   	subquery 子查询中的第一个select
   	dependent subquery 子查询中的第一个select 取决于外面的查询
   	derived 导出表的select（from子句的子查询）
   3.table 显示是查询的 哪张表
   4.type 显示使用了那种类别，是否使用索引
   结果值从好到坏依次是
   system	>	const	>	eq_ref > ref > fulltext > ref_or_null > index_merge > unique_subquery >index_subquery >range >index >All
   保证查询达到range级别，最好达到ref
   5.possible_keys ：指出mysql能使用哪个索引在该表中找到行
   6.key ：显示mysql实际决定使用的键（索引）如果没有选择索引键是null
   7.key_len ：显示mysql决定使用的键长度，如果键是null，长度为null。长度越短越好
   8.ref ： 显示使用哪个列或者常数与key一起从表中选择行
   9.rows：显示mysql认为他执行查询时必须检查的行数
   10.Extra ：包含msyql解决查询的详细信息
   ```

   ```
   当type 显示为 “index” 时，并且Extra显示为“Using Index”， 表明使用了覆盖索引。 
   
   ```

### 2.group by 、order by、group_concat(列转串)

```
写的顺序：select ... from... where.... group by... having... order by..
执行顺序：from... where...group by... having.... select ... order by...

```



### #TODO

### 3.日期处理

```

```

### 4.字符串处理

```

```

### 5.索引

### 6.视图

### 7.触发器

### 8.缓存、导出

### 9.分库分表

### 10.mycat

### 11.mysql练习

```mysql
#查询语句练习
# 1、查询'01'课程比'02'课程成绩高的学生的信息及课程分数
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

# 2.查询'01'课程比'02'课程成绩低的学生的信息及课程分
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


# 6、查询'李'姓老师的数量
select count(1) from Teacher where t_name like '%李%';
# 7、查询学过'张三'老师授课的同学的信息
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

# # 8、查询没学过'张三'老师授课的同学的信息
select stu.* from Student stu where stu.s_id not in (
    select  stu.s_id from Student stu join Score sc on sc.s_id=stu.s_id where sc.c_id  in(
            select  c.c_id from Course c join Teacher t on t.t_id=c.t_id where t.t_name='张三'
    ));

# 9、查询学过编号为'01'并且也学过编号为'02'的课程的同学的信息
select stu.* from Student stu
    join Score sc1 on stu.s_id=sc1.s_id and sc1.c_id='01'
    join Score sc2 on stu.s_id=sc2.s_id and sc2.c_id='02';

select a.* from
	student a,score b,score c
	where a.s_id = b.s_id  and a.s_id = c.s_id and b.c_id='01' and c.c_id='02';

#  10、查询学过编号为'01'但是没有学过编号为'02'的课程的同学的信息
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

# 12、查询至少有一门课与学号为'01'的同学所学相同的同学的信息

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


# 13、查询和'01'号的同学学习的课程完全相同的其他同学的信息
# 分组按照sc.s_id 顺序就对了
select * from Student where s_id in (
    select t1.s_id from (
      (select group_concat(sc.c_id) s, stu.s_id
        from Student stu
                join Score sc on
                stu.s_id = sc.s_id and stu.s_id != '01'
                group by sc.s_id) t1
        join
      (select group_concat(sc.c_id) s, stu.s_id
        from Score sc
               join Student stu on stu.s_id = sc.s_id
               where sc.s_id = '01') t2
               on t1.s = t2.s )
    );

# 14、查询没学过'张三'老师讲授的任一门课程的学生姓名

select Student.s_name from Student where s_id not in (
    select s_id from score where c_id in(
        select c_id from Course join
            Teacher on Course.t_id=Teacher.t_id where t_name='张三') );


select a.s_name from student a where a.s_id not in (
	select s_id from score where c_id in
				(select c_id from course where t_id =(
					select t_id from teacher where t_name = '张三')));


# 15、查询两门及其以上不及格课程的同学的学号，姓名及其平均成绩
select stu.s_id,stu.s_name,avg(sc.s_score) as ac from Student stu
    left join Score sc
    on stu.s_id=sc.s_id
    and sc.s_score<60
    or sc.s_score is null
    group by stu.s_id, stu.s_name having count(sc.s_id)>=2;

select st.s_id,st.s_name,avg(sc.s_score) from student st
left join score sc on sc.s_id=st.s_id
where sc.s_id in (
select sc.s_id from score sc
where sc.s_score<60 or sc.s_score is NULL
group by sc.s_id having COUNT(sc.s_id)>=2
)
group by st.s_id;


# 16、检索'01'课程分数小于60，按分数降序排列的学生信息
select stu.* from Student stu join Score sc
    on stu.s_id=sc.s_id
       and sc.c_id='01' and sc.s_score<60 order by sc.s_score desc;

select * from Student where s_id in (
select s_id from Score where s_score<60 and c_id='01' order by s_score desc ) ;

# 17、按平均成绩从高到低显示所有学生的所有课程的成绩以及平均成绩
select stu.*,sum(s_score),avg(s_score) ac from Student stu
    left join  Score sc on
        stu.s_id=sc.s_id group by stu.s_id order by ac desc ;

select st.s_id,st.s_name,avg(sc5.s_score) '平均分',sc.s_score '语文',sc2.s_score '数学',sc3.s_score '英语' ,sc4.s_score 'java' from student st
left join score sc  on sc.s_id=st.s_id  and sc.c_id='01'
left join score sc2 on sc2.s_id=st.s_id and sc2.c_id='02'
left join score sc3 on sc3.s_id=st.s_id and sc3.c_id='03'
left join score sc4 on sc4.s_id=st.s_id and sc4.c_id='04'
left join score sc5 on sc5.s_id=st.s_id
group by st.s_id
order by SUM(sc5.s_score) desc;

# 18.查询各科成绩最高分、最低分和平均分：以如下形式显示：课程ID，课程name，最高分，最低分，平均分，及格率，中等率，优良率，优秀率
# 及格为>=60，中等为：70-80，优良为：80-90，优秀为：>=90
select c.c_id,c.c_name,
       max(sc.s_score) '最高分',
       MIN(sc2.s_score) '最低分',
       avg(sc3.s_score) '平均分'
,((select count(s_id) from score where s_score>=60 and c_id=c.c_id )/(select count(s_id) from score where c_id=c.c_id)) '及格率'
,((select count(s_id) from score where s_score>=70 and s_score<80 and c_id=c.c_id )/(select count(s_id) from score where c_id=c.c_id)) '中等率'
,((select count(s_id) from score where s_score>=80 and s_score<90 and c_id=c.c_id )/(select count(s_id) from score where c_id=c.c_id)) '优良率'
,((select count(s_id) from score where s_score>=90 and c_id=c.c_id )/(select count(s_id) from score where c_id=c.c_id)) '优秀率'
from course c
left join score sc on sc.c_id=c.c_id
left join score sc2 on sc2.c_id=c.c_id
left join score sc3 on sc3.c_id=c.c_id
group by c.c_id;

# 19、按各科成绩进行排序，并显示排名(实现不完全)
set @i=0;
select c1.s_id,c1.c_id,c1.c_name,@score:=c1.s_score,@i:=@i+1 as '排名' from
 (select c.c_name,sc.* from course c left join score sc
     on sc.c_id=c.c_id
     where c.c_id='01' order by sc.s_score desc) c1 ,
            (select @i:=0) a
union all 
select c2.s_id,c2.c_id,c2.c_name,c2.s_score,@ii:=@ii+1 from (select c.c_name,sc.* from course c 
    left join score sc on sc.c_id=c.c_id
        where c.c_id='02' order by sc.s_score desc) c2 ,
            (select @ii:=0) aa
union all
select c3.s_id,c3.c_id,c3.c_name,c3.s_score,@iii:=@iii+1 from (select c.c_name,sc.* from course c 
    left join score sc on sc.c_id=c.c_id
        where c.c_id='03' order by sc.s_score desc) c3
#         set @iii=0
union all
select c4.s_id,c4.c_id,c4.c_name,c4.s_score,@iiii:=@iii+1 from (
    select c.c_name,sc.* from Course c
        left join  Score sc on sc.c_id=c.c_id
            where c.c_id='04' order by sc.s_score desc )c4;

# 20、查询学生的总成绩并进行排名
set @tt=0;
select r.s_id,r.s_name,r.sum,@tt:=@tt+1 from (
select stu.s_id,stu.s_name, sum(s_score) as sum from Student stu
    left join Score sc on stu.s_id=sc.s_id group by stu.s_id order by sum desc )r;

select st.s_id,st.s_name
,(case when sum(sc.s_score) is null then 0 else sum(sc.s_score) end)
 from student st
left join score sc on sc.s_id=st.s_id
group by st.s_id order by sum(sc.s_score) desc;

# 21、查询不同老师所教不同课程平均分从高到低显示
select t.t_id,t.t_name,c.c_name,avg(sc.s_score) from Teacher t left join Course c
    on t.t_id=c.t_id
    left join Score sc on sc.c_id=c.c_id
    group by t.t_id,t.t_name ,c.c_name order by avg(sc.s_score) desc ;

select t.t_id,t.t_name,c.c_name,avg(sc.s_score) from teacher t
left join course c on c.t_id=t.t_id
left join score sc on sc.c_id =c.c_id
group by t.t_id, t.t_name, c.c_name
order by avg(sc.s_score) desc;

# 22、查询所有课程的成绩第2名到第3名的学生信息及该课程成绩
select t_a.*
from (
         (select stu.*, c.c_name,sc.s_score
          from Student stu
                   left join Score sc
                             on stu.s_id = sc.s_id and sc.c_id = '01'
                   join Course c on c.c_id = sc.c_id
          group by stu.s_id
          order by sc.s_score desc
          limit 1,2
         )) t_a
union all
select t_b.*
from (
         select stu.*, c.c_name, sc.s_score
         from Student stu
                  left join Score sc
                            on stu.s_id = sc.s_id and sc.c_id = '02'
                  join Course c on sc.c_id = c.c_id
         group by stu.s_id
         order by sc.s_score desc
         limit 1, 2
     ) t_b
union all
select t_c.*
from (
         select stu.*, c.c_name, sc.s_score
         from Student stu
                  left join Score sc
                            on stu.s_id = sc.s_id and sc.c_id = '03'
                  join Course c on sc.c_id = c.c_id
         group by stu.s_id
         order by sc.s_score desc
         limit 1, 2
     ) t_c
union all
select t_d.*
from (
         select stu.*, c.c_name, sc.s_score
         from Student stu
                  left join Score sc
                            on stu.s_id = sc.s_id and sc.c_id = '04'
                  join Course c on sc.c_id = c.c_id
         group by stu.s_id
         order by sc.s_score desc
         limit 1, 2
     ) t_d;


select a.* from (
select st.*,c.c_id,c.c_name,sc.s_score from student st
left join score sc on sc.s_id=st.s_id
inner join course c on c.c_id =sc.c_id and c.c_id='01'
order by sc.s_score desc LIMIT 1,2 ) a
union all
select b.* from (
select st.*,c.c_id,c.c_name,sc.s_score from student st
left join score sc on sc.s_id=st.s_id
inner join course c on c.c_id =sc.c_id and c.c_id='02'
order by sc.s_score desc LIMIT 1,2) b
union all
select c.* from (
select st.*,c.c_id,c.c_name,sc.s_score from student st
left join score sc on sc.s_id=st.s_id
inner join course c on c.c_id =sc.c_id and c.c_id='03'
order by sc.s_score desc LIMIT 1,2) c;

#  23、统计各科成绩各分数段人数：课程编号,课程名称,[100-85],[85-70],[70-60],[0-60]及所占百分比

select c.c_id,c.c_name,

       (select count(1) from  Score sc where sc.c_id=c.c_id and  sc.s_score>=0 and sc.s_score<=60) as hc ,
       (select count(1) from  Score sc where sc.c_id=c.c_id and  sc.s_score>=0 and sc.s_score<=60)/(select count(1) from Score sc where sc.c_id=c.c_id ) as '百分比'
from Course c order by c.c_id  ;


select c.c_id,c.c_name
,((select count(1) from score sc where sc.c_id=c.c_id and sc.s_score<=100 and sc.s_score>85)/(select count(1) from score sc where sc.c_id=c.c_id )) "100-85"
,((select count(1) from score sc where sc.c_id=c.c_id and sc.s_score<=85 and sc.s_score>70)/(select count(1) from score sc where sc.c_id=c.c_id )) "85-70"
,((select count(1) from score sc where sc.c_id=c.c_id and sc.s_score<=70 and sc.s_score>60)/(select count(1) from score sc where sc.c_id=c.c_id )) "70-60"
,((select count(1) from score sc where sc.c_id=c.c_id and sc.s_score<=60 and sc.s_score>=0)/(select count(1) from score sc where sc.c_id=c.c_id )) "60-0"
from course c order by c.c_id;


# 24、查询学生平均成绩及其名次
set @tt=0;
select s.*,@tt:=@tt+1 from(
    select stu.s_id,stu.s_name ,round(avg(sc.s_score),2) as ac from Student stu
    left join Score sc on stu.s_id=sc.s_id group by stu.s_id order by avg(sc.s_score) desc) s;

set @i=0;
select a.*,@i:=@i+1 from (
    select st.s_id,st.s_name,round((case when avg(sc.s_score) is null then 0 else avg(sc.s_score) end),2) as ac from student st
        left join score sc on sc.s_id=st.s_id
        group by st.s_id  order by ac desc) a;

# 25、查询各科成绩前三名的记录

select a.* from

(select stu.s_id,stu.s_name,sc.s_score,c.c_name from Score sc join Course c
    on sc.c_id=c.c_id and sc.c_id='01' join Student stu on stu.s_id=sc.s_id group by sc.s_id order by s_score desc limit 3) a
union all
select b.* from
(select stu.s_id,stu.s_name,sc.s_score,c.c_name from Score sc join Course c
    on sc.c_id=c.c_id and sc.c_id='02' join Student stu on stu.s_id=sc.s_id group by sc.s_id order by s_score desc limit 3)b
union all
select c.* from

(select stu.s_id,stu.s_name,sc.s_score,c.c_name from Score sc join Course c
    on sc.c_id=c.c_id and sc.c_id='03' join Student stu on stu.s_id=sc.s_id group by sc.s_id order by s_score desc limit 3)c
union all
select d.* from
(select stu.s_id,stu.s_name,sc.s_score,c.c_name from Score sc join Course c
    on sc.c_id=c.c_id and sc.c_id='04' join Student stu on stu.s_id=sc.s_id group by sc.s_id order by s_score desc limit 3)d


select a.* from (
 select st.s_id,st.s_name,c.c_id,c.c_name,sc.s_score from student st
 left join score sc on sc.s_id=st.s_id
 inner join course c on c.c_id=sc.c_id and c.c_id='01'
 order by sc.s_score desc LIMIT 0,3) a
union all
select b.* from (
 select st.s_id,st.s_name,c.c_id,c.c_name,sc.s_score from student st
 left join score sc on sc.s_id=st.s_id
 inner join course c on c.c_id=sc.c_id and c.c_id='02'
 order by sc.s_score desc LIMIT 0,3) b
union all
select c.* from (
 select st.s_id,st.s_name,c.c_id,c.c_name,sc.s_score from student st
 left join score sc on sc.s_id=st.s_id
 inner join course c on c.c_id=sc.c_id and c.c_id='03'
 order by sc.s_score desc LIMIT 0,3) c;

# 26、查询每门课程被选修的学生数
select count(c.c_id),c.c_name from Score sc right join Course c
    on sc.c_id=c.c_id group by c.c_name;

select c.c_id,c.c_name,count(1) from course c
left join score sc on sc.c_id=c.c_id
inner join student st on st.s_id=c.c_id
group by st.s_id;

# 27、查询出只有两门课程的全部学生的学号和姓名
select stu.s_id,stu.s_name from Student stu
    join Score sc on sc.s_id=stu.s_id group by s_id having count(sc.c_id)=2;

select st.s_id,st.s_name from student st
left join score sc on sc.s_id=st.s_id
inner join course c on c.c_id=sc.c_id
group by st.s_id having count(1)=2;

# 28、查询男生、女生人数
select st.s_sex,count(1) from student st group by st.s_sex;

# 29、查询名字中含有"风"字的学生信息
select * from Student where s_name like '%风%';

# 30、查询同名同性学生名单，并统计同名人数
select st.s_id,st. s_name,count(1)+1 from Student stu join Student st
    on stu.s_name=st.s_name where stu.s_id!=st.s_id group by st.s_id;

# 31、查询1990年出生的学生名单
select * from Student where s_birth like '1990%';


# 32、查询每门课程的平均成绩，结果按平均成绩降序排列，平均成绩相同时，按课程编号升序排列
select c.c_id,c.c_name,avg(sc.s_score) ac from Score sc join Course c on sc.c_id=c.c_id
  group by c.c_id order by  ac desc ,c.c_id ;

select c.c_id,c.c_name,avg(sc.s_score) from course c
inner join score sc on sc.c_id=c.c_id
group by c.c_id order by avg(sc.s_score) desc,c.c_id asc;

# 33、查询平均成绩大于等于85的所有学生的学号、姓名和平均成绩
select stu.s_id,stu.s_name,avg(sc.s_score) ac from Student stu join Score sc
    on sc.s_id=stu.s_id group by stu.s_id having ac>=85;

select st.s_id,st.s_name,avg(sc.s_score) from student st
left join score sc on sc.s_id=st.s_id
group by st.s_id having avg(sc.s_score)>=85;


# 34、查询课程名称为"数学"，且分数低于60的学生姓名和分数
select stu.s_name,sc.s_score from Student stu
     join Score sc
        on stu.s_id=sc.s_id
     join Course c on sc.c_id=c.c_id where c.c_name='数学' and sc.s_score<60;

select st.s_id,st.s_name,sc.s_score from student st
inner join score sc on sc.s_id=st.s_id and sc.s_score<60
inner join course c on c.c_id=sc.c_id and c.c_name ='数学';

# 35、查询所有学生的课程及分数情况；
select stu.s_id,stu.s_name, c.c_name,sc.s_score from Student stu
    left join Score sc
        on stu.s_id=sc.s_id
    left join Course c on c.c_id=sc.c_id  group by stu.s_id, stu.s_name, c.c_name, sc.s_score;

select st.s_id,st.s_name,c.c_name,sc.s_score from student st
left join score sc on sc.s_id=st.s_id
left join course c on c.c_id =sc.c_id
order by st.s_id,c.c_name;
# 36、查询任何一门课程成绩在70分以上的姓名、课程名称和分数
select stu.s_name ,c.c_name,sc.s_score from Student stu
  right  join Score sc on sc.s_id=stu.s_id
  right  join Course c on c.c_id=sc.c_id where  sc.s_score>=70;

select a.s_name,b.c_name,c.s_score from course b left join score c on b.c_id = c.c_id
				left join student a on a.s_id=c.s_id where c.s_score>=70;

# 37、查询不及格的课程
select distinct c.c_name from Course c join Score sc
    on sc.c_id=c.c_id and sc.s_score<60;

select c.c_name from Course c where c.c_id in(
select sc.c_id from Score sc where sc.s_score<60);

select c.c_name from Course c join Score sc
    on c.c_id=sc.c_id and sc.s_score<60 group by c.c_name;


# 38、查询课程编号为01且课程成绩在80分以上的学生的学号和姓名
select stu.s_id,stu.s_name from Student stu
    join Score sc on sc.s_id=stu.s_id and sc.c_id='01' and sc.s_score>=80 ;

# 39、求每门课程的学生人数
select c.c_name,count(1) from Course c join Score sc
    on sc.c_id=c.c_id group by c.c_name  ;

select count(1) from Score GROUP BY c_id;

# 40、查询选修"张三"老师所授课程的学生中，成绩最高的学生信息及其成绩

select stu.* from Student stu join Score sc on sc.s_id=stu.s_id join
(select max(ss.s_score) sa from (
select sc.s_score,sc.s_id from Score sc join Course c
    on c.c_id=sc.c_id
    join Teacher t on t.t_id=c.t_id and t.t_name='张三'
group by sc.s_id,sc.s_score) ss) sss on sss.sa=sc.s_score;

# 41、查询不同课程成绩相同的学生的学生编号、课程编号、学生成绩

select DISTINCT b.s_id,b.c_id,b.s_score
    from score a,score b
        where a.c_id != b.c_id and a.s_score = b.s_score;

select stu.s_id,sc.c_id,sc.s_score from Student stu
    join Score sc on sc.s_id=stu.s_id
    join Course c on c.c_id=sc.c_id
    where (
        select count(1) from Student stu1
            join Score sc1 on sc1.s_id=stu1.s_id
            join Course c1 on c1.c_id=sc1.c_id where c.c_id!=c1.c_id and sc.s_score=sc1.s_score
              )>1;

select st.s_id,st.s_name,sc.c_id,sc.s_score from student st
left join score sc on sc.s_id=st.s_id
left join course c on c.c_id=sc.c_id
where (
select count(1) from student st2
left join score sc2 on sc2.s_id=st2.s_id
left join course c2 on c2.c_id=sc2.c_id
where sc.s_score=sc2.s_score and c.c_id!=c2.c_id
)>1;

#  42、查询每门功成绩最好的前两名
select a.* from (select st.s_id,st.s_name,c.c_name,sc.s_score from student st
left join score sc on sc.s_id=st.s_id
inner join course c on c.c_id=sc.c_id and c.c_id='01'
order by sc.s_score desc limit 0,2) a
union all
select b.* from (select st.s_id,st.s_name,c.c_name,sc.s_score from student st
left join score sc on sc.s_id=st.s_id
inner join course c on c.c_id=sc.c_id and c.c_id='02'
order by sc.s_score desc limit 0,2) b
union all
select cc.* from (select st.s_id,st.s_name,c.c_name,sc.s_score from student st
left join score sc on sc.s_id=st.s_id
inner join course c on c.c_id=sc.c_id and c.c_id='03'
order by sc.s_score desc limit 0,2) cc
union all
select d.* from (select st.s_id,st.s_name,c.c_name,sc.s_score from student st
left join score sc on sc.s_id=st.s_id
inner join course c on c.c_id=sc.c_id and c.c_id='04'
order by sc.s_score desc limit 0,2) d;

 select a.s_id,a.c_id,a.s_score from score a
    where (select COUNT(1) from score b
     where b.c_id=a.c_id and b.s_score>=a.s_score)<=2 order by a.c_id;

# 43、统计每门课程的学生选修人数（超过5人的课程才统计）。要求输出课程号和选修人数，查询结果按人数降序排列，若人数相同，按课程号升序排列
select sc.c_id,count(c_id) resu from Score sc group by sc.c_id having resu>5 order by resu desc ,c_id asc ;

select c_id,count(*) as total from score GROUP BY c_id HAVING total>5 ORDER BY total,c_id ASC;

#  44、检索至少选修两门课程的学生学号
select sc.s_id from Score sc group by sc.s_id having count(sc.s_id)>=2;

select s_id,count(*) as sel from score GROUP BY s_id HAVING sel>=2

# 45、查询选修了全部课程的学生信息
select stu.* from Student stu  where stu.s_id in (
            select sc.s_id from Score sc group by sc.s_id having count(s_id)=(
                select count(1) from Course
                )
);

# 46、查询各学生的年龄
select stu.s_name,year(now())-year(stu.s_birth) from Student stu;

select s_name,(DATE_FORMAT(NOW(),'%Y')-DATE_FORMAT(s_birth,'%Y') -
				(case when DATE_FORMAT(NOW(),'%m%d')>DATE_FORMAT(s_birth,'%m%d') then 0 else 1 end)) as age
		from student;


# 47、查询本周过生日的学生
```

