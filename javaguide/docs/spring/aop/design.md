### 原理
* 就是在initializeBean的时候，通过BeanPostProcessor去做
* 它的这个processor是AnnotationAwareAspectJAutoProxyCreator
* 然后它是AbstractAutoProxyCreator的子类，而AbstractAutoProxyCreator是对SmartInstantiationAwareBeanPostProcessor的实现
    * 也就是说，解决循环依赖那边，会先暴露代理过后的bean
* AbstractAutoProxyCreator
    * postProcessAfterInitialization
        * if (this.earlyProxyReferences.remove(cacheKey) != bean) 
        * return wrapIfNecessary(bean, beanName, cacheKey);
* 
