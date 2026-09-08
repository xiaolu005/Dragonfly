# @Autowired 注入找不到类型时，会不会按名称找？@Resource 是不是也一样？

## 一、先给核心结论

1. **@Autowired**
     默认**优先按类型 ByType 注入**；
   
       按类型找不到 → **直接报错**，**不会自动降级按名称找**。
       只有配合 **@Qualifier** 才会「类型+名称」匹配。

2. **@Resource**
     默认**先按名称 ByName**；
   
       按名称找不到 → **降级按类型 ByType** 再找一遍。
       和 @Autowired 逻辑**完全反过来**。

---

## 二、@Autowired 完整规则

### 规则流程

1. 优先 **根据变量类型** 去 Spring 容器找 Bean
2. 找到**唯一** → 直接注入
3. 找到**多个同类型** → 抛异常 `NoUniqueBeanDefinitionException`
4. **按类型找不到** → 直接报错，**不会拿变量名去匹配 BeanId**

### 想让 @Autowired 按名称匹配

必须加 `@Qualifier("beanId")`

```java
@Autowired
@Qualifier("userServiceImpl")
private UserService userService;
```

变成：**先按类型，再限定名称**。

### 补充：@Autowired 可以 required=false

```java
@Autowired(required = false)
private UserService userService;
```

找不到不报错，直接注入 null，**依然不会按名称兜底**。
找不到不报错，直接注入 null，**依然不会按名称兜底**。
---

## 三、@Resource 完整规则（JSR-250）

### 执行顺序（重点）

1. **第一步：优先按 name 属性 / 变量名** 匹配容器 BeanId
   - 写了 `@Resource(name="xxx")` 就按指定 name
   - 没写 name，默认用**字段名**当 bean 名称去找
2. **按名称找不到** → 自动降级 **按类型 ByType** 再搜一遍
3. 还找不到 → 报错

### 示例

```java
// 先找容器中 id = userService 的bean
// 找不到再按 UserService 类型找
@Resource
private UserService userService;
```

---

## 四、两者核心区别对照表

| 注解         | 匹配顺序          | 类型找不到会按名称兜底吗 | 多同类型处理         |
| ---------- | ------------- | ------------ | -------------- |
| @Autowired | 先**按类型**      | ❌ 不会，直接报错    | 需配合 @Qualifier |
| @Resource  | 先**按名称**，再按类型 | ✅ 会自动降级按类型   | 自动兼容           |

---

## 五、一句话总结

- **@Autowired**：认准**类型**，类型不对直接挂，不自己猜名字。
- **@Resource**：先认**名字**，名字找不到再退而求其次按类型找。
- 两者**查找逻辑完全不一样**，不是同一个机制。

# 标记了@Qualifier("beanId")，但容器中没有这个beanId会怎么样？

## 一、直接结论

**会直接报错，启动失败！**
报错信息：
`NoSuchBeanDefinitionException: No bean named 'xxx' available`

Spring 会**严格按照你指定的名字去查找**，找不到就直接抛异常，**不会再按类型兜底**。

---

## 二、为什么会报错？（核心原理）

`@Autowired + @Qualifier("beanId")` 组合的规则是：

1. **先按类型**找到所有候选Bean
2. **再按你指定的 beanId 精确匹配**
3. **匹配不到 → 直接报错**
4. **绝对不会再退化成按类型自动匹配**

这是 Spring 的**强约束机制**：
你既然明确指定了名字，Spring 就认为你**必须要这个名字的Bean**，找不到就是配置错误。

---

## 三、代码示例（一定会报错）

```java
// 容器中只有 userService1、userService2，没有 userServiceABC
@Autowired
@Qualifier("userServiceABC")  // 指定不存在的beanId
private UserService userService;
```

**启动直接报错：**

```
NoSuchBeanDefinitionException:
No bean named 'userServiceABC' available
```

---

## 四、和 @Resource 对比（非常重要）

### 1. @Autowired + @Qualifier("不存在")

❌ **直接报错，不兜底**

### 2. @Resource(name = "不存在")

❌ **直接报错，不兜底**

### 3. @Resource（不指定name，字段名不存在）

✅ **会自动降级按类型匹配**

---

## 五、一句话总结

1. **只要你手动指定了Bean名称（@Qualifier / @Resource(name=...)）**
   Spring **必须找到完全匹配的Bean**，找不到就**启动失败**。

2. **只有 @Resource 不写name时**，才会先按字段名找 → 找不到再按类型找。

3. **@Autowired 无论任何情况，都不会自动按字段名兜底。**

---

### 最简单记忆

**指定名字 = 强匹配 = 找不到就报错
不指定名字 = 自动匹配 = 会兜底**

# 怎么解决 Spring 循环依赖的问题

## 一、直接给最实用结论

Spring **默认只能解决【单例 + setter 注入】的循环依赖**。
如果出现以下情况，**会直接报错，无法自动解决**：

1. **构造器注入**循环依赖
2. **多例（prototype）**循环依赖
3. **@Async** 导致的循环依赖

解决方式分 **3 种（从最简单到最通用）**：

---

# 二、最简单方案：改用 **setter / field 注入**（最常用）

## 问题原因

**构造器注入 → 循环依赖无法解决**

```java
@Component
public class A {
    // 构造器注入 B
    public A(B b) {}
}

@Component
public class B {
    // 构造器注入 A
    public A(A a) {}
}
```

启动直接报错：
`The dependencies of some of the beans in the application context form a cycle`

## 解决

把**构造器注入改成字段注入 / setter 注入**

```java
@Autowired
private B b;   // 不报错，Spring 自动解决
```

✅ **Spring 三级缓存就是为了解决这个场景**

---

# 三、次简单方案：使用 **@Lazy 懒加载**

不需要改结构，**加个注解就解决**。

原理：先注入一个**代理对象**，真正使用时才初始化。

```java
@Component
public class A {
    @Lazy  // 加这里
    @Autowired
    private B b;
}

@Component
public class B {
    @Autowired
    private A a;
}
```

✅ **构造器循环依赖也能解决**
✅ **最简单、侵入性最小**

---

# 四、最通用方案：使用 **@Autowired(required = false) 弱依赖**

让 Spring 不强制初始化，跳过循环检查。

```java
@Autowired(required = false)
private B b;
```

适合：

- 启动时不需要马上用
- 动态依赖
- 循环依赖无法避免

---

# 五、终极方案：**拆分类，解除循环依赖**（企业标准做法）

真正的架构优化：**把公共部分抽出来**。

例如：
A依赖B，B依赖A
→ 抽一个 **C 工具类/公共Service**
→ A、B 都依赖 C
→ **循环依赖消失**

这是 **最规范、最推荐** 的方案。

---

# 六、Spring 循环依赖总结（必背）

## Spring **能自动解决**

- 单例 Bean
- **字段注入 / setter 注入**

## Spring **不能自动解决**

- **构造器注入循环**
- **多例 Bean**
- **@Async 增强**

## 最快解决顺序

1. **加 @Lazy**（最快）
2. 改成 **字段注入**
3. **拆分类**（最规范）
4. 使用 **required=false**

---

# 七、一句话终极记忆

**构造器循环 → 加 @Lazy
字段循环 → Spring 自动解决
架构问题 → 抽公共类**

---

需要我给你**写一段可直接运行的循环依赖报错代码 + 修复代码**吗？
你一跑就彻底懂原理。

# @Bean 标记的方法参数上可以单独使用 @Qualifier 吗

## 一、直接结论

**可以！完全可以，而且是合法常用写法**  
在 `@Bean` 方法的**方法参数**上，**不用 @Autowired**，直接单独写 `@Qualifier("beanName")` 完全生效。

## 二、原理

Spring 在实例化 @Bean 方法时：

1. 需要给方法**自动注入参数**
2. 参数默认**先按类型找 Bean**
3. 如果同类型多个 Bean，**参数上加 @Qualifier** 就能**按名称精准匹配**
4. 这里**不需要 @Autowired**，因为 @Bean 方法参数本身就是 Spring 自动装配的入参

## 三、正确示例（标准合法）

```java
@Configuration
public class BeanConfig {

    @Bean("user1")
    public User user1(){
        return new User("张三");
    }

    @Bean("user2")
    public User user2(){
        return new User("李四");
    }

    // 参数上单独用 @Qualifier，完全生效
    @Bean
    public UserService userService(
            @Qualifier("user1") User user
    ){
        return new UserService(user);
    }
}
```

效果：
自动从同类型多个 User 里，**强制注入 beanName = user1** 的 Bean。

## 四、为什么这里可以单独用？

- 字段上：`@Qualifier` 依赖 `@Autowired` 触发注入
- **@Bean 方法参数**：Spring 本身就会自动进行依赖注入，
  `@Qualifier` 只做**筛选限定**，不需要再加任何注入注解。

## 五、对比总结

1. **普通成员字段**
   
   ```java
   // 不行，单独@Qualifier无效
   @Qualifier("user1")
   private User user;
   ```
   
   必须搭配 `@Autowired`。

2. **@Bean 方法参数**
   
   ```java
   // 可以，单独@Qualifier合法生效
   @Bean
   public Xxx xxx(@Qualifier("user1") User user){}
   ```

## 六、一句话记住

- 类属性字段：**@Qualifier 不能单独用，必须配 @Autowired**
- @Bean 方法参数：**@Qualifier 可以单独直接用，用来按名称选Bean**

没有人是可以ip