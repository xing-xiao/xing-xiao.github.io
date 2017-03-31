# Sklearn 特征抽取

`sklearn.feature_extraction`从将学习数据抽取为机器学习算法可使用的数据。

## 从字典中加载特征

`DictVectorizer`类用于从`dict`中抽取特征，速度慢，易于使用，稀疏存储。

如下例子抽取了字符属性（city）和数字型属性（temperature），对于每个字符类属性，均抽取为一个特征，使用`0`或`1`表示是否存在，字符类属性则抽象为一个浮点型特征。

```python
...     {'city': 'Dubai', 'temperature': 33.},
...     {'city': 'London', 'temperature': 12.},
...     {'city': 'San Fransisco', 'temperature': 18.},
... ]

>>> from sklearn.feature_extraction import DictVectorizer
>>> vec = DictVectorizer()

>>> vec.fit_transform(measurements).toarray()
array([[  1.,   0.,   0.,  33.],
       [  0.,   1.,   0.,  12.],
       [  0.,   0.,   1.,  18.]])

>>> vec.get_feature_names()
['city=Dubai', 'city=London', 'city=San Fransisco', 'temperature']
```

`DictVectorizer`在自然语言处理的`sequence classifiers`中非常有用，如下是一个二维数据（字符位置／字符内容）特征抽取。如果字符过多`DictVectorizer`抽取的特征将会非常的多，特征中的值很多会是`0`，`DictVectorizer`默认使用`scipy.sparse`矩阵，而不是`numpy.ndarray`以避免内存消耗过大。

```python
>>> pos_window = [
...     {
...         'word-2': 'the',
...         'pos-2': 'DT',
...         'word-1': 'cat',
...         'pos-1': 'NN',
...         'word+1': 'on',
...         'pos+1': 'PP',
...     },
...     # in a real application one would extract many such dictionaries
... ]
>>> vec = DictVectorizer()
>>> pos_vectorized = vec.fit_transform(pos_window)
>>> pos_vectorized                
<1x6 sparse matrix of type '<... 'numpy.float64'>'
    with 6 stored elements in Compressed Sparse ... format>
>>> pos_vectorized.toarray()
array([[ 1.,  1.,  1.,  1.,  1.,  1.]])
>>> vec.get_feature_names()
['pos+1=PP', 'pos-1=NN', 'pos-2=DT', 'word+1=on', 'word-1=cat', 'word-2=the']
```


## 特征哈希

`FeatureHasher`类使用`feature hashing`技术达到特征化速度快，内存消耗低，他不保持输入的特征，所以不支持`inverse_transform`方法。

## 文本类特征抽取

### 语句的矢量化方法

文本数据不能直接被算法使用，我们期望输入给算法的数据是固定size的数字型特征向量，而非不定长的文本。`sk-learn`使用如下方法解决这个问题：

- tokenizing，符号化字符串，并且对于每个可能的符号一个整形id。可以使用n-grams或者特殊字符（空格、符号）进行分词
- counting，计算每个符号在文本中出现时的频率
- normalizing，对于每个符号进行归一化和权重评估

这个过程被称为`vectorization`（矢量化），它将文本转化为数字型的特征向量，上述策略被称为`Bag of Words`，它仅关心每个字符（符号）出现的频率，而不关心其所在的位置关系。

### 稀疏问题

由于每个文本使用的字符仅占整个语料库中的字符的一小部分，所以会出现矩阵中大量特征值为0的情况。`sk-learn`使用`scipy.sapse`包来解决这个问题。

### `CountVectorizer`矢量化

`CountVectorizer`类执行矢量化和出现频率计算，如下是一个简单的例子，`CountVectorizer`的参数参见[文档链接](http://scikit-learn.org/stable/modules/generated/sklearn.feature_extraction.text.CountVectorizer.html#sklearn.feature_extraction.text.CountVectorizer)。

```python
>>> from sklearn.feature_extraction.text import CountVectorizer
>>> ###使用CountVectorizer，参数如下
>>> ### analyzer={'word', 'char', 'char_wb', callable}，规定了分词的方法：word是数字字母；char是n-gram字符分词；char_wb是仅从数字字母中使用n-gram分词；调用函数名则将对每条数据使用该函数进行处理，输出为list
>>> ### binary, boolean型参数，将非0的数值全部置为1
>>> ### decode_error={‘strict’, ‘ignore’, ‘replace’}解码失败后的处理方法：strict表示处理失败后会raise一个UnicodeDecodeError；ignore则会忽略改条的处理
>>> ### dtype：fit_transform() or transform()返回的矩阵类型
>>> ### encoding，解码的方式
>>> ### input={‘filename’, ‘file’, ‘content’}，输入分析数据方式：filename则需要在fit_transform中输入一个文件名list；file则需要在fit_transform中输入python的file类型变量list；content需要在fit_transform中输入string或者bytes的list
>>> ### lowercase, boolean型参数，向量化之前将字符转为小写
>>> ### vocabulary，Mapping或iterable类型，默认为None，如果不为None，则选择其中的key作为矩阵的特征属性。
>>> ### max_df，0.0到1.0之间的浮点型，或无限制的整型值，默认是1.0。当数值为浮点型时，如果(含有某个词的文档数/总文档数)大于max_df的值，那么这个词就不会被用作关键词；当数值为整型时，如果某个词出现的次数超过max_df的值时则不会被用作关键词；当参数中vocabulary值不为None时这个参数无效。
>>> ### min_df，0.0到1.0之间的浮点型，或无限制的整型值，默认是1。当数值为浮点型时，如果(含有某个词的文档数/总文档数)小于min_df的值，那么这个词就不会被用作关键词；当数值为整型时，如果某个词出现的次数小于min_df的值时则不会被用作关键词；当参数中vocabulary值不为None时这个参数无效。
>>> ### max_features，对(含有某个词的文档数/总文档数)进行降序排列，仅取前max_features个，当参数中vocabulary值不为None时这个参数无效。
>>> ### ngram_range，分词的长度
>>> ### preprocessor
>>> ### stop_words
>>> ### strip_accents
>>> ### token_pattern
>>> ### tokenizer
>>> vectorizer = CountVectorizer(min_df=1)
>>> vectorizer                     
CountVectorizer(analyzer='word', binary=False, decode_error='strict',
        dtype=<class 'numpy.int64'>, encoding='utf-8', input='content',
        lowercase=True, max_df=1.0, max_features=None, min_df=1,
        ngram_range=(1, 1), preprocessor=None, stop_words=None,
        strip_accents=None, token_pattern='(?u)\\b\\w\\w+\\b',
        tokenizer=None, vocabulary=None)
>>> corpus = [
...     'This is the first document.',
...     'This is the second second document.',
...     'And the third one.',
...     'Is this the first document?',
... ]
>>> X = vectorizer.fit_transform(corpus)
>>> X                       
<4x9 sparse matrix of type '<... 'numpy.int64'>'
    with 19 stored elements in Compressed Sparse ... format>
>>> ### 查看分词结果
>>> analyze = vectorizer.build_analyzer()
>>> analyze("This is a text document to analyze.")
['this', 'is', 'text', 'document', 'to', 'analyze']
>>> ### 获取特征列表和对应的值
>>> vectorizer.get_feature_names()
['and', 'document', 'first', 'is', 'one', 'second', 'the', 'third', 'this']
>>> X.toarray()
array([[0, 1, 1, 1, 0, 0, 1, 0, 1],
       [0, 1, 0, 1, 0, 2, 1, 0, 1],
       [1, 0, 0, 0, 1, 0, 1, 1, 0],
       [0, 1, 1, 1, 0, 0, 1, 0, 1]], dtype=int64)
>>> ### 获取其中的某个特征的列index
>>> vectorizer.vocabulary_.get('document')
1
>>> vectorizer.vocabulary_.get('one')
4
>>> ### 对新样本进行转换
>>> vectorizer.transform(['Something completely new.']).toarray()
array([[0, 0, 0, 0, 0, 0, 0, 0, 0]])
>>> vectorizer.transform(['Something completely first.']).toarray()
array([[0, 0, 1, 0, 0, 0, 0, 0, 0]])
```

我们注意到上个例子中，第一条和最后一条文本转换后结果是一样的，这样就丢失了部分词的前后的顺序，如果想保留这个信息，那么可以使用n-grams来抓换，如下使用2-grams转换。

```python
>>> bigram_vectorizer = CountVectorizer(ngram_range=(1, 2),token_pattern=r'\b\w+\b', min_df=1)
>>> analyze = bigram_vectorizer.build_analyzer()
>>> analyze('Bi-grams are cool!')
['bi', 'grams', 'are', 'cool', 'bi grams', 'grams are', 'are cool']
>>> X_2 = bigram_vectorizer.fit_transform(corpus).toarray()
>>> X_2
array([[0, 0, 1, 1, 1, 1, 1, 0, 0, 0, 0, 0, 1, 1, 0, 0, 0, 0, 1, 1, 0],
       [0, 0, 1, 0, 0, 1, 1, 0, 0, 2, 1, 1, 1, 0, 1, 0, 0, 0, 1, 1, 0],
       [1, 1, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 1, 0, 0, 1, 1, 1, 0, 0, 0],
       [0, 0, 1, 1, 1, 1, 0, 1, 0, 0, 0, 0, 1, 1, 0, 0, 0, 0, 1, 0, 1]], dtype=int64)
>>> bigram_vectorizer.get_feature_names()                  
['and', 'and the', 'document', 'first', 'first document', 'is', 'is the', 'is this', 'one', 'second', 'second document', 'second second', 'the', 'the first', 'the second', 'the third', 'third', 'third one', 'this', 'this is', 'this the']
>>> bigram_vectorizer.vocabulary_.get('is this')
7
>>> X_2[:, feature_index] 
array([0, 0, 0, 1], dtype=int64)
```

### Tf-idf加权矢量化

在规模较大的语料库中，像`the`、`a`、`is`等词出现的概率会非常高，但对实际的学习内容并没有意义，而tf-idf转换则用于解决这个问题。

原理：TBD

使用：

```python
>>> from sklearn.feature_extraction.text import TfidfTransformer
>>> transformer = TfidfTransformer(smooth_idf=False)
>>> transformer   
TfidfTransformer(norm=...'l2', smooth_idf=False, sublinear_tf=False,
                 use_idf=True)
>>> counts = [[3, 0, 1],
...           [2, 0, 0],
...           [3, 0, 0],
...           [4, 0, 0],
...           [3, 2, 0],
...           [3, 0, 2]]
...
>>> tfidf = transformer.fit_transform(counts)
>>> tfidf                         
<6x3 sparse matrix of type '<... 'numpy.float64'>'
    with 9 stored elements in Compressed Sparse ... format>

>>> tfidf.toarray()                        
array([[ 0.81940995,  0.        ,  0.57320793],
       [ 1.        ,  0.        ,  0.        ],
       [ 1.        ,  0.        ,  0.        ],
       [ 1.        ,  0.        ,  0.        ],
       [ 0.47330339,  0.88089948,  0.        ],
       [ 0.58149261,  0.        ,  0.81355169]])
```

```python
>>> transformer = TfidfTransformer()
>>> transformer.fit_transform(counts).toarray()
array([[ 0.85151335,  0.        ,  0.52433293],
       [ 1.        ,  0.        ,  0.        ],
       [ 1.        ,  0.        ,  0.        ],
       [ 1.        ,  0.        ,  0.        ],
       [ 0.55422893,  0.83236428,  0.        ],
       [ 0.63035731,  0.        ,  0.77630514]])
>>> transformer.idf_                       
array([ 1. ...,  2.25...,  1.84...])
```


