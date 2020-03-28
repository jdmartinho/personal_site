---
title: Creating Web Integration Tests for Spring Boot REST Controller
date: "2018-04-29T10:48:07+00:00"
template: "post"
draft: false
slug: "creating-web-integration-tests-for-spring-boot-rest-controller"
category: "Software Engineering"
tags:
    - "API"
    - "Controller"
    - "Java"
    - "REST"
    - "Spring"
    - "Spring Boot"
    - "Web Integration Test"
description: "I’ve been working on a simple REST API for a CRUD backend application using Spring Boot. Nothing revolutionary, but it’s simply a new way of writing less code that I haven’t tried before."
---

I’ve been working on a simple REST API for a CRUD backend application using Spring Boot. Nothing revolutionary, but it’s simply a new way of writing less code that I haven’t tried before.

[Spring Boot](https://projects.spring.io/spring-boot/) is part of the Spring web application framework. The beauty of the Spring Boot project lies in using [convention over configuration](https://en.wikipedia.org/wiki/Convention_over_configuration) (similar to Ruby on Rails) to speed up development of web applications. It’s fast to run, you don’t need to worry about containers since the application has Tomcat embedded, and you can write a REST API in just a few lines of code with minimal configuration.

In this article, I will not go in depth into how to configure a Spring Boot application from the beginning. I will focus on creating a Controller class that will expose a REST API and how to write web integration tests to make sure it’s working as intended. I’ll be using Spring Boot version 2.0.1.

#### Controller Class

In my simple application, I’m just making a [CRUD](https://en.wikipedia.org/wiki/Create,_read,_update_and_delete) interface for a Fabric object. I want to expose five methods to start: listing all fabrics, creating a new fabric, retrieving a specific fabric, updating a specific fabric and deleting a specific fabric.

```java
@RestController
@RequestMapping("api/v1/")
public class FabricController {
    @Autowired
    private FabricRepository _fabricRepository;
}
```

In Spring Boot you can create a REST Controller class by using the ***@RestController*** annotation. I also added the ***@RequestMapping*** annotation to give a prefix of **“api/v1/”** to all actions which I’m mapping on each method.

The next thing to do is to create our interface to a **JpaRepository** and use the ***@AutoWired*** annotation to [inject our dependency](https://en.wikipedia.org/wiki/Dependency_injection) via the constructor.

```java
public interface FabricRepository extends JpaRepository<Fabric, Long> {
}
```

The repository will be responsible for allowing access to the data source. To keep this post short I’ll not show how to create a data source but you can read more about it [here](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-sql.html#boot-features-configure-datasource).

Now that we can access data the first action to map is a way to list all fabrics in the database. We can do this by simply calling the findAll() method on the repository. Notice the ***@RequestMapping*** annotation to specify the request method and the last part of the URL. We’ll follow this pattern for all methods.

```java
@RequestMapping(value = "fabrics", method = RequestMethod.GET)
public List<Fabric> list() {
    return _fabricRepository.findAll();
}
```

We can create a new Fabric object by doing a POST to the same endpoint. The method will return the object.

```java
@RequestMapping(value = "fabrics", method = RequestMethod.POST)
public Fabric create(@RequestBody Fabric fabric) {
    return _fabricRepository.saveAndFlush(fabric);
}
```

To get a specific object we access it by ID.

```java
@RequestMapping(value = "fabrics/{id}", method = RequestMethod.GET)
public Fabric get(@PathVariable Long id) {
    return _fabricRepository.getOne(id);
}
```

To update a specific object we retrieve it and then copy the properties of the passed in object to the retrieved one.

```java
@RequestMapping(value = "fabrics/{id}", method = RequestMethod.PUT)
public Fabric update(@PathVariable Long id, @RequestBody Fabric fabric) {
    Fabric existingFabric = _fabricRepository.getOne(id);
    BeanUtils.copyProperties(fabric, existingFabric);
    return _fabricRepository.saveAndFlush(existingFabric);
}
```

Finally, we also want to be able to delete a specific object.

```java
@RequestMapping(value = "fabrics/{id}", method = RequestMethod.DELETE)
public Fabric delete(@PathVariable Long id) {
    Fabric existingFabric = _fabricRepository.getOne(id);
    _fabricRepository.delete(existingFabric);
    return existingFabric;
}
```

This concludes our Controller class and our five methods. As you can see we can implement a very simple CRUD API in just over 30 lines of code.

#### Web Integration Test Class

Now I want to make sure that the API is working as intended. In order to do that I can write some web integration tests.

The first thing to do is to create my class.

```java
@RunWith(SpringJUnit4ClassRunner.class)
@SpringBootTest(webEnvironment=SpringBootTest.WebEnvironment.RANDOM_PORT)
@DirtiesContext(classMode = DirtiesContext.ClassMode.BEFORE_EACH_TEST_METHOD)
public class FabricControllerWebIntegrationTest {

    @Autowired
    private TestRestTemplate _restTemplate;

    private long _idToTest = 1L;
    private String _nameToTest = "jersey red";
    private String _materialToTest = "jersey";
    private float _lengthToTest = 100f;
}
```

We use the ***@RunWith*** annotation to specify that we’ll be running this class with the Spring JUnit class runner. The ***@SpringBootTest*** explicitly sets this a test class for our application. Finally, the ***@DirtiesContext*** forces our tests to run with a clean database each time so that we don’t have to clean it ourselves.

As seen before, we can use the ***@Autowired*** annotation to inject objects. In this case, we can inject a **TestRestTemplate** which will allow us to make calls to our API.

We’re also setting a few private variables that will be used in all test methods when creating a Fabric object.

Our first test is to make sure that, on an empty database, we retrieve an empty list of objects. Notice the ***@Test*** annotation to mark this method as a runnable test. We can call the **getForEntity** method on our **TestRestTemplate** object and then inspect the **ResponseEntity** object to make sure the Status Code is set to 200 and that the list is indeed empty.

```java
@Test
public void testListAll() throws IOException {
    ResponseEntity<String> response = _restTemplate.getForEntity("/api/v1/fabrics", String.class);

    assertThat(response.getStatusCode(), equalTo(HttpStatus.OK));

    ObjectMapper objectMapper = new ObjectMapper();
    JsonNode responseJson = objectMapper.readTree(response.getBody());

    assertThat(responseJson.isMissingNode(), is(false));
    assertThat(responseJson.toString(), equalTo("[]"));
}
```

In order to test if we can create a new object, we’ll follow a similar pattern. We create the object, pass it in our request, which this time is using the POST method, and then inspect the response to see if all our parameters are the same. Remember, our API methods always retrieve an object.

```java
@Test
public void testCreateNewFabric() {
    Fabric fabricForTest = new Fabric(_idToTest, _nameToTest, _materialToTest, _lengthToTest);
    ResponseEntity<Fabric> response = _restTemplate.postForEntity("/api/v1/fabrics", fabricForTest, Fabric.class);

    assertThat(response.getStatusCode(), equalTo(HttpStatus.OK));
    assertEquals(_idToTest, response.getBody().get_id().longValue());
    assertEquals(_nameToTest, response.getBody().get_name());
    assertEquals(_materialToTest, response.getBody().get_material());
    assertEquals(_lengthToTest, response.getBody().get_length(), 0f);
}
```

To test retrieving a specific object we’ll start by creating one and then finding it. We could have used mocking in this test but I wanted to keep these tests mock free so that the real API was always used.

```java
@Test
public void testGetById() {
    Fabric fabricForTest = new Fabric(_idToTest, _nameToTest, _materialToTest, _lengthToTest);

    ResponseEntity<Fabric> response = _restTemplate.postForEntity("/api/v1/fabrics", fabricForTest, Fabric.class);

    assertThat(response.getStatusCode(), equalTo(HttpStatus.OK));
    assertEquals(_idToTest, response.getBody().get_id().longValue());
    assertEquals(_nameToTest, response.getBody().get_name());
    assertEquals(_materialToTest, response.getBody().get_material());
    assertEquals(_lengthToTest, response.getBody().get_length(), 0f);

    response = _restTemplate.getForEntity("/api/v1/fabrics/1", Fabric.class);

    assertThat(response.getStatusCode(), equalTo(HttpStatus.OK));
    assertEquals(_idToTest, response.getBody().get_id().longValue());
    assertEquals(_nameToTest, response.getBody().get_name());
    assertEquals(_materialToTest, response.getBody().get_material());
    assertEquals(_lengthToTest, response.getBody().get_length(), 0f);
}
```

Now we want to test if we can update an existing object. We’ll have to create it first, before calling our update method.

```java
@Test
public void testUpdateById() {
    Fabric fabricForTest = new Fabric(_idToTest, _nameToTest, _materialToTest, _lengthToTest);
    ResponseEntity<Fabric> response = _restTemplate.postForEntity("/api/v1/fabrics", fabricForTest, Fabric.class);

    assertThat(response.getStatusCode(), equalTo(HttpStatus.OK));
    assertEquals(_idToTest, response.getBody().get_id().longValue());
    assertEquals(_nameToTest, response.getBody().get_name());
    assertEquals(_materialToTest, response.getBody().get_material());
    assertEquals(_lengthToTest, response.getBody().get_length(), 0f);

    fabricForTest.set_name("updated name");
    HttpEntity<Fabric> requestEntity = new HttpEntity<>(fabricForTest);
    ResponseEntity<Fabric> responsePut = _restTemplate.exchange("/api/v1/fabrics/1", HttpMethod.PUT, requestEntity, Fabric.class);

    assertThat(responsePut.getStatusCode(), equalTo(HttpStatus.OK));

    assertEquals(_idToTest, responsePut.getBody().get_id().longValue());
    assertEquals("updated name", responsePut.getBody().get_name());
}
```

Finally, we’ll test if we can delete an existing object.

```java
@Test
public void testDeleteById() {
    Fabric fabricForTest = new Fabric(_idToTest, _nameToTest, _materialToTest, _lengthToTest);
    ResponseEntity<Fabric> response = _restTemplate.postForEntity("/api/v1/fabrics", fabricForTest, Fabric.class);

    assertThat(response.getStatusCode(), equalTo(HttpStatus.OK));
    assertEquals(_idToTest, response.getBody().get_id().longValue());
    assertEquals(_nameToTest, response.getBody().get_name());
    assertEquals(_materialToTest, response.getBody().get_material());
    assertEquals(_lengthToTest, response.getBody().get_length(), 0f);

    ResponseEntity<Fabric> responseDelete = _restTemplate.exchange("/api/v1/fabrics/1", HttpMethod.DELETE, null, Fabric.class);

    assertThat(responseDelete.getStatusCode(), equalTo(HttpStatus.OK));

    assertEquals(_idToTest, responseDelete.getBody().get_id().longValue());
}
```

So there you have. We’ve implemented and tested a simple CRUD interface using Spring Boot.