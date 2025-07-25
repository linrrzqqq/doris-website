---
{
    "title": "QUANTILE_STATE",
    "language": "en"
}
---

## QUANTILE_STATE
### description

QUANTILE_STATE

**In 2.0, we support the [agg_state](AGG-STATE.md) function, and it is recommended to use agg_state quantile_union(quantile_state not null) instead of this type.**

QUANTILE_STATE cannot be used as a key column. The columns of the HLL type can be used in Aggregate tables, Duplicate tables and Unique tables. When used in an Aggregate table, the aggregation type is HLL_UNION when building table.

The user does not need to specify the length and default value. The length is controlled within the system according to the degree of data aggregation.
And the QUANTILE_STATE column can only be queried or used through the supporting QUANTILE_PERCENT, QUANTILE_UNION and TO_QUANTILE_STATE functions.    
QUANTILE_STATE is a type for calculating the approximate value of quantiles. Different values with the same key are pre-aggregated during loading process. When the number of aggregated values does not exceed 2048, all data are recorded in detail. When the number of aggregated values is greater than 2048, [TDigest] is used. (https://github.com/tdunning/t-digest/blob/main/docs/t-digest-paper/histo.pdf) algorithm to aggregate (cluster) the data and save the centroid points after clustering.

related functions:
    
    QUANTILE_UNION(QUANTILE_STATE):
      
      This function is an aggregation function, which is used to aggregate the intermediate results of different quantile calculations. The result returned by this function is still QUANTILE_STATE

    
    TO_QUANTILE_STATE(DOUBLE raw_data [,FLOAT compression]):
       
       This function converts a numeric type to a QUANTILE_STATE type
       The compression parameter is optional and can be set in the range [2048, 10000]. 
       The larger the value, the higher the precision of quantile approximation calculations, the greater the memory consumption, and the longer the calculation time.
       An unspecified or set value for the compression parameter is outside the range [2048, 10000], run with the default value of 2048

    QUANTILE_PERCENT(QUANTILE_STATE, percent):
       This function converts the intermediate result variable (QUANTILE_STATE) of the quantile calculation into a specific quantile value

    
### notice

Now QUANTILE_STATE can only used in Aggregate Model Tables. We should turn on the switch for the QUANTILE_STATE types feature with the following command before use:

```
$ mysql-client > admin set frontend config("enable_quantile_state_type"="true");
```

In this way the config will be reset after the FE process restarts. For permanent setting, you can add config `enable_quantile_state_type=true` inside fe.conf.
    

### example
    select QUANTILE_PERCENT(QUANTILE_UNION(v1), 0.5) from test_table group by k1, k2, k3;
    

### keywords

    QUANTILE_STATE, QUANTILE_UNION, TO_QUANTILE_STATE, QUANTILE_PERCENT
