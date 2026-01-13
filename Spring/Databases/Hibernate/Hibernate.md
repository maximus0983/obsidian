
Курс Дениса Матвиенко


![[Снимок экрана 2023-08-03 в 1.19.02.png]]
Уточнение! Persistent не обязательно есть уже в БД

Transient -  нет idшника.
Persistent - есть в кэше сущность и есть idшник
Detached - есть idшник, но нет сущности в контексте(кеше).


![[Снимок экрана 2023-08-03 в 8.28.34.png]]

![[Снимок экрана 2023-08-03 в 8.28.54.png]]

![[Снимок экрана 2023-08-03 в 8.29.17.png]]

![[Снимок экрана 2023-08-03 в 8.29.36.png]]



```java
@DisplayName(" должен изменять имя заданного студента по его id")  
@Test  
void shouldUpdateStudentNameById() {  
    val firstStudent = em.find(OtusStudent.class, FIRST_STUDENT_ID);  
    String oldName = firstStudent.getName();  
    //em.detach(firstStudent);  
  
    repositoryJpa.updateNameById(FIRST_STUDENT_ID, STUDENT_NAME);  
    val updatedStudent = em.find(OtusStudent.class, FIRST_STUDENT_ID);  
  
    assertThat(updatedStudent.getName()).isNotEqualTo(oldName).isEqualTo(STUDENT_NAME);  
}  
  
@DisplayName(" должен удалять заданного студента по его id")  
@Test  
void shouldDeleteStudentNameById() {  
    val firstStudent = em.find(OtusStudent.class, FIRST_STUDENT_ID);  
    assertThat(firstStudent).isNotNull();  
    //em.detach(firstStudent);  
  
    repositoryJpa.deleteById(FIRST_STUDENT_ID);  
    val deletedStudent = em.find(OtusStudent.class, FIRST_STUDENT_ID);  
  
    assertThat(deletedStudent).isNull();  
}
```

ВАЖНО!!!!   
 Запросы jpql на обновление(и удаление вроде) идут мимо persistence contexta.  Поэтому рекомендуется делать через jpql запросы только массовые операции.  А по id лучше сделать merge. 

![[Снимок экрана 2023-08-03 в 8.33.11.png]]