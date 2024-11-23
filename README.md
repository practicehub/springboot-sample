# Spring Boot Sample Project

这是一个基于Spring Boot 3.2.1的多模块项目示例，包含认证和业务逻辑两个主要模块。

## 技术栈

- Java 21
- Spring Boot 3.2.1
- Gradle 8.5
- 数据库：SQLite & PostgreSQL
- 缓存：Redis
- 消息队列：Apache Kafka & AWS SQS
- 认证：Spring Security, OAuth2, JWT
- ORM：MyBatis

## 项目结构

```
springboot-sample
├── sample-auth                  // 认证相关模块
│   ├── sample-auth-batch       // 认证相关批处理任务
│   ├── sample-auth-consumer    // 认证消息消费者
│   ├── sample-auth-producer    // 认证消息生产者
│   ├── sample-auth-devtool     // 开发工具
│   ├── sample-auth-controller  // 认证接口控制器
│   ├── sample-auth-mybatis     // 认证数据访问
│   └── sample-auth-service     // 认证业务逻辑
└── sample-business             // 业务逻辑模块
    ├── sample-business-batch       // 业务批处理任务
    ├── sample-business-consumer    // 业务消息消费者
    ├── sample-business-producer    // 业务消息生产者
    ├── sample-business-devtool     // 开发工具
    ├── sample-business-controller  // 业务接口控制器
    ├── sample-business-mybatis     // 业务数据访问
    └── sample-business-service     // 业务逻辑服务
```

## 模块说明

### Auth模块
- **sample-auth-controller**: 提供认证相关的REST API接口，包括登录、注册、OAuth2认证等
- **sample-auth-service**: 实现认证相关的业务逻辑，包括JWT token管理、用户认证等
- **sample-auth-mybatis**: 处理用户认证相关的数据库访问
- **sample-auth-consumer**: 处理认证相关的消息消费
- **sample-auth-producer**: 处理认证相关的消息生产
- **sample-auth-batch**: 处理认证相关的批量任务
- **sample-auth-devtool**: 认证模块的开发工具和辅助功能

### Business模块
- **sample-business-controller**: 提供业务相关的REST API接口
- **sample-business-service**: 实现核心业务逻辑
- **sample-business-mybatis**: 处理业务数据的数据库访问
- **sample-business-consumer**: 处理业务相关的消息消费
- **sample-business-producer**: 处理业务相关的消息生产
- **sample-business-batch**: 处理业务相关的批量任务
- **sample-business-devtool**: 业务模块的开发工具和辅助功能

## 特性

1. **认证支持**
   - JWT Token认证
   - OAuth2认证
   - Session管理
   - 用户权限管理

2. **数据库支持**
   - SQLite（开发环境）
   - PostgreSQL（生产环境）
   - 使用MyBatis进行ORM映射

3. **缓存支持**
   - Redis缓存
   - 分布式Session支持

4. **消息队列**
   - Kafka支持
   - AWS SQS支持
   - 异步消息处理

5. **开发工具**
   - Spring Boot DevTools
   - Spring Boot Actuator
   - 自定义开发工具

## 配置说明

### 数据库配置

#### SQLite配置
```properties
spring.datasource.url=jdbc:sqlite:path/to/database.db
spring.datasource.driver-class-name=org.sqlite.JDBC
```

#### PostgreSQL配置
```properties
spring.datasource.url=jdbc:postgresql://localhost:5432/dbname
spring.datasource.username=username
spring.datasource.password=password
spring.datasource.driver-class-name=org.postgresql.Driver
```

### Redis配置
```properties
spring.redis.host=localhost
spring.redis.port=6379
```

### Kafka配置
```properties
spring.kafka.bootstrap-servers=localhost:9092
spring.kafka.consumer.group-id=your-group-id
```

### AWS SQS配置
```properties
aws.accessKeyId=your-access-key
aws.secretKey=your-secret-key
aws.region=your-region
```

## 构建和运行

1. **构建项目**
```bash
./gradlew clean build
```

2. **运行特定模块**
```bash
./gradlew :sample-auth-controller:bootRun     # 运行认证控制器
./gradlew :sample-business-controller:bootRun  # 运行业务控制器
```

## 开发指南

1. **认证模块开发**
   - 在auth-controller中添加新的认证接口
   - 在auth-service中实现认证逻辑
   - 使用auth-mybatis处理用户数据
   - 使用consumer/producer处理异步认证事件

2. **业务模块开发**
   - 在business-controller中添加新的业务接口
   - 在business-service中实现业务逻辑
   - 使用business-mybatis处理业务数据
   - 使用consumer/producer处理异步业务事件

## 最佳实践

1. **模块化原则**
   - 保持模块间的低耦合
   - 遵循单一职责原则
   - 通过接口进行模块间通信

2. **安全性考虑**
   - 所有业务接口都应该经过认证
   - 敏感数据加密存储
   - 使用HTTPS进行通信

3. **性能优化**
   - 合理使用缓存
   - 异步处理耗时操作
   - 批量处理大量数据

4. **可维护性**
   - 遵循代码规范
   - 添加适当的注释和文档
   - 编写单元测试和集成测试
