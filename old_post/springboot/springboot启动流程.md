### 1 框架初始化

1. 配置资源加载器
2. 配置primarySources，启动类
3. 应用环境检测
4. 配置系统初始化器
5. 配置应用监听器
6. 配置main方法所在类

### 2 框架启动

1. 开始计时
2. headless模式赋值
3. 发送ApplicationStartingEvent
4. 配置环境模块
5. 发送ApplicationEnviromentPreparaeEvent
6. 打印banner
7. 创建应用上下文对象
8. 初始化失败分析器
9. 关联springboot组件与应用上下文对象
10. 发送ApplicationContextInitializedEvent
11. 加载sources
12. 发送ApplicationPreparedEvent
13. 刷新上下文
14. 计时器停止
15. 发送ApplicationStartedEnent
16. 调用框架启动扩展类
17. 发送ApplicationReadyEvent

### 3 自动化装配

1. 手机配置文件中的配置工厂类
2. 加载组件公尺长
3. 注册组件内定义bean