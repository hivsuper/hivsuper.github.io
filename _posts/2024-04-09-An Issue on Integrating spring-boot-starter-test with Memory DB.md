---
title: An Issue on Integrating spring-boot-starter-test with Memory DB  
date: 2024-04-09 20:00:00 +0800  
categories: [Technology, Java Learning Journey]  
tags: [docker]  
---
I encountered the following exception when integrating spring-boot-start-test with memory db.
```shell
Caused by: org.h2.jdbc.JdbcSQLSyntaxErrorException: Table "ACCOUNT" not found (this database is empty); SQL statement:
INSERT INTO `account` (`id`, `account_name`, `password`, `entity_id`, `created_by`) VALUES(1, 'test_account_1 in MemoryDB', 'test1', 1, 1) [42104-224]
	at org.h2.message.DbException.getJdbcSQLException(DbException.java:514)
	at org.h2.message.DbException.getJdbcSQLException(DbException.java:489)
	at org.h2.message.DbException.get(DbException.java:223)
	at org.h2.message.DbException.get(DbException.java:199)
	at org.h2.command.Parser.getTableOrViewNotFoundDbException(Parser.java:8051)
	at org.h2.command.Parser.getTableOrViewNotFoundDbException(Parser.java:8035)
	at org.h2.command.Parser.readTableOrView(Parser.java:8024)
	at org.h2.command.Parser.readTableOrView(Parser.java:7990)
	at org.h2.command.Parser.parseInsert(Parser.java:1540)
	at org.h2.command.Parser.parsePrepared(Parser.java:719)
	at org.h2.command.Parser.parse(Parser.java:592)
	at org.h2.command.Parser.parse(Parser.java:564)
	at org.h2.command.Parser.prepareCommand(Parser.java:483)
	at org.h2.engine.SessionLocal.prepareLocal(SessionLocal.java:639)
	at org.h2.engine.SessionLocal.prepareCommand(SessionLocal.java:559)
	at org.h2.jdbc.JdbcConnection.prepareCommand(JdbcConnection.java:1166)
	at org.h2.jdbc.JdbcStatement.executeInternal(JdbcStatement.java:245)
	at org.h2.jdbc.JdbcStatement.execute(JdbcStatement.java:231)
	at com.zaxxer.hikari.pool.ProxyStatement.execute(ProxyStatement.java:94)
	at com.zaxxer.hikari.pool.HikariProxyStatement.execute(HikariProxyStatement.java)
	at org.springframework.jdbc.datasource.init.ScriptUtils.executeSqlScript(ScriptUtils.java:261)
	... 17 more

```
Though some materials are found through search engine or ChatGPT, they couldn't really fix the issue. Fortunately I found the article [Spring @Sql, @SqlGroup and @SqlMergeMode](https://howtodoinjava.com/spring/spring-sql-sqlgroup-and-sqlmergemode/). I write this note in case I'd forget it one day in the future.
## Create REST API
The files below should be created properly.
<details open><summary markdown="span">pom.xml</summary>

```xml
...
  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>3.2.4</version>
    <relativePath/> <!-- lookup parent from repository -->
  </parent>
  <dependencies>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-test</artifactId>
      <scope>test</scope>
    </dependency>
  </dependencies>
...
```
</details>

<details open><summary markdown="span">com.demo.ledger.account.controller.AccountController.java</summary>

```java
package com.demo.ledger.account.controller;

import com.demo.ledger.account.pojo.Account;
import com.demo.ledger.account.service.IAccountQueryService;
import jakarta.annotation.Resource;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.List;

@RestController
@RequestMapping("/account")
public class AccountController {
    @Resource
    private IAccountQueryService accountQueryService;

    @GetMapping
    public ResponseEntity<List<Account>> findAllAccounts() {
        return ResponseEntity.ok(accountQueryService.findAllAccounts());
    }
}
```
</details>

<details open><summary markdown="span">com.demo.ledger.account.service.IAccountQueryService.java</summary>

```java
package com.demo.ledger.account.service;

import com.demo.ledger.account.pojo.Account;

import java.util.List;

public interface IAccountQueryService {
    List<Account> findAllAccounts();
}
```
</details>

<details open><summary markdown="span">com.demo.ledger.account.service.impl.AccountQueryService.java</summary>

```java
package com.demo.ledger.account.service.impl;

import com.demo.ledger.account.mapper.AccountMapper;
import com.demo.ledger.account.pojo.Account;
import com.demo.ledger.account.service.IAccountQueryService;
import jakarta.annotation.Resource;
import org.springframework.stereotype.Service;

import java.util.List;

@Service
public class AccountQueryService implements IAccountQueryService {
    @Resource
    private AccountMapper accountMapper;

    @Override
    public List<Account> findAllAccounts() {
        return accountMapper.findAll();
    }
}

```
</details>

<details open><summary markdown="span">com.demo.ledger.account.pojo.Account.java</summary>

```java
package com.demo.ledger.account.pojo;

import java.util.Date;

public class Account {
    private int id;
    private String accountName;
    private String password;
    private int entityId;
    private int createdBy;
    private Date createdTime;
    private Date lastModifiedTime;

    ...
}

```
</details>

<details open><summary markdown="span">com.demo.ledger.account.mapper.AccountMapper.java</summary>

```java
package com.demo.ledger.account.mapper;


import com.demo.ledger.account.pojo.Account;
import org.apache.ibatis.annotations.Result;
import org.apache.ibatis.annotations.Results;
import org.apache.ibatis.annotations.Select;

import java.util.List;

public interface AccountMapper {
    @Select("""
            SELECT a.id, a.account_name, NULL AS password, a.entity_id, a.created_by, a.created_time, a.last_modified_time
            FROM account AS a
            """)
    @Results(id = "accountMap", value = {
            @Result(property = "id", column = "id"),
            @Result(property = "accountName", column = "account_name"),
            @Result(property = "password", column = "password"),
            @Result(property = "entityId", column = "entity_id"),
            @Result(property = "createdBy", column = "created_by"),
            @Result(property = "createdTime", column = "created_time"),
            @Result(property = "lastModifiedTime", column = "last_modified_time")
    })
    List<Account> findAll();
}

```
</details>

## Create Unit Test
<details open><summary markdown="span">test/resources/application-test.yml</summary>

```yaml
spring:
  datasource:
    driver-class-name: org.h2.Driver
    url: jdbc:h2:mem:testdb;DB_CLOSE_DELAY=-1;DB_CLOSE_ON_EXIT=false;MODE=MYSQL

  test:
    database:
      replace: NONE
```
</details>

<details open><summary markdown="span">test/resources/test-db-schema.sql</summary>

```
DROP TABLE IF EXISTS `account`;;
CREATE TABLE `account` (
	`id` INT NOT NULL AUTO_INCREMENT,
	`account_name` VARCHAR(50) NOT NULL,
	`password` VARCHAR(50) NOT NULL,
	`entity_id` INT NULL DEFAULT NULL,
	`created_by` INT NOT NULL,
	`created_time` TIMESTAMP NULL DEFAULT NULL,
	`last_modified_time` TIMESTAMP NULL DEFAULT NULL,
	PRIMARY KEY (`id`)
);;

```
</details>

<details open><summary markdown="span">com.demo.ledger.account.config.MemoryDBTest.java</summary>

```java
package com.demo.ledger.account.config;

import org.springframework.test.context.ActiveProfiles;
import org.springframework.test.context.jdbc.Sql;
import org.springframework.test.context.jdbc.SqlMergeMode;
import org.springframework.test.context.web.WebAppConfiguration;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

import static org.springframework.test.context.jdbc.SqlMergeMode.MergeMode.MERGE;

@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@WebAppConfiguration
@SqlMergeMode(MERGE)
@Sql(scripts = "classpath:test-db-schema.sql")
@ActiveProfiles("test")
public @interface MemoryDBTest {
}
```
</details>
<details open><summary markdown="span">com.demo.ledger.account.controller.AccountControllerTest.java</summary>

```java
package com.demo.ledger.account.controller;

import com.demo.ledger.account.config.MemoryDBTest;
import jakarta.annotation.Resource;
import org.hamcrest.Matchers;
import org.junit.jupiter.api.Test;
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.jdbc.Sql;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.ResultActions;

import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.result.MockMvcResultHandlers.print;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.jsonPath;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

@SpringBootTest
@AutoConfigureMockMvc
@MemoryDBTest
public class AccountControllerTest {
    @Resource
    private MockMvc mockMvc;

    @Test
    @Sql(statements = """
            INSERT INTO `account` (`id`, `account_name`, `password`, `entity_id`, `created_by`)
            VALUES(1, 'test_account_1 in MemoryDB', 'test1', 1, 1);;
            """)
    public void findAllAccounts() throws Exception {
        ResultActions action = this.mockMvc.perform(get("/account"))
                .andDo(print());
        action.andExpect(status().isOk());
        action.andExpect(jsonPath("$.[0].accountName").value(Matchers.is("test_account_1 in MemoryDB")));
    }
}

```
</details>

## Reproduce the Exception
The exception can be reproduced by removing the `@SqlMergeMode(MERGE)` from `com.demo.ledger.account.config.MemoryDBTest.java`.

## Root Cause
Only the latter one will be invoked When there are `@Sql` annotations at class and method level, which has resulted the exception showed at the beginning of the article. 
