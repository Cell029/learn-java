
- **Oracle JDK**：由 Oracle 提供的 Java 开发工具包。它是最初由 Sun Microsystems 开发的 JDK 的延续，Oracle 收购 Sun 后继续提供 JDK 的商业支持和更新。

- **OpenJDK**：是 Oracle 提供的 Java 平台的开源实现。它由 Java 社区推动和维护，OpenJDK 项目遵循 GNU 通用公共许可证（GPL）v2 许可。

### 相同点：

1、 **功能一致性**：Oracle JDK 和 OpenJDK 都是基于同样的 Java 标准（JVM 规范和 Java API 规范）。因此，它们都包含了相同的功能，允许开发者在两者之间进行互换，运行同样的 Java 程序。

2、**代码实现**：两者大部分代码是相同的，特别是 Java 核心库（如 `java.util`、`java.lang` 等）和 JVM 都是由 OpenJDK 提供的。

3、**JVM 实现**：二者使用相同的 HotSpot JVM，提供 Java 程序的执行引擎。

### 区别：

1、是否开源：OpenJDK 是一个参考模型并且是完全开源的，而 Oracle JDK 是基于 OpenJDK 实现的，并不是完全开源的

2、是否免费：Oracle JDK 会提供免费版本，但一般有时间限制。JDK17 之后的版本可以免费分发和商用，但是仅有 3 年时间，3 年后无法免费商用。不过，JDK8u221 之前只要不升级可以无限期免费。OpenJDK 是完全免费的。

3、功能性：Oracle JDK 在 OpenJDK 的基础上添加了一些特有的功能和工具，比如 Java Flight Recorder（JFR，一种监控工具）、Java Mission Control（JMC，一种监控工具）等工具。不过，在 Java 11 之后，OracleJDK 和 OpenJDK 的功能基本一致，之前 OracleJDK 中的私有组件大多数也已经被捐赠给开源组织。

4、稳定性：OpenJDK 不提供 LTS 服务，而 OracleJDK 大概每三年都会推出一个 LTS 版进行长期支持。不过，很多公司都基于 OpenJDK 提供了对应的和 OracleJDK 周期相同的 LTS 版。因此，两者稳定性其实也是差不多的。

5、协议：Oracle JDK 使用 BCL/OTN 协议获得许可，而 OpenJDK 根据 GPL v2 许可获得许可



