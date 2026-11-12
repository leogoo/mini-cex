# mini-cex

Mini Crypto Exchange — 基于 Spring Cloud 微服务的虚拟货币交易所后端。

## 版本矩阵（与 coinExchange 项目保持一致，依赖已在本地仓库缓存）

| 组件 | 版本 | 说明 |
|---|---|---|
| JDK | 21（Temurin 21.0.13） | 编译目标 21，兼容 17+ |
| Spring Boot | 3.5.16 | spring-boot-starter-parent |
| Spring Cloud | 2025.1.4 | spring-cloud-dependencies BOM |
| Spring Cloud Alibaba | 2025.1.0.0 | Nacos 注册/配置中心 |
| Maven | 3.10.0 / 3.9.16 | 无独立安装，见下方「Maven 说明」 |

> 官方兼容矩阵参考：SCA 2025.1.0.0 ↔ Spring Cloud 2025.1.x ↔ Spring Boot 3.5.x/4.0.x。
> 本项目采用 Boot 3.5 线（生态更成熟，MyBatis-Plus 等第三方兼容性最好）。

## Maven 说明（本机未独立安装 mvn）

- **IDEA**：Settings → Build Tools → Maven，Maven home 选「Bundled (Maven 3)」即 IDEA 内置的
  `/Applications/IntelliJ IDEA.app/Contents/plugins/maven-plugin/lib/maven3`（3.9.16）；
  user settings 指向 `~/.m2/settings.xml`（已配阿里云镜像）。
- **命令行**：`~/.m2/wrapper/dists/apache-maven-3.10.0-bin/.../bin/mvn`（3.10.0），
  可软链到 PATH：`ln -s ~/.m2/wrapper/dists/apache-maven-3.10.0-bin/*/apache-maven-3.10.0/bin/mvn /usr/local/bin/mvn`

## 模块结构

```
mini-cex
├── coin-common            公共模块（统一返回体、异常、工具类）
├── coin-iaas              基础服务设施（聚合，packaging=pom）
│   ├── gateway-server     网关          :8000
│   └── authorization-server 授权服务    :8001
├── coin-chan              撮汉服务（聚合）
│   ├── chan-api           Feign 接口 + DTO
│   └── chan-service       服务实现       :8101
├── coin-task              定时任务        :8108
├── coin-statistics        统计分析服务（聚合）
│   ├── statistics-api     （脑图中的 finance-api，因与 coin-finance 的 finance-api
│   │                       GAV 冲突而改名）
│   └── statistics-service 服务实现        :8102
├── match-engine           撮合服务（聚合）
│   ├── match-api
│   └── match-service                     :8106
├── exchange-engine        在线交易服务（聚合）
│   ├── exchange-api
│   └── exchange-service                  :8107
├── coin-admin             后台管理服务（聚合）
│   ├── admin-api
│   └── admin-service                     :8103
├── coin-member            会员服务（聚合）
│   ├── member-api
│   └── member-service                    :8104
└── coin-finance           清算服务（聚合）
    ├── finance-api
    └── finance-service                   :8105
```

## 关于「模块内不能建模块」

Maven 天然支持多级模块：`coin-chan` 等只是 `packaging=pom` 的**聚合模块**，
`chan-api` / `chan-service` 通过聚合 POM 的 `<modules>` 声明挂进来。
在 IDEA 中导入根 POM 后，它们会以普通 Maven 模块形式平铺展示，
不需要（也不能）在 IDEA 里对某个模块再执行 New → Module。

## 启动依赖

- Nacos Server：默认 `127.0.0.1:8848`，可用环境变量 `NACOS_ADDR` 覆盖。
- 服务实现类已带 `@EnableDiscoveryClient` + `@EnableFeignClients(basePackages = "com.minicex")`，
  api 模块中的 `@FeignClient` 接口会被自动扫描。
