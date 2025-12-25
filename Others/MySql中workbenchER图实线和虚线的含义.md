# 1:1 实线（识别关系）vs 虚线（非识别关系）生成的 SQL 差异

核心差异集中在 **子表的主键定义** 和 **外键约束的属性** 上，父表 SQL 基本一致，子表 SQL 因关系类型不同有明显区别。

## 前提说明

假设我们有两个关联表，以「用户系统」为例：

- 父表：`user`（用户基本信息，主键 `user_id`）
- 子表：`user_profile`（用户详细档案，1:1 关联 `user`）

## 一、虚线（非识别关系）生成的 SQL

非识别关系中，子表有 **独立的主键**，外键是「普通列」（可空，默认允许独立存在），外键仅用于关联，不参与主键构成。

### 完整 SQL

```sql
-- 父表：用户基本信息（无差异）
CREATE TABLE `user` (
  `user_id` INT NOT NULL AUTO_INCREMENT,
  `username` VARCHAR(50) NOT NULL,
  PRIMARY KEY (`user_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- 子表：用户详细档案（非识别关系核心特征）
CREATE TABLE `user_profile` (
  `profile_id` INT NOT NULL AUTO_INCREMENT,  -- 独立主键，与外键无关
  `user_id` INT NULL,  -- 外键是普通列，默认允许为 NULL（可独立创建记录）
  `real_name` VARCHAR(50) DEFAULT NULL,
  `phone` VARCHAR(20) DEFAULT NULL,
  PRIMARY KEY (`profile_id`),  -- 主键仅包含自身字段
  -- 外键约束：关联父表 user_id，允许 NULL（非强制关联）
  CONSTRAINT `fk_user_profile_user`
    FOREIGN KEY (`user_id`)
    REFERENCES `user` (`user_id`)
    ON DELETE SET NULL  -- 父表记录删除时，子表外键设为 NULL（默认行为，可修改）
    ON UPDATE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

### 关键特征（SQL 层面）

1. 子表有 **独立主键**（如 `profile_id`），主键不包含外键；
2. 外键 `user_id` 是「普通列」，默认定义为 `INT NULL`（允许子表记录不关联父表，独立存在）；
3. 外键约束仅声明关联关系，不强制子表依赖父表（`ON DELETE SET NULL` 是默认行为，可手动改为 `RESTRICT` 等）。

## 二、实线（识别关系）生成的 SQL

识别关系中，子表的 **主键包含外键列**（外键是主键的一部分），外键强制「非空」（子表记录必须依赖父表存在，无法独立创建）。

### 完整 SQL



```sql
CREATE TABLE IF NOT EXISTS `mydb`.`PetCatalog` (
  `idPetCatalog` INT NOT NULL,
  `cat` INT NULL,
  `dog` INT NULL,
  `bird` INT NULL,
  `Pet_idPet` INT NULL,
  PRIMARY KEY (`idPetCatalog`),
  INDEX `fk_PetCatalog_Pet1_idx` (`Pet_idPet` ASC) VISIBLE,
  CONSTRAINT `fk_PetCatalog_Pet1`
    FOREIGN KEY (`Pet_idPet`)
    REFERENCES `mydb`.`Pet` (`idPet`)
    ON DELETE SET NULL
    ON UPDATE SET NULL)
ENGINE = InnoDB


CREATE TABLE IF NOT EXISTS `mydb`.`PetCatalog` (
  `idPetCatalog` INT NOT NULL,
  `cat` INT NULL,
  `dog` INT NULL,
  `bird` INT NULL,
  `Pet_idPet` INT NULL,
  PRIMARY KEY (`idPetCatalog`),
  INDEX `fk_PetCatalog_Pet1_idx` (`Pet_idPet` ASC) VISIBLE,
  CONSTRAINT `fk_PetCatalog_Pet1`
    FOREIGN KEY (`Pet_idPet`)
    REFERENCES `mydb`.`Pet` (`idPet`)
    ON DELETE SET NULL
    ON UPDATE SET NULL)
ENGINE = InnoDB


CREATE TABLE IF NOT EXISTS `mydb`.`PetCatalog` (
  `idPetCatalog` INT NOT NULL,
  `cat` INT NULL,
  `dog` INT NULL,
  `bird` INT NULL,
  `Pet_idPet` INT NULL,
  PRIMARY KEY (`idPetCatalog`),
  INDEX `fk_PetCatalog_Pet1_idx` (`Pet_idPet` ASC) VISIBLE,
  CONSTRAINT `fk_PetCatalog_Pet1`
    FOREIGN KEY (`Pet_idPet`)
    REFERENCES `mydb`.`Pet` (`idPet`)
    ON DELETE SET NULL
    ON UPDATE SET NULL)
ENGINE = InnoDB


INSERT INTO `mydb`.`Pet` (`idPet`, `name`, `catalog`, `Person_idPerson`) VALUES (NULL, NULL, NULL, NULL);
```

sql

```sql
-- 父表：用户基本信息（无差异）
CREATE TABLE `user` (
  `user_id` INT NOT NULL AUTO_INCREMENT,
  `username` VARCHAR(50) NOT NULL,
  PRIMARY KEY (`user_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- 子表：用户详细档案（识别关系核心特征）
CREATE TABLE `user_profile` (
  `user_id` INT NOT NULL,  -- 外键列，同时是主键的一部分
  `real_name` VARCHAR(50) DEFAULT NULL,
  `phone` VARCHAR(20) DEFAULT NULL,
  PRIMARY KEY (`user_id`),  -- 主键直接复用父表外键（1:1 核心，避免重复）
  -- 外键约束：关联父表 user_id，强制 NOT NULL（依赖父表）
  CONSTRAINT `fk_user_profile_user`
    FOREIGN KEY (`user_id`)
    REFERENCES `user` (`user_id`)
    ON DELETE CASCADE  -- 父表记录删除时，子表同步删除（强依赖默认行为）
    ON UPDATE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

### 关键特征（SQL 层面）

1. 子表 **无独立主键**，主键直接是父表的外键 `user_id`（1:1 关系下，外键天然唯一，无需额外加唯一索引）；
2. 外键 `user_id` 是主键的一部分，强制声明为 `INT NOT NULL`（子表记录必须关联父表的有效 `user_id`，无法独立创建）；
3. 外键约束默认是「强关联」（如 `ON DELETE CASCADE`），父表记录删除 / 更新时，子表同步操作（符合强依赖逻辑）。

## 三、核心 SQL 差异对比表

| 对比维度            | 虚线（非识别关系）                           | 实线（识别关系）                                     |
| ------------------- | -------------------------------------------- | ---------------------------------------------------- |
| 子表主键构成        | 独立主键（如 `profile_id`），与外键无关      | 主键 = 外键（如 `user_id`），外键是主键一部分        |
| 外键列属性          | 默认 `NULL`（允许子表独立存在）              | 强制 `NOT NULL`（子表必须依赖父表）                  |
| 外键与主键的关系    | 外键是普通列，不参与主键                     | 外键是主键核心组成部分                               |
| 删除 / 更新默认行为 | `ON DELETE SET NULL`（父表删，子表外键置空） | `ON DELETE CASCADE`（父表删，子表同步删）            |
| 子表独立创建限制    | 可直接插入无关联的子表记录（`user_id=NULL`） | 无法插入无关联的子表记录（`user_id` 必须存在于父表） |

## 四、补充说明

1. **1:1 关系的唯一性保证**：虚线关系中，外键列默认仅创建普通索引，若需严格 1:1（避免一个父表对应多个子表），需手动给外键加 `UNIQUE` 约束（Workbench 会自动生成）：

   ```sql
   -- 虚线关系下，Workbench 自动添加的唯一索引（保证 1:1）
   UNIQUE INDEX `uk_user_profile_user_id` (`user_id` ASC) VISIBLE;
   ```

   实线关系中，外键本身是主键，主键天然唯一，无需额外加唯一索引。

2. **手动修改行为**：生成的 SQL 中，`ON DELETE`/`ON UPDATE` 规则可在 Workbench 的「Relationship Editor」中修改（如将实线的 `CASCADE` 改为 `RESTRICT`），但核心的「主键构成」和「非空约束」差异由关系类型（识别 / 非识别）决定，无法通过规则修改。

## 总结

两者生成的 SQL 核心差异是 **子表的主键设计和外键约束强度**：

- 虚线（非识别）：子表有独立主键，外键可空，关系松散 → 对应「可选关联」场景；
- 实线（识别）：子表主键 = 外键，外键非空，关系紧密 → 对应「必须关联」场景。

SQL 的差异本质是对「业务依赖关系」的语法落地，和 ER 图中「强弱依赖」的设计逻辑完全一致。