---
layout: ../../../layouts/ArticleLayout.astro
title: NestJS 后端框架实战教程：从零搭建到可上线 API
description: 一篇覆盖项目初始化、模块化设计、数据库、鉴权与部署建议的 NestJS 入门实战。
category: backend
pubDate: 2026-04-10
---

# NestJS 后端框架实战教程：从零搭建到可上线 API

NestJS 是一个基于 TypeScript 的 Node.js 后端框架，借鉴了 Angular 的模块化思想，适合构建中大型 API 服务。它的核心优势是结构清晰、约束明确、可测试性强。

## 1. 为什么选 NestJS

相比“纯 Express 手写”，NestJS 的优势在于：

- 统一架构：模块、控制器、服务分层明确。
- 开发体验好：TypeScript 一等支持，IDE 友好。
- 生态完整：内置 DI、校验、守卫、拦截器、异常过滤器。
- 适合团队协作：代码风格统一，便于多人维护。

## 2. 初始化项目

准备环境：

- Node.js 22+
- pnpm

创建项目：

```bash
pnpm dlx @nestjs/cli new nest-demo
```

进入目录并启动：

```bash
cd nest-demo
pnpm start:dev
```

默认服务启动后可访问：

- http://localhost:3000

## 3. NestJS 核心结构

初始项目中，你会看到几个关键概念：

- Module：功能模块边界，负责组织功能。
- Controller：处理 HTTP 请求，定义路由。
- Provider（Service）：业务逻辑层，可注入复用。
- Pipe：参数转换与校验。
- Guard：鉴权与访问控制。
- Interceptor：日志、响应包装、耗时统计。
- Exception Filter：统一异常处理。

一句话理解：Controller 收请求，Service 干业务，Module 管组织。

## 4. 实战：实现 Users 模块

下面做一个最小可用的用户模块，包含新增和查询。

### 4.1 生成模块代码

```bash
pnpm nest g module users
pnpm nest g service users
pnpm nest g controller users
```

### 4.2 定义 DTO 与参数校验

安装依赖：

```bash
pnpm add class-validator class-transformer
```

在 main.ts 开启全局校验管道：

```ts
app.useGlobalPipes(new ValidationPipe({
  whitelist: true,
  forbidNonWhitelisted: true,
  transform: true,
}));
```

创建 create-user.dto.ts：

```ts
import { IsEmail, IsString, MinLength } from 'class-validator';

export class CreateUserDto {
  @IsString()
  name: string;

  @IsEmail()
  email: string;

  @IsString()
  @MinLength(6)
  password: string;
}
```

### 4.3 编写 Service

```ts
@Injectable()
export class UsersService {
  private users: Array<{ id: number; name: string; email: string }> = [];

  create(dto: CreateUserDto) {
    const user = {
      id: this.users.length + 1,
      name: dto.name,
      email: dto.email,
    };
    this.users.push(user);
    return user;
  }

  findAll() {
    return this.users;
  }
}
```

### 4.4 编写 Controller

```ts
@Controller('users')
export class UsersController {
  constructor(private readonly usersService: UsersService) {}

  @Post()
  create(@Body() dto: CreateUserDto) {
    return this.usersService.create(dto);
  }

  @Get()
  findAll() {
    return this.usersService.findAll();
  }
}
```

到这里你已经有一个可运行的 REST API：

- POST /users
- GET /users

## 5. 接入数据库（PostgreSQL + TypeORM）

真实项目一般不会用内存数组，建议接入数据库。

安装依赖：

```bash
pnpm add @nestjs/typeorm typeorm pg
```

在 AppModule 中配置：

```ts
TypeOrmModule.forRoot({
  type: 'postgres',
  host: process.env.DB_HOST,
  port: Number(process.env.DB_PORT ?? 5432),
  username: process.env.DB_USER,
  password: process.env.DB_PASS,
  database: process.env.DB_NAME,
  autoLoadEntities: true,
  synchronize: false,
});
```

建议：

- 开发环境可短期使用 synchronize。
- 生产环境必须使用迁移（migration）。

## 6. 鉴权：JWT 常见落地方式

安装依赖：

```bash
pnpm add @nestjs/jwt @nestjs/passport passport passport-jwt bcrypt
pnpm add -D @types/passport-jwt @types/bcrypt
```

推荐流程：

1. AuthModule 提供登录接口。
2. 登录校验账号密码后签发 access token。
3. 受保护接口挂 JwtAuthGuard。
4. 在 Guard 中解析并注入当前用户。

接口分层建议：

- 公共接口：登录、注册、健康检查。
- 业务接口：统一走 Guard。
- 管理接口：叠加角色守卫（RBAC）。

## 7. 配置管理与环境变量

安装配置模块：

```bash
pnpm add @nestjs/config
```

实践建议：

- 使用 .env.development 与 .env.production 分环境。
- 对关键配置做 schema 校验，防止启动后才报错。
- 不要将密钥写死在代码里。

## 8. 错误处理、日志与监控

可上线服务至少应具备：

- 全局异常过滤器：统一返回结构。
- 请求日志：记录方法、路径、耗时、状态码。
- 健康检查：提供 /health 接口。
- 基础监控：CPU、内存、错误率、P95 延迟。

日志字段建议统一：

- traceId
- userId
- route
- latency
- statusCode

## 9. 测试策略

NestJS 默认支持 Jest，建议至少覆盖：

- 单元测试：Service 业务逻辑。
- 集成测试：Controller + Service。
- E2E 测试：从 HTTP 请求到响应全链路。

最小测试目标：

- 核心路径覆盖率 > 70%
- 关键业务接口具备回归用例

## 10. 部署建议

常见部署组合：

- Docker + Nginx + PostgreSQL
- 或托管平台（Railway、Render、Fly.io）

上线前检查清单：

- 关闭开发配置与调试输出
- 数据库迁移已执行
- 鉴权和权限策略已验证
- 关键指标可观测
- 回滚方案可执行

## 总结

NestJS 的价值不在“能不能写出接口”，而在“能否把接口写成可维护系统”。推荐你按以下节奏推进：

1. 先把模块化和分层习惯建立起来。
2. 再补数据库与鉴权。
3. 最后完善测试、监控和部署。

按这个路径走，3 到 4 周就能从入门到搭出一个具备生产思维的后端服务。