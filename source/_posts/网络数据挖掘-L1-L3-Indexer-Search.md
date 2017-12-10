---
title: 网络数据挖掘 L1-L3 Indexer&Search
date: 2017-03-28 15:39:09
categories: DataMining
mathjax: true
tags: [WebDataMining]
---
<!--more-->

## L1 Instruction
略略略

## L2 Architecture and Spiders
网页、网站架构、搜索引擎等

![image.png](http://upload-images.jianshu.io/upload_images/2812342-ea53cdec6d99e09d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

homework: Spider

## L3 Indexer and Search
### Indexer
![image.png](http://upload-images.jianshu.io/upload_images/2812342-4dfc45fa329a22f8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* Tokenizer
    * Tokenization
    * Stopping (a, the, this, that, of, etc.[Stop word list](http://www.lextek.com/manuals/onix/stopwords1.html))
* Stemmer词干器（[Porter stemming algorithm](http://tartarus.org/~martin/PorterStemmer/)）
    * Root: fish 
    * Words from the root: fishing, fished, fisher
* Lemmatization词形还原（BE for am, is, are, was, were, been）
* Spell Checking or Correction
    * Example: s1=“war”, s2=“peace”
    * war>par>pear>peac>peace
    * Operations are: type3, type1, type3,type1
    * Edit distance=4
    * ![image.png](http://upload-images.jianshu.io/upload_images/2812342-815a8a74e653d534.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* Token Sequence：Sequence of (Modified token, Document ID) pairs.
* sort：Sort by terms And then docID
* Dictionary and Postings：频率和doc list
    * ![image.png](http://upload-images.jianshu.io/upload_images/2812342-edc77005c2614589.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


### Search
* Boolean Model
    * Implementation
        * Three basic operations: AND, OR, NOT
        * Construct a binary tree to represent query
        * Process the input lists bottom-up 
    * 优点：精确查询、实现简单
    * 缺点：文档权重一致、用户需要很好的Boolean query
* Ranked Retrieval Model
    * Jaccard系数：存在Jaccard系数word权重相等的问题
    * TF&IDF
        * Term frequency tf：term在document中出现越多，该文档和该词越相关
        * Document frequency df：term在越多document出现，term的信息量越少
        * Inverse document frequency idf：$idf(t)=log_{10}(N/df(t))$，N是document的总量
        * Tf-idf:$tf.idf(t, d) = tf(t, d) * idf(t)$
        * Final Rank: $R(Q,d)=\sum_{t\in Q}tf.idf(t,d)$
* Vector Space Model向量空间模型VSM
    * 把文档看作一系列词(Term)，每一个词(Term)都有一个权重(Term weight)，不同的词(Term)根据自己在文档中的权重来影响文档相关性的打分计算。
    * 于是我们把所有此文档中词(term)的权重(term weight) 看作一个向量。
        * Document = {term1, term2, …… ,term N}
        * 计算R(Q,d)：Document Vector = {weight1, weight2, …… ,weight N}
    * 同样我们把查询语句看作一个简单的文档，也用向量来表示。
        * Query = {term1, term 2, …… , term N}
        * 计算R(Q,d)：Query Vector = {weight1, weight2, …… , weight N}
    * 我们把所有搜索出的文档向量及查询向量放到一个N维空间中，每个词(term)是一维。
    ![Vector space model.jpg](http://upload-images.jianshu.io/upload_images/2812342-b5f9934d3284745d.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
    * 两个向量之间的夹角越小，相关性越大。所以我们计算夹角的余弦值作为相关性的打分，夹角越小，余弦值越大，打分越高，相关性越大。
    * 正则化并计算cos：![image.png](//upload-images.jianshu.io/upload_images/2812342-35bc3d6d62542e01.png)

* Probabilistic Model
    * 
* Evaluation
    * Precision：The fraction of retrieved documents that are relevant
    * Recall：The fraction of relevant documents that are retrieved
    * ![image.png](http://upload-images.jianshu.io/upload_images/2812342-b23cd4f509948588.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
  

> homework
```Java
//倒排文件索引(Inverted File Index)
import java.io.*;
import java.util.*;

public class Main{
	public static void main(String[] args){
		Main test = new Main();
		test.MyInvertedFileBuilde();
	}
	private void MyInvertedFileBuilde(){
		File dir = new File("documents");
		File[] files = dir.listFiles();
		LinkedHashMap<Integer, String> src = new LinkedHashMap<>();
		LinkedHashMap<String, LinkedHashSet<Integer>> invertedFile = new LinkedHashMap<>();
		for (int id = 0; id < files.length; id++) {
			File file = files[id];
            String lineTxt = new String();
			if(file.isFile()){
				try {
					InputStreamReader read = new InputStreamReader(new FileInputStream(file),"GBK");
					BufferedReader bufferedReader = new BufferedReader(read);
					String tmp = null;
					while((tmp = bufferedReader.readLine()) != null){
						lineTxt = lineTxt + tmp;
					}
					src.put(id, lineTxt);//files ID map
					String[] words = lineTxt.split(" ");
					for (String word : words) {
						if(!invertedFile.containsKey(word)){
							invertedFile.put(word,new LinkedHashSet<Integer>());
							invertedFile.get(word).add(id);
						}else {
							invertedFile.get(word).add(id);	// words inveredFile
						}
						
					}
				} catch (Exception e) {
					e.printStackTrace();
				}
			}
		}
		for (String word : invertedFile.keySet()) {
			System.out.println(word+" "+invertedFile.get(word));
		}
	}
}
```

