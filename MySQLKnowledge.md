# MySQL 知识点整理

## 简答题

### 两段封锁协议

1. 对任何一个数据进行读/写操作之前，事务必须获得对数据的封锁
2. 在释放一个封锁之后，事务不再获得任何其他封锁

### 最小函数依赖集步骤

#### 函数依赖规则（Armstrong 公理系统）

| 规则 | 内容 | 表达式 |
|:---|:---|:---|
| A1(自反性) | 如果 $Y \subseteq X \subseteq U$ | 则 $X \rightarrow Y$ |
| A2(增广性) | 如果 $X \rightarrow Y$ 且 $Z \subseteq U$ | 则 $XZ \rightarrow YZ$ |
| A3(传递性) | 如果 $X \rightarrow Y$ 且 $Y \rightarrow Z$ | 则 $X \rightarrow Z$ |
| B1(合并性) | 如果 $X \rightarrow Y$ 且 $X \rightarrow Z$ | 则 $X \rightarrow YZ$ |
| B2(分解性) | 如果 $X \rightarrow YZ$ | 则 $X \rightarrow Y$、$X \rightarrow Z$ |
| B3(结合性/伪传递) | 如果 $X \rightarrow Y$ 且 $W \rightarrow Z$ | 则 $XW \rightarrow YZ$ |
| B4(伪传递性) | 如果 $X \rightarrow Y$ 且 $WY \rightarrow Z$ | 则 $XW \rightarrow Z$ |

【例题】设有关系模式 R，属性集 $U = \{A,B,C,X,Y\}$，函数依赖集 $F=\{Z\rightarrow A, B\rightarrow X, AX\rightarrow Y, ZB\rightarrow Y\}$，试证明 $ZB\rightarrow Y$ 可由 F 导出。

**解答：**
1. 因为 $Z\rightarrow A$，$B\rightarrow X$，由 B3 可知，$ZB\rightarrow AX$
2. 因为 $ZB\rightarrow AX$，$AX\rightarrow Y$，由 A3 可知，$ZB\rightarrow Y$

即 $ZB\rightarrow Y$ 可由 F 中其他函数依赖导出，所以 $ZB\rightarrow Y$ 是冗余的函数依赖。

---

如果函数依赖集 F 满足下列条件，则称 F 为一个**最小函数依赖集**：

1. 每个函数依赖的右边都是单属性（可通过 B2 实现）
2. 函数依赖集 F 中没有冗余的函数依赖
3. F 中每个函数依赖的左边没有多余的属性

显然，每个函数依赖集至少存在一个最小依赖集，但并不一定唯一。

【例题】设 F 是关系模式 R(A,B,C) 的函数依赖集，$F=\{A\rightarrow BC, B\rightarrow C, A\rightarrow B, AB\rightarrow C\}$，试求最小函数依赖集。

**解答：**
1. 先把 F 中的函数依赖写成右边是单属性的形式：
   $$F=\{A\rightarrow B, A\rightarrow C, B\rightarrow C, A\rightarrow B, AB\rightarrow C\}$$
   删除一个 $A\rightarrow B$，得：
   $$F = \{A\rightarrow B, A\rightarrow C, B\rightarrow C, AB\rightarrow C\}$$

2. 删去冗余函数依赖。F 中的 $A\rightarrow C$ 可从 $A\rightarrow B$ 和 $B\rightarrow C$ 推出，因此 $A\rightarrow C$ 是冗余的，删去，得：
   $$F=\{A\rightarrow B, B\rightarrow C, AB\rightarrow C\}$$

3. 消除函数依赖左边冗余的属性。F 中的 $AB\rightarrow C$，因为有 $B\rightarrow C$，所以 A 多余，删去，得到最小函数依赖集为：
   $$F=\{A\rightarrow B, B\rightarrow C\}$$

---

### 数据库三级模式以及两级映射

#### 数据库系统的三级模式
- **外模式**（用户模式/子模式）
- **概念模式**（模式/逻辑模式）
- **内模式**（存储模式/物理模式）

#### 数据库二级映射
- **外模式/模式映射** → 保证逻辑独立性
- **模式/内模式映射** → 保证物理独立性

数据库系统的 3 个抽象级之间通过二级映射进行相互转换，使得数据库抽象的三级模式形成一个统一的整体。

#### 数据独立性

数据独立性是指程序与数据之间的独立性，主要包括：

1. **物理独立性**：应用程序与存储在磁盘上的数据库中的数据是独立的。通过**模式/内模式映射**实现。
2. **逻辑独立性**：用户的应用程序与逻辑结构是相互独立的。通过**外模式/模式映射**实现。

---

### 并发执行会出现的问题

| 问题 | 说明 |
|:---|:---|
| **丢失更新** | 两个事务同时修改同一数据，后提交的事务覆盖了先提交的事务的修改 |
| **不可重复读** | 同一事务内多次读取同一数据，结果不一致（被其他事务修改） |
| **读脏数据** | 读取了其他事务未提交的数据，随后该事务回滚 |

---

### 为什么先写日志后写数据库（WAL 协议）

1. **保证故障可恢复** — 系统崩溃时，日志已记录操作，可通过 UNDO/REDO 恢复一致性
2. **防止数据不一致** — 若先写数据库而日志丢失，故障后无法撤销未提交事务的修改
3. **WAL 机制要求** — 日志是恢复的唯一依据，必须先落盘才能确保数据库状态可回溯

---

## SQL 代码和关系表达式

### 表结构

#### S（学生表）

| 列名 | 数据类型 | 允许为空 | 特殊限制 |
|:---|:---|:---|:---|
| Sno | char(8) | 不允许 | 主键 |
| Sname | char(8) | 不允许 | |
| Sex | char(2) | 允许 | 取"男"或"女" |
| Dept | varchar(50) | 允许 | |
| Age | int | 不允许 | |

#### C（课程表）

| 列名 | 数据类型 | 允许为空 | 特殊限制 |
|:---|:---|:---|:---|
| Cno | char(3) | 不允许 | 主键 |
| Cname | varchar(20) | 允许 | |
| Ccredit | int | 不允许 | |

#### SC（选课表）

| 列名 | 数据类型 | 允许为空 | 特殊限制 |
|:---|:---|:---|:---|
| Sno | char(8) | 不允许 | 组合主键，外键 → S(Sno) |
| Cno | char(3) | 不允许 | 组合主键，外键 → C(Cno) |
| Grade | int | 允许 | 取值范围 0~100 |

---

### 基础查询

**1. 查询课号为 '01' 的课程信息**

```sql
SELECT * FROM C WHERE Cno = '01';
```

**2. 查询学分为 3 的课程号和课程名**

关系表达式：
$$\pi_{\text{Cno}, \text{Cname}}(\sigma_{\text{Ccredit}=3}(C))$$

```sql
SELECT Cno, Cname FROM C WHERE Ccredit = 3;
```

---

### SQL 语句练习

**1. 查询选修 '02' 课程的学生的学号和姓名**

```sql
SELECT S.Sno AS 学号, S.Sname AS 姓名 
FROM S, SC 
WHERE S.Sno = SC.Sno AND SC.Cno = '02';
```

**2. 查询平均分 > 70 分的课程课号、选课人数和平均分**

```sql
SELECT Cno AS 课号, COUNT(*) AS 选课人数, AVG(Grade) AS 平均分 
FROM SC
WHERE Grade > 70
GROUP BY Cno;
```

**3. 将选修课程号为 '02' 的所有学生的分数加 5 分**

```sql
UPDATE SC
SET Grade = Grade + 5
WHERE Cno = '02';
```

**4. 创建存储过程：输入课号、课程名、学分，增加一条课程数据**

```sql
DELIMITER //
CREATE PROCEDURE InsertCourse(
    IN p_Cno CHAR(3),
    IN p_Cname VARCHAR(20),
    IN p_Ccredit INT 
)
BEGIN
    INSERT INTO C VALUES(p_Cno, p_Cname, p_Ccredit);
END //
DELIMITER ;

-- 调用存储过程
CALL InsertCourse('04', '数据管理', 4);
```

**5. 创建视图：每科最高分的学生**

```sql
CREATE VIEW v_max_grade AS
SELECT SC.Cno, S.Sno AS 学号, S.Sname AS 姓名, SC.Grade AS 成绩
FROM S, SC
WHERE S.Sno = SC.Sno 
AND SC.Grade = (
    SELECT MAX(Grade) 
    FROM SC AS SC2 
    WHERE SC2.Cno = SC.Cno
);
```

**6. 在 S 表中增加一列 School（学校）**

```sql
ALTER TABLE S
ADD School VARCHAR(50) DEFAULT '燕京理工学院';
```

**7. 把 S 表的查询权限授权给 root 用户**

```sql
GRANT SELECT ON S TO 'root'@'localhost';
```

---

## 关系代数表达式

### 常用符号对照表

| 运算 | 符号 | LaTeX |
|:---|:---|:---|
| 选择 | $\sigma$ | `\sigma` |
| 投影 | $\pi$ | `\pi` |
| 并 | $\cup$ | `\cup` |
| 交 | $\cap$ | `\cap` |
| 差 | $-$ | `-` |
| 笛卡尔积 | $\times$ | `\times` |
| 自然连接 | $\bowtie$ | `\bowtie` |
| θ 连接 | $\bowtie_{\theta}$ | `\bowtie_\theta` |
| 除 | $\div$ | `\div` |
| 重命名 | $\rho$ | `\rho` |

---

### 表达式示例

**1. 查询选修了张三选修的所有课程的学生学号**

$$\pi_{\text{Sno}}(\sigma_{\text{Sname}='\text{张三}'}(S) \bowtie SC)$$

**2. 查询 C 语言课程成绩不及格的学生的学号和姓名**

$$\pi_{\text{Sno}, \text{Sname}}(\sigma_{\text{Grade} < 60}(S \bowtie SC \bowtie \sigma_{\text{Cname}='\text{C语言}'}(C)))$$

---

## 第一范式到第三范式

```
1NF —— 一张大表（属性不可再分）
  |
2NF —— 消除部分函数依赖（非主属性完全依赖于候选键）
  |
3NF —— 消除传递依赖（非主属性不依赖于其他非主属性）
```

---

### 【例题】综合练习

设有关系模式 R(运动员编号, 比赛项目, 成绩, 比赛类别, 比赛主管)，用于存储运动员比赛成绩及比赛类别、主管等信息。

**语义规定：**
- 每个运动员每参加一个比赛项目只有一个成绩
- 每个比赛项目只属于一个比赛类别
- 每个比赛类别只有一个比赛主管

**解答：**

**1. 基本函数依赖集和候选键**

```
F = {
    (运动员编号, 比赛项目) → 成绩,
    比赛项目 → 比赛类别,
    比赛类别 → 比赛主管
}

候选键：(运动员编号, 比赛项目)
```

**2. 分解为 2NF**

R 中存在部分函数依赖：
- 比赛项目 → (比赛类别, 比赛主管)

分解为：
- R1(运动员编号, 比赛项目, 成绩) — 主键：(运动员编号, 比赛项目)
- R2(比赛项目, 比赛类别, 比赛主管) — 主键：比赛项目

**3. 分解为 3NF**

R2 中存在传递依赖：比赛项目 → 比赛类别 → 比赛主管

继续分解：
- R21(比赛项目, 比赛类别) — 主键：比赛项目
- R22(比赛类别, 比赛主管) — 主键：比赛类别

最终 3NF：
- R1(运动员编号, 比赛项目, 成绩)
- R21(比赛项目, 比赛类别)
- R22(比赛类别, 比赛主管)

---

### 【例题】医疗关系分解为 3NF

现有关系：医疗(病人编号, 病人姓名, 病人性别, 病人年龄, 医生编号, 医生姓名, 医生职称, 诊断日期, 诊断结果)

**函数依赖集：**
```
F = {
    病人编号 → (病人姓名, 病人性别, 病人年龄),
    医生编号 → (医生姓名, 医生职称),
    (病人编号, 医生编号, 诊断日期) → 诊断结果
}
```

**分解为 3NF：**

| 关系模式 | 函数依赖 | 主键 | 外键 |
|:---|:---|:---|:---|
| 病人(病人编号, 病人姓名, 病人性别, 病人年龄) | 病人编号 → (病人姓名, 病人性别, 病人年龄) | 病人编号 | |
| 医生(医生编号, 医生姓名, 医生职称) | 医生编号 → (医生姓名, 医生职称) | 医生编号 | |
| 医疗(病人编号, 医生编号, 诊断日期, 诊断结果) | (病人编号, 医生编号, 诊断日期) → 诊断结果 | (病人编号, 医生编号, 诊断日期) | 病人编号 → 病人，医生编号 → 医生 |

---

## E-R 图设计

### 【例题】车辆管理系统

**需求：**
1. 车队：车队号、车队名
2. 车辆：牌照号、厂家、出厂日期
3. 司机：司机编号、姓名、电话
4. 车队聘用司机（1:n），有聘期
5. 车队拥有车辆（1:n）
6. 司机使用车辆（m:n），有使用日期和公里数

**关系模式：**

| 关系模式 | 属性 | 主键 | 外键 |
|:---|:---|:---|:---|
| 车队 | (车队号, 车队名) | 车队号 | |
| 车辆 | (牌照号, 厂家, 出厂日期, 车队号) | 牌照号 | 车队号 → 车队 |
| 司机 | (司机编号, 姓名, 电话, 车队号, 聘期) | 司机编号 | 车队号 → 车队 |
| 使用 | (牌照号, 司机编号, 日期, 公里数) | (司机编号, 牌照号, 日期) | 司机编号 → 司机，牌照号 → 车辆 |

---

## 补充知识点

### 1. ALTER 语句（修改表结构）

ALTER 用于修改已存在的表结构，包括添加列、删除列、修改列类型、添加约束等。

**基于 S、SC、C 表的示例：**

#### 添加列

```sql
-- 在 S 表中添加 Phone 列
ALTER TABLE S ADD Phone CHAR(11);

-- 在 SC 表中添加 ExamDate 列，默认值为当前日期
ALTER TABLE SC ADD ExamDate DATE DEFAULT CURDATE();

-- 在 C 表中添加 Semester 列（开课学期），放在 Ccredit 列之后
ALTER TABLE C ADD Semester INT AFTER Ccredit;
```

#### 修改列

```sql
-- 修改 S 表中 Sname 的长度为 20
ALTER TABLE S MODIFY Sname VARCHAR(20);

-- 修改 S 表中 Age 列的数据类型为 SMALLINT，并添加注释
ALTER TABLE S MODIFY Age SMALLINT COMMENT '学生年龄';

-- 修改 SC 表中 Grade 列允许为空（原为不允许）
ALTER TABLE SC MODIFY Grade INT NULL;

-- 修改列名（将 S 表中的 Sex 改为 Gender）
ALTER TABLE S CHANGE Sex Gender CHAR(2);
```

#### 删除列

```sql
-- 删除 S 表中的 Phone 列
ALTER TABLE S DROP COLUMN Phone;

-- 删除 SC 表中的 ExamDate 列
ALTER TABLE SC DROP COLUMN ExamDate;
```

#### 添加约束

```sql
-- 为 S 表的 Age 添加 CHECK 约束（年龄必须在 15-50 之间）
ALTER TABLE S ADD CONSTRAINT chk_age CHECK (Age BETWEEN 15 AND 50);

-- 为 SC 表的 Grade 添加 CHECK 约束（成绩在 0-100 之间）
ALTER TABLE SC ADD CONSTRAINT chk_grade CHECK (Grade BETWEEN 0 AND 100);

-- 为 S 表的 Dept 添加默认值
ALTER TABLE S ALTER COLUMN Dept SET DEFAULT '计算机学院';
```

#### 删除约束

```sql
-- 删除 S 表中的 chk_age 约束
ALTER TABLE S DROP CONSTRAINT chk_age;

-- 删除 SC 表中的 chk_grade 约束
ALTER TABLE SC DROP CONSTRAINT chk_grade;
```

#### 修改表名

```sql
-- 将 S 表改名为 Student
ALTER TABLE S RENAME TO Student;

-- 将 C 表改名为 Course
ALTER TABLE C RENAME TO Course;
```

---

### 2. DROP 语句（删除数据库对象）

DROP 用于删除数据库、表、视图、索引等对象。**删除后不可恢复**，需谨慎使用。

**基于 S、SC、C 表的示例：**

#### 删除表

```sql
-- 删除 SC 表（选课表）
DROP TABLE SC;

-- 删除 S 表（学生表），如果存在则删除
DROP TABLE IF EXISTS S;

-- 删除 C 表（课程表），并级联删除外键约束
DROP TABLE C CASCADE;
```

#### 删除数据库

```sql
-- 删除名为 school 的数据库
DROP DATABASE school;

-- 如果存在则删除
DROP DATABASE IF EXISTS school;
```

#### 删除视图

```sql
-- 删除之前创建的 v_max_grade 视图
DROP VIEW v_max_grade;

-- 如果存在则删除
DROP VIEW IF EXISTS v_max_grade;
```

#### 删除索引

```sql
-- 删除 S 表上的 idx_sname 索引
DROP INDEX idx_sname ON S;
```

#### 删除存储过程

```sql
-- 删除之前创建的 InsertCourse 存储过程
DROP PROCEDURE InsertCourse;

-- 如果存在则删除
DROP PROCEDURE IF EXISTS InsertCourse;
```

---

### 3. DELETE 语句（删除表中的数据）

DELETE 用于删除表中的行记录，**保留表结构**，可以配合 WHERE 条件删除指定数据。

**基于 S、SC、C 表的示例：**

#### 删除指定条件的数据

```sql
-- 删除 S 表中 Dept 为 '数学系' 的所有学生
DELETE FROM S WHERE Dept = '数学系';

-- 删除 SC 表中 Cno 为 '01' 的所有选课记录
DELETE FROM SC WHERE Cno = '01';

-- 删除 C 表中 Ccredit < 2 的课程
DELETE FROM C WHERE Ccredit < 2;
```

#### 删除所有数据（保留表结构）

```sql
-- 清空 SC 表中的所有数据
DELETE FROM SC;

-- 或使用 TRUNCATE（更快，不可回滚）
TRUNCATE TABLE SC;
```

#### 多表关联删除

```sql
-- 删除没有选课记录的学生（SC 表中没有对应 Sno 的学生）
DELETE FROM S
WHERE Sno NOT IN (SELECT DISTINCT Sno FROM SC);

-- 删除 '张三' 的所有选课记录
DELETE SC FROM SC, S
WHERE SC.Sno = S.Sno AND S.Sname = '张三';
```

#### 带 LIMIT 的删除

```sql
-- 删除 SC 表中成绩最低的 10 条记录
DELETE FROM SC ORDER BY Grade ASC LIMIT 10;
```

---

### 4. TRUNCATE 与 DELETE 的区别

| 特性 | DELETE | TRUNCATE |
|:---|:---|:---|
| 语法 | `DELETE FROM 表名` | `TRUNCATE TABLE 表名` |
| WHERE 条件 | 支持 | 不支持 |
| 执行速度 | 较慢（逐行删除，记录日志） | 较快（直接删除数据页） |
| 回滚 | 支持（可 ROLLBACK） | 不支持（自动提交） |
| 触发器 | 会触发 DELETE 触发器 | 不会触发 |
| 自增计数器 | 不变 | 重置为初始值 |
| 外键约束 | 受外键约束限制 | 有外键约束时不能执行 |
| 使用场景 | 删除部分数据 | 清空整张表 |

```sql
-- 示例：清空 SC 表
DELETE FROM SC;        -- 可以回滚，逐行删除
TRUNCATE TABLE SC;     -- 不可回滚，快速清空
```

---

### 5. 其他补充知识点

| 知识点                     | 说明                                                                                          |
|:------------------------|:--------------------------------------------------------------------------------------------|
| 数据库物理结构设计与具体的 DBMS 系统有关 | 不同的 DBMS（MySQL、Oracle、SQL Server）物理存储方式不同                                                   |
| 规范化过程                   | 投影分解为两个及两个以上的关系模式                                                                           |
| 概念结构设计步骤                | 抽象 → 设计局部视图 → 合并取消冲突 → 修改重构消除冗余                                                             |
| 采用静态副本恢复后               | 数据处于一致性状态                                                                                   |
| 存储过程                    | 可以调用多次，提高代码复用性                                                                              |
| 视图                      | 虚拟数据表，对视图操作与实体表操作基本一致，但视图不属于模式                                                              |
| 建立视图和建立存储过程             | 都是数据库对象，但视图是查询结果的虚拟表，存储过程是预编译的 SQL 语句集合                                                     |
| 数据备份                    | 海量存储：每次复制整个数据库。增量存储：每次只存上次存储后被更新过的数据。静态存储：是指在系统无法运行事务时进行的存储操作。动态存储：是指转储期间允许数据库进行存取或修改对的转储修改 |
| 事务ACID 原子性(Atomicity)   | 要么全做，要么不做                                                                                   |
| 事务ACID 一致性(Consistency) | 转账A,B总额不变                                                                                   |
| 事务ACID 隔离性(Isolation)   | 数据库允许多个并发事务同时对其中的数据进行读/写和修改能力                                                               |
| 事务ACID 持久性(Durability)  | I/O磁盘永久化，故障后数据不会丢失                                                                          |
| 规则完整性，实体完整性规则           | 主键唯一且不为空                                                                                    |
| 规则完整性，参照完整性约束           | 参照完整性给出正确联系的约束条件                                                                            |
| 规则完整性，用户定义规则完整性         | 根据环境要求，加入特殊限制                                                                               |# MySQL 知识点整理

## 简答题

### 两段封锁协议

1. 对任何一个数据进行读/写操作之前，事务必须获得对数据的封锁
2. 在释放一个封锁之后，事务不再获得任何其他封锁

### 最小函数依赖集步骤

#### 函数依赖规则（Armstrong 公理系统）

| 规则 | 内容 | 表达式 |
|:---|:---|:---|
| A1(自反性) | 如果 $Y \subseteq X \subseteq U$ | 则 $X \rightarrow Y$ |
| A2(增广性) | 如果 $X \rightarrow Y$ 且 $Z \subseteq U$ | 则 $XZ \rightarrow YZ$ |
| A3(传递性) | 如果 $X \rightarrow Y$ 且 $Y \rightarrow Z$ | 则 $X \rightarrow Z$ |
| B1(合并性) | 如果 $X \rightarrow Y$ 且 $X \rightarrow Z$ | 则 $X \rightarrow YZ$ |
| B2(分解性) | 如果 $X \rightarrow YZ$ | 则 $X \rightarrow Y$、$X \rightarrow Z$ |
| B3(结合性/伪传递) | 如果 $X \rightarrow Y$ 且 $W \rightarrow Z$ | 则 $XW \rightarrow YZ$ |
| B4(伪传递性) | 如果 $X \rightarrow Y$ 且 $WY \rightarrow Z$ | 则 $XW \rightarrow Z$ |

【例题】设有关系模式 R，属性集 $U = \{A,B,C,X,Y\}$，函数依赖集 $F=\{Z\rightarrow A, B\rightarrow X, AX\rightarrow Y, ZB\rightarrow Y\}$，试证明 $ZB\rightarrow Y$ 可由 F 导出。

**解答：**
1. 因为 $Z\rightarrow A$，$B\rightarrow X$，由 B3 可知，$ZB\rightarrow AX$
2. 因为 $ZB\rightarrow AX$，$AX\rightarrow Y$，由 A3 可知，$ZB\rightarrow Y$

即 $ZB\rightarrow Y$ 可由 F 中其他函数依赖导出，所以 $ZB\rightarrow Y$ 是冗余的函数依赖。

---

如果函数依赖集 F 满足下列条件，则称 F 为一个**最小函数依赖集**：

1. 每个函数依赖的右边都是单属性（可通过 B2 实现）
2. 函数依赖集 F 中没有冗余的函数依赖
3. F 中每个函数依赖的左边没有多余的属性

显然，每个函数依赖集至少存在一个最小依赖集，但并不一定唯一。

【例题】设 F 是关系模式 R(A,B,C) 的函数依赖集，$F=\{A\rightarrow BC, B\rightarrow C, A\rightarrow B, AB\rightarrow C\}$，试求最小函数依赖集。

**解答：**
1. 先把 F 中的函数依赖写成右边是单属性的形式：
   $$F=\{A\rightarrow B, A\rightarrow C, B\rightarrow C, A\rightarrow B, AB\rightarrow C\}$$
   删除一个 $A\rightarrow B$，得：
   $$F = \{A\rightarrow B, A\rightarrow C, B\rightarrow C, AB\rightarrow C\}$$

2. 删去冗余函数依赖。F 中的 $A\rightarrow C$ 可从 $A\rightarrow B$ 和 $B\rightarrow C$ 推出，因此 $A\rightarrow C$ 是冗余的，删去，得：
   $$F=\{A\rightarrow B, B\rightarrow C, AB\rightarrow C\}$$

3. 消除函数依赖左边冗余的属性。F 中的 $AB\rightarrow C$，因为有 $B\rightarrow C$，所以 A 多余，删去，得到最小函数依赖集为：
   $$F=\{A\rightarrow B, B\rightarrow C\}$$

---

### 数据库三级模式以及两级映射

#### 数据库系统的三级模式
- **外模式**（用户模式/子模式）
- **概念模式**（模式/逻辑模式）
- **内模式**（存储模式/物理模式）

#### 数据库二级映射
- **外模式/模式映射** → 保证逻辑独立性
- **模式/内模式映射** → 保证物理独立性

数据库系统的 3 个抽象级之间通过二级映射进行相互转换，使得数据库抽象的三级模式形成一个统一的整体。

#### 数据独立性

数据独立性是指程序与数据之间的独立性，主要包括：

1. **物理独立性**：应用程序与存储在磁盘上的数据库中的数据是独立的。通过**模式/内模式映射**实现。
2. **逻辑独立性**：用户的应用程序与逻辑结构是相互独立的。通过**外模式/模式映射**实现。

---

### 并发执行会出现的问题

| 问题 | 说明 |
|:---|:---|
| **丢失更新** | 两个事务同时修改同一数据，后提交的事务覆盖了先提交的事务的修改 |
| **不可重复读** | 同一事务内多次读取同一数据，结果不一致（被其他事务修改） |
| **读脏数据** | 读取了其他事务未提交的数据，随后该事务回滚 |

---

### 为什么先写日志后写数据库（WAL 协议）

1. **保证故障可恢复** — 系统崩溃时，日志已记录操作，可通过 UNDO/REDO 恢复一致性
2. **防止数据不一致** — 若先写数据库而日志丢失，故障后无法撤销未提交事务的修改
3. **WAL 机制要求** — 日志是恢复的唯一依据，必须先落盘才能确保数据库状态可回溯

---

## SQL 代码和关系表达式

### 表结构

#### S（学生表）

| 列名 | 数据类型 | 允许为空 | 特殊限制 |
|:---|:---|:---|:---|
| Sno | char(8) | 不允许 | 主键 |
| Sname | char(8) | 不允许 | |
| Sex | char(2) | 允许 | 取"男"或"女" |
| Dept | varchar(50) | 允许 | |
| Age | int | 不允许 | |

#### C（课程表）

| 列名 | 数据类型 | 允许为空 | 特殊限制 |
|:---|:---|:---|:---|
| Cno | char(3) | 不允许 | 主键 |
| Cname | varchar(20) | 允许 | |
| Ccredit | int | 不允许 | |

#### SC（选课表）

| 列名 | 数据类型 | 允许为空 | 特殊限制 |
|:---|:---|:---|:---|
| Sno | char(8) | 不允许 | 组合主键，外键 → S(Sno) |
| Cno | char(3) | 不允许 | 组合主键，外键 → C(Cno) |
| Grade | int | 允许 | 取值范围 0~100 |

---

### 基础查询

**1. 查询课号为 '01' 的课程信息**

```sql
SELECT * FROM C WHERE Cno = '01';
```

**2. 查询学分为 3 的课程号和课程名**

关系表达式：
$$\pi_{\text{Cno}, \text{Cname}}(\sigma_{\text{Ccredit}=3}(C))$$

```sql
SELECT Cno, Cname FROM C WHERE Ccredit = 3;
```

---

### SQL 语句练习

**1. 查询选修 '02' 课程的学生的学号和姓名**

```sql
SELECT S.Sno AS 学号, S.Sname AS 姓名 
FROM S, SC 
WHERE S.Sno = SC.Sno AND SC.Cno = '02';
```

**2. 查询平均分 > 70 分的课程课号、选课人数和平均分**

```sql
SELECT Cno AS 课号, COUNT(*) AS 选课人数, AVG(Grade) AS 平均分 
FROM SC
WHERE Grade > 70
GROUP BY Cno;
```

**3. 将选修课程号为 '02' 的所有学生的分数加 5 分**

```sql
UPDATE SC
SET Grade = Grade + 5
WHERE Cno = '02';
```

**4. 创建存储过程：输入课号、课程名、学分，增加一条课程数据**

```sql
DELIMITER //
CREATE PROCEDURE InsertCourse(
    IN p_Cno CHAR(3),
    IN p_Cname VARCHAR(20),
    IN p_Ccredit INT 
)
BEGIN
    INSERT INTO C VALUES(p_Cno, p_Cname, p_Ccredit);
END //
DELIMITER ;

-- 调用存储过程
CALL InsertCourse('04', '数据管理', 4);
```

**5. 创建视图：每科最高分的学生**

```sql
CREATE VIEW v_max_grade AS
SELECT SC.Cno, S.Sno AS 学号, S.Sname AS 姓名, SC.Grade AS 成绩
FROM S, SC
WHERE S.Sno = SC.Sno 
AND SC.Grade = (
    SELECT MAX(Grade) 
    FROM SC AS SC2 
    WHERE SC2.Cno = SC.Cno
);
```

**6. 在 S 表中增加一列 School（学校）**

```sql
ALTER TABLE S
ADD School VARCHAR(50) DEFAULT '燕京理工学院';
```

**7. 把 S 表的查询权限授权给 root 用户**

```sql
GRANT SELECT ON S TO 'root'@'localhost';
```

---

## 关系代数表达式

### 常用符号对照表

| 运算 | 符号 | LaTeX |
|:---|:---|:---|
| 选择 | $\sigma$ | `\sigma` |
| 投影 | $\pi$ | `\pi` |
| 并 | $\cup$ | `\cup` |
| 交 | $\cap$ | `\cap` |
| 差 | $-$ | `-` |
| 笛卡尔积 | $\times$ | `\times` |
| 自然连接 | $\bowtie$ | `\bowtie` |
| θ 连接 | $\bowtie_{\theta}$ | `\bowtie_\theta` |
| 除 | $\div$ | `\div` |
| 重命名 | $\rho$ | `\rho` |

---

### 表达式示例

**1. 查询选修了张三选修的所有课程的学生学号**

$$\pi_{\text{Sno}}(\sigma_{\text{Sname}='\text{张三}'}(S) \bowtie SC)$$

**2. 查询 C 语言课程成绩不及格的学生的学号和姓名**

$$\pi_{\text{Sno}, \text{Sname}}(\sigma_{\text{Grade} < 60}(S \bowtie SC \bowtie \sigma_{\text{Cname}='\text{C语言}'}(C)))$$

---

## 第一范式到第三范式

```
1NF —— 一张大表（属性不可再分）
  |
2NF —— 消除部分函数依赖（非主属性完全依赖于候选键）
  |
3NF —— 消除传递依赖（非主属性不依赖于其他非主属性）
```

---

### 【例题】综合练习

设有关系模式 R(运动员编号, 比赛项目, 成绩, 比赛类别, 比赛主管)，用于存储运动员比赛成绩及比赛类别、主管等信息。

**语义规定：**
- 每个运动员每参加一个比赛项目只有一个成绩
- 每个比赛项目只属于一个比赛类别
- 每个比赛类别只有一个比赛主管

**解答：**

**1. 基本函数依赖集和候选键**

```
F = {
    (运动员编号, 比赛项目) → 成绩,
    比赛项目 → 比赛类别,
    比赛类别 → 比赛主管
}

候选键：(运动员编号, 比赛项目)
```

**2. 分解为 2NF**

R 中存在部分函数依赖：
- 比赛项目 → (比赛类别, 比赛主管)

分解为：
- R1(运动员编号, 比赛项目, 成绩) — 主键：(运动员编号, 比赛项目)
- R2(比赛项目, 比赛类别, 比赛主管) — 主键：比赛项目

**3. 分解为 3NF**

R2 中存在传递依赖：比赛项目 → 比赛类别 → 比赛主管

继续分解：
- R21(比赛项目, 比赛类别) — 主键：比赛项目
- R22(比赛类别, 比赛主管) — 主键：比赛类别

最终 3NF：
- R1(运动员编号, 比赛项目, 成绩)
- R21(比赛项目, 比赛类别)
- R22(比赛类别, 比赛主管)

---

### 【例题】医疗关系分解为 3NF

现有关系：医疗(病人编号, 病人姓名, 病人性别, 病人年龄, 医生编号, 医生姓名, 医生职称, 诊断日期, 诊断结果)

**函数依赖集：**
```
F = {
    病人编号 → (病人姓名, 病人性别, 病人年龄),
    医生编号 → (医生姓名, 医生职称),
    (病人编号, 医生编号, 诊断日期) → 诊断结果
}
```

**分解为 3NF：**

| 关系模式 | 函数依赖 | 主键 | 外键 |
|:---|:---|:---|:---|
| 病人(病人编号, 病人姓名, 病人性别, 病人年龄) | 病人编号 → (病人姓名, 病人性别, 病人年龄) | 病人编号 | |
| 医生(医生编号, 医生姓名, 医生职称) | 医生编号 → (医生姓名, 医生职称) | 医生编号 | |
| 医疗(病人编号, 医生编号, 诊断日期, 诊断结果) | (病人编号, 医生编号, 诊断日期) → 诊断结果 | (病人编号, 医生编号, 诊断日期) | 病人编号 → 病人，医生编号 → 医生 |

---

## E-R 图设计

### 【例题】车辆管理系统

**需求：**
1. 车队：车队号、车队名
2. 车辆：牌照号、厂家、出厂日期
3. 司机：司机编号、姓名、电话
4. 车队聘用司机（1:n），有聘期
5. 车队拥有车辆（1:n）
6. 司机使用车辆（m:n），有使用日期和公里数

**关系模式：**

| 关系模式 | 属性 | 主键 | 外键 |
|:---|:---|:---|:---|
| 车队 | (车队号, 车队名) | 车队号 | |
| 车辆 | (牌照号, 厂家, 出厂日期, 车队号) | 牌照号 | 车队号 → 车队 |
| 司机 | (司机编号, 姓名, 电话, 车队号, 聘期) | 司机编号 | 车队号 → 车队 |
| 使用 | (牌照号, 司机编号, 日期, 公里数) | (司机编号, 牌照号, 日期) | 司机编号 → 司机，牌照号 → 车辆 |

---

## 补充知识点

### 1. ALTER 语句（修改表结构）

ALTER 用于修改已存在的表结构，包括添加列、删除列、修改列类型、添加约束等。

**基于 S、SC、C 表的示例：**

#### 添加列

```sql
-- 在 S 表中添加 Phone 列
ALTER TABLE S ADD Phone CHAR(11);

-- 在 SC 表中添加 ExamDate 列，默认值为当前日期
ALTER TABLE SC ADD ExamDate DATE DEFAULT CURDATE();

-- 在 C 表中添加 Semester 列（开课学期），放在 Ccredit 列之后
ALTER TABLE C ADD Semester INT AFTER Ccredit;
```

#### 修改列

```sql
-- 修改 S 表中 Sname 的长度为 20
ALTER TABLE S MODIFY Sname VARCHAR(20);

-- 修改 S 表中 Age 列的数据类型为 SMALLINT，并添加注释
ALTER TABLE S MODIFY Age SMALLINT COMMENT '学生年龄';

-- 修改 SC 表中 Grade 列允许为空（原为不允许）
ALTER TABLE SC MODIFY Grade INT NULL;

-- 修改列名（将 S 表中的 Sex 改为 Gender）
ALTER TABLE S CHANGE Sex Gender CHAR(2);
```

#### 删除列

```sql
-- 删除 S 表中的 Phone 列
ALTER TABLE S DROP COLUMN Phone;

-- 删除 SC 表中的 ExamDate 列
ALTER TABLE SC DROP COLUMN ExamDate;
```

#### 添加约束

```sql
-- 为 S 表的 Age 添加 CHECK 约束（年龄必须在 15-50 之间）
ALTER TABLE S ADD CONSTRAINT chk_age CHECK (Age BETWEEN 15 AND 50);

-- 为 SC 表的 Grade 添加 CHECK 约束（成绩在 0-100 之间）
ALTER TABLE SC ADD CONSTRAINT chk_grade CHECK (Grade BETWEEN 0 AND 100);

-- 为 S 表的 Dept 添加默认值
ALTER TABLE S ALTER COLUMN Dept SET DEFAULT '计算机学院';
```

#### 删除约束

```sql
-- 删除 S 表中的 chk_age 约束
ALTER TABLE S DROP CONSTRAINT chk_age;

-- 删除 SC 表中的 chk_grade 约束
ALTER TABLE SC DROP CONSTRAINT chk_grade;
```

#### 修改表名

```sql
-- 将 S 表改名为 Student
ALTER TABLE S RENAME TO Student;

-- 将 C 表改名为 Course
ALTER TABLE C RENAME TO Course;
```

---

### 2. DROP 语句（删除数据库对象）

DROP 用于删除数据库、表、视图、索引等对象。**删除后不可恢复**，需谨慎使用。

**基于 S、SC、C 表的示例：**

#### 删除表

```sql
-- 删除 SC 表（选课表）
DROP TABLE SC;

-- 删除 S 表（学生表），如果存在则删除
DROP TABLE IF EXISTS S;

-- 删除 C 表（课程表），并级联删除外键约束
DROP TABLE C CASCADE;
```

#### 删除数据库

```sql
-- 删除名为 school 的数据库
DROP DATABASE school;

-- 如果存在则删除
DROP DATABASE IF EXISTS school;
```

#### 删除视图

```sql
-- 删除之前创建的 v_max_grade 视图
DROP VIEW v_max_grade;

-- 如果存在则删除
DROP VIEW IF EXISTS v_max_grade;
```

#### 删除索引

```sql
-- 删除 S 表上的 idx_sname 索引
DROP INDEX idx_sname ON S;
```

#### 删除存储过程

```sql
-- 删除之前创建的 InsertCourse 存储过程
DROP PROCEDURE InsertCourse;

-- 如果存在则删除
DROP PROCEDURE IF EXISTS InsertCourse;
```

---

### 3. DELETE 语句（删除表中的数据）

DELETE 用于删除表中的行记录，**保留表结构**，可以配合 WHERE 条件删除指定数据。

**基于 S、SC、C 表的示例：**

#### 删除指定条件的数据

```sql
-- 删除 S 表中 Dept 为 '数学系' 的所有学生
DELETE FROM S WHERE Dept = '数学系';

-- 删除 SC 表中 Cno 为 '01' 的所有选课记录
DELETE FROM SC WHERE Cno = '01';

-- 删除 C 表中 Ccredit < 2 的课程
DELETE FROM C WHERE Ccredit < 2;
```

#### 删除所有数据（保留表结构）

```sql
-- 清空 SC 表中的所有数据
DELETE FROM SC;

-- 或使用 TRUNCATE（更快，不可回滚）
TRUNCATE TABLE SC;
```

#### 多表关联删除

```sql
-- 删除没有选课记录的学生（SC 表中没有对应 Sno 的学生）
DELETE FROM S
WHERE Sno NOT IN (SELECT DISTINCT Sno FROM SC);

-- 删除 '张三' 的所有选课记录
DELETE SC FROM SC, S
WHERE SC.Sno = S.Sno AND S.Sname = '张三';
```

#### 带 LIMIT 的删除

```sql
-- 删除 SC 表中成绩最低的 10 条记录
DELETE FROM SC ORDER BY Grade ASC LIMIT 10;
```

---

### 4. TRUNCATE 与 DELETE 的区别

| 特性 | DELETE | TRUNCATE |
|:---|:---|:---|
| 语法 | `DELETE FROM 表名` | `TRUNCATE TABLE 表名` |
| WHERE 条件 | 支持 | 不支持 |
| 执行速度 | 较慢（逐行删除，记录日志） | 较快（直接删除数据页） |
| 回滚 | 支持（可 ROLLBACK） | 不支持（自动提交） |
| 触发器 | 会触发 DELETE 触发器 | 不会触发 |
| 自增计数器 | 不变 | 重置为初始值 |
| 外键约束 | 受外键约束限制 | 有外键约束时不能执行 |
| 使用场景 | 删除部分数据 | 清空整张表 |

```sql
-- 示例：清空 SC 表
DELETE FROM SC;        -- 可以回滚，逐行删除
TRUNCATE TABLE SC;     -- 不可回滚，快速清空
```

---

### 5. 其他补充知识点

| 知识点                     | 说明                                                                                          |
|:------------------------|:--------------------------------------------------------------------------------------------|
| 数据库物理结构设计与具体的 DBMS 系统有关 | 不同的 DBMS（MySQL、Oracle、SQL Server）物理存储方式不同                                                   |
| 规范化过程                   | 投影分解为两个及两个以上的关系模式                                                                           |
| 概念结构设计步骤                | 抽象 → 设计局部视图 → 合并取消冲突 → 修改重构消除冗余                                                             |
| 采用静态副本恢复后               | 数据处于一致性状态                                                                                   |
| 存储过程                    | 可以调用多次，提高代码复用性                                                                              |
| 视图                      | 虚拟数据表，对视图操作与实体表操作基本一致，但视图不属于模式                                                              |
| 建立视图和建立存储过程             | 都是数据库对象，但视图是查询结果的虚拟表，存储过程是预编译的 SQL 语句集合                                                     |
| 数据备份                    | 海量存储：每次复制整个数据库。增量存储：每次只存上次存储后被更新过的数据。静态存储：是指在系统无法运行事务时进行的存储操作。动态存储：是指转储期间允许数据库进行存取或修改对的转储修改 |
| 事务ACID 原子性(Atomicity)   | 要么全做，要么不做                                                                                   |
| 事务ACID 一致性(Consistency) | 转账A,B总额不变                                                                                   |
| 事务ACID 隔离性(Isolation)   | 数据库允许多个并发事务同时对其中的数据进行读/写和修改能力                                                               |
| 事务ACID 持久性(Durability)  | I/O磁盘永久化，故障后数据不会丢失                                                                          |
| 规则完整性，实体完整性规则           | 主键唯一且不为空                                                                                    |
| 规则完整性，参照完整性约束           | 参照完整性给出正确联系的约束条件                                                                            |
| 规则完整性，用户定义规则完整性         | 根据环境要求，加入特殊限制                                                                               |