---
layout: post
title: Redis - 布隆过滤器 vs 哈希
category: redis
tags: [redis]
excerpt: Redis Bloom Filter VS Hash
---

本文主要从`时间`、`空间`、`正确性`三个维度，对比`哈希`与`布隆过滤器`在判断元素是否存在的区别。  

先看下结论：  


![](https://yyc-images.oss-cn-beijing.aliyuncs.com/redis_bloom_filter_vs_hash.png)  


## 准备数据  

``` java
@Test
public void pushDataIntoRedisSetTest(){

    Jedis jedis = new Jedis("localhost", 6379, 5000);
    // Use 18363ms
    for (int i = 0; i < 1000000; i++){
        jedis.sadd("set:2", String.valueOf(generateRandomDigits(8)));
    }
}

@Test
public void addDataIntoBloomFilter(){
    BloomFilter<String> filter = new BloomFilter<>(0.0001, 1000000);
    Jedis jedis = new Jedis("127.0.0.1", 6379);
    filter.bind(new RedisBitSet(jedis, "bloomfilter:key:bf1"));

    for (int i = 0; i < 1000000; i++){
        filter.add(String.valueOf(generateRandomDigits(8)));
    }
}

public static int generateRandomDigits(int n) {
    int m = (int) Math.pow(10, n - 1);
    return m + new Random().nextInt(9 * m);
}
```

这里简单介绍下`BloomFilter`类，详情可参考底下引用：  

``` java

public class BloomFilter<E> implements Cloneable, Serializable {

    private BaseBitSet bitSet;
    private int bitSetSize;
    private double bitsPerElement;
    private int expectedNumberOfFilterElements; // expected (maximum) number of elements to be added
    private int numberOfAddedElements; // number of elements actually added to the Bloom filter
    private int k; // number of hash functions


    /**
     * Bind a bit set for Bloom filter. It can be any data structure that implements the BaseBitSet interface.
     * @param bitSet
     */
    public void bind(BaseBitSet bitSet) {
        this.bitSet = bitSet;
    }

    /**
     * Constructs an empty Bloom filter. The total length of the Bloom filter will be
     * c*n.
     *
     * @param c is the number of bits used per element.
     * @param n is the expected number of elements the filter will contain.
     * @param k is the number of hash functions used.
     */
    public BloomFilter(double c, int n, int k) {
        this.expectedNumberOfFilterElements = n;
        this.k = k;
        this.bitsPerElement = c;
        this.bitSetSize = (int) Math.ceil(c * n);
        numberOfAddedElements = 0;
    }

    /**
     * Constructs an empty Bloom filter with a given false positive probability. The number of bits per
     * element and the number of hash functions is estimated
     * to match the false positive probability.
     *
     * @param falsePositiveProbability is the desired false positive probability.
     * @param expectedNumberOfElements is the expected number of elements in the Bloom filter.
     */
    public BloomFilter(double falsePositiveProbability, int expectedNumberOfElements) {
        this(Math.ceil(-(Math.log(falsePositiveProbability) / Math.log(2.0))) / Math.log(2.0), // c = k / ln(2)
                expectedNumberOfElements,
                (int) Math.ceil(-(Math.log(falsePositiveProbability) / Math.log(2.0)))); // k = ceil(-log_2(false prob.))
    }

    /**
     * Adds an object to the Bloom filter. The output from the object's
     * toString() method is used as input to the hash functions.
     *
     * @param element is an element to register in the Bloom filter.
     */
    public void add(E element) {
        add(element.toString().getBytes(MessageDigestUtils.CHARSET));
    }

    /**
     * Adds an array of bytes to the Bloom filter.
     *
     * @param bytes array of bytes to add to the Bloom filter.
     */
    public void add(byte[] bytes) {
        int[] hashes = MessageDigestUtils.createHashes(bytes, k);
        for (int hash : hashes)
            bitSet.set(Math.abs(hash % bitSetSize), true);
        numberOfAddedElements++;
    }

    /**
     * Returns true if the element could have been inserted into the Bloom filter.
     * Use getFalsePositiveProbability() to calculate the probability of this
     * being correct.
     *
     * @param element element to check.
     * @return true if the element could have been inserted into the Bloom filter.
     */
    public boolean contains(E element) {
        return contains(element.toString().getBytes(MessageDigestUtils.CHARSET));
    }

    /**
     * Returns true if the array of bytes could have been inserted into the Bloom filter.
     * Use getFalsePositiveProbability() to calculate the probability of this
     * being correct.
     *
     * @param bytes array of bytes to check.
     * @return true if the array could have been inserted into the Bloom filter.
     */
    public boolean contains(byte[] bytes) {
        int[] hashes = MessageDigestUtils.createHashes(bytes, k);
        for (int hash : hashes) {
            if (!bitSet.get(Math.abs(hash % bitSetSize))) {
                return false;
            }
        }
        return true;
    }
}


public class RedisBitSet implements BaseBitSet {

    private JedisCluster jedisCluster;
    private Jedis jedis;
    private String name;

    private boolean isCluster = true;

    private RedisBitSet() {
    }

    /**
     * Create a redis bitset.
     * @param jedisCluster jedis cluster client.
     * @param name the redis bit key name.
     */
    public RedisBitSet(JedisCluster jedisCluster, String name) {
        this.jedisCluster = jedisCluster;
        this.name = name;
        this.isCluster = true;
    }

    /**
     * Create a redis bitset.
     * @param jedis jedis client.
     * @param name the redis bit key name.
     */
    public RedisBitSet(Jedis jedis, String name) {
        this.jedis = jedis;
        this.name = name;
        this.isCluster = false;
    }

    public void set(int bitIndex) {
        if (this.isCluster) {
            this.jedisCluster.setbit(this.name, bitIndex, true);
        } else {
            this.jedis.setbit(this.name, bitIndex, true);
        }
    }

    public void set(int bitIndex, boolean value) {
        if (this.isCluster) {
            this.jedisCluster.setbit(this.name, bitIndex, value);
        } else {
            this.jedis.setbit(this.name, bitIndex, value);
        }
    }

    public boolean get(int bitIndex) {
        if (this.isCluster) {
            return this.jedisCluster.getbit(this.name, bitIndex);
        } else {
            return this.jedis.getbit(this.name, bitIndex);
        }
    }

    public void clear(int bitIndex) {
        if (this.isCluster) {
            this.jedisCluster.setbit(this.name, bitIndex, false);
        } else {
            this.jedis.setbit(this.name, bitIndex, false);
        }
    }

    public void clear() {
        if (this.isCluster) {
            this.jedisCluster.del(this.name);
        } else {
            this.jedis.del(this.name);
        }
    }

    public long size() {
        if (this.isCluster) {
            return this.jedisCluster.bitcount(this.name);
        } else {
            return this.jedis.bitcount(this.name);
        }
    }

    public boolean isEmpty() {
        return size() <= 0;
    }
}
```


## 对比  

``` java
@Test
public void bloomFilterVsHashPerformanceTest(){
    // Prepare ten thousand random strings.
    ArrayList<String> randomStrList = new ArrayList<>();
    for (int i = 0; i < 10000; i++){
        randomStrList.add(String.valueOf(generateRandomDigits(8)));
    }

    // Bloom filter and hash preparation.
    int bfHint = 0;
    int hHint  = 0;
    BloomFilter<String> filter = new BloomFilter<>(0.0001, 1000000);
    Jedis jedis = new Jedis("127.0.0.1", 6379);
    filter.bind(new RedisBitSet(jedis, "bloomfilter:key:bf1"));

    // Step1: Bloom filter
    LocalDateTime bfStart = LocalDateTime.now();
    for (int i = 0; i < 10000; i++) {
        if (filter.contains(randomStrList.get(i))){
            bfHint++;
        }
    }

    LocalDateTime bfEnd = LocalDateTime.now();
    long bfMillis = Duration.between(bfStart, bfEnd).toMillis();
    System.out.println("Bloom filter use " + bfMillis + " ms");

    // Step2: Hash
    LocalDateTime hStart = LocalDateTime.now();
    for (int i = 0; i < 10000; i++) {
        if (jedis.sismember("set:2", randomStrList.get(i))){
            hHint++;
        }
    }

    LocalDateTime hEnd = LocalDateTime.now();
    long hMillis = Duration.between(hStart, hEnd).toMillis();
    System.out.println("Hash use " + hMillis + " ms");

    System.out.println("Correct rate = bloomHint / hashHint = " + (bfHint / (double) hHint));
}

-------------  Console  --------------
// Bloom filter use 1431 ms
// Hash use 2014 ms
// Correct rate = bloomHint / hashHint = 0.7226277372262774
```


## `Reference`
- [基于Redis的BloomFilter算法去重](https://www.cnblogs.com/wxisme/p/5742456.html){:target="_blank"}  
- [wxisme/bloomfilter](https://github.com/wxisme/bloomfilter){:target="_blank"}  
- [Java: Generating a random number of a certain length](https://programming.guide/java/generate-random-number-of-given-length.html){:target="_blank"}  