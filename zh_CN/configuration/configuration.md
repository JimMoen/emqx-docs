# 配置文件简介

EMQX 支持通过修改配置文件或使用环境变量来设置 EMQX。本章节主要介绍了 EMQX 配置文件的基本信息，同时提供了 EMQX 中常用功能的基本配置指导。想要了解全面的配置项以及配置项的详细介绍，请参考 [EMQX 开源版配置手册](https://docs.emqx.com/zh/emqx/v@CE_VERSION@/hocon/)和 [EMQX 企业版配置手册](https://docs.emqx.com/zh/enterprise/v@EE_VERSION@/hocon/)。

## 配置目录

安装 EMQX 后，它会创建一组目录来管理其配置和运行时数据。这些目录分为两大类：

- **静态配置目录（`etc`）**：只读，包含不可变或静态的配置文件。
- **动态配置目录（`data/configs`）**：可写，存储运行时生成或动态更新的配置文件。

### 静态配置目录（`etc`）

`etc` 目录包含定义 EMQX 初始设置的配置文件。这些文件通常在部署或升级时进行修改，并在运行时是只读的，以确保系统稳定性。`etc` 目录的位置取决于安装方式：

| 安装方式               | 路径            |
| ---------------------- | --------------- |
| 使用 RPM 或 DEB 包安装 | `/etc/emqx`     |
| 在 Docker 容器中运行   | `/opt/emqx/etc` |
| 从便携式压缩包中提取   | `./etc`         |

### 动态配置目录（`data/configs`）

在运行时，EMQX 允许通过 Dashboard、REST API 或 CLI 进行动态重新配置。通过这些工具进行的更改将存储在 `data/configs` 目录中，以确保跨会话的数据持久性。该目录的位置同样取决于安装方式：

| 安装方式               | 路径                     |
| ---------------------- | ------------------------ |
| 使用 RPM 或 DEB 包安装 | `/var/lib/emqx/configs`  |
| 在 Docker 容器中运行   | `/opt/emqx/data/configs` |
| 从便携式压缩包中提取   | `./data/configs`         |

::: tip

可以通过修改配置中的 `node.data_dir` 设置或 `EMQX_NODE__DATA_DIR` 环境变量来更改数据目录。但是，在运行集群时，所有节点必须使用相同的目录路径。 

:::

尽管不推荐这样做，配置文件的内容可以重叠。如果发生重叠，冲突将根据预定义的覆盖规则解决，详情请见 [配置覆盖规则](#配置覆盖规则)。

## 配置示例

虽然 [Schema](#schema) 部分提供了详细的参考信息，但配置示例有助于理解和应用 EMQX 的设置。你可以在 `etc/examples` 目录中找到多个配置示例。

## 基础配置文件

从 EMQX 5.8.4 开始，`etc` 目录中新增了一个名为 `base.hocon` 的基础配置文件。该文件包含默认设置，可以在运行时被更高层次的配置文件覆盖。

例如，您可能希望在部署时使用一个基本的身份认证配置，然后在运行时通过 Dashboard UI 覆盖它为更复杂的配置。

对于 `node` 和 `cluster` 等不可变配置，**不推荐**将它们设置在 `base.hocon` 文件中。有关更多信息，请参阅[不可变配置文件](#不可变配置文件) 部分。

::: tip

`base.hocon` 文件不会在集群中同步，只适用于该文件所在的节点。 

:::

## 配置重写文件

在 `data/configs` 目录中，`cluster.hocon` 文件包含整个集群的配置项。通过 Dashboard、REST API 或 CLI 进行的配置更改会持久化到此文件中。

如果集群中的某个节点被重启或添加了新节点，该节点将自动从集群中的其他节点复制并应用 `cluster.hocon` 文件。因此，不建议手动修改此文件。

此文件中的配置会覆盖 `base.hocon` 文件中的配置。有关配置覆盖优先级的详细信息，请参阅[配置覆盖规则](#配覆盖规则)。

自 EMQX 5.1 版本起，对集群配置的任何更改都会在覆盖 `cluster.hocon` 文件之前备份该文件。备份文件会带有节点本地时间的时间戳，最多可以保留 10 个备份文件。

## 不可变配置文件

出于向后兼容的考虑，`emqx.conf` 文件仍然是用于配置关键系统设置的主要配置文件，包括 `node` 和 `cluster` 配置。该文件的优先级高于 `base.hocon` 和 `cluster.hocon`，但低于环境变量。

有关配置覆盖的更多细节，请参阅[配置覆盖规则](#配置覆盖规则)部分。

## 配置路径

在 EMQX 中，可以使用点分隔路径来引用配置值，类似于树结构。从根节点（始终是一个 Struct）开始，路径中的每个段表示一个字段名称或一个 Map 键。对于数组元素，使用基于 1 的索引。

以下是一些配置路径示例：

```bash
node.name = "emqx.127.0.0.1"
zone.zone1.max_packet_size = "10M"
authentication.1.enable = true
```

## HOCON 配置格式

从 5.0 版本开始，EMQX 采用 [HOCON](https://github.com/emqx/hocon) 作为配置文件格式。

HOCON（Human-Optimized Config Object Notation）是一种可扩展的配置语言，它支持类似 JSON 的语法，易于阅读和编写。同时 HOCON 具有继承、合并、引用等功能，使得配置文件更加灵活可控。

**基本语法：**

HOCON 值可以被记为类似 JSON 的对象，例如：

```hcl
node {
  name = "emqx@127.0.0.1"
  cookie = "mysecret"
  cluster_call {
    retry_interval  =  1m
  }
}
```

也可以使用扁平化的方式：

```bash
node.name = "127.0.0.1"
node.cookie = "mysecret"
node.cluster_call.retry_interval = "1m"
```

这种类似 cuttlefish 的扁平格式一定程度向后兼容了之前版本的 EMQX，但使用时又有所不同：

HOCON 建议字符串两端加上引号，没有特殊字符的字符串也可以不加引号如 `foo`，`foo_bar`，而 cuttlefish 把 `=` 右边的所有字符都视为值。

更多有关 HOCON 的语法请参考 [HOCON 文档](https://github.com/lightbend/config/blob/main/HOCON.md)。

## 环境变量

除了配置文件外，EMQX 还可以通过环境变量设置配置。

比如 `EMQX_NODE__NAME=emqx2@127.0.0.1` 环境变量将覆盖以下配置：

```bash
# emqx.conf

node {
  name = "emqx@127.0.0.1"
}
```

配置项与环境变量之前可以通过以下规则转换：

1. 由于配置文件中的 `.` 分隔符不能使用于环境变量，因此 EMQX 选用双下划线 `__` 作为配置分割；
2. 为了与其他的环境变量有所区分，EMQX 还增加了一个前缀 `EMQX_` 来用作环境变量命名空间;
3. 环境变量的值是按 HOCON 值解析的，这也使得环境变量可以用来传递复杂数据类型的值，但要注意特殊字符如`:` 和 `=` 需要用双引号 `"` 包裹。

转换示例：

```bash
# 环境变量

## localhost:1883 会被解析成一个结构体 `{"localhost": 1883}`，因此需要使用双引号包裹
export EMQX_LISTENERS__SSL__DEFAULT__BIND='"127.0.0.1:8883"'

## 通过字符直接传递 HOCON 数组
export EMQX_LISTENERS__SSL__DEFAULT__SSL_OPTIONS__CIPHERS='["TLS_AES_256_GCM_SHA384"]'


# 配置文件
listeners.ssl.default {
  ...
    bind = "127.0.0.1:8883"
    ssl_options {
      ciphers = ["TLS_AES_256_GCM_SHA384"]
    }
  }
}
```

::: tip
未定义的根路径会被 EMQX 忽略，例如 `EMQX_UNKNOWN_ROOT__FOOBAR` 这个环境变量会被 EMQX 忽略，因为 `UNKNOWN_ROOT` 不是预先定义好的根路径。

已知的根路径设置了未知的字段名时，将在启动时输出 `warning` 日志，例如将 `enable` 错误的配置为 `enabled` 时将输出：

```bash
[warning] unknown_env_vars: ["EMQX_AUTHENTICATION__ENABLED"]
```

:::

## 配置覆盖规则

HOCON 的值是分层覆盖的，最简单的规则如下：

- 在同一个文件中，后（在文件底部）定义的值，覆盖前（在文件顶部）定义的值。
- 当按层级覆盖时，高层级的值覆盖低层级的值。

EMQX 配置按以下顺序进行优先级排序：

```
base.hocon < cluster.hocon < emqx.conf < 环境变量
```

这意味着，`base.hocon` 文件中的设置具有最低优先级，可以被更高优先级的配置文件覆盖。以 `EMQX_` 开头的环境变量具有最高优先级。

::: tip

在 5.8.4 版本之前，`base.hocon` 文件并不存在。优先级顺序保持不变，但没有 `base.hocon` 文件。 

:::

通过 EMQX Dashboard UI、HTTP API 或 CLI 所做的更改会在运行时持久化到 `cluster.hocon` 文件中，并立即生效。然而，如果在 `emqx.conf` 或环境变量中对相同的配置项进行了不同的设置，重启节点后这些更改可能会被恢复。

为了避免混淆，**不要在 `emqx.conf` 和 `cluster.hocon` 之间重叠配置设置**。

::: tip
1. 如果您正在使用较旧的 EMQX 版本，特别是 e5.0.2/v5.0.22 或更早的版本（即 cluster-override.conf 文件仍存在于 EMQX 的数据目录中），那么配置设置的优先顺序如下：`emqx.conf < ENV < HTTP API(cluster-override.conf)`。
3. 如果您正在从 e5.0.2/v5.0.22 或更早的版本升级到最新版本的 EMQX，配置的优先级将与以前的版本保持一致，以保持兼容性。
4. `cluster-override.conf` 机制在 5.1 版本中删除。
:::

### 合并覆盖

在如下配置中，最后一行的 `debug` 值会覆盖原先 `level` 字段的 `error` 值，但是 `enable` 字段保持不变：

```bash
log {
  console {
    enable = true
    level = error
  }
}

## 将 console 日志打印级别设置为 debug，其他配置保持不变
log.console.level = debug
```

报文大小限制最先被设置成 1MB，后被覆写为 10MB：

```bash
zone {
  zone1 {
    mqtt.max_packet_size = 1M
  }
}
zone.zone1.mqtt.max_packet_size = 10M
```

### 列表元素覆盖

EMQX 配置中的数组有两种表达方式：

- 列表格式，例如： `[1, 2, 3]`。
- 带下标的 Map 格式，例如： `{"1"=1, "2"=2, "3"=3}`。

以下 3 种格式是等价的：

```bash
authentication.1 = {...}
authentication = {"1": {...}}
authentication = [{...}]
```

基于这个特性，我们就可以轻松覆写数组某个元素的值，例如：

```bash
authentication  = [
  {
    enable = true,
    backend = "built_in_database",
    mechanism = "password_based"
  }
]

# 可以用下面的方式将第一个元素的 `enable` 字段覆写
authentication.1.enable = false
```

::: tip
列表格式是的数组将全量覆写而不是合并覆盖原有值，例如：

```bash
authentication = [
  {
    enable = true
    backend = "built_in_database"
    mechanism="password_based"
  }
]

## 下面这种方式会导致数组第一个元素的除了 `enable` 以外的其他字段全部丢失。
authentication = [{ enable = true }]
```

:::

### Zone 覆盖

EMQX 中的 Zone 是一种配置分组的概念。可以通过将监听器的 `zone` 字段设置为所需 Zone 的名称，将 Zone 与监听器关联。与某个 Zone 关联的监听器连接的 MQTT 客户端将继承该 Zone 的配置，这些配置可能会覆盖全局设置。

::: tip

默认情况下，监听器与一个名为 `default` 的 Zone 关联。`default` Zone 是一个逻辑分组，在配置文件中并不存在。

:::

以下配置项可以在 Zone 级别进行覆盖：

- `mqtt`：MQTT 连接和会话设置，例如允许在特定 Zone 内的 MQTT 消息具有更大的最大数据包大小。
- `force_shutdown`：强制关闭策略。
- `force_gc`：Erlang 进程垃圾回收的微调。
- `flapping_detect`：客户端抖动检测。
- `durable_sessions`：会话持久性设置，例如在特定 Zone 启用 MQTT 会话的持久存储。

在 EMQX 版本 5 中，默认配置文件没有包含任何 Zone，这与版本 4 不同，在版本 4 中有两个默认 Zone：`internal` 和 `external`。

要创建一个 Zone，需要在 `emqx.conf` 文件中定义，例如：

```bash
zones {
  # 可以定义多个 Zone
  my_zone1 {
    # Zone 使用与全局配置相同的配置模式
    mqtt {
      # 允许该 Zone 内的连接具有更大的数据包大小
      max_packet_size = 10M
    }
    force_shutdown {
      # Zone 特定的配置
      ...
    }
    durable_sessions {
      # 仅为该 Zone 的会话启用持久存储
      ...
    }
  }
  my_zone2 {
    ...
  }
}
```

可以用如下方式把一个监听器跟一个 Zone 关联起来。

```bash
listeners.tcp.default {
    bind = 1883
    zone = my_zone1
    ...
}
```

## Schema 手册

为了确保配置正确，EMQX 引入了 schema。schema 定义了数据类型，以及数据字段的名称和元数据，用于配置值的类型检查等。

[EMQX 开源版配置手册](https://docs.emqx.com/zh/emqx/v@CE_VERSION@/hocon/)和 [EMQX 企业版配置手册](https://docs.emqx.com/zh/enterprise/v@EE_VERSION@/hocon/) 就是从这个 Schema 生成的。

::: tip

Zone 配置的 schema 未包含在配置手册中，因为每个组的配置是相同的。例如，`zones.my_zone1.mqtt {...}` 与 `mqtt {...}` 具有相同的 schema。

:::

### 基本数据类型

配置手册中的原始数据类型基本上是自解释的，不需要太多文档说明。
以下是您将遇到的所有原始类型的列表，附有明确的示例：

#### 整数 `Integer`

表示一个整数。示例包括 `42`、`-3`、`0`。

#### 范围 `Integer(Min..Max)`

在指定范围内的整数。例如，`1..+inf` 表示从 `1` 到正无穷大（`+inf`），表示只接受正整数。

#### 枚举 `Enum(symbol1, symbol2, ...)`

定义一个只能取预定义符号之一的枚举类型。例如，`Enum(debug,info,warning,error)` 定义了可接受的日志级别。

#### 字符串 `String`

**字符串**数据类型代表一个字符序列，并支持几种格式以适应不同的使用场景：

- **无引号**：适合简单的标识符或名称，避免使用特殊字符（详见下文）。

- **引号字符串**：对于包含特殊字符或空白的字符串，使用双引号（`"`），并根据需要使用反斜杠（`\`）进行转义。示例：`"line1\nline2"`。

- **三引号字符串**：用三引号（`"""`）包围，这些字符串除了`\`外不需要转义，简化了复杂内容的包含。注意，紧邻三引号的引号必须被转义才能被视为字符串的一部分。

- **带缩进的三引号字符串**：从 EMQX 5.6 开始支持。由`"""~`和`~"""`包裹，此格式允许字符串内部进行缩进，以便在配置文件中更好地布局，适合多行或格式化文本。

**无引号字符串的特别注意事项：**
- 避免“禁止字符”：`$`, `"`, `{`, `}`, `[`, `]`, `:`, `=`, `,`, `+`, `#`, `` ` ``, `^`, `?`, `!`, `*`, `&`, `\`, 或空格。
- 不以`//`开头（这会引入注释）。
- 开头不用`true`, `false`, 或`null`，以免被误解为布尔值或空值。

**三引号字符串的指导原则：**

- 要包含紧邻三引号的引号字符，需进行转义或使用`~`分隔符以增加清晰度。
- 多行字符串支持使用空格（不是制表符）进行缩进以提高可读性。缩进级别由任何行上最小的前导空格数确定。

示例：

```
rule_xlu4 {
  sql = """~
    SELECT
      *
    FROM
      "t/#"
  ~"""
}
```

有关HOCON字符串引用约定的更多细节，请参阅[HOCON规范](https://github.com/lightbend/config/blob/main/HOCON.md#unquoted-strings)。

有关EMQ对带缩进的三引号字符串的特殊适配信息，请参考[emqx/hocon.git README](https://github.com/emqx/hocon?tab=readme-ov-file#divergence-from-spec-and-caveats)。


#### 常量字符串 `String("constant")`

一个常量字符串值，实际上充当单值枚举（`Enum`）。可以用来定义不变的静态值。

#### 布尔 `Boolean`

只能是 `true` 或 `false`，区分大小写。

#### 浮点数 `Float`

小数的浮点数字。例如 `3.14`、`-0.001`。

#### 时间 `Duration`

以人类可读格式表示的时间跨度。
例如，`10s` 表示十秒，`2.5m` 表示两分半，`1h30m` 表示一小时三十分钟，`1W2D` 表示一周两天，或 `5ms` 表示五毫秒。
`ms` 是持续时间的最小单位。

#### 时间（秒）Duration(s)

秒级精度的 `Duration` 类型。指定持续时间的更细部分可能会被舍入或忽略。
例如，指定 `1200ms` 相当于 `1s`，表示不考虑毫秒。

#### 保密（Secret）

用于敏感信息的类型，如密码和令牌，应该用引号括起来，以确保它被视为单一的、安全的字符串。


### 复杂数据类型

EMQX 的 HOCON 配置中的复杂数据类型旨在封装可以包含其他复杂类型和原始值的数据结构。
这些数据类型支持灵活和层次化的数据表示。以下是可用的复杂数据类型：

#### 结构体 Struct(name)

代表一个带有字段的结构体，字段被大括号 `{}` 包裹。
`name` 用于指定一个 Schema 名称。Schema 定义了结构体应该包含哪些字段名称以及各字段的类型。

#### 映射 `Map($name-\>Type)`

类似于结构体（Struct），映射包含键值对，但不预定义字段名称。
`$name` 变量表示键可以是任何字符串（不能包含点（`.`）），代表实体或属性的名称。
`Type` 指定映射中所有值必须是相同的数据类型。

#### 联合 `OneOf(Type1, Type2, ...)`

定义一个联合类型，可以包含两个或以上不同类型。表示某字段可以是成员中的任何一个类型。
例如，某配置项可以是 `String(infinity)` 或 `Duration` 表示它要么是 infinity 要么是一个时间间隔字符串（例如 `1s`）。

#### 数组 `Array(Type)`

表示某个配置字段是一个数组，数组元素的类型是 `Type`。


::: tip
如果 Map 的字段名称是纯数字，它会被解析成一个数组。

例如：

```bash
myarray.1 = 74
myarray.2 = 75
```

会被解析成 `myarray = [74, 75]`，这个用法在重载数组元素的值时候非常有用。
:::

### Variform 表达式

Variform 是一种轻量级、富有表现力的语言,旨在进行字符串操作和运行时求值。它不是一种全功能的编程语言，而是一种专门的工具，可以嵌入配置中，用来态执行字符串操作。

::: tip
Variform 表达式仅适用于部分配置项中，如无明确说明请不要使用。
:::

::: tip 空值说明：
在 Variform 表达式中，一个变量引用或者表达式求值可能得到一个空值。 这个空值以空字符串的方式返回。
需要注意的是 JSON 解码得到的 `null` 被当作空值处理，而不是字符串 `"null"`。
:::

#### 语法概览

以下面的表达式作为示例：

```bash
function_call(clientid, another_function_call(username))
```

此表达式结合或操作 `clientid` 和 `username` 以生成新的字符串值。

Variform 支持以下字面量：

- 布尔值 ： `ture` 或者 `false`。
- 整数：例如，`42`。
- 浮点数：例如，`3.14`。
- 字符串：单引号 `'` 或双引号 `"` 之间的 ASCII 字符。
- 数组：元素位于 `[` 和 `]` 之间，以逗号 `,` 分隔。
- 变量：引用预定义的值，例如 `clientid`。
- 函数：预定义的函数，例如 `concat([...])`。

Variform 不支持以下功能：

- 算术运算
- 条件语句
- 循环
- 用户定义的变量
- 用户定义的函数
- 异常处理和错误恢复
- 字符串字面量中的转义序列。调用 `unescape` 函数来对特殊字符进行反转义。

以下是一个嵌入配置文件中的示例。

```bash
mqtt {
    client_attrs_init = [
        {
            # 提取客户端 ID 在第一个 `-` 字符前面的前缀
            expression = "nth(1,tokens(clientid, '-'))"
            # 然后把提取的字符串赋值给 client_attrs.group 这个字段
            set_as_attr = group
        }
    ]
}

```

::: tip
当表达式中需要使用 unescape 函数时，建议在 HOCON 配置中使用三引号（`"""`）字符串，这样就无需进行双重转义。

例如

```bash
#### 对于多行客户端 ID，取第一行。
expression = """nth(1, tokens(clientid, unescape('\n')))"""
```

:::

#### 预定义函数

EMQX 包含一系列丰富的字符串、数组、随机和散列函数，类似于规则引擎字符串函数中可用的那些。这些函数可以用来操作和格式化提取的数据。例如，`lower()`、`upper()` 和 `concat()` 可以帮助调整提取字符串的格式，而 `hash()` 和 `hash_to_range()` 可以基于数据创建散列或范围输出。

以下是可以在表达式中使用的函数：

- **字符串函数**：
  - [字符串操作函数](../data-integration/rule-sql-builtin-functions.md#string-operation-functions)
  - 还添加了一个新函数 `any_to_string/1`，用于将任何中间非字符串值转换为字符串。
- **数组函数**：[nth/2](../data-integration/rule-sql-builtin-functions.md#nth-n-integer-array-array-any)
- **随机函数**：rand_str, rand_int
- **无模式编码/解码函数**：
  - [bin2hexstr/1](../data-integration/rule-sql-builtin-functions.md#bin2hexstr-data-binary-string)
  - [hexstr2bin/1](../data-integration/rule-sql-builtin-functions.md#hexstr2bin-data-string-binary)
  - [base64_decode/1](../data-integration/rule-sql-builtin-functions.md#base64-decode-data-string-bytes-string)
  - [base64_encode/1](../data-integration/rule-sql-builtin-functions.md#base64-encode-data-string-bytes-string)
  - `int2hexstr(Integer)`: Encode an integer to hex string. e.g. 15 as 'F' (uppercase).
- **散列函数**：
  - `hash(Algorithm, Data)`：其中算法可以是以下之一：md4 | md5, sha (或 sha1) | sha224 | sha256 | sha384 | sha512 | sha3_224 | sha3_256 | sha3_384 | sha3_512 | shake128 | shake256 | blake2b | blake2s
  - `hash_to_range(Input, Min, Max)`：使用 sha256 散列输入数据，并将散列映射到最小值和最大值之间的整数（包括 Min 和 Max：Min =< X =< Max））。
  - `map_to_range(Input, Min, Max)`：将输入映射到最小值和最大值之间的整数（包括 Min 和 Max：Min =< X =< Max）。
- **比较函数**：
  - `num_eq(A, B)`：如果两个数字相同，则返回 'true'，否则返回 'false'。
  - `num_neq(A, B)`：如果两个数字不相同，则返回 'true'，否则返回 'false'。
  - `num_gt(A, B)`：如果 A 大于 B，则返回 'true'，否则返回 'false'。
  - `num_gte(A, B)`：如果 A 不小于 B，则返回 'true'，否则返回 'false'。
  - `num_lt(A, B)`：如果 A 小于 B，则返回 'true'，否则返回 'false'。
  - `num_lte(A, B)`：如果 A不大于 B，则返回 'true'，否则返回 'false'。
  - `str_eq(A, B)`：如果两个字符串相同，则返回 'true'，否则返回 'false'。
  - `str_neq(A, B)`：如果两个字符串不相同，则返回 'true'，否则返回 'false'。
  - `str_gt(A, B)`：如果 A 在字典顺序上位于 B 之后，则返回 'true'，否则返回 'false'。
  - `str_gte(A, B)`：如果 A 在字典顺序上不位于 B 之前，则返回 'true'，否则返回 'false'。
  - `str_lt(A, B)`：如果 A 在字典顺序上位于 B 之前，则返回 'true'，否则返回 'false'。
  - `str_lte(A, B)`：如果 A 在字典顺序上不位于 B 之后，则返回 'true'，否则返回 'false'。
  - `is_empty_var(V)`: 检查一个变量是否是空值。空值包括：对一个未知变量的引用 (`undefined`)，JSON 的 `null` 字段，或者一个空字符串`''`。
  - `not(Bool)`: 取反操作，如果 `Bool` 是 `true`，则返回 `false`，如果 `Bool` 是 `false`，则返回 `true`。 该函数亦支持字符串参数，如果输入是字符串，返回值也是字符串。

- **系统函数**:
  - `getenv(Name)`：返回环境变量 `Name` 的值，并遵循以下限制：
    - 在读取操作系统环境变量之前，会自动添加前缀 `EMQXVAR_`。例如，调用 `getenv('FOO_BAR')` 将读取 `EMQXVAR_FOO_BAR`。
    - 这些值一旦从操作系统环境加载便不会再改变。

#### 条件

到目前为止，Variform 表达式没有全面的控制流程。
下列函数可以用于简单的根据条件来选择不同取值：

- `iif(Condition, ThenExpression, ElseExpression)`: 如果 `Conditions` 是 `true` 或者非空字符串，则返回 `ThenExpression`，否则 `ElseExpression`。
- `coalesce(Arg1, Arg2, ...)`: 取参数列表中第一个非空值。
- `coalesce([Element1, Element2, ...])`: 取数组中的第一个非空值。

#### 错误处理

与 Bash 等脚本环境的默认行为一样，Variform 表达式在出现错误时，如遇到未绑定的变量或运行时异常，会生成空字符串（""）。

- 未绑定的变量：如果表达式引用了未定义或超出作用域（未绑定）的变量，该表达式将被评估为一个空字符串。
- 运行时异常：在表达式执行过程中发生的任何异常，无论是由于函数使用不当、数据类型无效还是其他未预见的问题，都会导致表达式产生空字符串。例如，数组索引超出范围。

#### 示例表达式

- `nth(1, tokens(clientid, '.'))`：提取以点分隔的客户端 ID 的前缀。
- `strlen(username, 0, 5)`：提取部分用户名。
- `coalesce(regex_extract(clientid,'[0-9]+'),'vin-1000')`：使用正则表达式从客户端 ID 中提取数字。如果正则表达式产生空字符串，则返回 `'000'`。
- `iif(true, "如果为真的值", "如果为假的值")`：返回 `如果为真的值`
- `iif("", "如果为真的值", "如果为假的值")`：返回 `如果为假的值`
- `iif("hello", "如果为真的值", "如果为假的值")`：返回 `如果为真的值`
- `iif(regex_match(clientid,'^foo\.+*'),'foo','bar')`：如果 `clientid` 以 `foo.` 开头，则返回 `foo`，否则返回 `bar`。
