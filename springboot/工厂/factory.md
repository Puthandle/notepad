# BeanFactory 的那些事儿



```java
public interface BeanFactory {

   // 根据名称获取Bean
   Object getBean(String name) throws BeansException;

   // 根据名称和类型获取Bean
   <T> T getBean(String name, Class<T> requiredType) throws BeansException;

   // 根据名称获取Bean 使用给定的构造函数参数或者工厂方法参数构造对象并返回，会重写Bean定义中的默认参数。
   //  PS：该方法只适用于prototype的Bean，默认作用域的Bean不能重写其参数。
   Object getBean(String name, Object... args) throws BeansException;

   // 根据类型获取Bean
   <T> T getBean(Class<T> requiredType) throws BeansException;

   // 使用Bean名称寻找对应的Bean，使用给定的构造函数参数或者工厂方法参数构造对象并返回，会重写Bean定义中的默认参数。
   //  PS：该方法只适用于prototype的Bean，默认作用域的Bean不能重写其参数。
   <T> T getBean(Class<T> requiredType, Object... args) throws BeansException;


   <T> ObjectProvider<T> getBeanProvider(Class<T> requiredType);


   <T> ObjectProvider<T> getBeanProvider(ResolvableType requiredType);

	// 是否包含Bean
    boolean containsBean(String name);

	// 是否是单例Bean
   	boolean isSingleton(String name) throws NoSuchBeanDefinitionException;


	// 是否是多例Bean
    boolean isPrototype(String name) throws NoSuchBeanDefinitionException;


    boolean isTypeMatch(String name, ResolvableType typeToMatch) throws NoSuchBeanDefinitionException;


    boolean isTypeMatch(String name, Class<?> typeToMatch) throws NoSuchBeanDefinitionException;

	// 获取类型
   	@Nullable
   	Class<?> getType(String name) throws NoSuchBeanDefinitionException;

	// 获取类型
    @Nullable
    Class<?> getType(String name, boolean allowFactoryBeanInit) throws NoSuchBeanDefinitionException;

	// 获取别名
   	String[] getAliases(String name);

}
```