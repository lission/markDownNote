[toc]

# Maven 常用命令如下：

## 基础构建命令

* `mvn clean` - 清理项目，删除 target 目录
* `mvn compile` - 编译项目源代码
* `mvn test` - 运行测试用例
* `mvn package` - 打包项目，生成 jar/war 文件
* `mvn install` - 安装项目到本地仓库
* `mvn deploy` - 部署项目到远程仓库

## 依赖相关命令

* `mvn dependency:tree` - 查看依赖树
* `mvn dependency:resolve` - 解析依赖
* `mvn dependency:copy-dependencies` - 复制依赖到指定目录
* `mvn dependency:analyze` - 分析依赖使用情况

## 强制更新命令

* `mvn dependency:resolve -U` - 强制更新依赖
* `mvn clean install -U` - 清理并强制更新安装

## 其他常用命令

* `mvn help:effective-pom` - 查看有效的 POM 信息
* `mvn archetype:generate` - 生成项目骨架
* `mvn site` - 生成项目站点
* `mvn jetty:run` - 运行项目（需要 jetty 插件）
* `mvn exec:java` - 执行主类

## 命令参数说明

* `-DskipTests` - 跳过测试执行
* `-Dmaven.test.failure.ignore=true` - 忽略测试失败
* `-PprofileName` - 指定激活的 profile
* `-X` - 显示详细日志信息



# Docker-oracle

```
基本选项卡：
- 连接名：Oracle-Docker
- 主机：127.0.0.1 或 localhost
- 端口：1521
- 服务名：XE
- 用户名：system
- 密码：Oracle123

高级选项卡（可选）：
- 角色：Default（或 SYSDBA）
- 编码：建议 UTF-8
```

- 公共用户（Common User）：以 C## 或 c## 开头，在所有容器中都存在
- 本地用户（Local User）：只在特定 PDB 中存在

## **总结**

| 场景                    | 服务名   | 用户前缀   | 示例        |
| ----------------------- | -------- | ---------- | ----------- |
| **容器数据库（CDB）**   | `XE`     | `C##` 开头 | `C##LSDATA` |
| **可插拔数据库（PDB）** | `XEPDB1` | 无前缀     | `LSDATA`    |

**推荐做法：**

1. 使用服务名 `XEPDB1` 连接到 PDB
2. 创建普通用户 `LSDATA`
3. 用 LSDATA 用户连接，创建表和序列

-- 1. 先创建表空间（在 XEPDB1 中）
CREATE TABLESPACE LSD_DATA
DATAFILE '/opt/oracle/oradata/XE/XEPDB1/lsd_data01.dbf'
SIZE 100M
AUTOEXTEND ON NEXT 50M
MAXSIZE UNLIMITED
LOGGING
EXTENT MANAGEMENT LOCAL
SEGMENT SPACE MANAGEMENT AUTO;

-- 2. 创建临时表空间（可选）
CREATE TEMPORARY TABLESPACE LSD_TEMP
TEMPFILE '/opt/oracle/oradata/XE/XEPDB1/lsd_temp01.dbf'
SIZE 50M
AUTOEXTEND ON NEXT 20M
MAXSIZE UNLIMITED;

-- 3. 现在创建用户
CREATE USER LSDATA IDENTIFIED BY LSDATA123
DEFAULT TABLESPACE LSD_DATA
TEMPORARY TABLESPACE LSD_TEMP
QUOTA UNLIMITED ON LSD_DATA
ACCOUNT UNLOCK;

-- 4. 授权
GRANT CONNECT, RESOURCE TO LSDATA;
GRANT CREATE SESSION TO LSDATA;

-- 5. 验证
SELECT
    username,
    default_tablespace,
    temporary_tablespace,
    account_status,
    created
FROM dba_users
WHERE username = 'LSDATA';

## 建表相关

1、创建序列（先创建序列）
-- ============================================
-- 创建序列（必须先于表创建）
-- ============================================

CREATE SEQUENCE ORACLE_HISTORY_TABLE_SEQ
    START WITH 1
    INCREMENT BY 1
    NOMINVALUE
    NOMAXVALUE
    NOCYCLE
    NOCACHE
    NOORDER;

-- 验证序列创建
SELECT
    sequence_name,
    last_number,
    increment_by,
    min_value,
    max_value
FROM user_sequences
WHERE sequence_name = 'ORACLE_HISTORY_TABLE_SEQ';

第三步：创建表（根据你的实体类）
sql
-- ============================================
-- 创建 ORACLE_HISTORY_TABLE 表
-- ============================================

CREATE TABLE ORACLE_HISTORY_TABLE (
    -- 主键ID - 对应 @TableId(type = IdType.INPUT)
    ID NUMBER(19) NOT NULL,

    -- 数据内容 - 对应 private String dataContent;
    DATA_CONTENT VARCHAR2(4000),

    -- CLOB大文本 - 对应 private String largeContent;
    LARGE_CONTENT CLOB,

    -- 创建日期 - 对应 private LocalDateTime createDate;
    -- 注意：这里避免与自动填充字段冲突，使用 BIZ_CREATE_DATE
    BIZ_CREATE_DATE TIMESTAMP,

    -- 逻辑删除 - 对应 @TableLogic private Integer deleted;
    IS_DELETED NUMBER(1) DEFAULT 0,

    -- 自动填充创建时间 - 对应 @TableField(fill = FieldFill.INSERT)
    CREATE_TIME TIMESTAMP DEFAULT SYSTIMESTAMP,

    -- 自动填充更新时间 - 对应 @TableField(fill = FieldFill.INSERT_UPDATE)
    UPDATE_TIME TIMESTAMP DEFAULT SYSTIMESTAMP,

    -- 主键约束
    CONSTRAINT PK_ORACLE_HISTORY PRIMARY KEY (ID),

    -- 逻辑删除检查约束
    CONSTRAINT CHK_IS_DELETED CHECK (IS_DELETED IN (0, 1))
);

-- ============================================
-- 添加注释
-- ============================================

COMMENT ON TABLE ORACLE_HISTORY_TABLE IS 'Oracle历史记录表';
COMMENT ON COLUMN ORACLE_HISTORY_TABLE.ID IS '主键ID';
COMMENT ON COLUMN ORACLE_HISTORY_TABLE.DATA_CONTENT IS '数据内容';
COMMENT ON COLUMN ORACLE_HISTORY_TABLE.LARGE_CONTENT IS '大文本内容（CLOB类型）';
COMMENT ON COLUMN ORACLE_HISTORY_TABLE.BIZ_CREATE_DATE IS '业务创建日期';
COMMENT ON COLUMN ORACLE_HISTORY_TABLE.IS_DELETED IS '逻辑删除标识：0-未删除，1-已删除';
COMMENT ON COLUMN ORACLE_HISTORY_TABLE.CREATE_TIME IS '系统创建时间（自动填充）';
COMMENT ON COLUMN ORACLE_HISTORY_TABLE.UPDATE_TIME IS '系统更新时间（自动填充）';
第四步：创建索引
sql
-- ============================================
-- 创建索引
-- ============================================

-- 逻辑删除字段索引（MyBatis Plus 逻辑删除查询需要）
CREATE INDEX IDX_HISTORY_DELETED ON ORACLE_HISTORY_TABLE(IS_DELETED);

-- 创建时间索引（常用于排序和范围查询）
CREATE INDEX IDX_HISTORY_CREATE_TIME ON ORACLE_HISTORY_TABLE(CREATE_TIME);

-- 更新时间索引
CREATE INDEX IDX_HISTORY_UPDATE_TIME ON ORACLE_HISTORY_TABLE(UPDATE_TIME);

-- 业务创建日期索引（如果你需要按业务日期查询）
CREATE INDEX IDX_HISTORY_BIZ_DATE ON ORACLE_HISTORY_TABLE(BIZ_CREATE_DATE);
第五步：创建触发器（可选）
sql
-- ============================================
-- 创建触发器：自动更新UPDATE_TIME
-- ============================================

CREATE OR REPLACE TRIGGER TRG_ORACLE_HISTORY_UPDATE
    BEFORE UPDATE ON ORACLE_HISTORY_TABLE
    FOR EACH ROW
BEGIN
    :NEW.UPDATE_TIME := SYSTIMESTAMP;
END;
/

-- ============================================
-- 创建触发器：自动生成ID（如果不使用MyBatis Plus生成）
-- ============================================

CREATE OR REPLACE TRIGGER TRG_ORACLE_HISTORY_ID
    BEFORE INSERT ON ORACLE_HISTORY_TABLE
    FOR EACH ROW
DECLARE
    v_next_id NUMBER;
BEGIN
    IF :NEW.ID IS NULL OR :NEW.ID = 0 THEN
        SELECT ORACLE_HISTORY_TABLE_SEQ.NEXTVAL INTO v_next_id FROM DUAL;
        :NEW.ID := v_next_id;
    END IF;

    -- 确保CREATE_TIME有默认值
    IF :NEW.CREATE_TIME IS NULL THEN
        :NEW.CREATE_TIME := SYSTIMESTAMP;
    END IF;

    -- 确保UPDATE_TIME有默认值
    IF :NEW.UPDATE_TIME IS NULL THEN
        :NEW.UPDATE_TIME := SYSTIMESTAMP;
    END IF;
END;
/
第六步：验证创建结果
sql
-- ============================================
-- 验证表结构
-- ============================================

-- 1. 查看表信息
SELECT
    table_name,
    tablespace_name,
    status,
    num_rows
FROM user_tables
WHERE table_name = 'ORACLE_HISTORY_TABLE';

-- 2. 查看字段信息
SELECT
    column_id AS "序号",
    column_name AS "字段名",
    data_type AS "数据类型",
    data_length AS "长度",
    data_precision AS "精度",
    data_scale AS "小数位",
    nullable AS "可空",
    data_default AS "默认值"
FROM user_tab_columns
WHERE table_name = 'ORACLE_HISTORY_TABLE'
ORDER BY column_id;

-- 3. 查看约束
SELECT
    constraint_name AS "约束名",
    constraint_type AS "类型",
    search_condition AS "检查条件",
    status AS "状态"
FROM user_constraints
WHERE table_name = 'ORACLE_HISTORY_TABLE';

-- 4. 查看索引
SELECT
    index_name AS "索引名",
    index_type AS "索引类型",
    uniqueness AS "唯一性",
    status AS "状态"
FROM user_indexes
WHERE table_name = 'ORACLE_HISTORY_TABLE';

-- 5. 查看触发器
SELECT
    trigger_name AS "触发器名",
    trigger_type AS "类型",
    triggering_event AS "触发事件",
    status AS "状态"
FROM user_triggers
WHERE table_name = 'ORACLE_HISTORY_TABLE';

第七步：插入测试数据
sql
-- ============================================
-- 插入测试数据
-- ============================================

-- 测试数据1：使用序列（触发器的ID为NULL时会自动生成）
INSERT INTO ORACLE_HISTORY_TABLE (
    DATA_CONTENT,
    LARGE_CONTENT,
    BIZ_CREATE_DATE,
    IS_DELETED
) VALUES (
    '第一条测试数据',
    '这是一个CLOB字段的测试内容。CLOB可以存储大量文本数据，最大4GB。',
    SYSTIMESTAMP - INTERVAL '2' DAY,
    0
);

-- 测试数据2：显式指定ID（MyBatis Plus会这样操作）
INSERT INTO ORACLE_HISTORY_TABLE (
    ID,
    DATA_CONTENT,
    LARGE_CONTENT,
    BIZ_CREATE_DATE,
    IS_DELETED
) VALUES (
    ORACLE_HISTORY_TABLE_SEQ.NEXTVAL,
    '第二条测试数据',
    EMPTY_CLOB(),
    SYSTIMESTAMP - INTERVAL '1' DAY,
    0
);

-- 测试数据3：逻辑删除的数据
INSERT INTO ORACLE_HISTORY_TABLE (
    ID,
    DATA_CONTENT,
    LARGE_CONTENT,
    BIZ_CREATE_DATE,
    IS_DELETED
) VALUES (
    ORACLE_HISTORY_TABLE_SEQ.NEXTVAL,
    '已删除的数据',
    '这条数据将被逻辑删除',
    SYSTIMESTAMP,
    1
);

COMMIT;

-- ============================================
-- 查询测试数据
-- ============================================

-- 查询所有数据（包含已删除的）
SELECT
    ID,
    DATA_CONTENT,
    DBMS_LOB.GETLENGTH(LARGE_CONTENT) AS CLOB_LENGTH,
    TO_CHAR(BIZ_CREATE_DATE, 'YYYY-MM-DD HH24:MI:SS') AS BIZ_DATE,
    IS_DELETED,
    TO_CHAR(CREATE_TIME, 'YYYY-MM-DD HH24:MI:SS') AS CREATE_TIME,
    TO_CHAR(UPDATE_TIME, 'YYYY-MM-DD HH24:MI:SS') AS UPDATE_TIME
FROM ORACLE_HISTORY_TABLE
ORDER BY ID;

-- 查询未删除的数据（MyBatis Plus逻辑删除查询）
SELECT COUNT(*) AS ACTIVE_RECORDS
FROM ORACLE_HISTORY_TABLE
WHERE IS_DELETED = 0;

-- 测试更新（验证UPDATE_TIME触发器）
UPDATE ORACLE_HISTORY_TABLE
SET DATA_CONTENT = DATA_CONTENT || ' [已更新]'
WHERE ID = 1;

COMMIT;

-- 查看更新后的数据
SELECT
    ID,
    DATA_CONTENT,
    TO_CHAR(UPDATE_TIME, 'YYYY-MM-DD HH24:MI:SS.FF3') AS UPDATE_TIME
FROM ORACLE_HISTORY_TABLE
WHERE ID = 1;

# mysql

```
localhost:3306
root
123456
```

## 搜索 mysql.server 文件

```bash
sudo find / -name "mysql.server" 2>/dev/null
```

## 编辑 zsh 配置文件

```bash
vim ~/.zshrc
```

## 添加以下内容（根据你的实际路径调整）

```bash
alias mysql.start='sudo /usr/local/mysql-8.0.44-macos15-x86_64/support-files/mysql.server start'
alias mysql.stop='sudo /usr/local/mysql-8.0.44-macos15-x86_64/support-files/mysql.server stop'
alias mysql.status='sudo /usr/local/mysql-8.0.44-macos15-x86_64/support-files/mysql.server status'
alias mysql.restart='sudo /usr/local/mysql-8.0.44-macos15-x86_64/support-files/mysql.server restart'
```

## 保存后重新加载

```bash
source ~/.zshrc
```

## 现在可以使用简化的命令

```bash
mysql.status
mysql.start
mysql.stop
mysql.restart
```

## 创建数据库

```
-- 创建数据中⼼数据库
CREATE DATABASE `data_center`
CHARACTER SET = utf8mb3
COLLATE = utf8mb3_general_ci
COMMENT = '数据中⼼数据库';
```
