
# OnlineBook-front
网上书店前端系统
# 在线图书商城管理系统

## 项目概述

在线图书商城管理系统是一个基于若依（RuoYi）前后端分离框架二次开发的综合性图书电商平台，旨在为用户提供完整的在线购书体验，同时为商家提供高效的图书管理与订单处理功能。系统包含超级管理员、商家和普通用户三种角色，支持图书浏览、购物车、下单支付、商家管理、数据统计等核心业务功能。

本项目为后端服务，基于 Spring Boot 4.x 构建，配套前端项目位于 [bookstore-frontend](../bookstore-frontend/vue-project)。

### 核心功能

- **商城端（用户）**：图书浏览与搜索、购物车管理、订单提交与付款、订单历史查询、个人信息管理、用户注册登录
- **商城端（商家）**：图书上架与管理、订单管理与发货、销售数据统计、店铺运营
- **系统管理端**：用户管理、部门管理、岗位管理、菜单权限管理、角色管理、字典管理、参数配置
- **系统监控**：在线用户监控、服务监控（CPU/内存/磁盘）、缓存监控、数据库连接池监控
- **特色功能**：代码生成器（一键生成前后端CRUD代码）、定时任务调度、JWT身份认证、数据权限隔离（商家仅可见自己图书的订单）、验证码登录、Excel导入导出

## 技术栈

### 后端核心
- Spring Boot 4.0.3（JDK 17+）
- Spring Security 安全框架
- MyBatis ORM框架 + PageHelper 分页插件
- MySQL 数据库 + Druid 连接池
- Redis 缓存
- JWT (JSON Web Token) 无状态身份认证
- Springdoc OpenAPI 3.0 接口文档

### 工具与组件
- Apache POI Excel处理
- Apache Velocity 代码模板引擎
- OSHI 服务器监控
- Kaptcha 验证码生成
- Fastjson2 JSON序列化
- Yauaa 客户端设备解析
- Lombok 代码简化
- Quartz 定时任务

### 开发工具
- Maven 依赖管理与多模块构建
- IntelliJ IDEA 集成开发环境
- Navicat / DBeaver 数据库管理

## 项目架构

### 整体架构设计

系统采用经典的前后端分离架构，通过 RESTful API 进行数据交互：

```
前端(Vue3/Element Plus) <--HTTP/JSON--> 后端(Spring Boot) <--MyBatis--> 数据库(MySQL)
                                                    ↕
                                               Redis(缓存/Session)
```

### 后端多模块架构

项目采用 Maven 多模块结构，各模块职责清晰：

```
ruoyi (父工程)
├── ruoyi-admin       # 管理后台 - Controller层、启动类、配置文件
├── ruoyi-common      # 通用模块 - 注解、工具类、基础实体、常量、枚举
├── ruoyi-framework   # 框架模块 - 安全认证、AOP切面、过滤器、配置
├── ruoyi-system      # 系统模块 - 图书/购物车/订单业务、Service/Mapper
├── ruoyi-generator   # 代码生成器 - 根据数据表生成前后端代码
└── ruoyi-quartz      # 定时任务 - Quartz任务调度管理
```

### 分层架构（以 ruoyi-system 为例）

- **Controller层**：接收HTTP请求，参数校验，返回统一响应
- **Service层**：业务逻辑处理，事务控制（如订单提交含库存扣减）
- **Mapper层**：数据持久化，MyBatis XML映射SQL
- **Domain层**：数据模型定义（Book、Order、Cart、OrderItem等）
- **DTO层**：数据传输对象（如SubmitOrderDTO）

### 数据流向

1. 用户在前端界面触发操作（如添加购物车、提交订单）
2. 前端 Axios 发送 HTTP 请求到后端 Controller
3. Security 过滤器验证 JWT Token 与用户权限
4. Controller 调用 Service 进行业务处理
5. Service 通过 Mapper 访问数据库（含事务控制）
6. 数据以 AjaxResult 统一格式返回前端渲染展示

### 认证机制

系统采用 Spring Security + JWT Token 进行身份验证：

- 用户登录成功后，后端生成 JWT Token 并返回给前端
- 前端将 Token 存储在 localStorage 中
- 后续请求在 Header 中携带 `Authorization: Bearer <token>`
- JwtAuthenticationTokenFilter 拦截请求并验证 Token 有效性
- @PreAuthorize 注解实现方法级权限控制
- 支持多终端认证，Token 有效期默认30分钟（可配置）
- 支持验证码登录（数学计算/字符验证两种模式）

### 数据权限隔离（商家维度）

- 商家登录后只能看到自己发布的图书及对应的订单
- 管理员可查看全部数据
- 通过 `create_by` 字段和 `SecurityUtils.getUsername()` 实现数据行级隔离

## 目录结构

```
RuoYi-Vue-master/
├── ruoyi-admin/                              # 管理后台模块（启动入口）
│   ├── src/main/java/com/ruoyi/
│   │   ├── RuoYiApplication.java             # Spring Boot 启动类
│   │   ├── RuoYiServletInitializer.java      # Servlet容器初始化（支持war部署）
│   │   └── web/
│   │       ├── controller/
│   │       │   ├── common/
│   │       │   │   ├── CaptchaController.java    # 验证码生成
│   │       │   │   └── CommonController.java     # 通用请求（文件上传/下载）
│   │       │   ├── monitor/
│   │       │   │   ├── CacheController.java      # 缓存监控
│   │       │   │   ├── ServerController.java     # 服务监控（CPU/内存/JVM）
│   │       │   │   ├── SysLogininforController.java  # 登录日志
│   │       │   │   ├── SysOperlogController.java     # 操作日志
│   │       │   │   └── SysUserOnlineController.java  # 在线用户
│   │       │   ├── shop/                          # 商城业务接口
│   │       │   │   ├── ShopBookController.java    # 商城图书列表/详情
│   │       │   │   ├── MerchantOrderController.java   # 商家订单管理
│   │       │   │   └── MerchantStatsController.java   # 商家数据统计
│   │       │   ├── system/                        # 系统管理接口
│   │       │   │   ├── SysLoginController.java    # 登录认证
│   │       │   │   ├── SysRegisterController.java # 用户注册
│   │       │   │   ├── SysUserController.java     # 用户管理
│   │       │   │   ├── SysRoleController.java     # 角色管理
│   │       │   │   ├── SysMenuController.java     # 菜单管理
│   │       │   │   ├── SysDeptController.java     # 部门管理
│   │       │   │   ├── SysPostController.java     # 岗位管理
│   │       │   │   ├── SysConfigController.java   # 参数配置
│   │       │   │   ├── SysDictDataController.java # 字典数据
│   │       │   │   ├── SysDictTypeController.java # 字典类型
│   │       │   │   ├── SysNoticeController.java   # 通知公告
│   │       │   │   ├── SysProfileController.java  # 个人信息
│   │       │   │   └── SysIndexController.java    # 首页
│   │       │   └── tool/
│   │       │       └── TestController.java        # 测试接口
│   │       └── core/config/
│   │           └── SwaggerConfig.java             # Swagger接口文档配置
│   ├── src/main/resources/
│   │   ├── application.yml                    # 主配置文件（端口/Redis/Token/MyBatis）
│   │   ├── application-druid.yml              # Druid数据源配置
│   │   ├── banner.txt                         # 启动横幅
│   │   ├── logback.xml                        # 日志配置
│   │   ├── i18n/messages.properties           # 国际化资源
│   │   ├── mybatis/mybatis-config.xml         # MyBatis全局配置
│   │   └── META-INF/spring-devtools.properties # 热部署配置
│   └── pom.xml
│
├── ruoyi-common/                              # 通用模块
│   └── src/main/java/com/ruoyi/common/
│       ├── annotation/                        # 自定义注解
│       │   ├── Anonymous.java                 # 匿名访问标记
│       │   ├── DataScope.java                 # 数据权限注解
│       │   ├── DataSource.java                # 多数据源注解
│       │   ├── Excel.java                     # Excel导入导出注解
│       │   ├── Log.java                       # 操作日志注解
│       │   ├── RateLimiter.java               # 限流注解
│       │   ├── RepeatSubmit.java              # 防重复提交注解
│       │   └── Sensitive.java                 # 敏感字段脱敏注解
│       ├── config/
│       │   └── RuoYiConfig.java               # RuoYi全局配置读取
│       ├── constant/                          # 常量定义
│       │   ├── Constants.java                 # 通用常量
│       │   ├── CacheConstants.java            # 缓存Key常量
│       │   ├── UserConstants.java             # 用户相关常量
│       │   ├── HttpStatus.java                # HTTP状态码
│       │   ├── GenConstants.java              # 代码生成器常量
│       │   └── ScheduleConstants.java         # 定时任务常量
│       ├── core/
│       │   ├── controller/BaseController.java # 控制器基类
│       │   ├── domain/
│       │   │   ├── AjaxResult.java            # 统一响应体（code/message/data）
│       │   │   ├── R.java                     # 通用响应封装
│       │   │   ├── BaseEntity.java            # 实体基类
│       │   │   ├── TreeEntity.java            # 树形实体基类
│       │   │   └── model/
│       │   │       ├── LoginBody.java         # 登录请求体
│       │   │       ├── LoginUser.java         # 登录用户信息
│       │   │       └── RegisterBody.java      # 注册请求体
│       │   ├── page/                          # 分页工具
│       │   │   ├── TableDataInfo.java         # 分页响应封装
│       │   │   ├── PageDomain.java            # 分页参数
│       │   │   └── TableSupport.java          # 分页支持
│       │   └── redis/RedisCache.java          # Redis缓存操作
│       ├── enums/                             # 枚举类
│       │   └── BusinessStatus.java            # 业务操作状态
│       ├── exception/                         # 异常定义
│       │   └── ServiceException.java          # 业务异常
│       ├── utils/                             # 通用工具类
│       │   ├── SecurityUtils.java             # 安全工具（获取当前用户）
│       │   └── DateUtils.java                 # 日期工具
│       └── xss/                               # XSS过滤
│
├── ruoyi-framework/                           # 框架核心模块
│   └── src/main/java/com/ruoyi/framework/
│       ├── aspectj/                           # AOP切面
│       │   ├── LogAspect.java                 # 操作日志切面
│       │   ├── DataScopeAspect.java           # 数据权限切面
│       │   ├── DataSourceAspect.java          # 多数据源切面
│       │   └── RateLimiterAspect.java         # 限流切面
│       ├── config/                            # 框架配置
│       │   ├── SecurityConfig.java            # Spring Security安全配置
│       │   ├── RedisConfig.java               # Redis配置
│       │   ├── DruidConfig.java               # Druid连接池配置
│       │   ├── MyBatisConfig.java             # MyBatis配置
│       │   ├── CaptchaConfig.java             # 验证码配置
│       │   ├── ResourcesConfig.java           # 资源映射配置
│       │   ├── FilterConfig.java              # 过滤器配置
│       │   ├── ThreadPoolConfig.java          # 线程池配置
│       │   ├── I18nConfig.java                # 国际化配置
│       │   └── properties/
│       │       ├── DruidProperties.java       # Druid属性配置
│       │       └── PermitAllUrlProperties.java # 匿名访问URL配置
│       ├── datasource/                        # 动态数据源
│       │   ├── DynamicDataSource.java         # 动态数据源实现
│       │   └── DynamicDataSourceContextHolder.java # 数据源上下文
│       ├── interceptor/                       # 拦截器
│       │   ├── RepeatSubmitInterceptor.java   # 防重复提交拦截器
│       │   └── impl/SameUrlDataInterceptor.java # 相同URL数据拦截
│       ├── manager/
│       │   ├── AsyncManager.java              # 异步任务管理器
│       │   ├── ShutdownManager.java           # 应用关闭管理器
│       │   └── factory/AsyncFactory.java      # 异步任务工厂
│       ├── security/                          # 安全认证
│       │   ├── context/
│       │   │   ├── AuthenticationContextHolder.java # 认证上下文
│       │   │   └── PermissionContextHolder.java     # 权限上下文
│       │   ├── filter/JwtAuthenticationTokenFilter.java # JWT认证过滤器
│       │   └── handle/
│       │       ├── AuthenticationEntryPointImpl.java  # 认证失败处理
│       │       └── LogoutSuccessHandlerImpl.java      # 退出成功处理
│       └── web/
│           ├── exception/GlobalExceptionHandler.java # 全局异常处理
│           ├── service/
│           │   ├── PermissionService.java      # 权限校验服务
│           │   ├── SysLoginService.java        # 登录服务
│           │   ├── SysRegisterService.java     # 注册服务
│           │   └── TokenService.java           # Token管理服务
│           └── domain/Server.java              # 服务器监控信息
│
├── ruoyi-system/                              # 系统业务模块
│   └── src/main/java/com/ruoyi/system/
│       ├── controller/                        # 前台业务接口
│       │   ├── BookController.java            # 图书管理CRUD（后台）
│       │   ├── CartController.java            # 购物车管理
│       │   └── OrderController.java           # 订单管理
│       ├── domain/                            # 数据实体
│       │   ├── Book.java                      # 图书实体（名称/作者/价格/库存/封面/分类）
│       │   ├── Cart.java                      # 购物车实体
│       │   ├── Order.java                     # 订单实体（编号/金额/状态/收货信息）
│       │   ├── OrderItem.java                 # 订单明细实体（价格快照）
│       │   ├── SysConfig.java                 # 参数配置实体
│       │   ├── SysNotice.java                 # 通知公告实体
│       │   ├── SysNoticeRead.java             # 通知已读实体
│       │   ├── SysLogininfor.java             # 登录日志实体
│       │   ├── SysOperLog.java                # 操作日志实体
│       │   ├── SysPost.java                   # 岗位实体
│       │   ├── SysRoleDept.java               # 角色部门关联
│       │   ├── SysRoleMenu.java               # 角色菜单关联
│       │   ├── SysUserOnline.java             # 在线用户实体
│       │   ├── SysUserPost.java               # 用户岗位关联
│       │   ├── SysUserRole.java               # 用户角色关联
│       │   ├── dto/
│       │   │   └── SubmitOrderDTO.java        # 提交订单请求DTO
│       │   └── vo/
│       │       ├── RouterVo.java              # 路由配置VO
│       │       └── MetaVo.java                # 路由元信息VO
│       ├── mapper/                            # MyBatis Mapper接口
│       │   ├── BookMapper.java                # 图书数据访问
│       │   ├── CartMapper.java                # 购物车数据访问
│       │   ├── OrderMapper.java               # 订单数据访问
│       │   ├── OrderItemMapper.java           # 订单明细数据访问
│       │   ├── SysUserMapper.java             # 用户数据访问
│       │   ├── SysRoleMapper.java             # 角色数据访问
│       │   ├── SysMenuMapper.java             # 菜单数据访问
│       │   ├── SysDeptMapper.java             # 部门数据访问
│       │   ├── SysPostMapper.java             # 岗位数据访问
│       │   ├── SysConfigMapper.java           # 参数配置数据访问
│       │   ├── SysDictDataMapper.java         # 字典数据访问
│       │   ├── SysDictTypeMapper.java         # 字典类型访问
│       │   ├── SysNoticeMapper.java           # 通知公告数据访问
│       │   ├── SysNoticeReadMapper.java       # 通知已读数据访问
│       │   ├── SysLogininforMapper.java       # 登录日志数据访问
│       │   ├── SysOperLogMapper.java          # 操作日志数据访问
│       │   ├── SysUserPostMapper.java         # 用户岗位关联
│       │   ├── SysUserRoleMapper.java         # 用户角色关联
│       │   ├── SysRoleDeptMapper.java         # 角色部门关联
│       │   └── SysRoleMenuMapper.java         # 角色菜单关联
│       ├── service/                           # 业务逻辑接口
│       │   ├── IBookService.java              # 图书业务接口
│       │   ├── IOrderService.java             # 订单业务接口
│       │   ├── ISysUserService.java           # 用户业务接口
│       │   ├── ISysRoleService.java           # 角色业务接口
│       │   ├── ISysMenuService.java           # 菜单业务接口
│       │   ├── ISysDeptService.java           # 部门业务接口
│       │   ├── ISysPostService.java           # 岗位业务接口
│       │   ├── ISysConfigService.java         # 参数配置业务接口
│       │   ├── ISysDictDataService.java       # 字典数据业务接口
│       │   ├── ISysDictTypeService.java       # 字典类型业务接口
│       │   ├── ISysNoticeService.java         # 通知公告业务接口
│       │   ├── ISysNoticeReadService.java     # 通知已读业务接口
│       │   ├── ISysLogininforService.java     # 登录日志业务接口
│       │   ├── ISysOperLogService.java        # 操作日志业务接口
│       │   └── ISysUserOnlineService.java     # 在线用户业务接口
│       └── service/impl/                      # 业务逻辑实现
│           ├── BookServiceImpl.java           # 图书CRUD实现
│           ├── OrderServiceImpl.java          # 订单业务实现（提交/付款/取消/发货/完成）
│           ├── SysUserServiceImpl.java        # 用户业务实现
│           ├── SysRoleServiceImpl.java        # 角色业务实现
│           ├── SysMenuServiceImpl.java        # 菜单业务实现
│           ├── SysDeptServiceImpl.java        # 部门业务实现
│           ├── SysPostServiceImpl.java        # 岗位业务实现
│           ├── SysConfigServiceImpl.java      # 参数配置业务实现
│           ├── SysDictDataServiceImpl.java    # 字典数据业务实现
│           ├── SysDictTypeServiceImpl.java    # 字典类型业务实现
│           ├── SysNoticeServiceImpl.java      # 通知公告业务实现
│           ├── SysNoticeReadServiceImpl.java  # 通知已读业务实现
│           ├── SysLogininforServiceImpl.java  # 登录日志业务实现
│           ├── SysOperLogServiceImpl.java     # 操作日志业务实现
│           └── SysUserOnlineServiceImpl.java  # 在线用户业务实现
│   └── src/main/resources/mapper/system/
│       ├── BookMapper.xml                     # 图书SQL映射（CRUD、库存扣减）
│       ├── CartMapper.xml                     # 购物车SQL映射
│       ├── OrderMapper.xml                    # 订单SQL映射
│       ├── OrderItemMapper.xml                # 订单明细SQL映射
│       ├── SysUserMapper.xml                  # 用户SQL映射
│       ├── SysRoleMapper.xml                  # 角色SQL映射
│       ├── SysMenuMapper.xml                  # 菜单SQL映射
│       ├── SysDeptMapper.xml                  # 部门SQL映射
│       └── ...                                # 其他系统表XML映射
│
├── ruoyi-generator/                           # 代码生成器模块
│   └── src/main/java/com/ruoyi/generator/
│       ├── config/GenConfig.java              # 生成器配置
│       ├── controller/GenController.java      # 代码生成控制器
│       ├── domain/
│       │   ├── GenTable.java                  # 业务表信息
│       │   └── GenTableColumn.java            # 业务表字段信息
│       ├── mapper/
│       │   ├── GenTableMapper.java            # 业务表数据访问
│       │   └── GenTableColumnMapper.java      # 字段数据访问
│       ├── service/
│       │   ├── IGenTableService.java          # 代码生成接口
│       │   └── impl/GenTableServiceImpl.java  # 代码生成实现
│       └── util/
│           ├── GenUtils.java                  # 代码生成工具
│           ├── VelocityUtils.java             # Velocity模板工具
│           └── VelocityInitializer.java       # Velocity初始化
│
├── ruoyi-quartz/                              # 定时任务模块
│   └── src/main/java/com/ruoyi/quartz/
│       ├── config/ScheduleConfig.java         # 调度配置
│       ├── controller/
│       │   ├── SysJobController.java          # 定时任务管理
│       │   └── SysJobLogController.java       # 任务日志管理
│       ├── domain/
│       │   ├── SysJob.java                    # 定时任务实体
│       │   └── SysJobLog.java                 # 任务日志实体
│       ├── mapper/
│       │   ├── SysJobMapper.java              # 任务数据访问
│       │   └── SysJobLogMapper.java           # 任务日志数据访问
│       ├── service/
│       │   ├── ISysJobService.java            # 任务业务接口
│       │   ├── ISysJobLogService.java         # 任务日志业务接口
│       │   └── impl/
│       │       ├── SysJobServiceImpl.java     # 任务业务实现
│       │       └── SysJobLogServiceImpl.java  # 任务日志业务实现
│       ├── task/RyTask.java                   # 系统内置任务
│       └── util/
│           ├── AbstractQuartzJob.java         # Quartz任务抽象类
│           ├── CronUtils.java                 # Cron表达式工具
│           ├── JobInvokeUtil.java             # 任务调用工具
│           ├── QuartzJobExecution.java        # 任务执行（允许并发）
│           ├── QuartzDisallowConcurrentExecution.java # 任务执行（禁止并发）
│           └── ScheduleUtils.java             # 调度工具类
│
├── ruoyi-ui/                                  # 前端Vue项目（RuoYi原有前端）
│   └── src/
│       ├── views/                             # 页面视图
│       ├── api/                               # API接口封装
│       ├── router/                            # 路由配置
│       ├── store/                             # Vuex状态管理
│       ├── components/                        # 公共组件
│       └── utils/                             # 工具函数
│
├── sql/                                       # 数据库脚本
│   ├── ry_20260417.sql                        # 若依系统表结构+初始数据
│   └── quartz.sql                             # Quartz定时任务表结构
├── bookstore.sql                              # 图书商城业务表+初始数据
├── merchant_fix.sql                           # 商家数据修复脚本
├── merchant_isolation.sql                     # 商家数据隔离SQL
├── merchant_menu_update.sql                   # 商家菜单更新SQL
├── book_category_update.sql                   # 图书分类更新SQL
├── pom.xml                                    # Maven父工程配置
├── ry.bat                                     # Windows快速启动脚本
├── ry.sh                                      # Linux快速启动脚本
└── doc/
    └── 若依环境使用手册.docx                    # 环境使用手册
```

## 核心文件说明

### 项目入口和配置文件

**后端入口**：`ruoyi-admin/src/main/java/com/ruoyi/RuoYiApplication.java`
- Spring Boot 应用启动类
- `@SpringBootApplication(exclude = {DataSourceAutoConfiguration.class})` 排除默认数据源，使用 Druid 动态数据源
- 内置 Tomcat 服务器，默认监听 8080 端口
- 启动成功打印若依 ASCII Art 横幅

**主要配置文件**：

| 文件 | 说明 |
|------|------|
| `application.yml` | 主配置（端口8080、Redis连接、JWT密钥/有效期、MyBatis、文件上传、验证码类型、XSS防护等） |
| `application-druid.yml` | Druid 数据源配置（数据库连接、连接池参数、监控页面） |
| `logback.xml` | 日志配置（控制台输出、文件滚动、日志级别） |
| `mybatis/mybatis-config.xml` | MyBatis 全局配置（驼峰映射、缓存等） |

### 核心业务逻辑实现

**订单服务**：`OrderServiceImpl.java`
- `submitOrder()`：提交订单的核心事务方法
  1. 查询购物车中要结算的商品
  2. 校验库存是否充足
  3. 计算订单总金额
  4. 生成订单编号（yyyyMMddHHmmss + 4位随机数）
  5. 创建订单记录，状态设为"0-待付款"
  6. 批量插入订单明细（含价格快照，防止后续价格变动影响）
  7. 扣减图书库存
  8. 删除已结算的购物车记录
- `cancelOrder()`：取消订单（仅待付款可取消，恢复库存）
- `payOrder()`：模拟付款（状态: 0→1）
- `shipOrder()`：商家发货（状态: 1→2）
- `completeOrder()`：完成订单（状态: 2→3）
- `deleteOrder()`：删除订单及关联明细

**购物车服务**：`CartController.java`
- 添加商品时自动合并相同图书的数量
- 查询购物车列表时关联图书名称、价格、封面信息
- 支持单条删除购物车记录

**图书服务**：`BookServiceImpl.java`
- 标准的 CRUD 操作（增删改查）
- 支持按分类、关键词模糊搜索
- 创建图书时自动填充 `create_by` 字段（商家用户名），实现数据隔离

**商家订单管理**：`MerchantOrderController.java`
- 商家仅能查看自己图书产生的订单
- 支持订单发货、完成等状态流转
- 数据行级隔离：通过 `getMerchantBookIds()` 获取当前商家发布的图书ID列表

**商家数据统计**：`MerchantStatsController.java`
- 统计当前商家的图书数量、订单数量、销售额
- admin 角色可查看全局统计数据

**认证与授权**：`SecurityConfig.java` + `JwtAuthenticationTokenFilter.java`
- Spring Security 配置：放行匿名URL（登录、注册、验证码、商城图书列表等）
- JWT 过滤器：从请求头提取 Token，验证有效性，设置 SecurityContext
- `@PreAuthorize("@ss.hasPermi('xxx')")` 实现细粒度方法级权限控制
- `PermissionService` 提供权限校验服务

### 数据模型和API接口

**核心数据模型**：

`Book.java` - 图书实体
- 基本信息：bookId、bookName（图书名称）、author（作者）、price（价格）、stock（库存）
- 展示信息：cover（封面URL）、description（描述）、category（分类）
- 运营字段：createBy（创建者/商家）、createTime（创建时间）

`Cart.java` - 购物车实体
- cartId（购物车ID）、userId（用户ID）、bookId（图书ID）、quantity（数量）
- 关联展示字段：bookName、price、cover、stock（通过SQL JOIN查询填充）

`Order.java` - 订单实体
- 订单信息：orderId、orderNo（订单编号）、userId、totalPrice（总金额）
- 状态流转：status（0-待付款 / 1-已付款 / 2-已发货 / 3-已完成 / 4-已取消）
- 收货信息：receiverName、receiverPhone、receiverAddress
- 关联列表：orderItems（订单明细列表，通过 orderItemMapper 关联查询）

`OrderItem.java` - 订单明细（价格快照）
- itemId、orderId、bookId
- 快照字段：bookName、cover、price（下单时的价格）、quantity、totalPrice

`SubmitOrderDTO.java` - 提交订单请求
- receiverName、receiverPhone、receiverAddress（收货信息）
- cartIds（要结算的购物车项ID列表）

**API 接口设计**：

所有接口遵循 RESTful 风格，统一返回 AjaxResult 格式：
```json
{
  "code": 200,
  "msg": "操作成功",
  "data": {...}
}
```

分页数据返回 TableDataInfo 格式：
```json
{
  "code": 200,
  "msg": "查询成功",
  "rows": [...],
  "total": 100
}
```

**商城业务接口（/shop/**）**：

| 接口 | 方法 | 说明 | 认证 |
|------|------|------|------|
| `/shop/book/list` | GET | 图书列表（支持分类/关键词筛选） | 匿名 |
| `/shop/book/{bookId}` | GET | 图书详情 | 匿名 |
| `/shop/cart/add` | POST | 添加购物车（已有则合并数量） | 需登录 |
| `/shop/cart/list` | GET | 购物车列表 | 需登录 |
| `/shop/cart/delete/{cartId}` | DELETE | 删除购物车记录 | 需登录 |
| `/shop/order/submit` | POST | 提交订单（库存校验+价格快照） | 需登录 |
| `/shop/order/list` | GET | 我的订单列表 | 需登录 |
| `/shop/order/{orderId}` | GET | 订单详情（含明细） | 需登录 |
| `/shop/order/cancel/{orderId}` | PUT | 取消订单（恢复库存） | 需登录 |
| `/shop/order/pay/{orderId}` | PUT | 模拟付款 | 需登录 |
| `/shop/order/{orderId}` | DELETE | 删除订单 | 需登录 |

**商家管理接口（/merchant/**）**：

| 接口 | 方法 | 说明 | 权限 |
|------|------|------|------|
| `/merchant/order/list` | GET | 商家订单列表（仅自己图书的订单） | `order:manage:list` |
| `/merchant/order/ship/{orderId}` | PUT | 发货 | `order:manage:ship` |
| `/merchant/stats/summary` | GET | 商家数据统计 | 商家/管理员 |

**系统管理接口（/system/**）**：

| 接口分类 | 主要接口 | 说明 |
|----------|----------|------|
| 用户管理 | `/system/user/**` | 用户CRUD、重置密码、状态修改、导出 |
| 角色管理 | `/system/role/**` | 角色CRUD、权限分配、数据范围设置 |
| 菜单管理 | `/system/menu/**` | 菜单树查询、新增/修改/删除（支持按钮权限） |
| 部门管理 | `/system/dept/**` | 部门树管理 |
| 岗位管理 | `/system/post/**` | 岗位CRUD |
| 字典管理 | `/system/dict/**` | 字典类型CRUD、字典数据CRUD |
| 参数配置 | `/system/config/**` | 系统参数增删改查 |
| 通知公告 | `/system/notice/**` | 公告发布、已读标记 |
| 登录认证 | `/system/login`、`/system/register` | 登录、注册、验证码、获取用户信息/路由 |

**系统监控接口（/monitor/**）**：

| 接口 | 说明 |
|------|------|
| `/monitor/server` | 服务器信息（CPU/内存/JVM/磁盘） |
| `/monitor/cache/**` | Redis 缓存监控 |
| `/monitor/logininfor/**` | 登录日志 |
| `/monitor/operlog/**` | 操作日志 |
| `/monitor/online/**` | 在线用户管理（强退） |

### 关键组件和服务模块

**代码生成器**：`ruoyi-generator/`
- 读取数据库表结构，使用 Apache Velocity 模板引擎生成代码
- 一键生成：Entity、Mapper、Service、Controller、Vue页面、SQL脚本
- 支持 CRUD 全部功能，大幅提升开发效率

**定时任务**：`ruoyi-quartz/`
- 基于 Quartz 实现，支持 Cron 表达式
- 在线管理：新增/修改/暂停/恢复/删除任务
- 支持并

发执行和禁止并发两种模式
- 任务执行日志记录与查询

**AOP 切面增强**：
- `LogAspect.java`：记录操作日志（基于 @Log 注解）
- `DataScopeAspect.java`：数据权限过滤（基于 @DataScope 注解）
- `RateLimiterAspect.java`：接口限流（基于 @RateLimiter 注解）
- `DataSourceAspect.java`：动态数据源切换（基于 @DataSource 注解）

**全局功能**：
- XSS 脚本过滤（可配置排除路径）
- 文件上传/下载（本地存储 + 虚拟路径映射）
- 国际化支持（i18n）
- 验证码（数学计算 / 字符验证两种模式）
- 密码加密（BCrypt）
- 防重复提交（基于 Redis 缓存 + 拦截器）

### 技术亮点

1. **Spring Boot 4.x + JDK 17+**：采用最新版 Spring Boot，享受虚拟线程等新特性
2. **JWT 无状态认证**：支持分布式部署，无需 Session 同步，多终端兼容
3. **RBAC 权限模型**：用户-角色-菜单-部门，支持按钮级权限控制和数据行级隔离
4. **动态数据源**：支持多数据源运行时切换，Druid 连接池监控
5. **代码生成器**：Velocity 模板引擎一键生成前后端 CRUD 全套代码
6. **商家数据隔离**：基于 `create_by` 字段实现商家仅管理自己的图书和订单
7. **订单状态机**：待付款→已付款→已发货→已完成，取消/恢复库存的事务完整性
8. **价格快照机制**：订单明细存储下单时的价格快照，防止历史订单受价格变动影响
9. **库存并发安全**：使用 `UPDATE ... SET stock = stock - ? WHERE stock >= ?` SQL 保证库存扣减原子性
10. **防重复提交**：基于 Redis 缓存 + 注解驱动，防止表单重复提交
11. **操作日志**：AOP + 异步方式记录操作日志，不阻塞主业务流程
12. **服务监控**：集成 OSHI 实时监控服务器 CPU/内存/JVM/磁盘状态

## 运行环境要求

| 组件 | 版本要求 |
|------|----------|
| JDK | 17+ |
| MySQL | 8.0+ (或 MariaDB 10.3+) |
| Redis | 5.0+ |
| Maven | 3.6+ |
| Node.js | 16+ (前端项目需要) |

## 启动说明

### 1. 环境准备

确保已安装 JDK 17+、MySQL、Redis、Maven，并启动 MySQL 和 Redis 服务。

### 2. 导入数据库

依次执行以下 SQL 脚本：

```bash
# 1. 导入若依系统表结构（用户/角色/菜单/部门等）
mysql -u root -p < sql/ry_20260417.sql

# 2. 导入 Quartz 定时任务表结构
mysql -u root -p < sql/quartz.sql

# 3. 导入图书商城业务表和数据
mysql -u root -p < bookstore.sql

# 4. 执行商家相关更新脚本
mysql -u root -p < merchant_fix.sql
mysql -u root -p < merchant_isolation.sql
mysql -u root -p < merchant_menu_update.sql
mysql -u root -p < book_category_update.sql
```

### 3. 修改配置

编辑 `ruoyi-admin/src/main/resources/application-druid.yml`，修改数据库连接信息：

```yaml
spring:
  datasource:
    druid:
      master:
        url: jdbc:mysql://localhost:3306/ry?useUnicode=true&characterEncoding=utf8&useSSL=false
        username: root          # 修改为你的数据库用户名
        password: your_password # 修改为你的数据库密码
```

编辑 `ruoyi-admin/src/main/resources/application.yml`，确认 Redis 配置：

```yaml
spring:
  data:
    redis:
      host: localhost
      port: 6379
      password:                # 如有密码请填写
```

### 4. 导入项目并启动

**IDE 方式（推荐）**：
1. 使用 IntelliJ IDEA 打开项目根目录（`RuoYi-Vue-master/`）
2. 等待 Maven 依赖下载完成
3. 找到 `ruoyi-admin/src/main/java/com/ruoyi/RuoYiApplication.java`
4. 右键运行 `main` 方法
5. 控制台看到若依启动横幅即启动成功

**命令行方式**：
```bash
cd RuoYi-Vue-master
mvn clean install -DskipTests
cd ruoyi-admin
mvn spring-boot:run
```

### 5. 访问地址

- **后端服务**：http://localhost:8080
- **Swagger 接口文档**：http://localhost:8080/swagger-ui.html
- **Druid 监控面板**：http://localhost:8080/druid（需登录，账号密码见 Druid 配置）

### 6. 前端启动

详见前端项目 README：[bookstore-frontend](../bookstore-frontend/vue-project)

```bash
cd bookstore-frontend/vue-project
npm install
npm run dev
```

## 默认账号

| 角色 | 用户名 | 密码 | 说明 |
|------|--------|------|------|
| 超级管理员 | admin | admin123 | 拥有全部权限 |
| 商家 | ry | admin123 | 可管理自己发布的图书和订单 |
| 普通用户 | 自行注册 | — | 浏览图书、购物下单 |

## 项目来源

本项目后端基于 **RuoYi-Vue v3.9.2**（Spring Boot 4.x 分支）二次开发，在原框架基础上新增了图书商城业务模块，包括图书管理、购物车、订单系统、商家数据隔离等功能。

- RuoYi 官网：http://ruoyi.vip
- RuoYi-Vue 仓库：https://gitee.com/y_project/RuoYi-Vue
- RuoYi 文档：http://doc.ruoyi.vip
