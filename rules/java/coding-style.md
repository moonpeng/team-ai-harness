# Java 编码风格规范

适用场景：新增或实质修改 Java 类、方法和数据对象结构，或调整、审查 Lombok、Javadoc 与注释。

仅约束新增代码及本次实质修改的部分，不顺带整改无关存量代码；明确要求规范化整改时按指定范围执行。

## Lombok 约束

在编写或修改 Java 实体类、DTO、VO 等结构时，必须严格遵守 Lombok 规范，禁止手动编写冗长的方法：

1. 必须使用注解：
    - Entity/DTO/VO 按需使用 Lombok 注解生成可等价替代的样板代码；不为统一风格额外引入 setter、字符串输出或相等性方法。
    - 需要无参或全参构造且不含额外校验时，分别使用 `@NoArgsConstructor` 或 `@AllArgsConstructor`；按需使用 `@Builder`，不得改变既有不可变性、构造校验或敏感字段输出限制。
2. 禁止手写样板代码：
    - 禁止手写可由 Lombok 等价生成的 `getter`、`setter`、`equals`、`hashCode` 和 `toString` 样板代码；承载业务语义或安全要求、无法由注解等价表达的实现除外。
3. 链式调用与安全：
    - DTO/VO 推荐开启链式调用 `@Accessors(chain = true)`。
    - 生成或自定义 `equals/hashCode` 时保留所需的继承语义，必要时使用 `@EqualsAndHashCode(callSuper = true)`。

## 注释

1. 所有类、抽象类和接口上方，必须使用 `/** ... */` (Javadoc) 格式编写类注释，说明类的职责、作者及创建意图。
2. 所有公开（public、protected）方法必须使用 `/** ... */` (Javadoc) 编写方法注释；有参数时包含对应的 @param，有返回值时包含 @return，有需要调用方了解的异常契约时说明异常，不添加不适用的标签。
3. 复杂算法、核心业务逻辑、状态判断或非常规写法，必须在代码行内使用 `//` 添加简明扼要的中文解释

新增类的作者信息取自 `Git config` 的 `user.name` 和 `user.email`；缺失时不编造，说明未填写原因，不为生成注释修改 Git 配置。修改已有类时保留原作者信息。
