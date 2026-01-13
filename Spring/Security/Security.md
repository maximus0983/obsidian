![[Снимок экрана 2023-10-01 в 16.08.07.png]]![[Снимок экрана 2023-10-01 в 16.09.32.png]]

HTTP Basic/ DIgest -  мерзкое окошко в браузере.
Form-based authentication - красивое окошко в браузере.
HTTP X.509 - по приватному ssh ключу (вроде)
OpenId/OAuth - когда мы аутентификацию отдаем внешним сервисам
LDAP - 






![[Снимок экрана 2024-01-12 в 07.16.42.png]]


![[Снимок экрана 2024-01-12 в 07.15.54.png]]


В конце фильтра кладем в контекстхолдер инфу:
SecurityContextHolder.getContext().authentication = UserAuthentication(calculatorUser)

calculatorUser наследуется от спрингового класса User. Потом в коде достаем из SecurityContextHolder этого юзера и его роли/инфу, когда нам надо.

