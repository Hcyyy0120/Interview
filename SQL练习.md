![image-20230420203148488](SQL练习.assets/image-20230420203148488.png)



### 一、

#### 查询课程编号为01的课程比02的课程成绩高的所有学生的学号（重要）

```sql

SELECT
	a.* 
FROM
	( SELECT s_id, s_score FROM score WHERE c_id = '01' ) a
	JOIN ( SELECT s_id, s_score FROM score WHERE c_id = '02' ) b ON a.s_id = b.s_id 
WHERE
	a.s_score > b.s_score
	
#自连接
SELECT
	s1.学号 
FROM
	score s1
	JOIN score s2 ON s1.学号 = s2.学号 
WHERE
	s1.课程号 = '01' 
	AND s2.课程号 = '02' 
	AND s1.成绩 > s2.成绩

```

### 二、

#### 查询平均成绩大于60分的学生的学号和平均成绩（重点）

```sql
#1
SELECT
	t.s_id,
	t.avgScore 
FROM
	( SELECT s_id, avg( s_score ) AS avgScore FROM score GROUP BY s_id ) AS t 
WHERE
	t.avgScore > 60
#2
SELECT
	s_id,
	avg( s_score ) 
FROM
	score 
GROUP BY
	s_id 
HAVING
	avg( s_score ) > 60
```

### 三、

#### 查询所有学生的学号、姓名、选课数、总成绩（不重要）

```sql
#1
SELECT
	student.s_id,
	student.s_name,
	IFNULL(t.courseNum,0),
	IFNULL(t.scoreSum,0)
FROM
	student
	LEFT JOIN ( SELECT score.s_id, count( c_id ) AS courseNum, sum( s_score ) AS scoreSum 
               FROM score 
               GROUP BY score.s_id ) AS t 
               ON student.s_id = t.s_id
#2               
SELECT
	a.s_id,
	a.s_name,
	IFNULL( count( b.c_id ), 0 ),
	IFNULL( sum( b.s_score ), 0 ) 
FROM
	student AS a
	LEFT JOIN score AS b 
	ON a.s_id = b.s_id 
GROUP BY
	a.s_id
```

### 四、

#### 查询姓张的老师的个数（不重要）

```sql
select count(t_id) from teacher where t_name like "张%"

#查询老师中有几种姓氏
select count(DISTINCT t_name) from teacher 
```

### 五、

#### 查询没学过张三老师课的学生的学号和姓名(重要)

```sql
SELECT
	student.s_id,
	student.s_name 
FROM
	student 
WHERE
	s_id NOT IN (
	SELECT
		score.s_id 
	FROM
		score
	WHERE
		score.c_id = ( SELECT a.c_id FROM course AS a JOIN teacher AS b ON a.t_id = b.t_id WHERE b.t_name = "张三" ))
```

### 六、

#### 查询学过"张三"老师所教的所有课的同学的学号、姓名（重要）

```sql
#创建张三所代课程的课程号视图 mc
DROP VIEW
IF EXISTS mc;
CREATE VIEW mc AS ( SELECT c.c_id FROM course c JOIN teacher t ON c.t_id = t.t_id WHERE t.t_name = '张三' );

#查询选过mc中所有课程的学生学号、姓名
SELECT
	a.s_id,
	st.s_name 
FROM
	( 
		SELECT s_id, count(*) cnt FROM score WHERE c_id IN ( SELECT c_id FROM mc ) GROUP BY s_id 
	) a
	JOIN student st ON a.s_id = st.s_id 
WHERE
	a.cnt = 
	(
		SELECT
			count(*) 
		FROM
			mc
	)
```

### 七、

#### 查询学过编号为"01”的课程并且也学过编号为"02"的课程的学生的学号、姓名（重要）

```sql
SELECT
	st.s_id,
	st.s_name 
FROM
	student st
	JOIN score s 
	ON st.s_id = s.s_id 
WHERE
	s.c_id = "02" 
	AND s.s_id IN 
	(
		SELECT
			s_id 
		FROM
			score 
		WHERE
			c_id = "01"
    )
#自连接 
SELECT
	s1.学号,姓名 
FROM
	score s1
	JOIN score s2 ON s1.学号 = s2.学号
	JOIN student s ON s.学号 = s2.学号 
WHERE
	s1.课程号 = '01' AND s2.课程号 = '02';
```

### 八、

#### 查询课程编号为“02”的总成绩

```sql
select c_id,sum(s_score) from score GROUP BY c_id having c_id = "02"
```

### 九、

#### 查询  所有课程  的成绩都小于60分的学生的学号、姓名

```sql
#1
SELECT
	t1.s_id,
	st.s_name 
FROM
	( SELECT s_id, count( c_id ) cnt1 FROM score WHERE s_score < 60 GROUP BY s_id ) t1
	JOIN ( SELECT s_id, count( c_id ) cnt2 FROM score GROUP BY s_id ) t2 ON t1.s_id = t2.s_id
	JOIN student st ON st.s_id = t1.s_id 
WHERE
	t1.cnt1 = t2.cnt2

#2
SELECT
	st.s_id,
	st.s_name 
FROM
	student st
	JOIN score s ON st.s_id = s.s_id 
GROUP BY
	s_id 
HAVING
	max( s_score ) < 60
```

### 十、

#### 查询没有学全所有课的学生的学号、姓名(重点)

```sql
#注意左连接，因为有 没有选课的学生
SELECT
	st.s_id,
	st.s_name 
FROM
	student as st
	LEFT JOIN score as sc ON st.s_id = sc.s_id 
GROUP BY
	st.s_id 
HAVING
	count( DISTINCT sc.c_id ) < ( SELECT count( c_id ) FROM course )
```

### 十一、

#### 查询至少有一门课与学号为“01”的学生所学课程相同的学生的学号和姓名（重点）

```sql
SELECT
	DISTINCT st.s_id,
	st.s_name	 
FROM
	score sc
	JOIN student st ON sc.s_id = st.s_id 
WHERE
	sc.s_id != "01" 
	AND sc.c_id IN 
	(
		SELECT
			c_id 
		FROM
			score 
		WHERE
			s_id = "01"
    )
```

### 十二、

#### 查询和“01”号同学所学课程完全相同的其他同学的学号(重点，和六相似)

```sql
DROP VIEW IF EXISTS v;
CREATE VIEW v AS ( SELECT c_id FROM score WHERE s_id = "01" );

SELECT
	st.s_id,
	st.s_name 
FROM
	( 
		SELECT s_id, count( DISTINCT c_id ) cnt FROM score WHERE c_id IN ( SELECT c_id FROM v ) GROUP BY s_id 
	) temp
	JOIN student st ON st.s_id = temp.s_id 
WHERE
	temp.cnt = ( SELECT count(*) FROM v ) 
	AND st.s_id != "01"
```

完全相同 / 所有 ，先分组，进行数量统计，然后对数量进行等值判断

### 十三、

#### 查询两门及其以上不及格课程的同学的学号，姓名及其平均成绩（重点）

```sql
SELECT
	sc.s_id,
	st.s_name,
	avg( s_score ) 
FROM
	student st
	JOIN score sc ON st.s_id = sc.s_id 
WHERE
	st.s_id IN ( SELECT s_id FROM score WHERE s_score < 60 GROUP BY s_id HAVING count( DISTINCT c_id )>= 2 ) 
GROUP BY
	s_id
```

### 十四、

#### 检索"01"课程分数小于60，按分数降序排列的学生信息

```sql
SELECT
	st.*,
	sc.s_score 
FROM
	score sc
	JOIN student st ON sc.s_id = st.s_id 
WHERE
	sc.c_id = "01" 
	AND sc.s_score < 60 
ORDER BY
	sc.s_score DESC
```

### 十五、

#### 按平均成绩从高到低显示所有学生的所有课程的成绩以及平均成绩(重点)

```sql
#1
SELECT
	a.s_id,
	a.c_id,
	a.s_score,
	b.avg_score 
FROM
	score a
	JOIN ( SELECT s_id, avg( s_score ) avg_score FROM score GROUP BY s_id ) b ON a.s_id = b.s_id 
ORDER BY
	b.avg_score DESC

#2
SELECT
	s_id "学号",
	max( CASE WHEN c_id = "01" THEN s_score ELSE NULL END ) "语文",
	max( CASE WHEN c_id = "02" THEN s_score ELSE NULL END ) "数学",
	max( CASE WHEN c_id = "03" THEN s_score ELSE NULL END ) "英语",
	max( CASE WHEN c_id = "04" THEN s_score ELSE NULL END ) "化学",
	max( CASE WHEN c_id = "05" THEN s_score ELSE NULL END ) "生物",
	avg( s_score ) "平均成绩" 
FROM
	score 
GROUP BY
	s_id 
ORDER BY
	avg( s_score ) DESC
```

### 十六、

#### 按各科成绩进行排序，并显示排名(重点row_number)

```sql
#窗口函数MySQL8支持
select s_id,c_id,s_score,row_number() over(order by s_score desc) from score
```

![image-20230426213140933](SQL练习.assets/image-20230426213140933.png)

### 十七、

#### 查询学生的总成绩并进行排名

```sql
SELECT
	st.s_id,
	st.s_name,
	sum( sc.s_score ) 
FROM
	student st
	JOIN score sc ON st.s_id = sc.s_id 
GROUP BY
	s_id 
ORDER BY
	sum( sc.s_score ) DESC
```

### 十八、

#### 查询不同老师所教不同课程平均分  从高到低显示

```sql
SELECT
	c.c_id,
	t.t_name,
	c.c_name,
	avg( sc.s_score ) 
FROM
	score sc
	JOIN course c ON sc.c_id = c.c_id
	JOIN teacher t ON t.t_id = c.t_id 
GROUP BY
	c.c_id,c.c_name
ORDER BY
	avg( s_score ) DESC
```

### 十九、

#### 查询所有课程的成绩第2名到第3名的学生信息及该课程成绩（重要）

```sql
SELECT
	* 
FROM
	(
	SELECT
		st.s_id,
		st.s_name,
		c_id,
		s_score,
		ROW_NUMBER() over ( PARTITION BY c_id ORDER BY s_score DESC ) m 
	FROM
		score sc jpin student st ON sc.s_id = st.s_id 
	) t 
WHERE
	m IN ( 2, 3 )
```

### 二十、

#### 使用分段[100-85],[85-70],[70-60],[<60]来统计各科成绩，分别统计各分数段人数：课程ID和课程名称(重点）

```sql
#统计总分，若要统计人数，then 1 else 0
select c.c_id,c.c_name,
sum(case when sc.s_score<=100 and sc.s_score>85 then sc.s_score else 0 end) as "(85,100]",
sum(case when sc.s_score<=85 and sc.s_score>70 then sc.s_score else 0 end) as "(70,85]",
sum(case when sc.s_score<=70 and sc.s_score>60 then sc.s_score else 0 end) as "[60,70]",
sum(case when sc.s_score<60 then sc.s_score else 0 end) as "[0,60)"
from score sc join course c on sc.c_id = c.c_id group by c.c_id 
```

### 二十一、

#### 查询学生平均成绩及其名次（重点）

```sql
SELECT
	st.s_name,
	avg( sc.s_score ),
	ROW_NUMBER() over ( ORDER BY avg( sc.s_score ) DESC ) 
FROM
	student st
	JOIN score sc ON st.s_id = sc.s_id 
GROUP BY
	sc.s_id
```

### 二十二、

#### 查询每门课程被选修的学生数

```sql
select sc.c_id,count(sc.s_id) from score sc GROUP BY sc.c_id
```

### 二十三、

#### 查询出只有两门课程的全部学生的学号和姓名

```sql
SELECT
	st.s_name,
	st.s_id 
FROM
	student st
	JOIN score sc ON st.s_id = sc.s_id 
GROUP BY
	sc.s_id 
HAVING
	count( DISTINCT sc.c_id ) = 2
```

### 二十四、

#### 查询男生、女生人数

```sql
select s_sex,count(s_id) from student group by s_sex
```

### 二十五、

#### 查询名字中含有"风"字的学生信息

```sql
select * from student where s_name like '%风%'
```

### 二十六、

#### 查询1990年出生的学生名单（重点）

```sql
#YEAR函数
select * from student where YEAR(s_birth) = '1990'
```

### 二十七、

#### 查询平均成绩大于等于85的所有学生的学号、姓名和平均成绩

```sql
SELECT
	sc.s_id,
	st.s_name,
	avg( sc.s_score ) 
FROM
	student st
	JOIN score sc ON st.s_id = sc.s_id 
GROUP BY
	sc.s_id 
HAVING
	avg( sc.s_score ) >= 85
```

### 二十八、

#### 查询每门课程的平均成绩，结果按平均成绩升序排序，平均成绩相同时，按课程号降序排列

```sql
SELECT
	c_id,
	avg( s_score ) 
FROM
	score 
GROUP BY
	c_id 
ORDER BY
	avg( s_score ) ASC,
	c_id DESC
```

### 二十九、

#### 查询课程名称为"数学"，且分数低于60的学生姓名和分数

```sql
SELECT
st.s_name,
sc.s_score
FROM
	student st
	JOIN score sc ON st.s_id = sc.s_id 
WHERE
	sc.c_id = ( SELECT c.c_id FROM course c WHERE c.c_name = "数学" ) 
	AND sc.s_score < 60
```

### 三十、

#### 查询所有学生的课程及分数情况（重点）

```sql
#用max是因为使用了Group by,必须使用聚合函数（其他聚合函数也可）
SELECT
	st.s_id,
	st.s_name,
	max( CASE WHEN c.c_name = "语文" THEN sc.s_score ELSE NULL END ) "语文",
	max( CASE WHEN c.c_name = "数学" THEN sc.s_score ELSE NULL END ) "数学",
	max( CASE WHEN c.c_name = "英语" THEN sc.s_score ELSE NULL END ) "英语",
	max( CASE WHEN c.c_name = "化学" THEN sc.s_score ELSE NULL END ) "化学",
	max( CASE WHEN c.c_name = "生物" THEN sc.s_score ELSE NULL END ) "生物" 
FROM
	score sc
	RIGHT JOIN student st ON st.s_id = sc.s_id
	LEFT JOIN course c ON sc.c_id = c.c_id 
GROUP BY
	st.s_id,
	st.s_name
```

### 三十一、

#### 查询课程成绩在70分以上的姓名、课程名称和分数

```sql
SELECT
	st.s_name,
	c.c_name,
	sc.s_score 
FROM
	student st
	JOIN score sc ON st.s_id = sc.s_id
	JOIN course c ON c.c_id = sc.c_id 
WHERE
	sc.s_score > 70
```

### 三十二、

#### 查询不及格的课程并按课程号从大到小排列

```sql
SELECT
	st.s_id,
	st.s_name,
	c.c_name,
	sc.s_score,
	c.c_id 
FROM
	course c
	JOIN score sc ON c.c_id = sc.c_id
	JOIN student st ON st.s_id = sc.s_id 
WHERE
	sc.s_score < 60 
ORDER BY
	sc.c_id desc
```

### 三十三、

#### 查询课程编号为03且课程成绩在80分以上的学生的学号和姓名

```sql
SELECT
	st.s_id,
	st.s_name 
FROM
	score sc
	JOIN student st ON sc.s_id = st.s_id 
WHERE
	sc.c_id = "03" 
	AND sc.s_score > 80
```

### 三十四、

#### 求每门课程的学生人数

```sql
select c_id ,count(distinct s_id) from score group by c_id
```

### 三十五、

#### 查询选修“张三”老师所授课程的学生中成绩最高的学生姓名及其成绩（重要）

```sql
# order by + limit，没有考虑有多个相同的最高分
SELECT
	st.s_name,
	sc.s_score 
FROM
	score sc
	JOIN student st ON sc.s_id = st.s_id
	JOIN course c ON c.c_id = sc.c_id 
WHERE
	c.t_id IN ( SELECT t.t_id FROM teacher t WHERE t.t_name = "张三" ) 
ORDER BY
	sc.s_score DESC 
	LIMIT 1
#较完整的解法（可能复杂化）
SELECT
	st.s_name,
	sc.c_id,
	sc.s_score 
FROM
	student st
	JOIN score sc ON st.s_id = sc.s_id
	JOIN temp_view ON sc.c_id = temp_view.c_id 
WHERE
	sc.s_score = temp_view.maxScore;
	
DROP VIEW IF EXISTS temp_view;
CREATE VIEW temp_view AS (
	SELECT
		sc.c_id,
		max( sc.s_score ) maxScore 
	FROM
		score sc
		JOIN student st ON sc.s_id = st.s_id
		JOIN course c ON c.c_id = sc.c_id 
	WHERE
		c.t_id IN ( SELECT t.t_id FROM teacher t WHERE t.t_name = "张三" ) 
	GROUP BY
	sc.c_id 
	);
```

### 三十六、

#### 查询一个学生中不同课程成绩相同的学生编号、课程编号、成绩 （重点）

```sql
#注意distinct
SELECT
	distinct a.s_id,
	a.c_id,
	a.s_score 
FROM
	score a
	JOIN score b ON a.s_id = b.s_id 
WHERE
	a.s_score = b.s_score 
	AND a.c_id != b.c_id
```

### 三十七、

#### 统计每门课程的学生选修人数

（超过5人的课程才统计）。要求输出课程号和选修人数，查询结果按人数降序排列，若人数相同，按课程号升序排列

```sql
SELECT
	sc.c_id,
	count( sc.s_id ) count 
FROM
	score sc 
GROUP BY
	sc.c_id 
HAVING
	count( sc.s_id ) > 5 
ORDER BY
	count( sc.s_id ) DESC,
	sc.c_id ASC
```

### 三十八、

#### 检索至少选修两门课程的学生学号

```sql
SELECT
	sc.s_id,
	count(sc.c_id)
FROM
	score sc 
GROUP BY
	sc.s_id 
HAVING
	count( sc.c_id ) >= 2
```

### 三十九、

#### 查询选修了全部课程的学生信息（重点）

```sql
SELECT
	s_id,
	count( distinct c_id ) #重修
FROM
	score sc 
GROUP BY
	sc.s_id 
HAVING
	count( c_id ) = 
	(
		SELECT
			count( c_id ) 
		FROM
		course 
	)
```

### 四十、

#### 查询各学生的年龄（精确到月份）

```sql
#不严谨
SELECT
	s_id,
	s_birth,
	FLOOR( DATEDIFF( NOW(), s_birth )/ 365 ) "年龄" 
FROM
	student
```

