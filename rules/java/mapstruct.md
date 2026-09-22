# MapStruct 对象转换规范

适用场景：新增、修改或审查对象映射，以及会影响映射的源/目标字段、类型、空值或可写范围。

仅约束新增代码及本次实质修改的部分，不顺带整改无关存量代码；明确要求规范化整改时按指定范围执行。

对象职责与层间边界由 [架构规范](architecture.md) 定义；本文件仅规定映射实现，无需因读取本文件而全文加载其他规范。

## 映射实现

- 对象结构映射统一使用 MapStruct，接口放在 Provider 模块的 `struct.<业务域>`，命名为 `<业务名称>Converter`；沿用 `Mappers.getMapper(...)` 获取 `INSTANCE`。
- Controller 调用接口对象与 DTO 之间的转换，Service 调用 DTO 与 Entity 之间的转换。不使用 `BeanUtils.copyProperties`、JSON 序列化反序列化或散落的逐字段复制代替结构映射；单个业务字段赋值不受此限制。
- Converter 仅负责字段映射及必要的纯数据转换，不查询数据库、调用远程服务、获取登录身份或执行业务校验。
- 新增、修改分别定义转换方法，如 `toCreateDTO`、`toUpdateDTO`、`toCreateEntity`、`toUpdateEntity`；输出转换使用 `toDTO`、`toResponse`，集合转换复用对应单对象映射。

## 字段与空值边界

- 写入链路的 `Request → DTO`、`DTO → Entity` 使用 `@BeanMapping(ignoreByDefault = true)` 和显式 `@Mapping`，仅映射本次操作允许写入的字段。
- 主键取自经过校验的参数；审计身份、不可编辑字段及数据库生成字段遵循持久化写入契约，不从客户端对象隐式复制受保护字段，不在转换器中擅自补值。
- 读取与响应映射允许语义一致的同名字段自动映射；名称、类型或语义不一致时显式声明。Response 只包含接口约定的对外字段。
- 明确单对象、集合和嵌套映射的 null 与空集合契约。写入转换须保留“清空字段”与“不修改字段”的业务区别，不以统一忽略 null 的策略替代业务判断。

涉及实际更新行为时，按 [持久化规范](mybatis-plus.md) 检查映射白名单与更新字段是否一致。
