## 0x01. Bean的生命周期

> 3和5需要自己写一个类来实现BeanPostProcessor接口，并重写接口中的两个方法。

1. 通过构造器创建bean实例（无参构造器）
2. 为bean的属性设置值和对其他bean的引用（调用set方法）
3. 把bean实例传递bean后置处理器的方法postProcessBeforeInitialization
4. 调用bean的初始化方法（需要进行配置初始化的方法）
5. 把bean 实例传递bean后置处理器的方法postProcessAfterInitialization
6. bean可以使用了（获取到了对象）
7. 当容器关闭时，调用bean的销毁的方法（需要进行配置销毁的方法）

## 0x02. 注解属性注入

1. `@Autowired`：根据属性类型进行自动装配
2. `@Qualifier`：根据属性名称进行注入
3. `@Resource`：可以根据类型注入，也可以根据名称注入
4. `@Value`：注入普通类型属性

