### BeanFactoryPostProcessor
* 跟BeanPostProcessor差不多，就是让你在所有的bean初始化前，能对这些元数据做些修改。
* 与BeanPostProcessor的差别：
    * 一个是在启动的时候，可以改所有的bean的配置
    * 一个是在bean的初始化过程中，对当前这个bean做点什么增强
* AbstractApplicationContext: refresh: invokeBeanFactoryPostProcessors(beanFactory);
* 相当于一个钩子。
* 应该是模板模式把
