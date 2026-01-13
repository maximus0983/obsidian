Í![[Pasted image 20230701185731.png]]

1.  

2.  Перед созданием компонентов, но после создания BeanDefinition-ов  есть этап BeanFactoryPostProcessor. Там в BeanFactory есть все  BeanDefinition. Можно кстати регистрировать свои Scope в ней.
   Также можно в BeanFactoryPostProcessor подложить свои BeanDefinition или изменить текущие.  Для бизнес-логики вряд ли пригодится это всё когда-то, но может позволить расширить инфраструктурные возможности фреймворка. 
3.  -
4. Этим занимается спринговый постпроцессор. 
5.  Set bean name, и прочая залупа.  Для бизнес логики aware интерфейсы лучше не использовать, а для расширения возможностей самого спринга можно. 
6.  По идее к этому моменту бин уже существует, и зависимости тоже в нем уже созданы. Для справки: 
   
   Initialization: Initialization is the process of preparing the bean for use after it has been instantiated. It involves setting the bean's properties, performing any necessary dependency injection, and **executing any initialization callbacks or custom initialization methods.** This phase occurs after the bean has been instantiated, and it ensures that the bean is in a valid and usable state.
   Теоретически можно вообще другой бин вернуть. И даже прокси самим замутить тут.
   
   
7.  The `afterPropertiesSet` method is an optional callback method defined in the `InitializingBean` interface. It is invoked by the Spring container after the bean's properties have been set through dependency injection but **before the bean is considered fully initialized and ready for use.**
    When the Spring container creates a bean, it first instantiates the bean and then performs dependency injection to set its properties. After the properties are set, the container checks if the bean implements the `InitializingBean` interface. If it does, the container calls the `afterPropertiesSet` method to allow the bean to perform any additional initialization tasks.
     So, at the time the `afterPropertiesSet` method is called, the bean instance already exists and has its properties set. The purpose of the `afterPropertiesSet` method is to provide a hook for the bean to perform any initialization logic or validation based on its dependencies and properties.
     
     PostConstruct работает до того, как все прокси настроились , включая Transactional. Поэтому прогрев кэша там рано делать. 
     PostConstruct пизже тем, что это не спринговая аннотация, а джава ЕЕ. 
     НАХУЯ ЭТОТ ИНИТ МЕТОД НУЖЕН ?  Евгений Борисов в потрошителе рассказывает про двухфазный конструктор.  https://youtu.be/BmBr5diz8WA?feature=shared&t=1129
     Кратко говоря, в констркуторе нельзя использовать штуки, которые спринг настраивает, потому что рано еще. Поэтому инит метод еще добавляем, чтобы там доделать то, что в конструкторе не можем. 
     
8.  После того, как все postProcessAfterInitialization методы вызваны,  ApplicationContext считается полностью готовым, и может принимать всякие запросы, и работать с events. 
   Зачем нам 2 метода постпроцессора?
   По конвенциям Спринга те бин пост процессоры, которые что-то меняют в классе должны это делать на этапе afterInitialization.  таким образом мы знаем что post construct  работает на оригинальный метод , до того как всякие прокси накрутились на него. Борисов в примере использует его для того, чтобы на метод оригинального бина накрутить логику(профилирование по времени показывать)
9.  Я так понял в этот момент бин в контейнер попадает
10. 
11.  Для prototype дестрой метод никогда не работает, т.к спринг их не хранит нигде, а синглтоны хранит в контексте
   

![[Снимок экрана 2023-07-09 в 2.03.31.png]]



Зачем 2  метода постпроцессора? Ответ:
По конвенциям Спринга те бин пост процессоры, которые что-то меняют в классе должны это делать на этапе afterInitialization.таким образом мы знаем что post construct  работает на оригинальный метод , до того как всякие прокси накрутились на него. Борисов в примере использует его для того, чтобы на метод оригинального бина накрутить логику(профилирование по времени показывать)