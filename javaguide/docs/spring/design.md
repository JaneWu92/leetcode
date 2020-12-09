
* ApplicationContext hierarchy
    * BeanFactory
    * HierarchicalBeanFactory, ListableBeanFactory
    * ApplicationContext
    * ConfigurableApplicationContext
    * AbstractApplicationContext
    * GenericApplicationContext
    * AnnotationConfigApplicationContext
    

* AnnotationConfigApplicationContext
    * 根据componentclass，拿reader读了内容
    * refresh() //这个函数在AbstractApplicationContext里
        
* AbstractApplicationContext
    * refresh
        * 做很多前期准备
        * finishBeanFactoryInitialization(beanFactory); 
        * //beanfactory的初始化。这里面就包括把所有的singleton的都初始化放缓存池里
        * //容易理解。因为beanfactory本身就是产生bean的工厂，提供getbean的功能。当然得先把它自己的东西（bean）准备好
            * beanFactory.preInstantiateSingletons();
                * DefaultListableBeanFactory#preInstantiateSingletons
                //作为一个beanfactory，把自己所有的singleton都预先实例化，是很make sense的


* DefaultListableBeanFactory
    * preInstantiateSingletons
        * for (String beanName : beanNames) getBean(beanName)
        //也就是说，作为一个beanFactory，我要预实例化singleton
        //只要去调用我的getBean方法。也就是说，这个getBean给外人用，也给内人用
        //作为一个beanFactory，你肯定对外有暴露getBean方法，无论你怎么做，结果肯定是拿到这个bean
            * 这个getBean方法是定义在AbstractBeanFactory里的
            //容易理解，getBean这么纯正的方法，当然要定义在很高的父类方法里面       
                * AbstractBeanFactory#doGetBean       
                    
* AbstractBeanFactory
    * doGetBean                  
        * getSingleton(beanName)                
            * 拿到：无论是半成品还是成品           
            * 没拿到。就要去开启。。。
                * getSingleton(beanName, ObjectFactory)
                    * 这个ObjectFactory就是用来产生这个bean的
                        * createBean
                            * AbstractAutowireCapableBeanFactory#createBean()
                            * AutowireCapable就是说用这个factory生成的类，fields也会被填充进去
                                * doCreateBean
                    
                    
* AbstractAutowireCapableBeanFactory
    * doCreateBean
       // 1: 实例化对象
       * instanceWrapper = createBeanInstance()
       * bean = instanceWrapper
       // 2: 暴露early reference。为循环引用做准备
       * addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean));
          *                    
       // 3: populate成员变量
       * populateBean(beanName, mbd, instanceWrapper);
       //4: 调用初始化函数
       * exposedObject = initializeBean(beanName, exposedObject, mbd);             
                    
                    
                    
                    
                    
                    
                    
                    
                    
                    
                    
                    
                    
                    
                    
                    
                    
                    
                    
                    
                    
                    
                    
                    
                    
                    
                
                