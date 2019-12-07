## Examining Media Coverage on Climate Change

You can use the [editor on GitHub](https://github.com/kvnchou/climate-change-media/edit/master/README.md) to maintain and preview the content for your website in Markdown files.


### Introduction

Media coverage on societal issues play a large role in shaping public opinion.  As seen in today’s “fake news” phenomenon, how citizens perceive the news can have wide reaching effects throughout the world.  In the past few decades, climate change has been a very divisive issue in the United States.  Based on polling conducted by Gallup, public opinion on climate change has often been nearly evenly split on either sides of the issue.

![https://news.gallup.com/poll/1615/environment.aspx](images/gallup1.png)

In this example survey conducted by Gallup in the United States, optimism surrounding the issue of climate change is measured.  In general, the amount of people who believe that public discourse on climate change is underestimated has been increasing over the last two decades.  This can also be interpreted as people becoming more pessimistic about this subject.  Public polling on the subject of climate change has been robust, and generally show that people are indeed becoming more pessimistic on the subject, especially as the effects of climate change have become more apparent in day to day life.
  
Polls such as these demonstrate the importance that other societal issues play in the discussion of climate change. In the poll above, economic conditions of the country is pitted against the interest of protecting the environment.  This raises the question: how does our news sources present this balance of issues?  If economic issues are important to Americans, does news coverage of climate change incorporate economics as well?

The goal of this research is to examine how news coverage of this issue has changed over the years, and potentially how this would correlate to public opinion over the years.
  
### Research Question

How has media coverage of climate change evolved over the years, in terms of optimism and topic choice?

### Methodology

There are two keys to answering this research question: sentiment analysis and article classification.  Sentiment analysis will be used to determine the optimism expressed in an article, while article classification will be used to determine the purpose and topics in an article.

#### Dictionary Method
Dictionary methods are a good way of performing sentiment analysis.  This method uses predefined dictionaries to categorize texts.  Categories can be split by topic, sentiment, or tone.  Therefore, dictionary methods work best when these categories are well defined, and a dictionary exists which codifies these categories.  These predetermined dictionaries are manually coded by humans, which identifies words and their associated categories.  Articles are then categorized based on the frequency that words appear in each category.  Using a dictionary method has a similar outcome as creating a supervised learning model.  However, while a supervised learning model requires large amounts of data for the training set, dictionary models can be employed with a small number of documents.  

Dictionary methods are context invariant, which means that words are weighted the same regardless of the context of their documents.  The problem with this is that a provided dictionary cannot ensure that the words are accurate within the context of the actual documents.  However, as the purpose of this research is to categorize the optimism of the articles, using a dictionary is appropriate, as well established dictionaries with this classification already exist.  The qdap dictionary in R will be used to classify documents by their positivity/negativity, which is an effective measurement of optimism.

#### Supervised Learning Method
In addition to using dictionary methods to classify the documents, a supervised learning model will be developed to further classify the documents by purpose.  A supervised learning model is used, because the categories used by the model will be predefined (political and economic articles).  Therefore, there is no need to for any sort of topic discovery.

The process of developing a supervised learning model requires for a training data set to be code manually first.  This data set is then used to train a model to classify articles into the predetermined categories.  For the most part, supervised learning models utilize the bag of words method, which is a very simple way of extracting features from text.  In essence, documents are reduced down to their individual words, or “tokens”.  The tokens can be counted and converted into a vector format, such as a Document Term Matrix (DTM).  These matrices represent how often each term, or token, appear in a certain document.  This data structure is used by machine learning algorithms to develop classifier models.

To develop a supervised learning model for this research, a portion of the collected news articles will need to be manually coded for the two categories: political and economic.  A lot of this process comes down to manual judgement, as sometimes it can be hard to tell if an article is “focusing” on the political issues involved with climate change.  The key is to remain consistent throughout the process.

### Collecting the Data
News articles containing the phrase “climate change” were collected from the New York Times over the years 2000-2019.  There are a multitude of terms to describe this phenomenon, such as “global warming” or “climate crisis”.  Ultimately, the phrase “climate change” was used as it is currently the most accepted term to describe the phenomenon.  The number of articles containing the phrase “climate change” is already very large, so collecting additional data did not seem necessary.

The news articles were collected using the New York Times public developer API.  This API was used because it provides keyword search functionality that returns the locations of all articles containing certain keywords.  The collected URLs to articles were then used to web scrape the New York Times website for full texts of articles.  The New York Times website provides uniform formatting for all articles through the years, making web scraping a simple and efficient process of acquiring large amounts of data.

The following code snippet was used to query the New York Times API, web scrape the retrieved HTML for the full news article content, and finally saved into CSV format for safe keeping.

```markdown
Syntax highlighted code block
library(jsonlite)
library(magrittr)

getYears <- function(start_year, end_year) {
  term <- "climate+change" # Need to use + to string together separate words
  begin_date <- paste0(start_year, "0101")
  end_date <- paste0(end_year, "0101")

 
  baseurl <- paste0("http://api.nytimes.com/svc/search/v2/articlesearch.json?q=",term,
                    "&begin_date=",begin_date,"&end_date=",end_date,
                    "&facet_filter=true&api-key=", apikey, sep="")
  initialQuery <- fromJSON(baseurl)
  maxPages <- round((initialQuery$response$meta$hits[1] / 10)-1)
 
  pages <- list()
  for(i in 0:maxPages) {
    possibleError <- tryCatch(
      nytSearch <- fromJSON(paste0(baseurl, "&page=", i), flatten = TRUE) %>% data.frame(),
      error = function(e) e
    )
    if(!inherits(possibleError, "error")) {
      
      message("Retrieving page ", i)
      pages[[i+1]] <- nytSearch
    }
    Sys.sleep(2)
  }
 
  allNYTSearch <- rbind_pages(pages)
  library(dplyr)
  allDocs <- select(allNYTSearch, response.docs.pub_date, response.docs.web_url)
 
  library(rvest)
 
  fullTexts <- vector()
  dates <- list()
 
  allDates <- allDocs$response.docs.pub_date
  allUrls <- allDocs$response.docs.web_url
  for (i in 1:length(allUrls)) {
    print(paste0("Retrieving document ", i))
    possibleError <- tryCatch(
      doc <- read_html(allUrls[i]),
      error = function(e) e
    )
    if(!inherits(possibleError, "error")) {
      results <- doc %>% html_nodes(".css-exrw3m.evys1bk0")
      article <- ""
      for (j in 1:length(results)) {
        article <- paste(article, toString(xml_contents(results[j])))
      }
      
      dates <- c(dates, allDates[i])
      fullTexts <- c(fullTexts, article)
    }
  }
 
  matrix <- do.call(rbind, Map(data.frame, dates=dates, text=fullTexts))
  write.csv(matrix, paste0(start_year, ".csv"))
}

cleanHtml <- function(htmlString) {
  return(gsub("<.*?>", "", htmlString))
}

# Header 1
## Header 2
### Header 3

- Bulleted
- List

1. Numbered
2. List

**Bold** and _Italic_ and `Code` text

[Link](url) and ![Image](src)
```
Notes: a few web scraped articles had to be “cleaned” of HTML tags, as some articles did have irregular content formats

### Processing the Data
  With the news articles stored as raw text, the next step was to build a corpus of documents to perform analysis on.  A few steps were taken to clean the data: standardizing to lowercase letters, removing stop words, stripping whitespaces, removing punctuation, and stemming.  The stop words removed are defined in the ‘tm’ R library.
  
```markdown
library(tm)

corpus <- Corpus(VectorSource(fullTexts))
tm_map(corpus, function(x) iconv(enc2utf8(x), sub = "byte"))

# standardize to lowercase
corpus <- tm_map(corpus, content_transformer(tolower))
# remove tm stopwords
corpus <- tm_map(corpus, removeWords, stopwords())
# strip whitespaces
corpus <- tm_map(corpus, stripWhitespace)
# remove punctuation
corpus <- tm_map(corpus, removePunctuation)
```
Next, document term matrices were built for use with the supervised learning model algorithms.

```markdown
dtm <- DocumentTermMatrix(corpus)
tdm <- TermDocumentMatrix(corpus2)
```
#### Coding a Training Set
A subset of the dataset was manually coded for the two categories as demonstrated below.

Due to the difficulty and tediousness of manually coded long articles, a total of 50 articles were coded.  The articles were chosen at random from the dataset, in order to get a wide range of scenarios.  These 50 articles will be used for training and validation of the supervised learning model.





For more details see [GitHub Flavored Markdown](https://guides.github.com/features/mastering-markdown/).

### Jekyll Themes

Your Pages site will use the layout and styles from the Jekyll theme you have selected in your [repository settings](https://github.com/kvnchou/climate-change-media/settings). The name of this theme is saved in the Jekyll `_config.yml` configuration file.

### Support or Contact

Having trouble with Pages? Check out our [documentation](https://help.github.com/categories/github-pages-basics/) or [contact support](https://github.com/contact) and we’ll help you sort it out.
