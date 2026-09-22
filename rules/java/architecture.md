# Java 架构规范

适用场景：新增、调整或审查模块、包位置、分层依赖、Controller/Service 契约，以及对象职责与命名。

仅约束新增代码及本次实质修改的部分，不顺带整改无关存量代码；明确要求规范化整改时按指定范围执行。

## 项目架构

严格遵守 Controller -> Service (Interface + Impl) -> Mapper -> Entity 的分层。Controller 不直接调用 Mapper；Service 负责业务逻辑，Mapper 负责数据访问。

## 项目目录

```text
abu-aos-ai/
├── pom.xml                         # Maven 聚合与依赖管理
├── abu-aos-ai-api/                 # 对外契约模块
│   ├── pom.xml
│   └── src/main/java/com/zt/abu/aos/ai/
│       ├── enums/                  # 契约使用的枚举
│       ├── feign/                  # 远程调用接口
│       ├── query/                  # 查询条件
│       ├── request/                # 请求对象
│       └── response/               # 响应对象
└── abu-aos-ai-provider/            # 服务实现模块
    ├── pom.xml
    └── src/
        ├── main/
        │   ├── java/com/zt/abu/aos/ai/
        │   │   ├── AIApplication.java  # Spring Boot 启动入口
        │   │   ├── annotation/         # 自定义注解
        │   │   ├── aspect/             # 切面
        │   │   ├── config/             # 框架与组件配置
        │   │   ├── constant/           # 内部常量
        │   │   ├── controller/         # HTTP 入口
        │   │   ├── dto/                # 服务内部传输对象
        │   │   ├── entity/             # 持久化实体，按业务域组织，类名以 Entity 结尾
        │   │   ├── mapper/             # MyBatis Mapper 接口
        │   │   ├── properties/         # 配置属性绑定
        │   │   ├── rabbitmq/
        │   │   │   ├── consumer/       # 消息消费
        │   │   │   └── producer/       # 消息生产
        │   │   ├── service/            # 业务接口与实现
        │   │   └── struct/             # 对象映射（MapStruct）
        │   └── resources/
        │       ├── bootstrap.properties
        │       ├── logback-spring.xml
        │       ├── META-INF/
        │       ├── i18n/
        │       └── mapper/         # 按业务域组织的 SQL 映射 XML
        └── test/java/com/zt/abu/aos/
            ├── common/            # 遗留公共工具测试
            └── dcs/               # 业务与工具测试
```

## 对象职责与命名

- `Request`：接收业务写入参数，放在 API 模块的 `request.<业务域>`。
- `Query`：接收查询条件，放在 API 模块的 `query.<业务域>`。
- `Response`：定义对外返回字段，放在 API 模块的 `response.<业务域>`。
- `DTO`：承载 Service 的输入、输出及内部业务数据，放在 Provider 模块的 `dto.<业务域>`。
- `Entity`：对应持久化数据，放在 Provider 模块的 `entity.<业务域>`；类名统一以 `Entity` 结尾，不使用 `PO` 后缀或 `entity.po` 包层级。

DTO 按业务用途定义，避免一个全字段 DTO 同时承担写入、查询和返回职责；结构与语义一致时可以复用。

## 分层与转换边界

- Controller 完成入口参数校验，将 `Request`、`Query` 转换为对应 DTO 后调用 Service；将返回 DTO 转换为 `Response`，再按统一响应规范包装。
- Service 接收 DTO 进行业务处理，持久化时将 DTO 转换为 Entity；查询得到 Entity 后，将其转换为 DTO 返回。
- Controller 不接触 Entity；Service 的业务接口不使用 `Request`、`Query`、`Response` 或 `ResultVo`，也不向 Controller 暴露 Entity。
- 查询 DTO 不必转换为 Entity，可由 Service 提取查询参数或传给 Mapper。主键、计数等简单参数与结果无需强制包装成 DTO；无返回数据的操作不构造空 DTO。
- 集合、分页和嵌套对象遵循相同边界，不能在 DTO 或 Response 中夹带 Entity。
- 不得为了复用框架 CRUD 接口而破坏上述对象和分层边界。

## 统一响应规范

1. Controller 的 HTTP 接口方法必须使用 `com.zt.digital.common.api.dto.response.ResultVo<T>` 作为返回类型，并明确具体泛型；不得直接返回业务对象、集合或另建统一响应包装类。内部辅助方法不受此限制。
2. 成功且有业务数据时使用 `ResultVo.success(data)`；无业务数据时返回 `ResultVo<Void>`，使用 `ResultVo.success()`。
3. 异常沿用项目既有异常处理机制，不为包装返回值而统一捕获异常，也不得将失败结果包装为成功。
