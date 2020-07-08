## MySQL50学习

```java
-- 建表语句
-- 学生表
CREATE TABLE `Student`(
`s_id` VARCHAR(20),
`s_name` VARCHAR(20) NOT NULL DEFAULT '',
`s_birth` VARCHAR(20) NOT NULL DEFAULT '',
`s_sex` VARCHAR(10) NOT NULL DEFAULT '',
PRIMARY KEY(`s_id`)
);
-- 课程表
CREATE TABLE `Course`(
`c_id` VARCHAR(20),
`c_name` VARCHAR(20) NOT NULL DEFAULT '',
`t_id` VARCHAR(20) NOT NULL,
PRIMARY KEY(`c_id`)
);
-- 教师表
CREATE TABLE `Teacher`(
`t_id` VARCHAR(20),
`t_name` VARCHAR(20) NOT NULL DEFAULT '',
PRIMARY KEY(`t_id`)
);
-- 成绩表
CREATE TABLE `Score`(
`s_id` VARCHAR(20),
`c_id` VARCHAR(20),
`s_score` INT(3),
PRIMARY KEY(`s_id`,`c_id`)
);
-- 插入学生表测试数据
insert into Student values('01' , '赵雷' , '1990-01-01' , '男');
insert into Student values('02' , '钱电' , '1990-12-21' , '男');
insert into Student values('03' , '孙风' , '1990-05-20' , '男');
insert into Student values('04' , '李云' , '1990-08-06' , '男');
insert into Student values('05' , '周梅' , '1991-12-01' , '女');
insert into Student values('06' , '吴兰' , '1992-03-01' , '女');
insert into Student values('07' , '郑竹' , '1989-07-01' , '女');
insert into Student values('08' , '王菊' , '1990-01-20' , '女');
-- 课程表测试数据
insert into Course values('01' , '语文' , '02');
insert into Course values('02' , '数学' , '01');
insert into Course values('03' , '英语' , '03');

-- 教师表测试数据
insert into Teacher values('01' , '张三');
insert into Teacher values('02' , '李四');
insert into Teacher values('03' , '王五');

-- 成绩表测试数据
insert into Score values('01' , '01' , 80);
insert into Score values('01' , '02' , 90);
insert into Score values('01' , '03' , 99);
insert into Score values('02' , '01' , 70);
insert into Score values('02' , '02' , 60);
insert into Score values('02' , '03' , 80);
insert into Score values('03' , '01' , 80);
insert into Score values('03' , '02' , 80);
insert into Score values('03' , '03' , 80);
insert into Score values('04' , '01' , 50);
insert into Score values('04' , '02' , 30);
insert into Score values('04' , '03' , 20);
insert into Score values('05' , '01' , 76);
insert into Score values('05' , '02' , 87);
insert into Score values('06' , '01' , 31);
insert into Score values('06' , '03' , 34);
insert into Score values('07' , '02' , 89);
insert into Score values('07' , '03' , 98);
########################################################################################################################
# 1. 查询课程编号为'01'的课程比'02'的课程成绩高的所有学生的学号(重点)
select b.s_id, c.s_name, a.s_score as '01', b.s_score as '02'
from (
         (select s_id, c_id, s_score
          from test_mysql.Score
          where c_id = '01') as a
             inner join
             (select s_id, c_id, s_score
              from test_mysql.Score
              where c_id = '02') as b
             on a.s_id = b.s_id
         )
         inner join test_mysql.Student as c
                    on b.c_id = c.s_id
where a.s_score > b.s_score;

########################################################################################################################
# 2. 查询平均成绩大于60分的学生的学号和平均成绩(第二道重点)
select a.s_id, round(avg(a.s_score), 2) as avg_score
from test_mysql.Score as a
group by a.s_id
having avg_score > 60;

########################################################################################################################
# 3. 查询所有学生的学号,姓名,选课数,总成绩
select a.s_id,
       a.s_name,
       count(b.c_id)                                              as 'count_num',
       -- 总成绩为null时取值0
       sum(if(b.s_score is null, 0, b.s_score))                   as 'sum_score',
       -- 总成绩为null时取值0
       sum(case when b.s_score is null then 0 else b.s_score end) as 'sum_score'
from test_mysql.Student as a
         -- 左关联数据更多
         left join test_mysql.Score as b
                   on a.s_id = b.s_id
group by a.s_id, a.s_name;

########################################################################################################################
# 4. 查询姓'李'的老师的个数(不重要)
select count(a.t_id) AS '姓李老师的个数'
from test_mysql.Teacher as a
where a.t_name like '李%';

########################################################################################################################
# 5. 查询没学过'张三'老师课的学生的学号,姓名
select a.s_id, a.s_name
from test_mysql.Student as a
where a.s_id not in (
    select sc.s_id
    from test_mysql.Score as sc
             inner join test_mysql.Course as co on sc.c_id = co.c_id
             inner join test_mysql.Teacher as te on co.t_id = te.t_id
    where te.t_name = '张三');

########################################################################################################################
# 6. 查询学过'张三'老师课的学生的学号,姓名
select a.s_id, a.s_name
from test_mysql.Student as a
where a.s_id in (
    select sc.s_id
    from test_mysql.Score as sc
             inner join test_mysql.Course as co on sc.c_id = co.c_id
             inner join test_mysql.Teacher as te on co.t_id = te.t_id
    where te.t_name = '张三');

########################################################################################################################
# 7. 查询学过编号为'01'的课程并且也学过编号为'02'的课程的学生的学号,姓名
select s_id, s_name
from test_mysql.Student
where s_id in
      (
          select a.s_id
          from (select s_id
                from test_mysql.Score
                where c_id = '01'
               ) as a
                   inner join
               (select s_id
                from test_mysql.Score
                where c_id = '02'
               ) as b
               on a.s_id = b.s_id
      );

########################################################################################################################
# 8. 查询课程编号为'02'的总成绩(不重点)
select sum(s_score) as '02编号课程总成绩'
from test_mysql.Score
where c_id = '02';

########################################################################################################################
# 9. 查询所有课程成绩小于60分的学生的学号,姓名
select a.s_id, c.s_name
from (
         (
             select s_id, count(c_id) as c_num
             from test_mysql.Score
             where s_score < 60
             group by s_id
         ) as a
             inner join
             (
                 select s_id, count(c_id) as c_num
                 from test_mysql.Score
                 group by s_id
             ) as b
             on a.s_id = b.s_id
    )
         inner join test_mysql.Student as c
                    on a.s_id = c.s_id
-- 判断两个子表的课程数一致
where a.c_num = b.c_num;

########################################################################################################################
# 10. 查询没有学全所有课的学生的学号,姓名(重点)
select a.s_id, a.s_name
from test_mysql.Student as a
         -- 左关联时,左表的数据不会被删减(左表数据肯定会出现)
         -- 内连接时会出现为null的那条数据08(选课0的情况)
         left join test_mysql.Score as b
                   on a.s_id = b.s_id
group by a.s_id, a.s_name
-- 表明课程树<3没学完全部课程(关键点)
having count(distinct c_id) < (select count(distinct c_id) from test_mysql.Course);

########################################################################################################################
# 11. 查询至少有一门课与学号为'01'的学生所学课程相同的学生的学号和姓名(重点)
-- 子查询
select s_id, s_name
from test_mysql.Student
where s_id in
      (
          select distinct s_id
          from test_mysql.Score as a
          where a.c_id in
                (
                    select c_id
                    from test_mysql.Score
                    where s_id = '01'
                )
      )
  -- 排除01
  and s_id != '01';

-- 关联查询
select a.s_id, a.s_name
from test_mysql.Student as a
         inner join
     (
         select distinct s_id
         from test_mysql.Score as a
         where a.c_id in
               (
                   select c_id
                   from test_mysql.Score
                   where s_id = '01'
               )
     ) as b
     on a.s_id = b.s_id
         -- 排除01
         and a.s_id != '01';

########################################################################################################################
# 12. 查询和'01'号同学所学课程完全相同的其他同学的学号(重点)
select s_id, s_name
from test_mysql.Student
where s_id in
      (
          select s_id
          from test_mysql.Score
          where s_id != '01'
          group by s_id
          having count(distinct c_id) =
                 (
                     select count(distinct c_id)
                     from test_mysql.Score
                     where s_id = '01'
                 )
      )
  -- 因为第一次过滤把1去掉了,防止1234/234/124情况出现
  and s_id not in
      (
          select distinct s_id
          from test_mysql.Score
          where c_id not in
                (
                    select c_id
                    from test_mysql.Score
                    where s_id = '01'
                )
      );

########################################################################################################################
# 13. 查询没学过"张三"老师讲授的任一门课程的学生姓名(重点)
select distinct a.s_name
from test_mysql.Student as a
         left join test_mysql.Score as b
                   on a.s_id = b.s_id
         left join test_mysql.Course as c
                   on b.c_id = c.c_id
         left join test_mysql.Teacher as d
                   on c.t_id = d.t_id
where a.s_id not in
      (
          -- 学过张三老师的课学生
          select a.s_id
          from test_mysql.Student as a
                   left join test_mysql.Score as b
                             on a.s_id = b.s_id
                   left join test_mysql.Course as c
                             on b.c_id = c.c_id
                   left join test_mysql.Teacher as d
                             on c.t_id = d.t_id
          where d.t_name = '张三'
      );

########################################################################################################################
# 15. 查询两门及其以上不及格课程的同学的学号,姓名及其平均成绩(重点)
select b.s_id, b.s_name, round(avg(c.s_score), 2) as '平均成绩'
from test_mysql.Student as b
         inner join test_mysql.Score as c
                    on b.s_id = c.s_id
where b.s_id in
      (
          -- 查询两门及其以上不及格课程的同学的学号
          select a.s_id
          from test_mysql.Score as a
          where a.s_score < 60
          group by a.s_id
          having count(distinct a.c_id) >= 2
      )
group by b.s_id, b.s_name;

########################################################################################################################
# 16. 检索"01"课程分数小于60,按分数降序排列的学生信息(不重点)
select a.*, b.s_score
from test_mysql.Student as a
         inner join test_mysql.Score as b
                    on a.s_id = b.s_id
where b.s_score < 60
  and b.c_id = '01'
-- 降序排序
order by b.s_score desc;

########################################################################################################################
# 17. 按平均成绩从高到低显示所有学生的所有课程的成绩以及平均成绩(重点)
-- IF函数(三目运算符)
select a.s_id,
       ROUND(avg(a.s_score), 2)            as '平均成绩',
       MAX(IF(c_id = '01', s_score, null)) as '语文',
       MAX(IF(c_id = '02', s_score, null)) as '数学',
       MAX(IF(c_id = '03', s_score, null)) as '英语'
from test_mysql.Score as a
group by a.s_id
order by avg(a.s_score) desc;

-- CASE WHEN
select a.s_id,
       ROUND(avg(a.s_score), 2)                              as '平均成绩',
       MAX(case when c_id = '01' then s_score else null end) as '语文',
       MAX(case when c_id = '02' then s_score else null end) as '数学',
       MAX(case when c_id = '03' then s_score else null end) as '英语'
from test_mysql.Score as a
group by a.s_id
order by avg(a.s_score) desc;

########################################################################################################################
# 18.查询各科成绩最高分,最低分和平均分: 以如下形式显示:课程ID,课程name,最高分,最低分,平均分,及格率,中等率,优良率,优秀率
select a.c_id,
       b.c_name,
       MIN(a.s_score)                                                                      as 'min',
       MAX(a.s_score)                                                                      as 'max',
       ROUND(avg(a.s_score), 2)                                                            as 'avg',
       CONCAT(round(avg(if(a.s_score > 60, 1.0, 0.0) * 100), 2), '%')                      as 'jg',
       CONCAT(round(avg(if(a.s_score <= 70 and a.s_score < 80, 1.0, 0.0) * 100), 2), '%')  as 'zd',
       CONCAT(round(avg(if(a.s_score <= 80 and a.s_score < 90, 1.0, 0.0) * 100), 2), '%')  as 'yl',
       CONCAT(round(avg(if(a.s_score <= 90 and a.s_score < 100, 1.0, 0.0) * 100), 2), '%') as 'yx'
from test_mysql.Score as a
         inner join test_mysql.Course as b
                    on a.c_id = b.c_id
group by a.c_id, b.c_name
order by avg(a.s_score) desc

########################################################################################################################
# 19. 按各科成绩进行排序,并显示排名(重点row_number)
select c.s_name, b.c_name, a.s_score, row_number() over (order by a.s_score desc) as 'row_number_rank'
from test_mysql.Score as a
         inner join test_mysql.Course as b
                    on a.c_id = b.c_id
         inner join test_mysql.Student as c
                    on c.s_id = a.s_id;

########################################################################################################################
# 20. 查询学生的总成绩并进行排名(不重点)
select a.s_id                                            as 'no',
       b.s_name                                          as 'name',
       sum(a.s_score)                                    as min_rank,
       dense_rank() over (order by sum(a.s_score) desc ) as 'rank'
from test_mysql.Score as a
         left join test_mysql.Student as b
                   on a.s_id = b.s_id
group by b.s_name, a.s_id
order by min_rank desc

########################################################################################################################
# 21. 查询不同老师所教不同课程平均分从高到低显示(不重点)
select a.c_id, c.t_name, round(avg(a.s_score), 2) as avg_score
from test_mysql.Score as a
         left join test_mysql.Course as b
                   on a.c_id = b.c_id
         left join test_mysql.Teacher as c
                   on b.t_id = c.t_id
group by a.c_id
order by avg_score desc;

########################################################################################################################
# 22. 查询所有课程的成绩第2名到第3名的学生信息及该课程成绩(重要25类似)
select c.s_id, s_name, s_birth, s_sex, s_score
from (
         -- 构建子表并且进行分组
         select b.s_id,
                s_name,
                s_birth,
                s_sex,
                a.s_score,
                -- 排名根据成绩排序并根据课程id分组
                row_number() over (partition by a.c_id order by a.s_score desc) as rank_num
         from test_mysql.Score as a
                  inner join test_mysql.Student as b
                             on a.s_id = b.s_id
     ) as c
where rank_num in (2, 3);

########################################################################################################################
# 23. 使用分段[100-85],[85-70],[70-60],[<60]来统计各科成绩,分别统计各分数段人数: 课程ID和课程名称(重点和18题类似)
select a.c_id,
       b.c_name,
       round(avg(a.s_score), 2)                                                         as 'avg_score',
       concat(round(avg(if(s_score >= 85 and s_score <= 100, 1.0, 0.0)) * 100, 2), '%') as '[100-85]',
       count(if(s_score >= 85 and s_score <= 100, 1, null))                             as 'people_num',
       concat(round(avg(if(s_score >= 70 and s_score <= 84, 1.0, 0.0)) * 100, 2), '%')  as '[85-70]',
       count(if(s_score >= 70 and s_score <= 84, 1, null))                              as 'people_num',
       concat(round(avg(if(s_score >= 60 and s_score <= 69, 1.0, 0.0)) * 100, 2), '%')  as '[70-60]',
       count(if(s_score >= 60 and s_score <= 69, 1, null))                              as 'people_num',
       concat(round(avg(if(s_score < 60 and s_score > 0, 1.0, 0.0)) * 100, 2), '%')     as '[<60]',
       count(if(s_score < 60 and s_score > 0, 1, null))                                 as 'people_num'
from test_mysql.Score as a
         inner join test_mysql.Course as b
                    on a.c_id = b.c_id
group by a.c_id, b.c_name;

########################################################################################################################
# 24. 查询学生平均成绩及其名次(重点)
select a.s_id,
       avg(a.s_score)                                     as avg_score,
       dense_rank() over ( order by avg(a.s_score) desc ) as avg_dense_rank
from test_mysql.Score as a
group by a.s_id;

########################################################################################################################
# 25. 查询各科成绩前三名的记录(不考虑成绩并列情况)
select c.s_id, s_name, s_birth, s_sex, s_score, c.c_name, c.rank_num
from (
         -- 构建子表并且进行分组
         select b.s_id,
                s_name,
                s_birth,
                s_sex,
                a.s_score,
                -- 排名根据成绩排序并根据课程id分组
                row_number() over (partition by a.c_id order by a.s_score desc) as rank_num,
                d.c_name
         from test_mysql.Score as a
                  inner join test_mysql.Student as b
                             on a.s_id = b.s_id
                  inner join test_mysql.Course as d
                             on a.c_id = d.c_id
     ) as c
where rank_num in (1, 2, 3);

########################################################################################################################
# 26. 查询每门课程被选修的学生数(不重点)
select c.c_id, c.c_name, count(distinct b.s_id)
from test_mysql.Score as b
         inner join test_mysql.Course as c
                    on c.c_id = b.c_id
group by c.c_id, c.c_name;

########################################################################################################################
# 27. 查询出只有两门课程的全部学生的学号和姓名(不重点)
select a.s_id, a.s_name
from test_mysql.Student as a
where s_id in
      (
          select b.s_id
          from test_mysql.Score as b
          group by b.s_id
          having count(distinct b.c_id) = 2
      );

########################################################################################################################
# 28. 查询男生,女生人数(不重点)
select s_sex as 'sex', count(distinct s_id) as 'people_num'
from test_mysql.Student
group by s_sex;

########################################################################################################################
# 29 查询名字中含有"风"字的学生信息(不重点)
select a.s_id, s_name, s_birth, s_sex
from test_mysql.Student as a
where a.s_name like '%风%';

########################################################################################################################
# 31. 查询1990年出生的学生名单(重点year)
select *
from test_mysql.Student as a
where year(a.s_birth) = 1990;

select *
from test_mysql.Student as a
where a.s_birth like '1990%';

########################################################################################################################
# 32. 查询平均成绩大于等于85的所有学生的学号,姓名和平均成绩(不重要)
select a.s_id, round(avg(a.s_score), 2) as avg_score
from test_mysql.Score as a
         inner join test_mysql.Student as b
                    on a.s_id = b.s_id
group by a.s_id
having avg_score >= 85
order by avg_score desc;

########################################################################################################################
# 33. 查询每门课程的平均成绩,结果按平均成绩升序排序,平均成绩相同时,按课程号降序排列(不重要)
select a.c_id, avg(a.s_score) as b
from test_mysql.Score as a
group by a.c_id
order by b desc, a.c_id;

########################################################################################################################
# 34. 查询课程名称为"数学",且分数低于60的学生姓名和分数(不重点)
select a.s_name, b.s_score
from test_mysql.Student as a
         inner join test_mysql.Score as b
                    on a.s_id = b.s_id
         inner join test_mysql.Course as c
                    on b.c_id = c.c_id
where c.c_name = '数学'
  and b.s_score < 60;

########################################################################################################################
# 35. 查询所有学生的课程及分数情况(重点)
select b.s_score, c.c_id, c_name, t_id
from test_mysql.Student as a
         inner join test_mysql.Score as b
                    on a.s_id = b.s_id
         inner join test_mysql.Course as c
                    on b.c_id = c.c_id

########################################################################################################################
# 36. 查询任何一门课程成绩在70分以上的姓名,课程名称和分数(重点)
select a.s_name, c.c_name, b.s_score
from test_mysql.Student as a
         inner join test_mysql.Score as b
                    on a.s_id = b.s_id
         inner join test_mysql.Course as c
                    on b.c_id = c.c_id
where b.s_score > 70;

########################################################################################################################
# 37. 查询不及格的课程并按课程号从大到小排列(不重点)
select a.s_id, c_id, s_score
from test_mysql.Score as a
where a.s_score < 60
order by a.c_id desc;

########################################################################################################################
# 38. 查询课程编号为03且课程成绩在80分以上的学生的学号和姓名(不重要)
select a.s_id, a.s_name
from test_mysql.Student as a
         inner join test_mysql.Score as b
                    on a.s_id = b.s_id
where b.c_id = '03'
  and b.s_score > 80;

########################################################################################################################
# 39. 求每门课程的学生人数(不重要)
select a.c_id, count(distinct a.s_id)
from test_mysql.Score as a
group by a.c_id

########################################################################################################################
# 40. 查询选修'张三'老师所授课程的学生中成绩最高的学生姓名及其成绩(limit/top)
select a.s_name, b.s_score
from test_mysql.Student as a
         inner join test_mysql.Score as b
                    on a.s_id = b.s_id
         inner join test_mysql.Course as c
                    on b.c_id = c.c_id
         inner join test_mysql.Teacher as d
                    on c.t_id = d.t_id
where d.t_name = '张三'
order by b.s_score desc
limit 1

########################################################################################################################
# 41. 查询不同课程成绩相同的学生的学生编号,课程编号,学生成绩(重点)
select a.s_id
from (
         -- 子表分组得到只有一门课程相同
         select b.s_id, b.s_score
         from test_mysql.Score as b
                  inner join
              (
                  -- 排除只有一门课程的情况
                  select s_id
                  from test_mysql.Score
                  group by s_id
                  having count(distinct c_id) > 1
              ) as c
              on b.s_id = c.s_id
         group by b.s_id, b.s_score
     ) as a
group by a.s_id
having count(a.s_id) = 1;

########################################################################################################################
# 42. 查询每门功成绩最好的前两名(同22和25题)
select c.s_id, s_name, s_birth, s_sex, s_score
from (
         -- 构建子表并且进行分组
         select b.s_id,
                s_name,
                s_birth,
                s_sex,
                a.s_score,
                -- 排名根据成绩排序并根据课程id分组
                row_number() over (partition by a.c_id order by a.s_score desc) as rank_num
         from test_mysql.Score as a
                  inner join test_mysql.Student as b
                             on a.s_id = b.s_id
     ) as c
where rank_num in (1, 2);

########################################################################################################################
# 43. 统计每门课程的学生选修人数(超过5人的课程才统计)要求输出课程号和选修人数,询结果按人数降序排列,若人数相同,按课程号升序排列(不重要)
select a.c_id, count(a.s_id) as num
from test_mysql.Score as a
group by a.c_id
having count(a.s_id) > 5
-- 默认为asc升序排序
order by num desc, a.c_id ;

########################################################################################################################
# 44. 检索至少选修两门课程的学生学号(不重要)
select a.s_id, count(b.c_id) as a
from test_mysql.Student as a
         inner join test_mysql.Score as b
                    on a.s_id = b.s_id
group by a.s_id
having count(b.c_id) > 2;

########################################################################################################################
# 45. 查询选修了全部课程的学生信息(重点划红线地方)
select a.s_id, count(distinct b.c_id) as num
from test_mysql.Student as a
         inner join test_mysql.Score as b
                    on a.s_id = b.s_id
group by a.s_id
having num =
       (select count(distinct c_id)
        from test_mysql.Score as a);

########################################################################################################################
# 46. 查询各学生的年龄(精确到月份)
select a.s_id, a.s_name, a.s_birth, floor(datediff(now(), a.s_birth) / 365) as 'age'
from test_mysql.Student as a

########################################################################################################################
# 47. 查询没学过'张三'老师讲授的任一门课程的学生姓名
select e.s_name
from test_mysql.Student as e
where e.s_id not in
      (
          select a.s_id
          from test_mysql.Student as a
                   inner join test_mysql.Score as b
                              on a.s_id = b.s_id
                   inner join test_mysql.Course as c
                              on b.c_id = c.c_id
                   inner join test_mysql.Teacher as d
                              on c.t_id = d.t_id
          where d.t_name = '张三'
      );

########################################################################################################################
# 48. 查询两门以上不及格课程的同学的学号及其平均成绩
select e.s_name
from test_mysql.Student as e
where e.s_id not in
      (
          select a.s_id
          from test_mysql.Student as a
                   inner join test_mysql.Score as b
                              on a.s_id = b.s_id
                   inner join test_mysql.Course as c
                              on b.c_id = c.c_id
                   inner join test_mysql.Teacher as d
                              on c.t_id = d.t_id
          where d.t_name = '张三'
      );

########################################################################################################################
# 49. 查询本月过生日的学生
select a.s_id, s_name, s_birth, s_sex
from test_mysql.Student as a
where month(a.s_birth) = month(date(now()));

########################################################################################################################
# 50. 查询本周过生日的学生
select a.s_id, s_name, s_birth, s_sex
from test_mysql.Student as a
where week(a.s_birth, 1) = week(date(now()), 1);

########################################################################################################################
# 50. 查询本周过生日的学生
select a.s_id, s_name, s_birth, s_sex
from test_mysql.Student as a
where day(a.s_birth) = day(date(now()));
########################################################################################################################
```