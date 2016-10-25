#Paging support for aggregations
Terms aggregation does not support a way to page through the buckets returned.  
Details refer to the [#4915](https://github.com/elastic/elasticsearch/issues/4915) discussions.

In my project, I should get large amount of terms(100,000,000+) from ES.
If I page on client-side, memory will have too much pressure.
According to the [#4915](https://github.com/elastic/elasticsearch/issues/4915) discussions, we can use top_hits to improve the ES's query speed and reduce client-side memory usage.

``` java
    aggTerms = AggregationBuilders.terms("keyAgg").field(WebUriFields.URI).size(0);
    aggTerms.subAggregation(AggregationBuilders.topHits(WebUriFields.URI).setFrom(0).setSize(1));
```
We use PageBean class defined by ourself to page terms on client-side after that are searched from ES,   
but its performance and inefficient is not high, so we hope ES can support paging for aggregations.
