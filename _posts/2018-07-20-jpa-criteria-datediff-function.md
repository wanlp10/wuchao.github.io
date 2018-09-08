---
layout: post
title: JPA Criteria 中 DATEDIFF 函数的使用
category : [问题记录]
tagline: "Supporting tagline"
tags : [JPA, DATEDIFF]
---
{% include JB/setup %}
# JPA Criteria 中 DATEDIFF 函数的使用
---

<!--break-->

项目中有个查询使用了很多 left join 语句，查询出来的总记录数有二十几万条，而每次查询时就只要查询某一天的数据，及时这样分页查询时依旧很慢。
优化时把条件一个一个删掉，发现日期条件比较那里消耗了很多时间。sql 里面的日期比较是使用大小比较的，改成 DATEDIFF 函数后速度明显提升了。

项目查询时使用的是 Spring-Data-JPA Criteria，其中 DATEDIFF 查询代码如下：
```
Specification<Xxx> specification = (root, query, cb) -> {
    List<Predicate> predicates = new ArrayList<>();
    ...
    Expression<String> day = new MyFunctionExpression(null, String.class, "DAY");
    Expression<Integer> diff = cb.function("DATEDIFF", Integer.class, day, cb.literal(paramDTO.getReportDate()), root.get("createdDate"));
    predicates.add(cb.equal(diff, 0));
    ...
    if (predicates.size() > 0) {
        Predicate[] pre = new Predicate[predicates.size()];
        return query.where(predicates.toArray(pre)).getRestriction();
    }
    return null;
};
```

```
import org.hibernate.query.criteria.internal.CriteriaBuilderImpl;
import org.hibernate.query.criteria.internal.compile.RenderingContext;
import org.hibernate.query.criteria.internal.expression.function.BasicFunctionExpression;

public class MyFunctionExpression extends BasicFunctionExpression {
    public MyFunctionExpression(CriteriaBuilderImpl criteriaBuilder, Class javaType, String functionName) {
        super(criteriaBuilder, javaType, functionName);
    }

    @Override
    public String render(RenderingContext renderingContext) {
        return getFunctionName();
    }

}
```

刚开始参考网上写成：
```
Expression<String> day = cb.literal("day");
Expression<Integer> diff = cb.function("DATEDIFF", Integer.class, day, cb.literal(paramDTO.getReportDate()), root.get("createdDate"));
predicates.add(cb.equal(diff, 0));
```
运行时会报：
```
为 datediff 指定的参数 1 无效。
```
原因是上面的写法，sql 解析时会把 `"day"` 字符串放到 DATEDIFF 函数的第一个参数位置上，即 DATEDIFF('day', x, y)，而不是 DATEDIFF(day, x, y) 这种形式。

> 参考：
>
>  [http://jpwh.org/examples/jpwh2/jpwh-2e-examples-20151103/examples/src/test/java/org/jpwh/test/querying/criteria/Restriction.java](http://jpwh.org/examples/jpwh2/jpwh-2e-examples-20151103/examples/src/test/java/org/jpwh/test/querying/criteria/Restriction.java)
>  
> [How to call SQL native function, requiring constant argument, with JPA?
](https://java.wekeepcoding.com/article/10706975/How+to+call+SQL+native+function%2C+requiring+constant+argument%2C+with+JPA%3F)
