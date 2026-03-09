# sa-token Node.js/TypeScript 移植计划

> 本文档描述将 Java 轻量级权限认证框架 **sa-token** 移植到 Node.js/TypeScript 的完整方案。

---

## 一、项目背景

### 1.1 源项目简介

[sa-token](https://sa-token.cc) 是一个轻量级 Java 权限认证框架，主要特性：

- **登录认证**：单端登录、多端登录、同端互斥登录、七天内免登录
- **权限认证**：权限认证、角色认证、会话二级认证
- **踢人下线**：根据账号id踢人下线、根据Token值踢人下线
- **Session会话**：全端共享Session，单端独享Session
- **单点登录**：三种模式支持同域、跨域、前后端分离
- **OAuth2.0**：支持授权码、隐藏式、密码式、客户端凭证式

### 1.2 移植目标

将 sa-token 的核心功能移植到 Node.js/TypeScript，实现：

| 功能模块 | 优先级 | 说明 |
|---------|--------|------|
| 核心认证 | P0 | 登录/注销、Token管理、会话管理 |
| 权限验证 | P0 | 角色/权限校验 |
| SSO单点登录 | P1 | Ticket验证、同域/跨域模式 |
| OAuth2.0 | P1 | 完整授权服务器实现 |
| Redis适配器 | P1 | 分布式存储支持 |

---

## 二、项目结构设计

### 2.1 目录结构

```
sa-token-ts/
├── packages/
│   ├── core/                              # 核心包
│   │   ├── src/
│   │   │   ├── config/
│   │   │   │   ├── SaTokenConfig.ts           # 配置类
│   │   │   │   ├── SaCookieConfig.ts          # Cookie配置
│   │   │   │   └── index.ts
│   │   │   ├── constants/
│   │   │   │   └── SaTokenConsts.ts           # 常量定义
│   │   │   ├── dao/
│   │   │   │   ├── SaTokenDao.ts              # DAO接口
│   │   │   │   ├── SaTokenDaoMemory.ts        # 内存实现
│   │   │   │   └── index.ts
│   │   │   ├── error/
│   │   │   │   ├── SaErrorCode.ts             # 错误码
│   │   │   │   └── index.ts
│   │   │   ├── exception/
│   │   │   │   ├── SaTokenException.ts        # 基础异常
│   │   │   │   ├── NotLoginException.ts       # 未登录异常
│   │   │   │   ├── NotPermissionException.ts  # 无权限异常
│   │   │   │   ├── NotRoleException.ts        # 无角色异常
│   │   │   │   ├── DisableServiceException.ts # 封禁异常
│   │   │   │   └── index.ts
│   │   │   ├── listener/
│   │   │   │   ├── SaTokenListener.ts         # 监听器接口
│   │   │   │   ├── SaTokenEventCenter.ts      # 事件中心
│   │   │   │   └── index.ts
│   │   │   ├── session/
│   │   │   │   ├── SaSession.ts               # 会话模型
│   │   │   │   ├── SaTerminalInfo.ts          # 终端信息
│   │   │   │   └── index.ts
│   │   │   ├── stp/
│   │   │   │   ├── StpLogic.ts                # 核心逻辑类
│   │   │   │   ├── StpUtil.ts                 # 静态工具类
│   │   │   │   ├── StpInterface.ts            # 权限接口
│   │   │   │   ├── parameter/
│   │   │   │   │   ├── SaLoginParameter.ts
│   │   │   │   │   ├── SaLogoutParameter.ts
│   │   │   │   │   └── enums.ts
│   │   │   │   └── index.ts
│   │   │   ├── strategy/
│   │   │   │   └── SaStrategy.ts              # 策略模式
│   │   │   ├── util/
│   │   │   │   └── SaFoxUtil.ts               # 工具类
│   │   │   ├── context/
│   │   │   │   ├── SaTokenContext.ts          # 上下文管理
│   │   │   │   └── index.ts
│   │   │   ├── SaManager.ts                   # 全局管理器
│   │   │   └── index.ts                       # 导出入口
│   │   ├── package.json
│   │   ├── tsconfig.json
│   │   └── README.md
│   │
│   ├── adapter-redis/                      # Redis 适配器
│   │   ├── src/
│   │   │   ├── SaTokenDaoRedis.ts
│   │   │   └── index.ts
│   │   └── package.json
│   │
│   ├── plugin-sso/                         # SSO 插件
│   │   ├── src/
│   │   │   ├── config/
│   │   │   │   ├── SaSsoServerConfig.ts
│   │   │   │   └── SaSsoClientConfig.ts
│   │   │   ├── template/
│   │   │   │   ├── SaSsoTemplate.ts
│   │   │   │   ├── SaSsoServerTemplate.ts
│   │   │   │   └── SaSsoClientTemplate.ts
│   │   │   ├── processor/
│   │   │   │   └── SaSsoProcessor.ts
│   │   │   └── index.ts
│   │   └── package.json
│   │
│   └── plugin-oauth2/                      # OAuth2 插件
│       ├── src/
│       │   ├── config/
│       │   │   └── SaOAuth2ServerConfig.ts
│       │   ├── dao/
│       │   │   └── SaOAuth2Dao.ts
│       │   ├── data/
│       │   │   ├── model/
│       │   │   │   ├── AccessTokenModel.ts
│       │   │   │   ├── RefreshTokenModel.ts
│       │   │   │   ├── CodeModel.ts
│       │   │   │   └── ClientTokenModel.ts
│       │   │   ├── loader/
│       │   │   │   └── SaOAuth2DataLoader.ts
│       │   │   └── generate/
│       │   │       └── SaOAuth2DataGenerate.ts
│       │   ├── template/
│       │   │   └── SaOAuth2Template.ts
│       │   ├── granttype/
│       │   │   └── handlers/
│       │   └── index.ts
│       └── package.json
│
├── examples/                               # 示例项目
│   ├── basic-usage/
│   ├── with-express/
│   └── with-nestjs/
│
├── docs/                                   # 文档
│   ├── getting-started.md
│   ├── configuration.md
│   └── api-reference.md
│
├── tests/                                  # 测试
│   ├── unit/
│   └── integration/
│
├── package.json                            # Monorepo 根配置
├── tsconfig.base.json
├── jest.config.js
├── pnpm-workspace.yaml
└── README.md
```

### 2.2 包依赖关系

```
@sa-token/core                    # 核心包，无外部依赖
    ↓
@sa-token/adapter-redis           # 依赖 core + ioredis
@sa-token/plugin-sso              # 依赖 core
@sa-token/plugin-oauth2           # 依赖 core
```

---

## 三、核心接口设计

### 3.1 配置接口 (SaTokenConfig)

```typescript
// packages/core/src/config/SaTokenConfig.ts

export interface SaTokenConfig {
  // ===== Token 基础配置 =====

  /** Token名称（Cookie名称、请求参数名、存储key前缀），默认 "satoken" */
  tokenName: string;

  /** Token有效期（秒），-1表示永久有效，默认 30天 */
  timeout: number;

  /** Token最低活跃频率（秒），-1表示不限制，默认 -1 */
  activeTimeout: number;

  /** 是否启用动态activeTimeout */
  dynamicActiveTimeout: boolean;

  // ===== 登录配置 =====

  /** 是否允许同一账号多地同时登录，默认 true */
  isConcurrent: boolean;

  /** 多人登录同一账号时是否共用一个token，默认 false */
  isShare: boolean;

  /** 同一账号最大登录数量，-1表示不限，默认 12 */
  maxLoginCount: number;

  /** Token风格：uuid, simple-uuid, random-32, random-64, random-128 */
  tokenStyle: TokenStyle;

  /** Token前缀，如 "Bearer" */
  tokenPrefix?: string;

  // ===== Token 读取配置 =====

  /** 是否尝试从请求体读取token，默认 true */
  isReadBody: boolean;

  /** 是否尝试从请求头读取token，默认 true */
  isReadHeader: boolean;

  /** 是否尝试从Cookie读取token，默认 true */
  isReadCookie: boolean;

  /** 是否为持久Cookie */
  isLastingCookie: boolean;

  /** 是否在登录后将token写入响应头 */
  isWriteHeader: boolean;

  // ===== 其他配置 =====

  /** 是否自动续签activeTimeout */
  autoRenew: boolean;

  /** 日志配置 */
  isLog: boolean;
  logLevel: LogLevel;

  /** Cookie配置 */
  cookie: SaCookieConfig;
}

export type TokenStyle = 'uuid' | 'simple-uuid' | 'random-32' | 'random-64' | 'random-128';
export type LogLevel = 'trace' | 'debug' | 'info' | 'warn' | 'error' | 'fatal';

// 默认配置
export const DEFAULT_SA_TOKEN_CONFIG: SaTokenConfig = {
  tokenName: 'satoken',
  timeout: 60 * 60 * 24 * 30, // 30天
  activeTimeout: -1,
  dynamicActiveTimeout: false,
  isConcurrent: true,
  isShare: false,
  maxLoginCount: 12,
  tokenStyle: 'uuid',
  isReadBody: true,
  isReadHeader: true,
  isReadCookie: true,
  isLastingCookie: true,
  isWriteHeader: false,
  autoRenew: true,
  isLog: false,
  logLevel: 'trace',
  cookie: {},
};
```

### 3.2 DAO 接口 (SaTokenDao)

```typescript
// packages/core/src/dao/SaTokenDao.ts

import type { SaSession } from '../session/SaSession';

/** DAO 常量 */
export const SaTokenDaoConstants = {
  /** 永不过期 */
  NEVER_EXPIRE: -1,
  /** 不存在 */
  NOT_VALUE_EXPIRE: -2,
} as const;

/**
 * Sa-Token 数据访问接口
 *
 * 实现此接口可将数据存储至任意位置：内存、Redis、数据库等
 */
export interface SaTokenDao {
  // ===== 常量访问 =====
  readonly NEVER_EXPIRE: number;
  readonly NOT_VALUE_EXPIRE: number;

  // ===== 字符串读写 =====

  /** 获取值 */
  get(key: string): string | null;

  /** 写入值并设置过期时间（秒） */
  set(key: string, value: string, timeout: number): void;

  /** 更新值（保持原过期时间） */
  update(key: string, value: string): void;

  /** 删除值 */
  delete(key: string): void;

  /** 获取剩余过期时间（秒） */
  getTimeout(key: string): number;

  /** 更新过期时间（秒） */
  updateTimeout(key: string, timeout: number): void;

  // ===== 对象读写 =====

  /** 获取对象 */
  getObject<T>(key: string): T | null;

  /** 写入对象并设置过期时间（秒） */
  setObject<T>(key: string, object: T, timeout: number): void;

  /** 更新对象 */
  updateObject<T>(key: string, object: T): void;

  /** 删除对象 */
  deleteObject(key: string): void;

  /** 获取对象剩余过期时间（秒） */
  getObjectTimeout(key: string): number;

  /** 更新对象过期时间（秒） */
  updateObjectTimeout(key: string, timeout: number): void;

  // ===== Session 读写 =====

  /** 获取Session */
  getSession(sessionId: string): SaSession | null;

  /** 写入Session */
  setSession(session: SaSession, timeout: number): void;

  /** 更新Session */
  updateSession(session: SaSession): void;

  /** 删除Session */
  deleteSession(sessionId: string): void;

  /** 获取Session剩余过期时间 */
  getSessionTimeout(sessionId: string): number;

  /** 更新Session过期时间 */
  updateSessionTimeout(sessionId: string, timeout: number): void;

  // ===== 数据搜索 =====

  /** 搜索数据 */
  searchData(
    prefix: string,
    keyword: string,
    start: number,
    size: number,
    sortType: boolean
  ): string[];

  // ===== 生命周期 =====

  /** 初始化 */
  init?(): void | Promise<void>;

  /** 销毁 */
  destroy?(): void | Promise<void>;
}
```

### 3.3 Session 模型 (SaSession)

```typescript
// packages/core/src/session/SaSession.ts

import type { SaTerminalInfo } from './SaTerminalInfo';

/** Session 类型 */
export type SessionType = 'account' | 'token' | 'custom';

/** Session 数据结构（用于序列化） */
export interface SaSessionData {
  id: string;
  type: SessionType;
  loginType: string;
  loginId?: string | number;
  token?: string;
  createTime: number;
  dataMap: Record<string, unknown>;
  terminalList: SaTerminalInfo[];
  historyTerminalCount: number;
}

/**
 * Session Model，会话作用域的读取值对象
 *
 * 在 Sa-Token 中，SaSession 分为三种：
 * - Account-Session: 为每个账号id分配的Session
 * - Token-Session: 为每个token分配的Session
 * - Custom-Session: 以特定值作为SessionId的Session
 */
export class SaSession {
  /** 存储用户对象时的建议key */
  public static readonly USER = 'USER';

  /** 存储角色列表时的建议key */
  public static readonly ROLE_LIST = 'ROLE_LIST';

  /** 存储权限列表时的建议key */
  public static readonly PERMISSION_LIST = 'PERMISSION_LIST';

  private id: string;
  private type: SessionType;
  private loginType: string;
  private loginId?: string | number;
  private token?: string;
  private createTime: number;
  private dataMap: Map<string, unknown>;
  private terminalList: SaTerminalInfo[];
  private historyTerminalCount: number;

  constructor(id: string);

  // ===== 存取值 =====

  /** 取值 */
  get<T = unknown>(key: string): T | undefined;

  /** 写值 */
  set(key: string, value: unknown): this;

  /** 写值（仅当key不存在时） */
  setByNull(key: string, value: unknown): this;

  /** 删值 */
  delete(key: string): this;

  /** 判断是否存在 */
  has(key: string): boolean;

  /** 获取所有key */
  keys(): string[];

  /** 清空所有数据 */
  clear(): void;

  // ===== 终端管理 =====

  /** 获取终端列表 */
  getTerminalList(): SaTerminalInfo[];

  /** 按设备类型获取终端列表 */
  getTerminalListByDeviceType(deviceType?: string): SaTerminalInfo[];

  /** 添加终端 */
  addTerminal(terminal: SaTerminalInfo): void;

  /** 移除终端 */
  removeTerminal(tokenValue: string): void;

  /** 获取终端 */
  getTerminal(tokenValue: string): SaTerminalInfo | null;

  // ===== 生命周期 =====

  /** 更新Session */
  update(): void;

  /** 注销Session */
  logout(): void;

  /** 获取剩余存活时间 */
  timeout(): number;

  /** 更新存活时间 */
  updateTimeout(timeout: number): void;

  // ===== 序列化 =====

  toJSON(): SaSessionData;
  static fromJSON(data: SaSessionData): SaSession;
}
```

### 3.4 核心逻辑类 (StpLogic)

```typescript
// packages/core/src/stp/StpLogic.ts

import type { SaSession } from '../session/SaSession';
import type { SaTerminalInfo } from '../session/SaTerminalInfo';
import type { SaLoginParameter } from './parameter/SaLoginParameter';
import type { SaLogoutParameter } from './parameter/SaLogoutParameter';

/**
 * Sa-Token 核心逻辑类
 *
 * 包含所有认证、权限、会话管理的核心方法
 */
export class StpLogic {
  private loginType: string;

  constructor(loginType: string);

  // ===== Token 相关 =====

  /** 获取Token名称 */
  getTokenName(): string;

  /** 获取当前Token值 */
  getTokenValue(): string | null;

  /** 获取Token信息 */
  getTokenInfo(): SaTokenInfo;

  /** 设置Token值 */
  setTokenValue(tokenValue: string, parameter?: SaLoginParameter): void;

  // ===== 登录相关 =====

  /** 会话登录 */
  login(id: string | number, parameter?: SaLoginParameter): void;

  /** 创建登录会话 */
  createLoginSession(id: string | number, parameter?: SaLoginParameter): string;

  /** 注销登录 */
  logout(parameter?: SaLogoutParameter): void;

  /** 根据Token注销 */
  logoutByTokenValue(tokenValue: string, parameter?: SaLogoutParameter): void;

  /** 踢人下线 */
  kickout(tokenValue: string, parameter?: SaLogoutParameter): void;

  /** 顶人下线 */
  replaced(tokenValue: string, parameter?: SaLogoutParameter): void;

  // ===== 会话查询 =====

  /** 判断是否登录 */
  isLogin(): boolean;

  /** 判断指定账号是否登录 */
  isLogin(loginId: string | number): boolean;

  /** 校验登录（未登录抛异常） */
  checkLogin(): void;

  /** 获取登录ID */
  getLoginId(): string | number;

  /** 获取登录ID（未登录返回null） */
  getLoginIdDefaultNull(): string | number | null;

  /** 获取登录ID（字符串） */
  getLoginIdAsString(): string;

  /** 获取登录ID（数字） */
  getLoginIdAsNumber(): number;

  // ===== Session 相关 =====

  /** 获取当前会话的Session */
  getSession(isCreate?: boolean): SaSession | null;

  /** 获取指定账号的Session */
  getSessionByLoginId(loginId: string | number, isCreate?: boolean): SaSession | null;

  /** 获取Token-Session */
  getTokenSession(): SaSession;

  /** 获取指定Token的Session */
  getTokenSessionByToken(tokenValue: string): SaSession;

  // ===== 权限验证 =====

  /** 获取权限列表 */
  getPermissionList(): string[];

  /** 获取指定账号的权限列表 */
  getPermissionList(loginId: string | number): string[];

  /** 判断是否有权限 */
  hasPermission(permission: string): boolean;

  /** 判断是否有全部权限 */
  hasPermissionAnd(...permissions: string[]): boolean;

  /** 判断是否有任一权限 */
  hasPermissionOr(...permissions: string[]): boolean;

  /** 校验权限 */
  checkPermission(permission: string): void;

  /** 校验全部权限 */
  checkPermissionAnd(...permissions: string[]): void;

  /** 校验任一权限 */
  checkPermissionOr(...permissions: string[]): void;

  // ===== 角色验证 =====

  /** 获取角色列表 */
  getRoleList(): string[];

  /** 获取指定账号的角色列表 */
  getRoleList(loginId: string | number): string[];

  /** 判断是否有角色 */
  hasRole(role: string): boolean;

  /** 校验角色 */
  checkRole(role: string): void;

  // ===== 账号封禁 =====

  /** 封禁账号 */
  disable(loginId: string | number, time: number, service?: string): void;

  /** 判断是否被封禁 */
  isDisable(loginId: string | number, service?: string): boolean;

  /** 校验封禁状态 */
  checkDisable(loginId: string | number, ...services: string[]): void;

  /** 获取剩余封禁时间 */
  getDisableTime(loginId: string | number, service?: string): number;

  /** 解除封禁 */
  untieDisable(loginId: string | number, ...services: string[]): void;

  // ===== 阶梯封禁 =====

  /** 封禁到指定等级 */
  disableLevel(loginId: string | number, level: number, time: number, service?: string): void;

  /** 判断是否被封禁到指定等级 */
  isDisableLevel(loginId: string | number, level: number, service?: string): boolean;

  /** 获取封禁等级 */
  getDisableLevel(loginId: string | number, service?: string): number;

  // ===== 二级认证 =====

  /** 开启二级认证 */
  openSafe(safeTime: number, service?: string): void;

  /** 判断是否处于二级认证 */
  isSafe(service?: string): boolean;

  /** 校验二级认证 */
  checkSafe(service?: string): void;

  /** 获取二级认证剩余时间 */
  getSafeTime(service?: string): number;

  /** 关闭二级认证 */
  closeSafe(service?: string): void;

  // ===== 身份切换 =====

  /** 临时切换身份 */
  switchTo(loginId: string | number): void;

  /** 结束身份切换 */
  endSwitch(): void;

  /** 判断是否处于身份切换 */
  isSwitch(): boolean;

  // ===== Token 管理 =====

  /** 获取Token剩余有效时间 */
  getTokenTimeout(): number;

  /** 续签Token */
  renewTimeout(timeout: number): void;

  // ===== 活跃超时 =====

  /** 更新最后活跃时间 */
  updateLastActiveToNow(): void;

  /** 校验活跃超时 */
  checkActiveTimeout(): void;
}
```

### 3.5 静态工具类 (StpUtil)

```typescript
// packages/core/src/stp/StpUtil.ts

import { StpLogic } from './StpLogic';
import { SaManager } from '../SaManager';

/**
 * Sa-Token 权限认证工具类
 *
 * 静态代理所有 StpLogic 方法，提供便捷的 API 调用
 */
export class StpUtil {
  private static _stpLogic: StpLogic | null = null;

  /** 账号类型标识 */
  public static readonly TYPE = 'login';

  private constructor() {}

  /** 获取底层 StpLogic 实例 */
  static get stpLogic(): StpLogic {
    if (!this._stpLogic) {
      this._stpLogic = new StpLogic(this.TYPE);
    }
    return this._stpLogic;
  }

  /** 设置自定义 StpLogic */
  static setStpLogic(stpLogic: StpLogic): void {
    this._stpLogic = stpLogic;
    SaManager.putStpLogic(stpLogic);
  }

  // ===== 以下为静态代理方法 =====

  /** 获取账号类型 */
  static getLoginType(): string {
    return this.stpLogic.getLoginType();
  }

  /** 获取Token名称 */
  static getTokenName(): string {
    return this.stpLogic.getTokenName();
  }

  /** 获取Token值 */
  static getTokenValue(): string | null {
    return this.stpLogic.getTokenValue();
  }

  /** 登录 */
  static login(id: string | number, parameter?: SaLoginParameter): void {
    this.stpLogic.login(id, parameter);
  }

  /** 注销 */
  static logout(): void {
    this.stpLogic.logout();
  }

  /** 判断是否登录 */
  static isLogin(): boolean {
    return this.stpLogic.isLogin();
  }

  /** 校验登录 */
  static checkLogin(): void {
    this.stpLogic.checkLogin();
  }

  /** 获取登录ID */
  static getLoginId(): string | number {
    return this.stpLogic.getLoginId();
  }

  /** 判断是否有权限 */
  static hasPermission(permission: string): boolean {
    return this.stpLogic.hasPermission(permission);
  }

  /** 校验权限 */
  static checkPermission(permission: string): void {
    this.stpLogic.checkPermission(permission);
  }

  /** 判断是否有角色 */
  static hasRole(role: string): boolean {
    return this.stpLogic.hasRole(role);
  }

  /** 校验角色 */
  static checkRole(role: string): void {
    this.stpLogic.checkRole(role);
  }

  /** 封禁账号 */
  static disable(loginId: string | number, time: number): void {
    this.stpLogic.disable(loginId, time);
  }

  /** 判断是否被封禁 */
  static isDisable(loginId: string | number): boolean {
    return this.stpLogic.isDisable(loginId);
  }

  /** 解除封禁 */
  static untieDisable(loginId: string | number): void {
    this.stpLogic.untieDisable(loginId);
  }

  /** 获取Session */
  static getSession(): SaSession {
    return this.stpLogic.getSession()!;
  }

  /** 获取Token-Session */
  static getTokenSession(): SaSession {
    return this.stpLogic.getTokenSession();
  }

  // ... 其他代理方法
}
```

### 3.6 异常类设计

```typescript
// packages/core/src/exception/SaTokenException.ts

/** Sa-Token 基础异常 */
export class SaTokenException extends Error {
  public code: number;

  constructor(message: string);
  constructor(code: number, message: string);
  constructor(message: string, cause: Error);

  setCode(code: number): this;

  /** 断言为真 */
  static notTrue(flag: boolean, message: string, code?: number): void;

  /** 断言非空 */
  static notEmpty(value: unknown, message: string, code?: number): void;
}

// packages/core/src/exception/NotLoginException.ts

/** 未登录异常 */
export class NotLoginException extends SaTokenException {
  /** 未提供Token */
  public static readonly NOT_TOKEN = '-1';
  /** Token无效 */
  public static readonly INVALID_TOKEN = '-2';
  /** Token已过期 */
  public static readonly TOKEN_TIMEOUT = '-3';
  /** 已被顶下线 */
  public static readonly BE_REPLACED = '-4';
  /** 已被踢下线 */
  public static readonly KICK_OUT = '-5';
  /** Token已被冻结 */
  public static readonly TOKEN_FREEZE = '-6';
  /** 无前缀 */
  public static readonly NO_PREFIX = '-7';

  private readonly type: string;
  private readonly loginType: string;

  constructor(message: string, loginType: string, type: string);

  static newInstance(
    loginType: string,
    type: string,
    message: string,
    token?: string
  ): NotLoginException;
}

// packages/core/src/exception/NotPermissionException.ts

/** 无权限异常 */
export class NotPermissionException extends SaTokenException {
  private readonly permission: string;
  private readonly loginType: string;

  constructor(permission: string, loginType: string);
}

// packages/core/src/exception/NotRoleException.ts

/** 无角色异常 */
export class NotRoleException extends SaTokenException {
  private readonly role: string;
  private readonly loginType: string;

  constructor(role: string, loginType: string);
}
```

### 3.7 全局管理器 (SaManager)

```typescript
// packages/core/src/SaManager.ts

import type { SaTokenConfig } from './config/SaTokenConfig';
import type { SaTokenDao } from './dao/SaTokenDao';
import type { StpInterface } from './stp/StpInterface';
import type { StpLogic } from './stp/StpLogic';

/**
 * Sa-Token 全局管理器
 *
 * 管理所有核心组件的生命周期
 */
export class SaManager {
  private static _config: SaTokenConfig | null = null;
  private static _dao: SaTokenDao | null = null;
  private static _stpInterface: StpInterface | null = null;
  private static _stpLogicMap: Map<string, StpLogic> = new Map();

  // ===== 配置管理 =====

  static setConfig(config: SaTokenConfig): void {
    this._config = config;
  }

  static getConfig(): SaTokenConfig {
    if (!this._config) {
      throw new SaTokenException('SaTokenConfig 未初始化');
    }
    return this._config;
  }

  // ===== DAO 管理 =====

  static setSaTokenDao(dao: SaTokenDao): void {
    this._dao = dao;
    dao.init?.();
  }

  static getSaTokenDao(): SaTokenDao {
    if (!this._dao) {
      throw new SaTokenException('SaTokenDao 未初始化');
    }
    return this._dao;
  }

  // ===== 权限接口管理 =====

  static setStpInterface(stpInterface: StpInterface): void {
    this._stpInterface = stpInterface;
  }

  static getStpInterface(): StpInterface {
    if (!this._stpInterface) {
      throw new SaTokenException('StpInterface 未初始化');
    }
    return this._stpInterface;
  }

  // ===== StpLogic 管理 =====

  static putStpLogic(stpLogic: StpLogic): void {
    this._stpLogicMap.set(stpLogic.getLoginType(), stpLogic);
  }

  static removeStpLogic(loginType: string): void {
    this._stpLogicMap.delete(loginType);
  }

  static getStpLogic(loginType: string, isCreate = true): StpLogic {
    let stpLogic = this._stpLogicMap.get(loginType);
    if (!stpLogic && isCreate) {
      stpLogic = new StpLogic(loginType);
      this._stpLogicMap.set(loginType, stpLogic);
    }
    if (!stpLogic) {
      throw new SaTokenException(`StpLogic[${loginType}] 不存在`);
    }
    return stpLogic;
  }
}
```

---

## 四、分阶段实施计划

### 阶段 1：基础架构（第 1-2 周）

**目标**：搭建项目骨架，实现基础类型和配置

| 序号 | 任务 | 优先级 | 关键文件 |
|------|------|--------|----------|
| 1 | 初始化 monorepo 项目结构 | P0 | package.json, tsconfig.json |
| 2 | 配置 TypeScript、Jest、ESLint | P0 | tsconfig.base.json, jest.config.js |
| 3 | 实现常量定义 | P0 | SaTokenConsts.ts |
| 4 | 实现错误码 | P0 | SaErrorCode.ts |
| 5 | 实现工具类 | P0 | SaFoxUtil.ts |
| 6 | 实现所有异常类 | P0 | SaTokenException.ts, NotLoginException.ts 等 |
| 7 | 实现配置类和默认值 | P0 | SaTokenConfig.ts |
| 8 | 编写单元测试 | P1 | tests/unit/ |

**验收标准**：
- ✅ 所有异常类可正常实例化和抛出
- ✅ 配置类支持默认值和部分覆盖
- ✅ 工具类函数覆盖率达到 90%+

### 阶段 2：数据层（第 3-4 周）

**目标**：实现数据存储接口和内存实现

| 序号 | 任务 | 优先级 | 关键文件 |
|------|------|--------|----------|
| 1 | 定义 DAO 接口 | P0 | SaTokenDao.ts |
| 2 | 实现内存 DAO（支持过期清理） | P0 | SaTokenDaoMemory.ts |
| 3 | 实现 Session 模型 | P0 | SaSession.ts |
| 4 | 实现 TerminalInfo 模型 | P0 | SaTerminalInfo.ts |
| 5 | 实现序列化/反序列化 | P0 | SaSession.toJSON/fromJSON |
| 6 | 编写 DAO 和 Session 单元测试 | P1 | tests/unit/dao/, tests/unit/session/ |

**验收标准**：
- ✅ 内存 DAO 支持所有 CRUD 操作
- ✅ 过期数据自动清理功能正常
- ✅ Session 序列化/反序列化正确

### 阶段 3：核心认证逻辑（第 5-8 周）

**目标**：实现完整的登录认证功能

| 序号 | 任务 | 优先级 | 关键文件 |
|------|------|--------|----------|
| 1 | 实现 Token 生成策略（多种风格） | P0 | SaStrategy.ts |
| 2 | 实现 StpLogic 核心逻辑 | P0 | StpLogic.ts |
| 3 | 实现登录参数模型 | P0 | SaLoginParameter.ts |
| 4 | 实现注销参数模型 | P0 | SaLogoutParameter.ts |
| 5 | 实现 StpUtil 静态代理 | P0 | StpUtil.ts |
| 6 | 实现 SaManager 全局管理 | P0 | SaManager.ts |
| 7 | 实现 SaTokenContext 上下文管理 | P0 | SaTokenContext.ts |
| 8 | 编写集成测试 | P1 | tests/integration/ |

**验收标准**：
- ✅ 登录/注销功能完整
- ✅ Token 有效期管理正确
- ✅ 多设备登录处理正确
- ✅ 并发登录配置生效

### 阶段 4：权限与高级功能（第 9-10 周）

**目标**：实现权限验证、账号封禁等高级功能

| 序号 | 任务 | 优先级 | 关键文件 |
|------|------|--------|----------|
| 1 | 实现 StpInterface 权限接口 | P0 | StpInterface.ts |
| 2 | 实现权限/角色验证方法 | P0 | StpLogic.ts (权限方法) |
| 3 | 实现账号封禁功能 | P0 | StpLogic.ts (封禁方法) |
| 4 | 实现二级认证功能 | P1 | StpLogic.ts (安全方法) |
| 5 | 实现身份切换功能 | P1 | StpLogic.ts (切换方法) |
| 6 | 实现踢人/顶人下线 | P1 | StpLogic.ts |
| 7 | 编写单元测试 | P1 | tests/unit/stp/ |

**验收标准**：
- ✅ 权限验证正确处理 AND/OR 逻辑
- ✅ 封禁支持基础、分类、阶梯三种模式
- ✅ 二级认证按服务隔离

### 阶段 5：适配器开发（第 11-13 周）

**目标**：提供 Redis 存储适配器

| 序号 | 任务 | 优先级 | 关键文件 |
|------|------|--------|----------|
| 1 | 实现 Redis DAO 适配器 | P0 | packages/adapter-redis/SaTokenDaoRedis.ts |
| 2 | 支持 ioredis 和 redis 库 | P0 | SaTokenDaoRedis.ts |
| 3 | 编写集成测试 | P1 | tests/integration/redis/ |
| 4 | 编写使用示例 | P2 | examples/with-redis/ |

**验收标准**：
- ✅ Redis 适配器通过所有 DAO 测试
- ✅ 支持连接池配置
- ✅ 示例项目可运行

### 阶段 6：SSO 插件（第 14-16 周）

**目标**：实现单点登录功能

| 序号 | 任务 | 优先级 | 关键文件 |
|------|------|--------|----------|
| 1 | 实现 SSO 配置模型 | P0 | SaSsoServerConfig.ts, SaSsoClientConfig.ts |
| 2 | 实现 Ticket 生成与验证 | P0 | SaSsoTemplate.ts |
| 3 | 实现 SSO 服务端处理器 | P0 | SaSsoServerTemplate.ts |
| 4 | 实现 SSO 客户端处理器 | P0 | SaSsoClientTemplate.ts |
| 5 | 实现消息处理机制 | P1 | SaSsoProcessor.ts |
| 6 | 编写集成测试 | P1 | tests/integration/sso/ |

**验收标准**：
- ✅ 服务端可下发和验证 Ticket
- ✅ 客户端可完成单点登录流程
- ✅ 支持同域和跨域模式

### 阶段 7：OAuth2 插件（第 17-21 周）

**目标**：实现 OAuth2.0 授权服务器功能

| 序号 | 任务 | 优先级 | 关键文件 |
|------|------|--------|----------|
| 1 | 实现 OAuth2 配置模型 | P0 | SaOAuth2ServerConfig.ts |
| 2 | 实现数据模型（Token、Code） | P0 | AccessTokenModel.ts 等 |
| 3 | 实现 OAuth2 DAO | P0 | SaOAuth2Dao.ts |
| 4 | 实现数据加载器接口 | P1 | SaOAuth2DataLoader.ts |
| 5 | 实现授权码授权类型 | P0 | AuthorizationCodeGrantTypeHandler.ts |
| 6 | 实现密码授权类型 | P1 | PasswordGrantTypeHandler.ts |
| 7 | 实现刷新令牌授权类型 | P1 | RefreshTokenGrantTypeHandler.ts |
| 8 | 实现 Scope 处理器 | P1 | SaOAuth2ScopeHandler.ts |
| 9 | 实现 OIDC 扩展（可选） | P2 | - |
| 10 | 编写集成测试 | P1 | tests/integration/oauth2/ |

**验收标准**：
- ✅ 支持授权码、密码、客户端凭证、刷新令牌四种授权类型
- ✅ Token 有效期管理正确
- ✅ Scope 权限验证生效

---

## 五、关键技术决策

### 5.1 异步 vs 同步

Java 版本是同步设计，Node.js 版本的策略：

```typescript
// DAO 接口同时支持同步和异步
interface SaTokenDao {
  // 同步版本（内存实现）
  get(key: string): string | null;

  // 异步版本（Redis实现）
  getAsync?(key: string): Promise<string | null>;
}

// 核心逻辑保持同步风格
class StpLogic {
  login(id: string | number): void; // 同步

  // 但提供异步版本供异步DAO使用
  loginAsync?(id: string | number): Promise<void>;
}
```

### 5.2 Token 上下文管理

使用 Node.js `AsyncLocalStorage` 实现请求级别的 Token 上下文隔离：

```typescript
import { AsyncLocalStorage } from 'async_hooks';

const tokenContextStorage = new AsyncLocalStorage<Map<string, unknown>>();

export class SaTokenContext {
  /** 在上下文中执行回调 */
  static run<T>(callback: () => T): T {
    return tokenContextStorage.run(new Map(), callback);
  }

  /** 获取当前 Token 值 */
  static getTokenValue(): string | null {
    const store = tokenContextStorage.getStore();
    return store?.get('tokenValue') ?? null;
  }

  /** 设置当前 Token 值 */
  static setTokenValue(token: string): void {
    const store = tokenContextStorage.getStore();
    store?.set('tokenValue', token);
  }
}
```

### 5.3 序列化策略

使用标准 JSON 序列化，支持自定义序列化器：

```typescript
export interface SaSerializerTemplate {
  serialize<T>(object: T): string;
  deserialize<T>(json: string): T;
}

export class SaJsonSerializer implements SaSerializerTemplate {
  serialize<T>(object: T): string {
    return JSON.stringify(object, (key, value) => {
      // 处理特殊类型
      if (value instanceof Date) {
        return { __type: 'Date', value: value.toISOString() };
      }
      return value;
    });
  }

  deserialize<T>(json: string): T {
    return JSON.parse(json, (key, value) => {
      if (value?.__type === 'Date') {
        return new Date(value.value);
      }
      return value;
    });
  }
}
```

---

## 六、测试策略

### 6.1 单元测试

```typescript
// tests/unit/dao/SaTokenDaoMemory.test.ts
describe('SaTokenDaoMemory', () => {
  let dao: SaTokenDaoMemory;

  beforeEach(() => {
    dao = new SaTokenDaoMemory();
  });

  describe('字符串操作', () => {
    it('应该正确设置和获取字符串值', () => {
      dao.set('key1', 'value1', 100);
      expect(dao.get('key1')).toBe('value1');
    });

    it('过期后应该返回 null', async () => {
      dao.set('key1', 'value1', 1);
      await sleep(1100);
      expect(dao.get('key1')).toBeNull();
    });
  });
});
```

### 6.2 集成测试

```typescript
// tests/integration/login-flow.test.ts
describe('登录流程集成测试', () => {
  beforeEach(() => {
    SaManager.setConfig(createDefaultConfig());
    SaManager.setSaTokenDao(new SaTokenDaoMemory());
  });

  it('完整登录流程', () => {
    // 登录
    StpUtil.login(10001);

    // 验证登录状态
    expect(StpUtil.isLogin()).toBe(true);
    expect(StpUtil.getLoginId()).toBe(10001);

    // 注销
    StpUtil.logout();

    // 验证注销状态
    expect(StpUtil.isLogin()).toBe(false);
  });

  it('多设备登录', () => {
    StpUtil.login(10001, new SaLoginParameter().setDeviceType('PC'));
    const pcToken = StpUtil.getTokenValue();

    StpUtil.login(10001, new SaLoginParameter().setDeviceType('MOBILE'));
    const mobileToken = StpUtil.getTokenValue();

    expect(pcToken).not.toBe(mobileToken);
    expect(StpUtil.getTokenValueListByLoginId(10001).length).toBe(2);
  });
});
```

---

## 七、关键参考文件

移植过程中主要参考以下 Java 源文件：

| 模块 | Java 源文件 | 说明 |
|------|-------------|------|
| 核心逻辑 | `sa-token-core/.../stp/StpLogic.java` | 包含所有认证方法的具体实现，约2000行 |
| 配置类 | `sa-token-core/.../config/SaTokenConfig.java` | 40+ 配置项定义 |
| DAO 接口 | `sa-token-core/.../dao/SaTokenDao.java` | 数据存储抽象层 |
| Session | `sa-token-core/.../session/SaSession.java` | 会话模型，包含终端管理 |
| 全局管理器 | `sa-token-core/.../SaManager.java` | 组件生命周期管理 |
| SSO 模块 | `sa-token-plugin/sa-token-sso/` | 单点登录实现 |
| OAuth2 模块 | `sa-token-plugin/sa-token-oauth2/` | OAuth2 实现 |

---

## 八、风险与缓解措施

| 风险 | 影响程度 | 缓解措施 |
|------|----------|----------|
| Java 与 TypeScript 语言差异 | 中 | 保持核心 API 一致，适当调整命名风格（camelCase） |
| 异步编程模型差异 | 高 | 提供同步和异步两套 API，核心逻辑保持同步 |
| Redis 客户端库选择 | 低 | 支持 ioredis 和 redis 两个主流库 |
| Session 序列化兼容性 | 中 | 使用 JSON 标准，提供自定义序列化器接口 |
| 性能差异 | 中 | 进行性能基准测试，优化热点路径 |

---

## 九、使用示例

### 基础使用

```typescript
import { StpUtil, SaManager, SaTokenDaoMemory } from '@sa-token/core';

// 初始化
SaManager.setConfig({
  tokenName: 'satoken',
  timeout: 60 * 60 * 24 * 30, // 30天
});
SaManager.setSaTokenDao(new SaTokenDaoMemory());

// 登录
StpUtil.login(10001);
console.log('登录成功，Token:', StpUtil.getTokenValue());

// 验证登录
if (StpUtil.isLogin()) {
  console.log('当前登录用户:', StpUtil.getLoginId());
}

// 注销
StpUtil.logout();
```

### 权限验证

```typescript
import { StpUtil, SaManager, StpInterface } from '@sa-token/core';

// 实现权限接口
class MyStpInterface implements StpInterface {
  getPermissionList(loginId: string | number, loginType: string): string[] {
    // 从数据库获取权限列表
    return ['user:add', 'user:delete', 'user:update'];
  }

  getRoleList(loginId: string | number, loginType: string): string[] {
    // 从数据库获取角色列表
    return ['admin', 'user'];
  }
}

SaManager.setStpInterface(new MyStpInterface());

// 权限校验
StpUtil.login(10001);

if (StpUtil.hasPermission('user:add')) {
  console.log('有添加用户的权限');
}

StpUtil.checkPermission('user:delete'); // 无权限会抛异常
```

### Redis 存储

```typescript
import { SaManager } from '@sa-token/core';
import { SaTokenDaoRedis } from '@sa-token/adapter-redis';
import Redis from 'ioredis';

const redis = new Redis({
  host: 'localhost',
  port: 6379,
});

SaManager.setSaTokenDao(new SaTokenDaoRedis(redis));
```

---

## 十、时间规划总结

| 阶段 | 内容 | 周数 |
|------|------|------|
| 阶段 1 | 基础架构 | 2 周 |
| 阶段 2 | 数据层 | 2 周 |
| 阶段 3 | 核心认证逻辑 | 4 周 |
| 阶段 4 | 权限与高级功能 | 2 周 |
| 阶段 5 | Redis 适配器 | 3 周 |
| 阶段 6 | SSO 插件 | 3 周 |
| 阶段 7 | OAuth2 插件 | 5 周 |
| **总计** | | **21 周** |

---

*文档版本: 1.0*
*创建日期: 2026-03-09*
*基于 sa-token Java v1.45.0*