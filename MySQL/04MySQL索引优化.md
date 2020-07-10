## like查询会不会走索引？

- %value 不走
- %value% 不走
- value%可能走

为什么value%还是可能走呢。其实极大可能是要走索引的。但是对于例如”湖人总冠军123“、湖人总冠军”456“这种，你来个`like 湖人总冠军%`，那对半走不了，因为有优化器，思量一下，这还不如全表扫描呢。

**那么。像%value这种能让他走索引吗？**

答案是肯定的。我可以使用mysql的反转语句让它走，怎么说呢，看代码：`SELECT * FROM jka WHERE REVERSE(v) LIKE REVERSE('%ABC');`



## MySQL查询优化--少不了EXPLAIN（执行计划）

