# 1. 相关文章：
> [【Gradle-4】Gradle的生命周期](https://juejin.cn/post/7170684769083555877)

# 2. 主要细节点
&emsp;&emsp;Gradle分三个阶段评估和运行构建，分别是`Initialization (初始化)`、`Configuration (配置)` 和 `Execution (执行)`，且任何的构建任务都会执行这个三个阶段。
> 1. 在 Initialization (初始化) 阶段，Gradle会决定构建中包含哪些项目，并会为每个项目创建Project实例。为了决定构建中会包含哪些项目，Gradle首先会寻找settings.gradle来决定此次为单项目构建还是多项目构建，单项目就是module，多项目即project+app+module(1+n)。
> 2. 在 Configuration (配置) 阶段，Gradle会评估构建项目中包含的所有构建脚本，随后应用插件、使用DSL配置构建，并在最后注册Task，同时惰性注册它们的输入，因为并不一定会执行。
> 3. 最后，在 Execution (执行) 阶段，Gradle会执行构建所需的Task集合。

