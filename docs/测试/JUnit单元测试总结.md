推荐参考 [SpringBoot2.x 单元测试](http://blinkfox.com/2019/03/02/hou-duan/spring/springboot2.x-dan-yuan-ce-shi/)

## 轻量化MockBean测试 -- 模拟HTTP请求

只需使用`@WebMvcTest(BeMockClassName.class)`和`@RunWith(SpringRunner.class)`即可只加载指定bean进行**轻量化**HTTP请求测试

### JUnit4

```java
/** 使用 JUnit 4 模拟HTTP请求轻量化测试 , 只加载指定bean */
// @WebMvcTest 包含 @AutoConfigureMockMvc 
// 且 JUnit4 中必须配合 @RunWith(SpringRunner.class) 一起使用
@WebMvcTest(AnnotationController.class) // 加载指定bean (轻量化测试,加快测试速度).
@RunWith(SpringRunner.class)
public class AnnotationControllerTest3 {
    // mock 对象
    @Resource // 使用 @AutoConfigureMockMvc 后还需手动注入该bean才可
    MockMvc mvc;
    @MockBean // annotationController 中需要依赖 helloService
    HelloService helloService;
      // ... 以下省略 
}
```

### JUnit5

如使用 JUnit5 , 则 `@RunWith(SpringRunner.class)` 可省略 , 其它照旧

```java
/** JUnit 5 模拟HTTP请求轻量化测试 , 只加载指定bean */
// @WebMvcTest 已包含 @AutoConfigureMockMvc ,@ExtendWith(SpringExtension.class) 
@WebMvcTest(AnnotationController.class) // 指定要测试的bean (轻量化测试,加快测试速度)
class AnnotationControllerTest2 {
    // mock 对象
    @Resource // 使用 @AutoConfigureMockMvc 后还需手动注入该bean才可
    MockMvc mvc;

    @MockBean // annotationController 中需要依赖 helloService
    HelloService helloService;
    // ... 以下省略 
}
```

>  以下为完整测试
>
>  ```java
>  @Slf4j
>  // @WebMvcTest 包含 @AutoConfigureMockMvc , 且 JUnit4 中必须配合 @RunWith(SpringRunner.class) 一起使用
>  @WebMvcTest(AnnotationController.class) // 只加载指定bean及其所依赖bean (轻量化测试,加快测试速度)
>  // When using JUnit 4, this annotation should be used
>  // in combination with @RunWith(SpringRunner.class).
>  @RunWith(SpringRunner.class) 
>  public class AnnotationControllerTest3 {
>   
>    // mock 对象
>   @Resource // 使用 @AutoConfigureMockMvc 后还需手动注入该bean才可
>   MockMvc mvc;
>   
>   @MockBean
>   HelloService helloService; // annotationController依赖helloService, 进行MockBean操作
>   
>   @SneakyThrows
>   @Test
>   public void testWhitelist() {
>  
>       User hello = new User("hello", 12, LocalDateTime.now());
>       hello.toBuilder().build();
>       Mockito.when(helloService.verifyWhitelist()).thenReturn(hello);
>       ResultActions resultActions = mvc.perform(MockMvcRequestBuilders.get("/aop/whitelist")
>               // 使用 MediaType.APPLICATION_JSON_UTF8 避免中文乱码
>               .accept(MediaType.APPLICATION_JSON_UTF8)
>       );
>  
>       MvcResult result = resultActions.andReturn();
>       result.getResponse().setCharacterEncoding("UTF-8");
>       resultActions
>               //.andDo(print())
>               .andExpect(MockMvcResultMatchers.status().isOk())
>               //验证响应contentType
>               //.andExpect(content().contentType(MediaType.APPLICATION_JSON))
>               //.andExpect(content().json("{\"code\":0,\"data\":\"\",\"msg\":\"执行成功\"}"))
>               //使用Json path验证JSON 请参考  http://goessner.net/articles/JsonPath/
>               .andExpect(MockMvcResultMatchers.jsonPath("$.code").value(1000))//;
>               .andExpect(MockMvcResultMatchers.jsonPath("$.data.name").value("hello"))
>               .andDo(MockMvcResultHandlers.print())
>       ;
>       // log.info(result.getResponse().getContentAsString());
>   }
>  }
>   
>  ```



## 全量化化MockBean测试 -- 模拟HTTP请求

只需使用`@SpringBootTest` 和`@AutoConfigureMockMvc`即可只加载指定bean进行**全量化**HTTP请求测试

### JUnit4

如使用 JUnit4 , 则 还需要添加`@RunWith(SpringRunner.class)` , 其它照旧.

```java
/** 使用 JUnit 4 模拟HTTP请求全量加载 bean测试 */
@SpringBootTest/*(classes = TestApplication.class)*/ // 加载所有bean
@AutoConfigureMockMvc
@RunWith(SpringRunner.class)
public class AnnotationControllerTest4 {

    // mock 对象
    @Resource // 使用 @AutoConfigureMockMvc 后还需手动注入该bean才可
    MockMvc mvc;
  
    @MockBean
    HelloService helloService;
    // ... 以下省略 
}
```

### JUnit5

JUnit5 如下

```java
/** JUnit5 模拟HTTP请求全量加载 bean测试 */
// @SpringBootTest已包含注解 @ExtendWith(SpringExtension.class)
@SpringBootTest // 加载所有bean
@AutoConfigureMockMvc
class AnnotationControllerTest {
    // mock 对象
    @Resource // 使用 @AutoConfigureMockMvc 后还需手动注入该bean才可
    MockMvc mvc;
    // ... 以下省略 
}
```

> 以下为完整测试
>
> ```java
> @Slf4j
> @SpringBootTest // 加载所有bean
> @AutoConfigureMockMvc
> // @ExtendWith(SpringExtension.class) // @SpringBootTest已包含该注解
> class AnnotationControllerTest {
>  // mock 对象
>  @Resource // 使用 @AutoConfigureMockMvc 后还需手动注入该bean才可
>  MockMvc mvc;
> 
>  @SneakyThrows
>  @Test
>  void testWhitelist() {
>      ResultActions resultActions = mvc.perform(MockMvcRequestBuilders.get("/aop/whitelist")
>              // 使用 MediaType.APPLICATION_JSON_UTF8 避免中文乱码
>              .accept(MediaType.APPLICATION_JSON_UTF8)
>      );
> 
>      //resultActions.andReturn().getResponse().setCharacterEncoding("UTF-8");
>      resultActions
>              // test fail 时自动会执行打印
>              // .andDo(MockMvcResultHandlers.print())
>              .andExpect(MockMvcResultMatchers.status().isOk())
>              //验证响应contentType
>              //.andExpect(content().contentType(MediaType.APPLICATION_JSON))
>              //.andExpect(content().json("{\"code\":0,\"data\":\"\",\"msg\":\"执行成功\"}"))
>              //使用Json path验证JSON 请参考  http://goessner.net/articles/JsonPath/
>              .andExpect(MockMvcResultMatchers.jsonPath("$.code").value(0));
>      ;
>      // log.info(resultActions.andReturn().getResponse().getContentAsString());
>  }
> 
> 
> }
> ```



## 进行Mock测试 -- 无需模拟HTTP请求

不需要使用MockMvc模拟HTTP请求时

### JUnit4

```java
@RunWith(MockitoJUnitRunner.StrictStubs.class) // JUnit 4 使用
public class UnitTest2 { 
    @Mock // 间接使用@Mock来进行初始化list成员变量
    List list;
    @Mock
    private HelloService helloService;
    @InjectMocks
    private AnnotationController annotationController;

    @Test
    public void unitTest() {
        list.add(100);
        // mock 调用
        User build = User.builder().name("hello").age(10).dateTime(LocalDateTime.now()).build();
        when(helloService.verifyWhitelist(/*anyLong()*/)).thenReturn(build);
        Assert.assertEquals(R.ok(build), annotationController.testWhitelist());
    }
}
```

### JUnit5

```java
@ExtendWith(MockitoExtension.class) // JUnit 5 使用
class UnitTest2 {

    @Mock
    List list;
    @Mock
    private HelloService helloService;
    @InjectMocks
    private AnnotationController annotationController;

    @Test
    void unitTest() {
        list.add(100);
        // mock 调用
        User build = User.builder()
          .name("hello")
          .age(10)
          .dateTime(LocalDateTime.now())
          .build();
        when(helloService.verifyWhitelist(/*anyLong()*/)).thenReturn(build);
        Assert.assertEquals(R.ok(build), annotationController.testWhitelist());
    }
}
```

