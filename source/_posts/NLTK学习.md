---
title: 自然语言处理包——NLTK
tags: NLP
categories: 自然语言处理
cover: false
date: 2021-03-20 00:00:00
keywords: 
	-自然语言处理
	-NLP
---

# 1NLTK库安装
## 1.1Anaconda安装

打开Anaconda目录下的`Anaconda prompt`，输入以下代码直接进行下载：

```python
import nltk
nltk.download()
```

![image-20210319195750473](https://cdn.jsdelivr.net/gh/small-brilliant/image/img1/20210319195917.png)

但是在没有`科学上网`的前提下，下载时非常慢的。

# 2NLTK常见操作

## 2.1文本切分

```python
import nltk
from nltk.tokenize import sent_tokenize
text="Don't hesitate to ask questions. Be positive."
print(sent_tokenize(text))
```

![image-20210319205038870](https://cdn.jsdelivr.net/gh/small-brilliant/image/img1/20210319205038.png)

## 2.2分词方法

- TreebankWordTokenizer缩略词会被分离

  ```python
  words = nltk.word_tokenize(text)
  print(words)
  ```

  ![image-20210319234108848](https://cdn.jsdelivr.net/gh/small-brilliant/image/img1/20210319234108.png)

- PunktWordTokenizer通过分离标点来实现切分，每一个单词都会被保留

  ```python
  from nltk.tokenize import WordPunctTokenizer
  tokenizer=WordPunctTokenizer()
  words = tokenizer.tokenize(text) 
  print(words)
  ```

  ![image-20210319234201231](https://cdn.jsdelivr.net/gh/small-brilliant/image/img1/20210319234201.png)

## 2.3停止词

无用词被称为停止词。

看下面例子![image-20210320091749214](https://cdn.jsdelivr.net/gh/small-brilliant/image/img1/20210320091756.png)

```python
from nltk.corpus import stopwords
from nltk.tokenize import word_tokenize

example_sent = "This is a sample sentence, showing off the stop words filtration."

stop_words = set(stopwords.words('english'))

word_tokens = word_tokenize(example_sent)
	
filtered_sentence = [w for w in word_tokens if not w in stop_words]

print(word_tokens)
print(filtered_sentence)
```

## 2.4语料库

- nltk自带的语料库

![image-20210320124655248](https://cdn.jsdelivr.net/gh/small-brilliant/image/img1/20210320124717.png)

- 语料库操作

![image-20210320124640746](https://cdn.jsdelivr.net/gh/small-brilliant/image/img1/20210320124726.png)

## 2.5提取词干

提取词干的原因是为了缩短查找的时间，使句子正常化。

考虑下面这种情况：

```text
I was taking a ride in the car. 
I was riding in the car.
```

表达的意思都是我在车上，没有必要区分`taking`和`ridinig`

下面是提取相干单词的词干：

```python
from nltk.stem import PorterStemmer
from nltk.tokenize import sent_tokenize,word_tokenize

ps = PorterStemmer()
example_words = ["python","pythoner","pythoning","pythoned","pythonly"]
for w in example_words:
    print(ps.stem(w))
```

![image-20210320103202489](https://cdn.jsdelivr.net/gh/small-brilliant/image/img1/20210320103202.png)

下面是实际中句子的提取词干：

```python
new_text = "It is important to by very pythonly while you are pythoning with python. All pythoners have pythoned poorly at least once."
words = word_tokenize(new_text)

for w in words:
    print(ps.stem(w),end=" ")
```

![image-20210320103339676](https://cdn.jsdelivr.net/gh/small-brilliant/image/img1/20210320103339.png)

## 2.6词性还原

和词干提取类似，不同之处在于**词干提取**会创造不存在的词汇，**词性还原**的结果是一个真正的词汇

```python
from nltk.stem import WordNetLemmatizer
lemmatizer= WordNetLemmatizer()
print(lemmatizer.lemmatize('increases'))
#输出结果为 increase
```

1. 有时，如果你试图还原一个词，比如 playing,还原的结果还是 playing。这是因为默认还原的结果是**名词**，如果你想得到动词，可以通过以下的方式指定。

   ```python
   print(lemmatizer.lemmatize('playing', pos="v"))
   print(lemmatizer.lemmatize('playing', pos="n"))
   print(lemmatizer.lemmatize('playing', pos="a"))
   print(lemmatizer.lemmatize('playing', pos="r"))
   '''
   结果为
   play
   playing
   playing
   playing
   '''
   ```

## 2.7词性标注

NLTK模块的一个更强大的方面是，它可以为你做词性标注。 意思是把一个句子中的单词标注为名词，形容词，动词等。下面是标签以及含义和例子

```text
POS tag list:

CC  coordinating conjunction
CD  cardinal digit
DT  determiner
EX  existential there (like: "there is" ... think of it like "there exists")
FW  foreign word
IN  preposition/subordinating conjunction
JJ  adjective   'big'
JJR adjective, comparative  'bigger'
JJS adjective, superlative  'biggest'
LS  list marker 1)
MD  modal   could, will
NN  noun, singular 'desk'
NNS noun plural 'desks'
NNP proper noun, singular   'Harrison'
NNPS    proper noun, plural 'Americans'
PDT predeterminer   'all the kids'
POS possessive ending   parent's
PRP personal pronoun    I, he, she
PRP$   possessive pronoun  my, his, hers
RB  adverb  very, silently,
RBR adverb, comparative better
RBS adverb, superlative best
RP  particle    give up
TO  to  go 'to' the store.
UH  interjection    errrrrrrrm
VB  verb, base form take
VBD verb, past tense    took
VBG verb, gerund/present participle taking
VBN verb, past participle   taken
VBP verb, sing. present, non-3d take
VBZ verb, 3rd person sing. present  takes
WDT wh-determiner   which
WP  wh-pronoun  who, what
WP$    possessive wh-pronoun   whose
WRB wh-abverb   where, when
```

下面是程序示例：

```python
import nltk
from nltk.corpus import inaugural
from nltk.tokenize import PunktSentenceTokenizer

#两篇演讲
train_text = inaugural.raw("2001-Bush.txt")
sample_text = inaugural.raw("2005-Bush.txt")

#训练
custom_sent_tokenizer = PunktSentenceTokenizer(train_text)
#切分
tokenized = custom_sent_tokenizer.tokenize(sample_text)

#标注函数
def process_content():
    try:
        for i in tokenized[:5]:
            #分词
            words = nltk.word_tokenize(i)
            #标注
            tagged = nltk.pos_tag(words)
            print(tagged)
    except Exception as e:
        print(str(e))
        
process_content()
```

![image-20210320110455717](https://cdn.jsdelivr.net/gh/small-brilliant/image/img1/20210320110455.png)

## 2.8文本分类

文本分类的目标可能相当宽泛。 也许我们试图将文本分类为政治或军事。 也许我们试图按照作者的性别来分类。 一个相当受欢迎的文本分类任务是，将文本的正文识别为垃圾邮件或非垃圾邮件，例如电子邮件过滤器。 在下面的例子中，我将尝试创建一个情感分析算法。

我们首先尝试使用属于 NLTK 语料库的电影评论数据库。 从那里，我们将尝试使用词汇作为“特征”，这是“正面”或“负面”电影评论的一部分。 NLTK 语料库`movie_reviews`数据集拥有评论，他们被标记为正面或负面。 这意味着我们可以训练和测试这些数据。 首先，让我们来预处理我们的数据。

```python
import nltk
import random
from nltk.corpus import movie_reviews
#在每个类别（正向和负向）选取所有文件ID，然后对文件ID存储word_tokenized版本，之后再加上一个正面或者负面标签
documents = [(list(movie_reviews.words(fileid)), category)
             for category in movie_reviews.categories()
             for fileid in movie_reviews.fileids(category)]
#因为前部分都是负面，后部分都是正面，所有打乱数据
random.shuffle(documents)

print(documents[1])

all_words = []
for w in movie_reviews.words():
    #将大写转化为小写
    all_words.append(w.lower())
#统计每个单词次数，FreqDist中的键为单词，值为单词的出现总次数。
#关于FreqDist的学习（https://blog.csdn.net/csdn_lzw/article/details/80390768）
all_words = nltk.FreqDist(all_words)
#找出出现频率最高的前15个单词
print(all_words.most_common(15))
#找出指定单词频数
print(all_words["stupid"])
```

![image-20210320133457819](https://cdn.jsdelivr.net/gh/small-brilliant/image/img1/20210320133458.png)

​		接下来将为单词，储存正面或负面的电影评论的特征

### 2.8.1使用NLTK将单词转化为特征

根据上一节的代码：

```python
import nltk
import random
from nltk.corpus import movie_reviews

documents = [(list(movie_reviews.words(fileid)), category)
             for category in movie_reviews.categories()
             for fileid in movie_reviews.fileids(category)]

random.shuffle(documents)

all_words = []

for w in movie_reviews.words():
    all_words.append(w.lower())

all_words = nltk.FreqDist(all_words)
#keys（）返回一个迭代器
word_features = list(all_words.keys())[:3000]
```

`		word_features`包含了前 3000 个最常用的单词。我们将建立一个简单的函数，在我们的正面和负面的文档中找到这些前 3000 个单词，将他们的存在标记为是或否：

```python
def find_features(document):
    words = set(document)
    features = {}
    for w in word_features:
        features[w] = (w in words)
    return features
#下面，我们可以打印除特征集
print((find_features(movie_reviews.words('neg/cv000_29416.txt'))))
```

![image-20210320141335773](https://cdn.jsdelivr.net/gh/small-brilliant/image/img1/20210320141335.png)

之后我们可以为我们所有的文档都做这件事情，通过做下列事情，保存特征存在性布尔值，以及它们各自的正面或负面的类别：

```python
featuresets = [(find_features(rev), category) for (rev, category) in documents]
```

​		现在，我们有了特征和标签，接下来是什么？通常，下一步是继续并训练算法，然后对其进行测试。 所以，让我们继续这样做，从下一个教程中的朴素贝叶斯分类器开始！

### 2.8.2NLTK朴素贝斯分类器

朴素贝叶斯分类器。这是一个非常受欢迎的文本分类算法，然而，在我们可以训练和测试我们的算法之前，我们需要先把数据分解成训练集和测试集。

我们将首先将包含正面和负面评论的 1900 个乱序评论作为训练集。然后，我们可以在最后的 100 个上测试，看看我们有多准确。

这被称为监督机器学习，因为我们正在向机器展示数据，并告诉它“这个数据是正面的”，或者“这个数据是负面的”。然后，在完成训练之后，我们向机器展示一些新的数据，并根据我们之前教过计算机的内容询问计算机，计算机认为新数据的类别是什么。

​		我们可以用以下方式分割数据：

```python
training_set = featuresets[:1900]
testing_set = featuresets[1900:]
```

​		下面，我们可以定义并训练我们的分类器：

```python
classifier = nltk.NaiveBayesClassifier.train(training_set)
```

​		测试：

```python
print("Classifier accuracy percent:",(nltk.classify.accuracy(classifier, testing_set))*100)
```

很快啊！就得到了我们的答案，准确度平均为 60-75%。

![image-20210320142609895](https://cdn.jsdelivr.net/gh/small-brilliant/image/img1/20210320142648.png)

接下来，我们可以进一步了解正面或负面评论中最具有价值的词汇：

```python
classifier.show_most_informative_features(15)
```

这对于每个人都不一样，但是你应该看到这样的东西：

![image-20210320143254541](https://cdn.jsdelivr.net/gh/small-brilliant/image/img1/20210320143254.png)

这个告诉你的是，每一个词的负面到正面的出现几率，或相反。 因此，在这里，我们可以看到，负面评论中的`schumacher `一词比正面评论多出现 11.7 倍。`sucks`是 10.1。

你完全满意你的结果，你想要继续，也许使用这个分类器来预测现在的事情。 训练分类器，并且每当你需要使用分类器时，都要重新训练，是非常不切实际的。 因此，您可以使用`pickle`模块保存分类器。 我们接下来做。

### 2.8.3使用NLTK保存分类器

训练分类器和机器学习算法可能需要很长时间，特别是如果您在更大的数据集上训练。你可以想象，每次你想开始使用分类器的时候，都要训练分类器吗？ 这么恐怖！ 相反，我们可以使用`pickle`模块，并序列化我们的分类器对象，这样我们所需要做的就是简单加载该文件。

#### **pickle**

pickle提供了一个简单的持久化功能。可以将对象以文件的形式存放在磁盘上。

- `pickle.dump(obj, file[, protocol])`
  　　**序列化对象**，并将结果数据流写入到文件对象中。参数`protocol`是序列化模式，默认值为0，表示以文本的形式序列化。`protocol`的值还可以是1或2，表示以二进制的形式序列化。

- `pickle.load(file)`
  　　**反序列化对象**。将文件中的数据解析为一个`Python`对象。

  其中要注意的是，在`load(file)`的时候，要让`python`能够找到类的定义，否则会报错：

1. 第一步是保存对象


```python
import pickle
save_classifier = open("naivebayes.pickle","wb")
pickle.dump(classifier, save_classifier)
save_classifier.close()
```

2. 将其读入内存，这与读取任何其他普通文件一样简单。

```python
classifier_f = open("naivebayes.pickle", "rb")
classifier = pickle.load(classifier_f)
classifier_f.close()
```

这样就可以使用了！我们可以使用这个对象`classifier`，每当我们想用它来分类时，我们不再需要训练我们的分类器。

## 2.9NLTK和Sklearn

NLTK 背后的人们更看重将 sklearn 模块纳入NLTK分类器方法的价值。 就这样，他们创建了各种`SklearnClassifier` API。 要使用它，你只需要像下面这样导入它：

```python
from nltk.classify.scikitlearn import SklearnClassifier
```

从这里开始，你可以使用任何`sklearn`分类器。 例如，让我们引入更多的朴素贝叶斯算法的变体：

```python
from sklearn.naive_bayes import MultinomialNB,BernoulliNB
```

之后：

```python
MNB_classifier = SklearnClassifier(MultinomialNB())
MNB_classifier.train(training_set)
print("MultinomialNB accuracy percent:",nltk.classify.accuracy(MNB_classifier, testing_set))

BNB_classifier = SklearnClassifier(BernoulliNB())
BNB_classifier.train(training_set)
print("BernoulliNB accuracy percent:",nltk.classify.accuracy(BNB_classifier, testing_set))
```

就是这么简单！让我们引入更多东西：

```python
from sklearn.linear_model import LogisticRegression,SGDClassifier
from sklearn.svm import SVC, LinearSVC, NuSVC
```

```python
print("Original Naive Bayes Algo accuracy percent:", (nltk.classify.accuracy(classifier, testing_set))*100)
classifier.show_most_informative_features(15)

MNB_classifier = SklearnClassifier(MultinomialNB())
MNB_classifier.train(training_set)
print("MNB_classifier accuracy percent:", (nltk.classify.accuracy(MNB_classifier, testing_set))*100)

BernoulliNB_classifier = SklearnClassifier(BernoulliNB())
BernoulliNB_classifier.train(training_set)
print("BernoulliNB_classifier accuracy percent:", (nltk.classify.accuracy(BernoulliNB_classifier, testing_set))*100)

LogisticRegression_classifier = SklearnClassifier(LogisticRegression())
LogisticRegression_classifier.train(training_set)
print("LogisticRegression_classifier accuracy percent:", (nltk.classify.accuracy(LogisticRegression_classifier, testing_set))*100)

SGDClassifier_classifier = SklearnClassifier(SGDClassifier())
SGDClassifier_classifier.train(training_set)
print("SGDClassifier_classifier accuracy percent:", (nltk.classify.accuracy(SGDClassifier_classifier, testing_set))*100)

SVC_classifier = SklearnClassifier(SVC())
SVC_classifier.train(training_set)
print("SVC_classifier accuracy percent:", (nltk.classify.accuracy(SVC_classifier, testing_set))*100)

LinearSVC_classifier = SklearnClassifier(LinearSVC())
LinearSVC_classifier.train(training_set)
print("LinearSVC_classifier accuracy percent:", (nltk.classify.accuracy(LinearSVC_classifier, testing_set))*100)

NuSVC_classifier = SklearnClassifier(NuSVC())
NuSVC_classifier.train(training_set)
print("NuSVC_classifier accuracy percent:", (nltk.classify.accuracy(NuSVC_classifier, testing_set))*100)
```

运行结果：

![image-20210320152751042](https://cdn.jsdelivr.net/gh/small-brilliant/image/img1/20210320152751.png)

# 参考

> https://blog.csdn.net/wizardforcel/article/details/79274443
>
> https://blog.csdn.net/dickdick111/article/details/88544624