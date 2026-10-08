# 用户中心 React 前端

本目录使用 React 17、Umi 3 和 Ant Design Pro。完整环境准备、数据库与后端启动方式见[根目录 README](../README.md)，源码阅读顺序见[学习路线](../源码学习路线.md)。

## 开发

在本目录执行：

```bash
npm ci
npm run start:dev
```

默认访问 [http://localhost:8000](http://localhost:8000)。`start:dev` 关闭 Mock，并通过开发代理将 `/api` 转发到 `http://localhost:8080`，需要同时启动后端。

保留 `.npmrc` 和 `package-lock.json`：它们分别提供旧依赖与 OpenSSL 的兼容配置，以及可复现的安装版本。

## 构建

```bash
npm run build
```

产物在 `dist/`。API 默认使用同源 `/api`；如需独立 API 域名，在构建前设置 `REACT_APP_API_BASE_URL`，具体部署说明见根目录 README。

## 主要源码

| 位置 | 内容 |
| --- | --- |
| `config/routes.ts` | 路由与管理员入口 |
| `config/proxy.ts` | 开发代理 |
| `src/pages/user/Login/index.tsx` | 登录表单 |
| `src/pages/user/Register/index.tsx` | 注册表单 |
| `src/pages/Admin/UserManage/index.tsx` | 用户查询、删除确认与表格刷新 |
| `src/services/ant-design-pro/api.ts` | 后端 API 调用 |
| `src/plugins/globalRequest.ts` | Cookie、请求头与响应处理 |
| `src/app.tsx`、`src/access.ts` | 当前用户加载与前端权限 |

页面基于 [Ant Design Pro](https://pro.ant.design) 原模板整理，原始项目作者为[程序员鱼皮](https://github.com/liyupi)。
