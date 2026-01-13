#PropertySource #Value 

![[Снимок экрана 2023-06-15 в 18.31.58.png]]

```java
@PropertySource(“classpath:application.properties") 
@Configuration 
class DaoConfig { 

@Bean 
public PersonDao personDao() { 
return new PersonDaoSimple();
}
   
}
```

![[Снимок экрана 2023-06-15 в 18.36.05.png]]