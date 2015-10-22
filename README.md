# RedisAPI

[![Build Status](https://travis-ci.org/ropensci/RedisAPI.png?branch=master)](https://travis-ci.org/ropensci/RedisAPI)

Automatically generated R6 interface to the full [Redis](http://redis.io) API.  The generated functions are faithful to the Redis [documentation](http://redis.io/commands) while attempting to match R's argument and vectorisation semantics.

As of version `0.3.0` RedisAPI supports binary serialisation of almost anything; keys, values, etc.  Just don't expect Redis to do anything sensible with the values - you won't be able to compute on directly.

```r
con <- RedisAPI::hiredis()
```

That connection has many (149) methods. Automatically generated methods are in all-caps, following the Redis documentation.  Unlike Redis they are *case sensitive*.

```r
con
## <redis_api>
##   Public:
##     APPEND: function
##     AUTH: function
##     BGREWRITEAOF: function
##     BGSAVE: function
##     ...
##     ZSCORE: function
##     ZUNIONSTORE: function
```

149 methods are available in the resulting class; these are designed to be straightforward to use following the Redis documentation.  For example, the Redis [`HMSET`](http://redis.io/commands/hmset) command is defined as

```
HMSET key field value [field value ...]
```

which sets one or more `field` / `value` pairs within a hash stored at a `key`.  In `RedisAPI`, the generated interface has arguments

```
function(key, field, value)
```

where `key` is a scalar and `field` / `value` are vectors of the same non-zero length (these requirements are enforced at runtime).

Because `RedisAPI` objects are `R6` objects, access methods using `$`:


```r
con$HMSET("myhash", c("a", "b", "c"), c(1, 2, 3))
```

```
## [1] "OK"
```

```r
con$HGET("myhash", "b")
```

```
## [1] "2"
```

Note that no clever type conversion will be done; R types are converted to strings and are not converted back when returned.

# Serialisation

To ease saving arbitrary R data into Redis, `RedisAPI` has two convenience functions for serialising to and deserialising from a string: `object_to_string` and `string_to_object`.


```r
object_to_string(2)
```

```
## [1] "A\n2\n197120\n131840\n14\n1\n2\n"
```

```r
string_to_object("A\n2\n196866\n131840\n14\n1\n2\n")
```

```
## [1] 2
```

This makes it easy (though reasonably verbose) to use arbitrary R values anywhere in Redis:


```r
con$SET(object_to_string(1:10), object_to_string(iris))
```

```
## [1] "OK"
```

```r
head(string_to_object(con$GET(object_to_string(1:10))))
```

```
##   Sepal.Length Sepal.Width Petal.Length Petal.Width Species
## 1          5.1         3.5          1.4         0.2  setosa
## 2          4.9         3.0          1.4         0.2  setosa
## 3          4.7         3.2          1.3         0.2  setosa
## 4          4.6         3.1          1.5         0.2  setosa
## 5          5.0         3.6          1.4         0.2  setosa
## 6          5.4         3.9          1.7         0.4  setosa
```

String serialisation can very slightly mess with floating point numbers, but should be reasonable for many uses.  Eventually I'd like to support binary serialisation here, but that requires C-level wrappers to be generated.

# High level support

As a simple example of what could be built on `RedisAPI` there is an example `rdb` class that does very basic key/value storage of R objects:


```r
r <- RedisAPI::rdb(RedisAPI::hiredis())
r$set("foo", runif(20))
r$get("foo")
```

```
##  [1] 0.26550866 0.37212390 0.57285336 0.90820779 0.20168193 0.89838968
##  [7] 0.94467527 0.66079779 0.62911404 0.06178627 0.20597457 0.17655675
## [13] 0.68702285 0.38410372 0.76984142 0.49769924 0.71761851 0.99190609
## [19] 0.38003518 0.77744522
```

```r
r$keys()
```

```
## [1] "mykey"      "mtcars"     "foo"        "a_function"
```

For a more interesting example, see [`storr`](https://github.com/richfitz/storr)

# rlite support

If [`rrlite`](https://github.com/ropensci/rrlite) is installed, you can create hirlite connections with `rrlite::hirlite()` which has the same set of generated interfaces.  Not all commands are supported (for example, `SCAN` and `BLPOP`) but `hirlite` will throw an error if unsupported commands are used.

# Documentation

There is a vignette (`vignette("RedisAPI")`) that outlines the basic idea.  Source is [here](vignette/RedisAPI.Rmd), and rendered [here](http://htmlpreview.github.io/?https://raw.githubusercontent.com/ropensci/RedisAPI/master/inst/doc/RedisAPI.html)

## Meta

* Please [report any issues or bugs](https://github.com/ropensci/RedisAPI/issues).
* License: BSD (2 clause)
* Get citation information for `RedisAPI` in R by doing `citation(package = 'RedisAPI')`

[![rofooter](http://ropensci.org/public_images/github_footer.png)](http://ropensci.org)
