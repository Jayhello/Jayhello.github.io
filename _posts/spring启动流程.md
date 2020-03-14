```mermaid
graph TD
A[扫描] --> B[加载类] --> C[包装为BeanDefinition] --> D[将这些BeanDefinition放到一个map中,key为类名,value为BeanDefinition] --> E[调用实现了BeanFactoryPostProcesser类的方法]
```