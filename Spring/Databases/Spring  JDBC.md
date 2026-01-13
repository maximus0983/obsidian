![[
]]![[Снимок экрана 2023-07-20 в 18.18.13.png]]![[]]
``` java 
private static class PersonMapper implements RowMapper<Person> {  
  
    @Override  
    public Person mapRow(ResultSet resultSet, int i) throws SQLException {  
        long id = resultSet.getLong("id");  
        String name = resultSet.getString("name");  
        return new Person(id, name);  
    }  
}

```



![[Снимок экрана 2023-07-20 в 18.29.29.png]]![[Снимок экрана 2023-07-20 в 18.29.52.png

```java
@Repository  
public class PersonDaoJdbc implements PersonDao {  
  
    private final JdbcOperations jdbc;  
    private final NamedParameterJdbcOperations namedParameterJdbcOperations;  
  
    public PersonDaoJdbc(NamedParameterJdbcOperations namedParameterJdbcOperations) {  
        // Это просто оставили, чтобы не переписывать код  
        // В идеале всё должно быть на NamedParameterJdbcOperations        this.jdbc = namedParameterJdbcOperations.getJdbcOperations();  
        this.namedParameterJdbcOperations = namedParameterJdbcOperations;  
    }  
  
    @Override  
    public int count() {  
        Integer count = jdbc.queryForObject("select count(*) from persons", Integer.class);  
        return count == null ? 0 : count;  
    }  
  
    @Override  
    public void insert(Person person) {  
        namedParameterJdbcOperations.update("insert into persons (id, name) values (:id, :name)",  
                Map.of("id", person.getId(), "name", person.getName()));  
    }  
  
    @Override  
    public Person getById(long id) {  
        Map<String, Object> params = Collections.singletonMap("id", id);  
        return namedParameterJdbcOperations.queryForObject(  
                "select id, name from persons where id = :id", params, new PersonMapper()  
        );  
    }  
  
    @Override  
    public List<Person> getAll() {  
        return jdbc.query("select id, name from persons", new PersonMapper());  
    }  
  
    @Override  
    public void deleteById(long id) {  
        Map<String, Object> params = Collections.singletonMap("id", id);  
        namedParameterJdbcOperations.update(  
                "delete from persons where id = :id", params  
        );  
    }  
  
    private static class PersonMapper implements RowMapper<Person> {  
  
        @Override  
        public Person mapRow(ResultSet resultSet, int i) throws SQLException {  
            long id = resultSet.getLong("id");  
            String name = resultSet.getString("name");  
            return new Person(id, name);  
        }  
    }  
}
```

![[Снимок экрана 2023-07-20 в 22.36.03.png]]

![[Снимок экрана 2023-07-20 в 22.36.32.png]]

![[Снимок экрана 2023-07-20 в 22.37.15.png]]