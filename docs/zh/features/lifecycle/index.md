# 数据生命周期管理和分层

随着数据的不断增长，针对访问、安全性和经济性进行协同优化的能力成为一项硬性要求，而不是锦上添花。这就是生命周期数据管理的作用。RustFS 提供了一套独特的功能来保护云内部和云之间的数据 - 包括公共和私有云。RustFS 的企业数据生命周期管理工具，包括版本控制、对象锁定和各种衍生组件，满足许多用例。

## 对象过期

数据不必永远存在：RustFS 生命周期管理工具允许您定义数据在删除之前在磁盘上保留多长时间。用户将时间长度定义为 RustFS 开始删除对象的特定日期或天数。

生命周期管理规则是按存储桶制定的，可以使用对象和标签筛选器的任意组合来构建。不指定筛选条件以设置整个存储桶的到期规则，或指定多个规则以制定更复杂的到期行为。

RustFS 对象过期规则也适用于版本控制存储桶，并附带一些特定于版本控制的风格。例如，您可以仅对对象的非当前版本指定到期规则，以最大限度地发挥对象版本控制的优势，而不会产生长期存储成本。同样，您可以创建生命周期管理规则，用于删除其唯一剩余版本为删除标记的对象。

存储桶过期规则完全符合 RustFS WORM 锁定和法定保留 - 处于锁定状态的对象将保留在磁盘上，直到锁定过期或被明确解除。一旦对象不再受锁定约束，RustFS 就会开始正常应用到期规则。

RustFS 对象过期生命周期管理规则在功能和语法上与 AWS Lifecycle Management 兼容。RustFS 还支持以 JSON 格式导入现有规则，从而轻松迁移现有 AWS 到期规则。

## 基于策略的对象分层

RustFS 可以以编程方式配置对象存储分层，以便对象根据任意数量的变量从一种状态或类转换到另一种状态或类 - 尽管最常用的是访问的时间和频率。最好在分层的上下文中理解此功能。分层允许用户优化存储成本或功能，以应对不断变化的数据访问模式。分层数据存储一般用于以下场景：

## 跨存储介质

跨存储介质分层是最著名和最直接的分层用例。在这里，RustFS 对底层介质进行了抽象，并针对性能和成本进行了协同优化。例如，对于性能或近线工作负载，数据可能存储在 NVMe 或 SSD 上，但在一段时间后分层到 HDD 介质，或者用于重视性能扩展的工作负载。随着时间的推移，如果合适，可以将该数据进一步迁移到长期存储中。

![跨存储介质分层](images/s9-2.png)

## 跨云类型

一个快速出现的用例涉及使用公有云的廉价存储和计算资源作为私有云的另一层。在此用例中，使用适当的私有云介质执行面向性能的近线工作负载。数据量无关紧要，但价值和性能预期无关紧要。随着数据量的增加和性能预期的降低，企业可以使用公有云的冷存储选项来优化与保留数据相关的成本和访问能力。

这是通过在私有云和公共云上运行 RustFS 来实现的。使用复制，RustFS 可以将数据移动到廉价的公共云选项上，并在必要时使用公共云中的 RustFS 来保护和访问它。在这种情况下，公有云成为 RustFS 的哑存储，就像 JBOD 成为 RustFS 的哑存储一样。此方法可避免替换和添加过时的磁带基础结构。

![跨云类型分层](images/s9-3.png)

## 在公有云中

RustFS 通常充当公有云中的主要应用程序存储层。在这种情况下，与其他用例一样，RustFS 是应用程序访问的唯一存储。应用程序（和开发人员）不需要知道存储终结点以外的任何内容。RustFS 根据管理参数确定哪些数据属于何处。例如，RustFS 可以确定块数据应移动到对象层，以及哪个对象层满足企业的性能和经济目标。

RustFS 结合了不同的存储分层层，并确定合适的介质，以在不影响性能的情况下提供更好的经济性。应用程序只是通过 RustFS 寻址对象，而 RustFS 透明地应用策略在层之间移动对象，并将该对象的元数据保留在块层中。

![公有云分层](images/s9-4.png)
