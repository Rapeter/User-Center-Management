# 用户中心管理系统

一个基于 Spring Boot 和 React 的用户中心练习项目，覆盖注册、登录、Session 登录态、管理员查询用户和逻辑删除用户。前后端源码分别位于 `backend/` 与 `frontend/`。

## 技术栈

| 部分 | 技术 |
| --- | --- |
| 后端 | Java 8、Spring Boot 2.6.4、MyBatis-Plus、MySQL |
| 前端 | React 17、Umi 3、Ant Design Pro、Umi Request |

## 目录结构

```text
backend/                 Spring Boot 后端 Maven 项目
  sql/create_table.sql   数据库与用户表初始化脚本
  src/main/java/         Controller、Service、Mapper、模型及异常处理
  src/main/resources/    Spring 配置与 Mapper XML
frontend/                React 前端
  config/                Umi 路由、代理与运行配置
  src/pages/              登录、注册和管理员用户页面
  src/services/           前端 API 调用
源码学习路线.md           分阶段源码阅读计划和练习
```

## 本地运行

请先安装 Java 8 和 Maven，并准备 MySQL。

### 1. 准备数据库

准备 MySQL，然后执行 [`backend/sql/create_table.sql`](backend/sql/create_table.sql)。脚本会创建 `yupi` 数据库和 `user` 表。

### 2. 启动后端

后端默认监听 `8080`，应用上下文为 `/api`。启动前设置数据库连接；下面是 PowerShell 示例：

```powershell
$env:DB_URL = "jdbc:mysql://localhost:3306/yupi"
$env:DB_USERNAME = "root"
$env:DB_PASSWORD = "你的本地 MySQL 密码"
Set-Location backend
mvn spring-boot:run
```

macOS/Linux 可以在 `backend/` 目录运行：

```bash
export DB_URL="jdbc:mysql://localhost:3306/yupi"
export DB_USERNAME="root"
export DB_PASSWORD="你的本地 MySQL 密码"
mvn spring-boot:run
```

如果不设置变量，本地配置默认连接 `localhost:3306/yupi`，用户名默认为 `root`，密码默认为空。也可以按自己的 MySQL 配置覆盖这些值。

### 3. 启动前端

在另一个终端运行：

```bash
cd frontend
npm install
npm start
```

Umi 开发代理会把 `/api` 请求转发到 `http://localhost:8080`。如后端地址或端口不同，修改 `frontend/config/proxy.ts` 的 `dev.target`。

## 管理员账号

先通过注册页面创建账号，再在 `yupi` 数据库中将该账号设为管理员：

```sql
UPDATE user
SET userRole = 1
WHERE userAccount = '你的账号' AND isDelete = 0;
```

管理员页面入口受前端角色控制；查询和删除接口也会在后端校验管理员角色。

## 生产配置

后端生产 profile 使用 `DB_URL`、`DB_USERNAME`、`DB_PASSWORD` 环境变量。启用方式：

```bash
cd backend
mvn -DskipTests package
java -jar target/user-center-backend-0.0.1-SNAPSHOT.jar --spring.profiles.active=prod
```

PowerShell 下可在 `backend/` 目录运行 `mvn -DskipTests package`，再运行 `java -jar target\user-center-backend-0.0.1-SNAPSHOT.jar --spring.profiles.active=prod`。打包命令跳过测试执行：现有 `UserServiceTest` 会直接更新和逻辑删除 ID 为 1 的记录，且注册断言与当前实现不一致；修复这些测试并配置隔离测试库后，再单独运行测试。

前端生产构建可设置 `REACT_APP_API_BASE_URL`。PowerShell 示例：

```powershell
$env:REACT_APP_API_BASE_URL = "https://api.example.com"
npm run build
```

macOS/Linux 示例：

```bash
REACT_APP_API_BASE_URL="https://api.example.com" npm run build
```

未设置时，前端使用同源的 `/api` 路径。部署时需要由 Nginx 或其他反向代理把 `/api` 转发到后端；同源代理也便于保留基于 Cookie 的 Session 登录态。当前 `frontend/docker/nginx.conf` 负责静态资源服务，部署时需按实际后端地址增加 `/api` 代理规则。源码不包含数据库密码或自有生产域名。

## 学习建议

从 [`源码学习路线.md`](源码学习路线.md) 开始，按照数据库、后端分层、Session 登录态、React 路由与页面、前后端接口调用的顺序阅读。源码包中的密码处理使用固定盐 MD5，仅用于学习示例；实际系统应改用适合密码存储的慢哈希算法并评估完整的部署安全配置。


