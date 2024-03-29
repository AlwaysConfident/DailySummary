# Maven

软件项目管理工具，基于项目对象模型(POM)的概念，Maven 可以从一条中心信息管理项目的构建、报告和文档

pom.xml 文件位于根目录，包含项目构建生命周期的详细信息

- 项目构建：提供标准、跨平台的自动化项目构建方式
- 依赖管理：方便快捷地管理项目依赖的资源，避免资源间的版本冲突问题
- 统一开发结构：提供标准、统一的项目结构

## Maven 坐标

Maven 通过依赖的唯一坐标在 Maven 仓库中定位并引入：

- groupId：Maven 项目隶属的组织
- artifactId：Maven 项目名称，项目的唯一标识符，对应项目根目录的名称
- version：定义 Maven 项目版本

### 依赖冲突

项目中的两个依赖同时引入了一个依赖，且两者引入的版本冲突

- 最短路径优先：选择依赖链路最短的
- 声明顺序优先：路径长度相同时，选择最先在 pom 中声明的

## Maven 仓库

- 本地仓库：计算机本地的目录，缓存远程下载的依赖
- 远程仓库
  - 中央仓库：Maven 社区维护的仓库
  - 私有仓库：局域网中的私有仓库，一般为远程仓库的镜像
  - 公共仓库：用作加速访问

依赖查找顺序：本地仓库 → 远程仓库 → 报错

## Maven 生命周期

Maven 的生命周期是为了对所有依赖过程进行抽象和统一，包含了项目的清理、初始化、编译、测试、打包、集成测试、验证、部署和站点生成等所有构建步骤

Maven 中定义了 3 个生命周期：

- default：在没有任何关联依赖的情况下定义的，Maven 的主要生命周期，用于构建应用程序
- clean：包含 3 个阶段，用于移除上一次构建生成的文件
  - pre-clean
  - clean
  - post-clean
- site：用于建立和发布项目站点
  - pre-site
  - site
  - post-site
  - site-deploy

生命周期相互独立，每个生命周期都包含有序的多个阶段

