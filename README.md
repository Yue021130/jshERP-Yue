# 本仓库说明
* 本仓库是开源 ERP 项目 **jshERP（管伊佳ERP，原名：华夏ERP）** 的源码副本，代码拉取自 GitHub
* 原始仓库地址：
  * GitHub：https://github.com/jishenghua/JSH_ERP
  * Gitee（镜像）：https://gitee.com/jishenghua/jshERP
* 本地代码版本：后端 `3.6-SNAPSHOT`、前端 `3.5.0`
* 本地的 git 提交历史已删除，本副本仅作为个人学习 ERP 及二次开发的代码基础
* 版权说明：原项目归原作者及管伊佳ERP 团队所有，遵循 Apache-2.0 协议，感谢原作者的开源贡献

# 项目简介
* jshERP 是一款面向中小企业的开源 ERP 系统，功能覆盖进销存、财务、生产等核心业务
* 系统采用前后端分离架构，支持多租户（可 SaaS 化部署），也可私有化部署
* 本仓库包含两个子工程：`jshERP-boot`（后端服务）、`jshERP-web`（前端工程）

# 总体架构

```
浏览器 ──► jshERP-web（Vue2，dev 端口 3000）──► jshERP-boot（SpringBoot，端口 9999）
                                                     │
                                              ┌──────┴──────┐
                                              ▼             ▼
                                     Redis（会话/缓存）   MySQL 8（jsh_erp 库）
```

* 前端通过 axios 统一封装请求，登录成功后由后端建立 Redis 会话，后续请求头携带 session 标识
* 后端通过全局 Servlet Filter 做登录校验与路径拦截，业务接口按模块划分为 REST Controller
* 数据库初始化脚本：`jshERP-boot/docs/jsh_erp.sql`
* 开发环境下前端通过 `vue.config.js` 将 `/jshERP-boot` 代理到 `http://localhost:9999`

# 后端架构（jshERP-boot）

单一 SpringBoot 工程，主类 `com.jsh.erp.ErpApplication`，默认端口 `9999`，上下文路径 `/jshERP-boot`。

| 包 | 职责 |
|---|---|
| `com.jsh.erp.controller` | REST 接口层，30+ 个控制器，按业务模块划分（商品、单据、财务、系统等） |
| `com.jsh.erp.service` | 业务逻辑层，含 `RedisService` 及序列号、租户等业务服务 |
| `com.jsh.erp.datasource` | 数据层：`entities` 表实体、`mappers` Mapper 接口、`vo` 视图对象 |
| `com.jsh.erp.base` | 基础支撑：统一响应 `AjaxResult`、分页 `TableSupport`、基类 `BaseController` |
| `com.jsh.erp.config` | 配置类：Swagger 文档、租户 `TenantConfig`、插件 `PluginConfiguration` 等 |
| `com.jsh.erp.filter` | `LogCostFilter` 全局过滤器：登录校验（Redis 会话）、非法路径拦截 |
| `com.jsh.erp.utils` / `constants` / `exception` | 工具类、常量、业务异常 |

关键机制：
* **登录鉴权**：`LogCostFilter` 对除登录、注册、接口文档等白名单外的所有请求，校验 Redis 会话中的 `userId`
* **插件机制**：基于 `springboot-plugin-framework`，插件 jar 放入 `plugins/` 目录可热加载，可扩展菜单、接口与 MyBatis 映射
* **多租户**：租户间数据隔离，`application.yml` 中配置租户可建用户数、试用天数等限制
* **接口文档**：Swagger + swagger-bootstrap-ui，启动后访问 `http://localhost:9999/jshERP-boot/doc.html`
* **持久层**：MyBatis-Plus，Mapper XML 位于 `resources/mapper_xml/`，集成 mybatis-generator 代码生成插件

# 前端架构（jshERP-web）

Vue CLI 工程，基于 Jeecg-Boot 2.2 模板开发，UI 组件库为 Ant Design Vue。

* `src/views`：业务页面，按模块组织：
  * `bill` —— 采购/销售/零售/库存单据（入库、出库、调拨、组装拆卸等）
  * `financial` —— 财务（收付款、转账、预收预付、其他收支）
  * `report` —— 报表中心（出入库明细、库存、对账、统计）
  * `material` —— 商品资料（商品、分类、属性、单位）
  * `system` —— 系统管理（用户、角色、菜单、机构、租户、插件等）
* `src/api`：后端接口封装；`src/utils/request.js` 为 axios 统一封装（baseURL、token 头、异常处理）
* `src/router` + `src/store/modules/permission.js`：登录后按用户权限动态生成路由
* `src/store`：Vuex 模块（user / permission / app / dict / enhance）
* `src/components`：通用组件库，含 `jeecg` 系列封装组件、表格、布局等

# 核心业务模块

| 模块 | 主要页面 / 接口 | 说明 |
|---|---|---|
| 基础资料 | Material、Unit、Supplier、Depot、Person、Organization | 商品、单位、供应商/客户/会员、仓库、职员 |
| 采购管理 | PurchaseOrder / In / Back / Apply | 采购订单、采购入库、采购退货、采购申请 |
| 销售管理 | SaleOrder / Out / Back、RetailOut / Back | 销售订单、销售出库、销售退货、零售 |
| 库存管理 | OtherIn / Out、AllocationOut、Assemble / Disassemble | 其他出入库、调拨、组装、拆卸 |
| 财务 | MoneyIn / Out、Giro、AdvanceIn、ItemIn / Out | 收款、付款、转账、预收预付、其他收支 |
| 报表中心 | In / OutDetail、MaterialStock、各类对账与统计报表 | 进销存全链路报表 |
| 系统管理 | User、Role、Function、Tenant、Dict、Log、Plugin | 用户、角色、菜单、多租户、字典、日志、插件 |

单据类业务统一由 `DepotHead`（单据主表）+ `DepotItem`（单据明细）承载出入库流水，财务单据由 `AccountHead`（主表）+ `AccountItem`（明细）承载。

# 技术框架
* 核心框架：SpringBoot 2.0.0
* 持久层框架：MyBatis-Plus（MyBatis 增强）
* 日志管理：SLF4J 1.7
* 缓存 / 会话：Redis 6.2.1
* 接口文档：Swagger + swagger-bootstrap-ui
* 插件框架：springboot-plugin-framework
* 前端框架：Vue 2.7.16
* UI 框架：Ant Design Vue 1.5.2
* 前端模板：Jeecg-Boot 2.2.0
* 项目管理：Maven 3.3.9

# 开发环境
建议开发者使用以下环境，可以避免版本带来的问题
* IDE：IntelliJ IDEA 2025.1.4.1
* DB：MySQL 8.0.24
* JDK：JDK 1.8
* Node：Node 20.17.0
* Maven：3.3.9
* Redis：6.2.1
* Nginx：1.12.2

# 服务器环境
* 数据库：MySQL 8.0.24
* JAVA 平台：JRE 1.8
* Redis 库：Redis 6.2.1
* Nginx 代理：Nginx 1.12.2
* 操作系统：Windows、Linux 等
