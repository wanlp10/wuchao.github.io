---
layout: post
title: JPA Criteria 
category : [JavaEE]
tagline: "Supporting tagline"
tags : [Java, JPA, Criteria, Querydsl]
---
{% include JB/setup %}
# JPA Criteria 查询 

> [JPA criteria 查询: 类型安全与面向对象](http://blog.csdn.net/dracotianlong/article/details/28445725) 



### 元模型（Metamodel）

> [Hibernate JPA 2 Metamodel Generator](http://docs.jboss.org/hibernate/jpamodelgen/1.0/reference/en-US/html_single/) 



### JPA Criteria Multiselect 

``` 
@Entity
public class Advisor extends AbstractAuditingEntity implements Serializable{
	private static final long serialVersionUID = 1L;

    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "sequenceGenerator")
    @SequenceGenerator(name = "sequenceGenerator")
    private Long id;

	@NotNull
    @Column(name = "name", nullable = false)
    private String name;
    
    @NotNull
    @Column(name = "activated", nullable = false)
    private Boolean activated;

  	@OneToMany(mappedBy = "advisor", fetch = FetchType.LAZY)
    private Set<Product> products = new HashSet<>();

    @OneToMany(mappedBy = "advisor", fetch = FetchType.LAZY)
    @OrderBy("lastModifiedDate desc")
    private Set<Manager> managers = new HashSet<>();
    
    //...
}
```

``` 
@Entity
public class Product extends AbstractAuditingEntity implements Serializable {
  	private static final long serialVersionUID = 1L;

    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "sequenceGenerator")
    @SequenceGenerator(name = "sequenceGenerator")
    private Long id;
    
    @NotNull
    @Column(name = "name", nullable = false)
    private String name;
    
    @NotNull
    @Column(name = "rate", nullable = false)
    private Double rate;
    
    @ManyToOne(fetch = FetchType.LAZY)
    private Advisor advisor;
    
    //...
}
```

``` 
@Entity
public class Manager extends AbstractAuditingEntity implements Serializable {
  	private static final long serialVersionUID = 1L;

    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "sequenceGenerator")
    @SequenceGenerator(name = "sequenceGenerator")
    private Long id;

    @NotNull
    @Column(name = "name", nullable = false)
    private String name;
    
    @ManyToOne(fetch = FetchType.LAZY)
    private Advisor advisor;
    
    //...
}
```

### multi select 查询
现在要查询所有 Advisor 对象,并且查出 Advisor 对象中 managers 集合的个数和 products 集合中 rate 属性的最大值: 

定义一个 DTO 对象,保存查询结果

``` 
public class AdvisorDto implements Serializable {
  private Long id;
  private String name;
  private Long managerNum;
  private Double maxRate;
  
  //multiselect查询语句返回值需要参数个数和类型都匹配的一个构造方法
  public AdvisorDto(Long id, String name, Long managerNum, Double maxRate) {
    this.id = id;
    this.name = name;
    this.managerNum = managerNum;
    this.maxRate = maxRate;
  }
  //...
}
```

定义查询接口和实现类

``` 
public interface AdvisorDtoRepository {
  Page<AdvisorDto> findAll(Boolean activated, String name, Pageable pageable);
}
```

``` 
public class AdvisorDtoRepositoryImpl implements AdvisorDtoRepository {

	@PersistenceContext
    private EntityManager em;
    
  	public Page<AdvisorDto> findAll(Boolean activated, String name, Pageable pageable) {
    	CriteriaBuilder cb = em.getCriteriaBuilder();
        CriteriaQuery<AdvisorDto> multiselectQuery = cb.createQuery(AdvisorDto.class);
        //定义查询根的实体类型
        Root<Advisor> root = multiselectQuery.from(Advisor.class);
        
        /*创建 select 部分要查询的属性,并分别定义别名*/
        Selection idSelection = root.get("id").alias("id");
        Selection nameSelection = root.get("name").alias("name");
        Selection managerNumSelection = cb.count(root.join("managers", JoinType.LEFT)).alias("managerNum");
        Selection maxRateSelection = cb.max(root.join("products", JoinType.LEFT).get("rate")).alias("maxRate");

        multiselectQuery.multiselect(idSelection, nameSelection, managerNumSelection, maxRateSelection);

		/*创建 where 字句的查询条件*/
        List<Predicate> predicates = new ArrayList<>();

        if (activated != null) {
            predicates.add(cb.equal(root.get(Advisor_.activated), activated));
        }
        if (!StringUtils.isEmpty(name)) {
            predicates.add(cb.like(root.get(Advisor_.name), "%" + name + "%"));
        }

        Predicate wherePredicate = cb.and(predicates.toArray(new Predicate[0]));

        //(Column "Advisor_.ID" must be in the GROUP BY list)   	multiselectQuery.where(wherePredicate).groupBy(root.get(Advisor_.id)).orderBy(cb.desc(root.get(Advisor_.lastModifiedDate)));
        
        /*获取查询结果*/
        List<AdvisorDto> results = em.createQuery(multiselectQuery)
            .setFirstResult(pageable.getOffset())
            .setMaxResults(pageable.getPageSize())
            .getResultList();

		/*查询总元素的个数*/
        CriteriaQuery<Long> countQuery = cb.createQuery(Long.class);
        countQuery.select(cb.count(countQuery.from(Advisor.class)));
        countQuery.where(wherePredicate);
        Long totalElements = em.createQuery(countQuery).getSingleResult();

		//创建 Page 对象
        Page<AdvisorDto> advisors = new PageImpl<AdvisorDto>((totalElements > pageable.getOffset() ? results : Collections.<AdvisorDto>emptyList()), pageable, totalElements);

        return advisors;
  	}
}
```



### Spring Data Jpa 的 Specification 查询 

> [SpringDataJpa 的 Specification 查询](http://blog.csdn.net/baijunzhijiang_01/article/details/51557125)  

Spring Data JPA 支持 JPA2.0 的 Criteria 查询，相应的接口是 JpaSpecificationExecutor。Criteria 查询：是一种类型安全和更面向对象的查询 。

这个接口基本是围绕着 Specification 接口来定义的， Specification 接口中只定义了如下一个方法：

``` 
Predicate toPredicate(Root<T> root, CriteriaQuery<?> query, CriteriaBuilder cb);  
```



Specification 查询: 

``` 
public class AdvisorSpecs {
  public static Specification<Advisor> findAdvisors( final Boolean activated, final String name, final Long managerNum) {
    return (root, query, cb) -> {
    	List<Predicate> predicates = new ArrayList<>(); 
    
    	if (managerNum != null) {   		query.groupBy(root.get(Advisor_.id)).orderBy(cb.desc(root.get(Advisor_.lastModifiedDate))).having(cb.greaterThan((cb.countDistinct(root.join(Advisor_.managers, JoinType.LEFT))), managerNum));
        }
        if (activated != null) {
        	predicates.add(cb.equal(root.get(Advisor_.activated), activated));
        }
    	if (!StringUtils.isEmpty(name){
    		predicates.add(cb.like(root.get(Advisor_.name), "%" + name + "%"));
    	}
  
  		return cb.and(predicates.toArray(new Predicate[0]));
  	}
  }
}
```

Spring Data Jpa 查询: 

``` 
@GetMapping("/advisors")
public String index(@RequestParam(value = "activated", required = false, defaultValue = "") Boolean activated,
@RequestParam(value = "name", required = false, defaultValue = "") String name,
@RequestParam(value = "managerNum", required = false, defaultValue = "") Long managerNum,
@PageableDefault(sort = "lastModifiedDate", direction = Sort.Direction.DESC) Pageable pageable, Model model) {
	Specification<Advisor> specification = 	Specifications.where(AdvisorSpecs.findAdvisors(activated, name, managerNum));

	Page<Advisor> advisors = advisorRepository.findAll(specification, pageable);

	model.addAttribute("activated", activated);
	model.addAttribute("name", name);
	model.addAttribute("managerNum", managerNum);

	return "advisors/list";
}
```

### Querydsl

> 介绍： [Spring Data JPA with QueryDSL: Repositories made easy](http://dontpanic.42.nl/2011/06/spring-data-jpa-with-querydsl.html) 
> 
> [QueryDSL+gradle+idea](http://blog.csdn.net/plm609337931/article/details/66968444?locationNum=4&fps=1)  
>
> [spring boot-jpa整合QueryDSL来简化复杂操作](http://blog.csdn.net/liuchuanhong1/article/details/70244261?utm_source=gold_browser_extension) 
>
> [Spring Boot JPA - Querydsl](https://lufficc.com/blog/spring-boot-jpa-querydsl) 

#### 配置 
build.gradle 
``` 
dependencies {
    ... 
    compile "com.querydsl:querydsl-jpa:4.1.4"
    compile "com.querydsl:querydsl-apt:4.1.4:jpa"
    ...
}
``` 
启动项目后，会将项目中所有添加了 `@Entity` 注解的实体类会在 `src/main/generated/` 目录下生成对应的查询类型。每个查询类型都以大写字母Q开头。

修改默认生成路径，在 build.gradle 中添加： 
``` 
apply plugin: 'idea'
idea {
    module {
        sourceDirs += file('generated/')
        generatedSourceDirs += file('generated/')
    }
}
``` 

然后在 idea 中设置：
![](/images/2017-12-06-store-generated-sources-relative-to.png)   


#### QueryDslPredicateExecutor 接口  
继承了 QueryDslPredicateExecutor 接口和继承了 JpaRepository 接口一样，可以直接使用其自带的方法： 
``` 
public interface QueryDslPredicateExecutor<T> {  
  
    T findOne(Predicate predicate);  
  
    Iterable<T> findAll(Predicate predicate);  
  
    Iterable<T> findAll(Predicate predicate, Sort sort);  
  
    Iterable<T> findAll(Predicate predicate, OrderSpecifier<?>... orders);  
  
    Iterable<T> findAll(OrderSpecifier<?>... orders);  
  
    Page<T> findAll(Predicate predicate, Pageable pageable);  
  
    long count(Predicate predicate);  
  
    boolean exists(Predicate predicate);  
}  
```  
示例： 
``` 
@GetMapping(value = {"", "/", "home", "/index", "/index.html"})
public String index(@PageableDefault Pageable pageable, Model model) {
    QBlog qBlog = QBlog.blog;
    Predicate predicate = qBlog.id.gt(2);
    //likeIgnoreCase() 方法实参前后要加上 % 
    predicate = qBlog.title.containsIgnoreCase("spring").and(predicate);
    Page<Blog> blogs = blogRepository.findAll(predicate, pageable);
    model.addAttribute("articles", blogs);
    return "blogs/index";
}
``` 
然后请求： 
``` 
http://localhost:8008/blog?id=2&title=spring // 搜索id大于2并且标题忽略大小写匹配“spring”字符串
```


#### QueryDslPredicateExecutor 接口  
Spring 参数支持解析 com.querydsl.core.types.Predicate，根据用户请求的参数自动生成 Predicate，这样搜索方法不用自己写啦，比如：
``` 
@GetMapping(value = {"", "/", "home", "/index", "/index.html"})
public String index(@QuerydslPredicate(root = Blog.class) Predicate predicate, 
                    @PageableDefault Pageable pageable, Model model) {
    Page<Blog> blogs = blogRepository.findAll(predicate, pageable);
    model.addAttribute("articles", blogs);
    return "blogs/index";
}
``` 
然后请求：
``` 
http://localhost:8080/blog?id=1 //列表页面仅显示id为1的博客
``` 
还可以自定义 Predicate，继承 QueryDslPredicateExecutor 接口： 
``` 
@GetMapping(value = {"", "/", "home", "/index", "/index.html"})
public String index(@QuerydslPredicate(root = Blog.class) Predicate predicate,
                    @PageableDefault Pageable pageable, Model model) {
    Page<Blog> blogs = blogRepository.findAll(predicate, pageable);
    model.addAttribute("articles", blogs);
    return "blogs/index";
}
``` 

``` 
public interface BlogRepository extends JpaRepository<Blog, Long>, QueryDslPredicateExecutor<Blog>, QuerydslBinderCustomizer<QBlog> {

    default void customize(QuerydslBindings bindings, QBlog root) {
        bindings.bind(root.id).first((path, value) -> path.gt(value));
        bindings.bind(root.title).first((StringPath path, String value) -> path.contains(value));
    }
}
```
然后请求：
``` 
http://localhost:8008/blog?id=2&title=xx //列表页面仅显示id大于2且标题含有xx字符串的博客
```

#### 单表操作 
  
