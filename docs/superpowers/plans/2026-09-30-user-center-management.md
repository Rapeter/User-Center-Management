# 用户中心管理项目 Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use `superpowers:subagent-driven-development` or `superpowers:executing-plans` to implement this plan task-by-task. Each task ends with a separate commit.

**Goal:** 将现有 Spring Boot 后端与 React 前端整理进目标仓库，补齐既定用户管理流程，并提交源码学习路线文档。

**Architecture:** 按 `backend/` 和 `frontend/` 保留原有前后端边界与技术栈。前端通过现有 `/api/user/*` 路由调用后端；后端保留 Controller、Service、Mapper 分层以及 Session 登录态。

**Tech Stack:** Java 8、Spring Boot 2.6.4、MyBatis-Plus 3.5.1、MySQL、React 17、Umi 3、Ant Design Pro。

**Spec:** [2026-09-30-user-center-management-design.md](../specs/2026-09-30-user-center-management-design.md)

## Global Constraints

- 保持 Java 8、Spring Boot 2.6.4、MyBatis-Plus 3.5.1、React 17、Umi 3。
- 使用后端与 React 版 ZIP 源码；不纳入 Vue 3 前端和通用初始化模板。
- 后端生产数据库读取 `DB_URL`、`DB_USERNAME`、`DB_PASSWORD`；不保留 ZIP 中的生产凭据。
- 前端生产 API 地址使用 `REACT_APP_API_BASE_URL`；默认使用同源 `/api` 路由。
- 保留原作者标注和已有测试源码；本次不新增或运行测试。
- 不升级框架、不改造密码存储算法、不增加用户编辑或分页接口。

## Review Focus

以下情形需在代码审阅中留意；本次依照任务要求不新增或运行测试：

1. 注册服务返回 `-1` 时，接口不能将其作为成功的用户 ID 返回。
2. 登录凭据错误时，响应需包含前端可显示的描述，且不能写入登录 Session。
3. Session 中用户记录已删除时，`/user/current` 需清理失效状态并返回未登录错误。
4. 用户名查询为空或有内容时，管理表格都需正确处理接口返回列表。
5. 删除失败或请求错误时，管理表格不能显示删除成功；非管理员仍由后端拒绝。

---

### Task 1: 导入源码并清理部署配置

**Files:**
- Create: `backend/**`（保留 `user-center-backend-master.zip` 内的相对路径）
- Create: `frontend/**`（保留 `user-center-frontend-master.zip` 内的相对路径）
- Create: `.gitignore`
- Modify: `backend/src/main/resources/application.yml`
- Modify: `backend/src/main/resources/application-prod.yml`
- Modify: `frontend/config/config.ts`
- Modify: `frontend/src/plugins/globalRequest.ts`

**Interfaces:**
- Produces: 后续任务使用 `backend/` 和 `frontend/` 作为唯一应用源码根目录；配置由 `DB_URL`、`DB_USERNAME`、`DB_PASSWORD` 和 `REACT_APP_API_BASE_URL` 控制。

- [ ] 将后端 ZIP 解压到 `backend/`，将 React 前端 ZIP 解压到 `frontend/`；保留各自 README、Dockerfile、SQL、资源及测试源码。
- [ ] 不导入 ZIP 文件本身、Vue 项目、Vue 包内 `.git/` 或两个通用初始化模板。
- [ ] 新增仓库根 `.gitignore`，忽略 `backend/target/`、`frontend/node_modules/`、`frontend/dist/`、`.env`、`.env.*` 及 IDE 缓存。
- [ ] 首次提交前，将本地数据库设置改为可覆盖的环境变量，将生产数据库设置改为只读取 `DB_URL`、`DB_USERNAME`、`DB_PASSWORD`；不把 ZIP 中的原始生产设置写入任何 Git 提交。
- [ ] 在 Umi 配置中定义 `process.env.REACT_APP_API_BASE_URL`，请求插件使用该值作为 prefix；未设置时不附加外部主机前缀。
- [ ] 提交：`chore: add backend and React sources with safe config`。

### Task 2: 统一注册、登录和当前用户错误处理

**Files:**
- Modify: `backend/src/main/java/com/yupi/usercenter/controller/UserController.java`

**Interfaces:**
- Consumes: `UserService.userRegister(...)` 现有返回值、`UserService.userLogin(...)` 现有返回值、`BusinessException`、`ErrorCode`。
- Produces: 注册失败抛出带可读描述的 `PARAMS_ERROR`；无效登录抛出带可读描述的凭据错误；失效 Session 抛出 `NOT_LOGIN`。

- [ ] 注册请求为空或必填字段为空时抛出 `BusinessException`，不返回 Java `null`。
- [ ] 注册服务返回值小于等于 0 时抛出参数错误；只有正数用户 ID 进入成功响应。
- [ ] 登录服务返回 `null` 时抛出含“账号或密码错误”描述的业务错误；成功登录仍返回脱敏用户。
- [ ] `/user/current` 取到的数据库用户为空时移除 Session 中的登录态并抛出 `NOT_LOGIN`。
- [ ] 提交：`fix: return consistent user authentication errors`。

### Task 3: 接通管理员用户搜索与删除

**Files:**
- Modify: `frontend/src/services/ant-design-pro/api.ts`
- Modify: `frontend/src/pages/Admin/UserManage/index.tsx`

**Interfaces:**
- Produces: `searchUsers(params?: { username?: string })` 向 `GET /api/user/search` 传 query 参数；`deleteUser(id: number)` 向 `POST /api/user/delete` 发送 JSON 数字 ID。

- [ ] 扩展 `searchUsers` 的参数类型并将 `username` 传入请求 `params`。
- [ ] 新增 `deleteUser(id: number)` API 方法，沿用现有请求插件和 `/api/user/delete` 路由。
- [ ] 表格查询请求传入 `params.username`，返回完整列表及正确的 total；由于接口不分页，关闭表格分页。
- [ ] 每行提供带二次确认的删除操作；成功时提示并刷新，不成功时提示失败。
- [ ] 移除当前没有后端实现的编辑、查看和无效下拉操作；保留现有管理员页面访问控制。
- [ ] 提交：`feat: connect admin user search and delete`。

### Task 4: 编写仓库启动说明

**Files:**
- Create: `README.md`

**Interfaces:**
- Produces: 从仓库根目录可查到项目结构、依赖、数据库初始化及前后端启动方式。

- [ ] 说明技术栈、`backend/` 与 `frontend/` 目录、用户表脚本 `backend/sql/create_table.sql`。
- [ ] 说明 MySQL 数据库准备、后端环境变量和启动命令，以及前端安装/启动命令。
- [ ] 说明 `/api` 上下文路径、开发代理目标、生产 API 基址配置和同源反向代理要求。
- [ ] 说明如何注册普通用户，以及如何通过数据库为一个账号授予管理员角色。
- [ ] 提交：`docs: add project setup guide`。

### Task 5: 编写源码学习路线

**Files:**
- Create: `源码学习路线.md`

**Interfaces:**
- Produces: 面向初学者的分阶段源码阅读顺序、准确文件路径、调用链和练习。

- [ ] 依次讲解 SQL/User 实体、Mapper、Service 注册登录、Controller 与统一异常、Session 登录态、前端路由/权限、前端 API 封装、登录/注册页和管理页。
- [ ] 对每阶段列出仓库内具体源码路径和要理解的问题，确保路径指向 `backend/` 或 `frontend/` 中真实文件。
- [ ] 加入从 React 表单到 Controller、Service、Mapper、MySQL 再回到统一响应的端到端追踪练习。
- [ ] 标出固定盐 MD5、无分页用户列表等现状与后续可改进方向，不把这些改造纳入本次代码任务。
- [ ] 提交：`docs: add source code learning roadmap`。

### Task 6: 推送完整交付

**Files:**
- Review: 所有已提交源码与 Markdown 文件

**Interfaces:**
- Consumes: Task 1–5 的提交。
- Produces: `origin/main` 包含本次全部提交。

- [ ] 查看 Git 提交和工作区状态，确认待推送内容仅属于本项目交付。
- [ ] 推送 `main` 到 `https://github.com/Rapeter/User-Center-Management.git`。
- [ ] 汇报提交记录、学习路线文件位置，以及本次未新增/运行测试这一限制。
