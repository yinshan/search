IKAnalyzer2012FF_hf1_factory:
=====
1. 在Solr4.0发布以后，官方取消了BaseTokenizerFactory接口，而直接使用Lucene Analyzer标准接口。因此IK分词器2012 FF版本也取消了org.wltea.analyzer.solr.IKTokenizerFactory类，进而直接使用IKAnalyzer。

<fieldType name="text_cn" class="solr.TextField"> 
  <analyzer class="org.wltea.analyzer.lucene.IKAnalyzer"/> 
</fieldType> 


2. 此处修改IKAnalyzer2012FF_hf1增加IKAnalyzerTokenizerFactory，同时修改一处package定义进而支持solr 4.10.1。

<fieldType name="text_cn" class="solr.TextField" positionIncrementGap="100">
  <analyzer>
    <tokenizer class="org.wltea.analyzer.lucene.IKAnalyzerTokenizerFactory" useSmart="true"/>   
    <filter class="solr.StopFilterFactory" ignoreCase="true" words="stopwords.txt" />	
    <filter class="solr.LowerCaseFilterFactory"/> 
  </analyzer>
</fieldType>

IKAnalyzerTokenizerFactory本身也支持ext.dic和stopword.dic，所以可以考虑如何与Solr filterFactory合并使用。(同义词filter)

测试如下：
=====
http://localhost:8080/solr/#/collection1/analysis
测试字符串：   我最喜欢看的是诛仙，最近出了梦幻诛仙网游。

使用text_cn进行分析：(LCF对中文不起作用，所以省略)

1. stopwords.txt包含“的”，ext.dic包含“诛仙”以及“梦幻诛仙”

IKT: (IKTokenizer)
我，最喜欢，看，的，诛仙，最近，出了，梦幻诛仙，网游
SF: (stopFilter)
我，最喜欢，看，诛仙，最近，出了，梦幻诛仙，网游

2. stopword.dic包含“的”，无论stopwords.txt是否包含"的"，ext.dic包含“诛仙”以及“梦幻诛仙”

IKT:
我，最喜欢，看，诛仙，最近，出了，梦幻诛仙，网游
SF:
我，最喜欢，看，诛仙，最近，出了，梦幻诛仙，网游

3. stopword.dic以及stopwords.txt均不包含“的”，ext.dic空

IKT:SF:
我，最喜欢，看，的，诛，仙，最近，出了，梦幻，诛，仙，网游

Note:
=====
1. 编译本库依赖四个jar包：

lucene-analyzers-common-4.10.1.jar
lucene-core-4.10.1.jar
lucene-grouping-4.10.1.jar
lucene-queryparser-4.10.1.jar
2. 也可以和solr一起编译，这样就不用显示add上述四个jar包了。
