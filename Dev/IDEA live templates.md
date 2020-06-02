## IDEA live templates
Tags: idea live templates

### Тесты JPA

Template for generate Jpa repository test

```xml
<template name="jpatest" value="companion object {&#10;    private const val ENTITY_ID = &quot;fakeId&quot;&#10;}&#10;&#10;@Autowired&#10;lateinit var entityManager: TestEntityManager&#10;&#10;@Autowired&#10;lateinit var $VARIABLE_NAME$: $CLASS_NAME$&#10;&#10;@BeforeEach&#10;fun setUp() {&#10;&#10;    val entity =&#10;        $ENTITY_CALSS$(&#10;            id = ENTITY_ID,&#10;            createTime = Instant.now(),&#10;            createUser = &quot;fakeUser&quot;,&#10;            lastModifyTime = Instant.now(),&#10;            lastModifyUser = &quot;fakeUser&quot;,&#10;            partitionKey = generatePartitionKey()&#10;        )&#10;    entityManager.persistAndFlush(entity)&#10;    entityManager.clear()&#10;}&#10;&#10;@Test&#10;fun `should return an entity by id`() {&#10;    val response = $VARIABLE_NAME$.findById(ENTITY_ID)&#10;&#10;    val entity = response.get()&#10;    assertThat(entity).isNotNull&#10;    assertThat(entity.id).isEqualTo(ENTITY_ID)&#10;}" toReformat="false" toShortenFQNames="true">
  <variable name="CLASS_NAME" expression="className()" defaultValue="" alwaysStopAt="true" />
  <variable name="VARIABLE_NAME" expression="camelCase(CLASS_NAME)" defaultValue="" alwaysStopAt="true" />
  <variable name="ENTITY_CALSS" expression="" defaultValue="" alwaysStopAt="true" />
  <context>
    <option name="KOTLIN_CLASS" value="true" />
  </context>
</template>
```

Example

```kotlin
@ExtendWith(SpringExtension::class)
@DataJpaTest
class MetaScreenRepositoryTest {

    companion object {
        private const val SCREEN_ID = "ID"
    }

    @Autowired
    lateinit var entityManager: TestEntityManager

    @Autowired
    lateinit var metaScreenRepository: MetaScreenRepository

    @BeforeEach
    fun setUp() {
        val newEntity = MetaScreenEntity(
            screenId = SCREEN_ID,
            screenLoadBase = "",
            jsonArbitrary = "",
            note = "test note",
            createTime = Instant.now(),
            createUser = "TestUser",
            lastModifyTime = Instant.now(),
            lastModifyUser = "TestUser"
        )
        entityManager.persistAndFlush(newEntity)
        entityManager.clear()
    }

    @Test
    fun `should return an entity`() {
        val response = metaScreenRepository.findById(SCREEN_ID)


        val entity = response.get()
        Assertions.assertThat(entity).isNotNull
        Assertions.assertThat(entity.screenId).isEqualTo(SCREEN_ID)
    }
}
```



| Ключ                            | Значение по умолчанию | Описание                                           |
| ------------------------------- | --------------------- | -------------------------------------------------- |
| omni-logging.enable-request-log | False                 | Включение/отключение логирования входящих запросов |



### New spring bean



```kotlin
@org.springframework.context.annotation.Bean
fun $BEAN_NAME$($BEAN_DEPS$): $BEAN_CLASS$ {
    return $BEAN_CLASS$($END$)
}
```

| var         | expression               |
| ----------- | ------------------------ |
| $BEAN_NAME$ | decapitalize(BEAN_CLASS) |



