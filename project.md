---
title: "Report Submission"
author: "Francisco Moreno Arcas"
date: "Saturday, July 25, 2015"
output: html_document
---

#Sumary

We will explore the dateset given by ![Coursera and Swiftkey](https://d396qusza40orc.cloudfront.net/dsscapstone/dataset/Coursera-SwiftKey.zip); you can download it clicking the link before.

This dataset has a folder called **final** with the following:

* de_DE
    + blogs
    + news
    + twitter
* en_US
    + blogs
    + news
    + twitter
* fi_FI
    + blogs
    + news
    + twitter
* ru_RU
    + blogs
    + news
    + twitter

We will focus our analysis in the **en_US** folder, those files have text information that we will explore in the next steps.

#Preparation

Loading libraries:


```r
library("tm")
```

```
## Loading required package: NLP
```

```r
library("slam") #usefull to work with great matrixes
library("ggplot2")
```

```
## 
## Attaching package: 'ggplot2'
## 
## The following object is masked from 'package:NLP':
## 
##     annotate
```

```r
library("wordcloud")
```

```
## Loading required package: RColorBrewer
```

```r
library("NLP")
library("RWeka")
library("RColorBrewer")
```

After we set the working directory in the fold we downloaded and extracted the files:


```r
setwd("C:/Data Scientific/Project")
```

#Loading and cleaning the data:

We store the names of the files in some vars:


```r
fileNameTwitter <- "Docs/en_US.twitter.txt"
fileNameNews <- "Docs/en_US.news.txt"
fileNameBlogs <- "Docs/en_US.blogs.txt"
```

File sizes are:

- en_US.twitter.txt 159.364069 MB
- en_US.news.txt 196.2775126 MB
- en_US.blogs.txt 200.4242077 MB

Using in a bash shell wc -l *.txt we obtain the amount of lines of each file:

```
  899288 en_US.blogs.txt
 1010242 en_US.news.txt
 2360148 en_US.twitter.txt
```

The total of lines are 4269678 with more than the 50% are in the twitter file.

Also changing -l for -w to count the words of each file, wc -w *.txt:

```
 37334690 en_US.blogs.txt
 34372720 en_US.news.txt
 30374206 en_US.twitter.txt
```

The total of words are 102,081,616 among the three files.

With each of files we read from the stream and clean removing:
*html urls
*numbers
*punctuation
*stripping white spaces
*convert to plain text to avoid errors during term matrix conversion


```r
#Loading data section

#open the stream for each file
con <- file(fileNameTwitter,open="r")
#read only the first 10000 lines of each document because the frequency doesn't change too much when use the full text file
linesTwitter <- readLines(con,encoding="UTF-8",10000)
#close the stream
close(con)

con <- file(fileNameNews,open="r")
linesNews <- readLines(con,encoding="UTF-8",10000)
close(con)

con <- file(fileNameBlogs,open="r")
linesBlogs <- readLines(con,encoding="UTF-8",10000)
close(con)

#concatenation of the files
data <- paste(linesTwitter,linesNews,linesBlogs)

#create the main corpus
data <- VCorpus(VectorSource(data))

#Cleaning section

#transform to Lower Case, removing http patterns, numbers, puntuation and
#removing trailing spaces
#data <- tm_map(data, content_transformer(tolower))
removeURL <- function(x) gsub("http[[:alnum:]]*", "", x)
data <- tm_map(data, removeURL)

data <- tm_map(data, removeNumbers)
data <- tm_map(data, removePunctuation)
data <- tm_map(data, stripWhitespace)
data <- tm_map(data, PlainTextDocument)
```

With the package SnowballC we perform a stemming to the document.


```r
require(SnowballC)
```

```
## Loading required package: SnowballC
```

```r
corpus<- tm_map(data, stemDocument)
```

#NGrams tokens and Exploratory Analysis

With the corpus prepared, we made NGram tokenization with the data, means that we put the differents combinations of 1,2,3,4 words with the frequency associated to each one.


```r
# Create the corpus
corpus<-data.frame(text=unlist(sapply(corpus, `[`, "content")), stringsAsFactors=F)
```

##One word tokens 


```r
onetoken <- NGramTokenizer(corpus, Weka_control(min = 1, max = 1))
one <- data.frame(table(onetoken))
onesorted <- one[order(one$Freq,decreasing = TRUE),]
```

Wordcloud visualization with the top 20 ngram of 1 word


```r
wordcloud(onesorted[1:20,"onetoken"],onesorted[1:20,"Freq"], colors=brewer.pal(8, "Dark2"))
```

<img src="project_files/figure-html/unnamed-chunk-8-1.png" title="" alt="" width="672" />

Histogram plot with the top 40 ngram of 1 word


```r
ggplot(onesorted[1:40,], aes(x=onetoken,y=Freq)) + geom_bar(stat="Identity", fill="blue") + theme(axis.text.x = element_text(angle = 90, hjust = 1))
```

<img src="project_files/figure-html/unnamed-chunk-9-1.png" title="" alt="" width="672" />

##Two word tokens


```r
twotoken <- NGramTokenizer(corpus, Weka_control(min = 2, max = 2,delimiters = " \\r\\n\\t.,;:\"()?!"))
two <- data.frame(table(twotoken))
twosorted <- two[order(two$Freq,decreasing = TRUE),]
```

Wordcloud visualization with the top 20 ngram of 2 words


```r
wordcloud(twosorted[1:20,"twotoken"],twosorted[1:20,"Freq"], rot.per = 0.35,  colors=brewer.pal(8, "Dark2"))
```

<img src="project_files/figure-html/unnamed-chunk-11-1.png" title="" alt="" width="672" />

Histogram plot with the top 40 ngram of 2 words


```r
ggplot(twosorted[1:40,], aes(x=twotoken,y=Freq)) + geom_bar(stat="Identity", fill="blue") + theme(axis.text.x = element_text(angle = 90, hjust = 1))
```

<img src="project_files/figure-html/unnamed-chunk-12-1.png" title="" alt="" width="672" />

##Three word tokens


```r
threetoken <- NGramTokenizer(corpus, Weka_control(min = 3, max = 3, delimiters = " \\r\\n\\t.,;:\"()?!"))
three <- data.frame(table(threetoken))
threesorted <- three[order(three$Freq,decreasing = TRUE),]
```

Wordcloud visualization with the top 20 ngram of 3 words


```r
wordcloud(threesorted[1:20,"threetoken"], threesorted[1:20,"Freq"], rot.per = 0.35, colors=brewer.pal(8, "Dark2"))
```

<img src="project_files/figure-html/unnamed-chunk-14-1.png" title="" alt="" width="672" />

Histogram plot with the top 40 ngram of 3 words


```r
ggplot(threesorted[1:40,], aes(x=threetoken,y=Freq)) + geom_bar(stat="Identity", fill="blue") + theme(axis.text.x = element_text(angle = 90, hjust = 1))
```

<img src="project_files/figure-html/unnamed-chunk-15-1.png" title="" alt="" width="672" />

##Four word tokens


```r
fourtoken <- NGramTokenizer(corpus, Weka_control(min = 4, max = 4,delimiters = " \\r\\n\\t.,;:\"()?!"))
four <- data.frame(table(fourtoken))
foursorted <- four[order(four$Freq,decreasing = TRUE),]
```

Wordcloud visualization with the top 5 ngram of 4 words


```r
wordcloud(foursorted[1:5,"fourtoken"],foursorted[1:5,"Freq"], rot.per = 0.35, colors=brewer.pal(8, "Dark2"))
```

<img src="project_files/figure-html/unnamed-chunk-17-1.png" title="" alt="" width="672" />

Histogram plot with the top 40 ngram of 4 words


```r
ggplot(foursorted[1:40,], aes(x=fourtoken,y=Freq)) + geom_bar(stat="Identity", fill="blue") + theme(axis.text.x = element_text(angle = 90, hjust = 1))
```

<img src="project_files/figure-html/unnamed-chunk-18-1.png" title="" alt="" width="672" />
