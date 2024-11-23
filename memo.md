# Spring Boot Sample Project Memo

## 项目概述

这是一个基于Spring Boot 3.2.1的多模块项目示例，包含认证和业务逻辑两个主要模块。

### 技术栈
- Java 21
- Spring Boot 3.2.1
- Gradle 8.5
- 数据库：SQLite & PostgreSQL
- 缓存：Redis
- 消息队列：Apache Kafka & AWS SQS
- 认证：Spring Security, OAuth2, JWT
- ORM：MyBatis

## 项目结构

### 模块说明

#### Auth模块
- **sample-auth-controller**: 提供认证相关的REST API接口，包括登录、注册、OAuth2认证等
- **sample-auth-service**: 实现认证相关的业务逻辑，包括JWT token管理、用户认证等
- **sample-auth-mybatis**: 处理用户认证相关的数据库访问
- **sample-auth-consumer**: 处理认证相关的消息消费
- **sample-auth-producer**: 处理认证相关的消息生产
- **sample-auth-batch**: 处理认证相关的批量任务
- **sample-auth-devtool**: 认证模块的开发工具和辅助功能

#### Business模块
- **sample-business-controller**: 提供业务相关的REST API接口
- **sample-business-service**: 实现核心业务逻辑
- **sample-business-mybatis**: 处理业务数据的数据库访问
- **sample-business-consumer**: 处理业务相关的消息消费
- **sample-business-producer**: 处理业务相关的消息生产
- **sample-business-batch**: 处理业务相关的批量任务
- **sample-business-devtool**: 业务模块的开发工具和辅助功能

## 配置说明

### 构建配置（build.gradle）

#### 1. 基础依赖
```gradle
dependencies {
    // Spring Boot
    implementation 'org.springframework.boot:spring-boot-starter'
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    implementation 'org.springframework.boot:spring-boot-starter-security'
    
    // Database
    implementation 'org.xerial:sqlite-jdbc:3.44.1.0'
    implementation 'org.postgresql:postgresql:42.7.1'
    implementation 'com.github.gwenn:sqlite-dialect:0.1.2'
    
    // Redis
    implementation 'org.springframework.boot:spring-boot-starter-data-redis'
    
    // Kafka
    implementation 'org.springframework.kafka:spring-kafka'
    
    // AWS SQS
    implementation platform('software.amazon.awssdk:bom:2.22.9')
    implementation 'software.amazon.awssdk:sqs'
    
    // MyBatis
    implementation 'org.mybatis.spring.boot:mybatis-spring-boot-starter:3.0.3'
}
```

#### 2. JUnit & JaCoCo配置
```gradle
dependencies {
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
    testImplementation 'org.springframework.security:spring-security-test'
    testImplementation 'org.springframework.kafka:spring-kafka-test'
}

// JUnit配置
tasks.named('test') {
    useJUnitPlatform()
    finalizedBy jacocoTestReport
}

// JaCoCo配置
jacocoTestReport {
    dependsOn test
    reports {
        xml.required = true
        csv.required = false
        html.outputLocation = layout.buildDirectory.dir('jacocoHtml')
    }
}

jacocoTestCoverageVerification {
    violationRules {
        rule {
            limit {
                minimum = 0.7
            }
        }
    }
}
```

#### 3. Docker镜像构建（Jib）配置
```gradle
jib {
    from {
        image = 'eclipse-temurin:21-jre'
    }
    to {
        image = "ghcr.io/${findProperty('gpr.user') ?: 'unknown-user'}/${project.name}"
        tags = ['latest', project.version]
        auth {
            username = findProperty('gpr.user') ?: 'unknown-user'
            password = findProperty('gpr.key') ?: ''
        }
    }
    container {
        jvmFlags = ['-Xms512m', '-Xmx512m']
        ports = ['8080']
        environment = [
            'SPRING_PROFILES_ACTIVE': 'prod'
        ]
    }
}
```

### 运行时配置

#### 1. 数据库配置

##### SQLite（开发环境）
```properties
spring.datasource.url=jdbc:sqlite:path/to/database.db
spring.datasource.driver-class-name=org.sqlite.JDBC
```

##### PostgreSQL（生产环境）
```properties
spring.datasource.url=jdbc:postgresql://localhost:5432/dbname
spring.datasource.username=username
spring.datasource.password=password
spring.datasource.driver-class-name=org.postgresql.Driver
```

#### 2. Redis配置
```properties
spring.redis.host=localhost
spring.redis.port=6379
```

#### 3. Kafka配置
```properties
spring.kafka.bootstrap-servers=localhost:9092
spring.kafka.consumer.group-id=your-group-id
```

#### 4. AWS SQS配置
```properties
aws.accessKeyId=your-access-key
aws.secretKey=your-secret-key
aws.region=your-region
```

## 监控配置

项目使用Spring Boot Actuator和Prometheus进行监控和指标收集。

#### 1. 监控依赖
```gradle
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-actuator'
    implementation 'io.micrometer:micrometer-registry-prometheus'
}
```

#### 2. 监控配置（application-monitoring.yml）
```yaml
management:
  endpoints:
    web:
      exposure:
        include: health,metrics,prometheus,info,env,loggers,threaddump,heapdump
      base-path: /actuator
  endpoint:
    health:
      show-details: always
    prometheus:
      enabled: true
  metrics:
    tags:
      application: ${spring.application.name}
    export:
      prometheus:
        enabled: true
    distribution:
      percentiles-histogram:
        http.server.requests: true
      sla:
        http.server.requests: 10ms, 50ms, 100ms, 200ms, 500ms
```

#### 3. 可用的监控端点
- `/actuator/health`: 应用健康状态
- `/actuator/metrics`: 应用指标
- `/actuator/prometheus`: Prometheus格式的指标数据
- `/actuator/info`: 应用信息
- `/actuator/env`: 环境变量
- `/actuator/loggers`: 日志级别管理
- `/actuator/threaddump`: 线程转储
- `/actuator/heapdump`: 堆转储

#### 4. 监控指标
1. **JVM指标**
   - 内存使用情况
   - 垃圾回收统计
   - 线程使用情况
   - 类加载统计

2. **HTTP请求指标**
   - 请求计数
   - 请求延迟分布
   - 请求错误率

3. **数据库连接池指标**
   - 活跃连接数
   - 空闲连接数
   - 等待连接数

4. **缓存指标**
   - 缓存命中率
   - 缓存大小
   - 缓存驱逐统计

5. **自定义业务指标**
   - 可以使用`@Timed`注解或MeterRegistry添加自定义指标

#### 5. Prometheus集成
1. **配置Prometheus服务器**
```yaml
scrape_configs:
  - job_name: 'spring-boot-app'
    metrics_path: '/actuator/prometheus'
    static_configs:
      - targets: ['localhost:8080']
```

2. **Grafana仪表板**
   - 可以导入现成的Spring Boot仪表板模板
   - 仪表板ID: 12900（推荐使用）

#### 6. 最佳实践
1. **安全性**
   - 在生产环境中限制actuator端点的访问
   - 配置适当的CORS策略
   - 考虑使用Spring Security保护监控端点

2. **性能考虑**
   - 合理配置指标收集频率
   - 避免过多的自定义指标
   - 适当配置指标保留时间

3. **告警配置**
   - 配置关键指标的告警规则
   - 设置合适的告警阈值
   - 建立告警通知渠道

4. **监控数据保留**
   - 配置适当的数据保留策略
   - 定期备份重要的监控数据
   - 考虑使用时序数据库长期存储

## Prometheus和Grafana设置

项目在`sample-auth-devtool/docker`目录下提供了完整的监控环境配置。

#### 1. 目录结构
```
docker/
├── docker-compose.yml              # Docker Compose配置文件
├── prometheus/
│   └── prometheus.yml             # Prometheus配置
└── grafana/
    └── provisioning/
        ├── dashboards/            # Grafana仪表板配置
        │   └── dashboard.yml
        └── datasources/           # Grafana数据源配置
            └── prometheus.yml
```

#### 2. 配置说明

##### Docker Compose配置
```yaml
services:
  prometheus:
    image: prom/prometheus:v2.48.1
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus/prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus_data:/prometheus

  grafana:
    image: grafana/grafana:10.2.3
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_USER=admin
      - GF_SECURITY_ADMIN_PASSWORD=admin123
```

##### Prometheus配置
```yaml
scrape_configs:
  - job_name: 'spring-boot-app'
    metrics_path: '/actuator/prometheus'
    scrape_interval: 5s
    static_configs:
      - targets: ['host.docker.internal:8080']
        labels:
          application: 'spring-boot-sample'
```

#### 3. 使用方法

1. **启动监控环境**
```bash
cd sample-auth/sample-auth-devtool/docker
docker-compose up -d
```

2. **访问监控界面**
   - Prometheus: http://localhost:9090
   - Grafana: http://localhost:3000
     - 用户名: admin
     - 密码: admin123

3. **导入Grafana仪表板**
   - 登录Grafana
   - 点击 "+" -> "Import"
   - 输入仪表板ID: 12900（Spring Boot 2.1 Statistics）
   - 选择Prometheus数据源
   - 点击"Import"

4. **查看指标**
   - 在Prometheus中：
     - 访问 http://localhost:9090/targets 查看目标状态
     - 使用查询浏览器测试PromQL查询
   - 在Grafana中：
     - 查看导入的仪表板
     - 可以创建自定义仪表板和告警规则

#### 4. 常用监控指标

1. **JVM相关**
   ```promql
   # JVM内存使用
   jvm_memory_used_bytes
   
   # GC次数
   jvm_gc_collection_seconds_count
   
   # 线程数
   jvm_threads_live_threads
   ```

2. **HTTP请求**
   ```promql
   # 请求总数
   http_server_requests_seconds_count
   
   # 请求延迟
   http_server_requests_seconds_sum
   
   # 错误率
   sum(rate(http_server_requests_seconds_count{status="5xx"}[5m]))
   ```

3. **系统资源**
   ```promql
   # CPU使用
   process_cpu_usage
   
   # 系统负载
   system_load_average_1m
   ```

#### 5. 告警配置

1. **在Grafana中设置告警**
   - 编辑仪表板面板
   - 切换到"Alert"标签
   - 配置告警条件和通知

2. **常用告警规则示例**
   - JVM堆内存使用超过80%
   - HTTP请求错误率超过1%
   - API响应时间超过500ms
   - 系统负载过高

#### 6. 最佳实践

1. **数据保留**
   - Prometheus默认保留15天数据
   - 可以通过`--storage.tsdb.retention.time`参数调整

2. **性能优化**
   - 适当调整抓取间隔
   - 避免过多的时间序列数据
   - 合理使用标签

3. **安全性**
   - 在生产环境中修改默认密码
   - 配置反向代理和SSL
   - 限制访问IP

4. **备份**
   - 定期备份Grafana配置
   - 导出重要的仪表板配置
   - 保存自定义的告警规则

## 使用指南

### 1. 构建和运行

#### 构建整个项目
```bash
./gradlew clean build
```

#### 运行特定模块
```bash
# 运行认证控制器
./gradlew :sample-auth-controller:bootRun

# 运行业务控制器     
./gradlew :sample-business-controller:bootRun
```

### 2. 测试

#### 运行测试
```bash
# 运行所有测试
./gradlew test

# 运行特定模块的测试
./gradlew :sample-auth-controller:test
```

#### 生成测试覆盖率报告
```bash
# 生成报告
./gradlew jacocoTestReport

# 验证覆盖率
./gradlew jacocoTestCoverageVerification
```

### 3. Docker镜像构建

#### 配置GitHub Container Registry
1. 创建GitHub Personal Access Token (PAT):
   - 访问路径：GitHub -> Settings -> Developer settings -> Personal access tokens -> Tokens (classic)
   - 所需权限：
     - `read:packages`
     - `write:packages`
     - `delete:packages`

2. 配置认证信息：
   - 创建`gradle.properties`文件：
     ```properties
     gpr.user=你的GitHub用户名
     gpr.key=你的GitHub Personal Access Token
     ```

#### 构建和推送镜像
```bash
# 构建到本地
./gradlew jibDockerBuild

# 推送到GitHub Container Registry
./gradlew jib
```

## 开发指南

### 1. 添加新功能
1. 确定功能属于认证模块还是业务模块
2. 在相应的controller模块中添加API接口
3. 在service模块中实现业务逻辑
4. 在mybatis模块中添加数据访问代码
5. 如需异步处理，使用producer发送消息，consumer处理消息

### 2. 最佳实践
1. **模块化开发**
   - 保持模块间的低耦合
   - 遵循单一职责原则
   - 通过接口进行模块间通信

2. **安全性**
   - 所有业务接口都应该经过认证
   - 敏感数据加密存储
   - 使用HTTPS进行通信

3. **性能优化**
   - 合理使用Redis缓存
   - 使用消息队列处理异步任务
   - 批量处理大量数据

4. **代码质量**
   - 遵循代码规范
   - 保持测试覆盖率在70%以上
   - 添加适当的注释和文档

## 注意事项
1. 不要将敏感配置（如gradle.properties）提交到Git仓库
2. 开发环境使用SQLite，生产环境使用PostgreSQL
3. 所有模块的修改都需要编写对应的单元测试
4. 定期运行jacocoTestCoverageVerification确保测试覆盖率
5. 在提交代码前运行完整的构建和测试流程

## 项目配置说明

### JUnit & JaCoCo 配置

项目已配置JUnit测试框架和JaCoCo代码覆盖率工具。

#### JUnit配置
- 使用Spring Boot测试启动器：`org.springframework.boot:spring-boot-starter-test`
- 配置位于`build.gradle`
```gradle
dependencies {
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
    testImplementation 'org.springframework.security:spring-security-test'
    testImplementation 'org.springframework.kafka:spring-kafka-test'
}

tasks.named('test') {
    useJUnitPlatform()
}
```

#### JaCoCo配置
- 插件版本：最新版本（通过Spring Boot依赖管理）
- 配置位于`build.gradle`
```gradle
plugins {
    id 'jacoco'
}

subprojects {
    apply plugin: 'jacoco'
    
    tasks.named('test') {
        finalizedBy jacocoTestReport
    }

    jacocoTestReport {
        dependsOn test
        reports {
            xml.required = true
            csv.required = false
            html.outputLocation = layout.buildDirectory.dir('jacocoHtml')
        }
    }

    jacocoTestCoverageVerification {
        violationRules {
            rule {
                limit {
                    minimum = 0.7  // 设置最低覆盖率为70%
                }
            }
        }
    }
}
```

##### JaCoCo使用方法
1. 运行测试并生成覆盖率报告：
```bash
./gradlew test
```

2. 只生成覆盖率报告：
```bash
./gradlew jacocoTestReport
```

3. 验证代码覆盖率：
```bash
./gradlew jacocoTestCoverageVerification
```

4. 查看报告：测试完成后，可以在`build/jacocoHtml`目录下找到HTML格式的详细覆盖率报告。

### Docker镜像构建配置（Jib）

项目使用Google的Jib插件进行Docker镜像构建，并配置了GitHub Container Registry (ghcr.io)作为镜像仓库。

#### Jib基础配置
- 插件版本：3.4.0
- 基础镜像：eclipse-temurin:21-jre
- 配置位于`build.gradle`
```gradle
plugins {
    id 'com.google.cloud.tools.jib' version '3.4.0'
}

subprojects {
    apply plugin: 'com.google.cloud.tools.jib'

    jib {
        from {
            image = 'eclipse-temurin:21-jre'
        }
        to {
            image = "ghcr.io/${findProperty('gpr.user') ?: 'unknown-user'}/${project.name}"
            tags = ['latest', project.version]
            auth {
                username = findProperty('gpr.user') ?: 'unknown-user'
                password = findProperty('gpr.key') ?: ''
            }
        }
        container {
            jvmFlags = ['-Xms512m', '-Xmx512m']
            ports = ['8080']
            environment = [
                'SPRING_PROFILES_ACTIVE': 'prod'
            ]
        }
    }
}
```

#### GitHub Container Registry配置
1. 创建GitHub Personal Access Token (PAT):
   - 访问路径：GitHub -> Settings -> Developer settings -> Personal access tokens -> Tokens (classic)
   - 所需权限：
     - `read:packages`
     - `write:packages`
     - `delete:packages`

2. 配置认证信息：
   - 在项目根目录创建`gradle.properties`文件（已被添加到.gitignore）
   - 文件内容：
     ```properties
     gpr.user=你的GitHub用户名
     gpr.key=你的GitHub Personal Access Token
     ```
   - 参考模板文件：`gradle.properties.example`

#### Jib使用方法
1. 构建镜像到本地Docker daemon：
```bash
./gradlew jibDockerBuild
```

2. 构建并推送到GitHub Container Registry：
```bash
./gradlew jib
```

### 重要文件说明
- `build.gradle`: 主要构建配置文件
- `gradle.properties`: 存储GitHub认证信息（本地文件，不提交到Git）
- `gradle.properties.example`: 配置文件模板（提交到Git）
- `.gitignore`: Git忽略配置，已配置忽略`gradle.properties`

### 注意事项
1. 不要将`gradle.properties`文件提交到Git仓库
2. 首次使用需要根据`gradle.properties.example`创建并配置`gradle.properties`
3. 确保GitHub Token有正确的权限设置
4. JaCoCo覆盖率要求设置为70%，可根据需要在`build.gradle`中调整
