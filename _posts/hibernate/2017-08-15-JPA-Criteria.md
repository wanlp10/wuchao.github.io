---
layout: post
title: JPA Criteria 
category : [JavaEE, Hibernate]
tagline: "Supporting tagline"
tags : [Java, Hibernate]
---
{% include JB/setup %}
# JPA Criteria 查询 

> [JPA criteria 查询: 类型安全与面向对象](http://blog.csdn.net/dracotianlong/article/details/28445725) 



###  

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
    @OrderBy("created_date desc")
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

现在要查询所有 Advisor 对象,并且查出 Advisor 对象中 managers 集合的个数和 products 集合中 rate 属性的最大值: 

定义一个 DTO 对象,保存查询结果

``` 
public class AdvisorDto implements Serializable {
  private Long id;
  private String name;
  private Long managerNum;
  private Double maxRate;
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

