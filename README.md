# Independent_Study_Module_170017784
Code for MT5990 Independent Study Module

# ================ Source Data through establishing API Connections ================ #
require(coindeskr)
require(RMariaDB)
require(DBI)
require(cronR)
require(mailR)

# ================ Establish Connection to Maria Database (MySQL)
con <- dbConnect(RMariaDB::MariaDB(), host = "rwr3.host.cs.st-andrews.ac.uk",
                 user = "rwr3", 
                 password = "!8Wq2!vJY8WT6v",
                 db = "rwr3_Bitcoin")

# ================ Get Bitcoin API Data
BTC_Data <- get_current_price("BTC")
BTC_Data <- BTC_Data[c(1, 5, 6)]


# ================ Write Data to the Database
dbWriteTable(conn = con, name = 'Bitcoin_Data', value = BTC_Data, overwrite = FALSE, append = TRUE)

# ================ Email Receipt of completed process 
sender <- "roryroberts2104@gmail.com"
recipients <- "roryroberts2104@gmail.com"
xxx <- "St.Andrews2018"
#BTC_File <- write.csv(BTC_Data, file = paste(timestamp(),"_Bitcoin.csv"))
email <- send.mail(from = sender, 
                    to = recipients, 
                    subject = paste("BTC_Data_",timestamp()), 
                    body = "See Attached", 
                    encoding = "utf-8",
                    html = TRUE, 
                    inline = TRUE, 
                    authenticate = TRUE,
                    smtp = list(host.name = "smtp.gmail.com",
                                port = 465,
                                user.name = "roryroberts2104@gmail.com",
                                passwd = xxx,
                                ssl = TRUE), 
                    send = TRUE,
                    debug = TRUE)

# ================ Read Data from the Database
# dbReadTable(con, 'Bitcoin_Data')

require(twitteR)
require(ROAuth)
require(RMariaDB)
require(mailR)

# ================ Establish Connection to Maria Database (MySQL)
con_tweets <- dbConnect(RMariaDB::MariaDB(), host = "rwr3.host.cs.st-andrews.ac.uk",
                 user = "rwr3", 
                 password = "!8Wq2!vJY8WT6v",
                 db = "rwr3_Twitter")

# ================== Create Twitter API ================== #
api_key <- "E8N3eRd1OvIUO3x8zZeFDn5lD"
api_secret <- "nsT8MXEje4BY31UCbSNLfV3bfxJfMVdgwSploOzePSYJfCJOP7"
access_token <- "967357803672297472-YOiw8qFU7XMvvzNP28hZ5qolyullnLR"
access_token_secret <- "GuJSCnI8fgjeZrbqHoMDt1A2AAw6ubgzlS9aphWcYN9jY"

# Access Twitter Api
twitter_api = setup_twitter_oauth(api_key,api_secret,access_token,access_token_secret) # Twitter Feed API

# ================== Fetch tweets by keywords ================== #
bitcoin <- searchTwitter("bitcoin", n = 1000) # create a for loop for this to run in a loop
tweetsDF <- twListToDF(bitcoin) # Convert the tweets to a Dataframe
tweetsDF <- tweetsDF[c(1,2,5,8,12,13,14)]
colnames(tweetsDF) <- c("tweets", "favorited", "created", "id", "retweetCount", "isRetweet", "retweeted")

# ================ Write Data to the Database
dbWriteTable(conn = con_tweets, name = 'Twitter_Feed', value = tweetsDF, overwrite = FALSE, append = TRUE)

# ================ Email Receipt of completed process 
sender <- "roryroberts2104@gmail.com"
recipients <- "roryroberts2104@gmail.com"
xxx <- "St.Andrews2018"
#BTC_File <- write.csv(BTC_Data, file = paste(timestamp(),"_Bitcoin.csv"))
email <- send.mail(from = sender, 
                   to = recipients, 
                   subject = paste("Twitter_Data_",timestamp()), 
                   body = "See Attached", 
                   encoding = "utf-8",
                   html = TRUE, 
                   inline = TRUE, 
                   authenticate = TRUE,
                   smtp = list(host.name = "smtp.gmail.com",
                               port = 465,
                               user.name = "roryroberts2104@gmail.com",
                               passwd = xxx,
                               ssl = TRUE), 
                   send = TRUE,
                   debug = TRUE)

# ================ Write Data t=from the Database
#dbReadTable(con_tweets, 'Twitter_Feed')

# ========== Obtain data from the database ========== #
devtools::session_info()
install.packages("devtools")
require("devtools")
devtools::install_deps(dependencies = TRUE)
# ================ Establish Connection to Maria Database (MySQL)
require(RMySQL)
m<-MySQL()
summary(m)
connect<-dbConnect(m, dbname = "rwr3_Bitcoin", host="rwr3.host.cs.st-andrews.ac.uk", 
               port=3306, 
               user="rwr3", 
               pass="!8Wq2!vJY8WT6v") 
               #unix.sock="/Applications/MAMP/tmp/mysql/mysql.sock") # in case you are using MAMP 
tables <- dbListTables(connect)
Bitcoin_Data <- dbReadTable(conn = connect, name = tables)


connect_Twitter <-dbConnect(m, dbname = "rwr3_Twitter", host="rwr3.host.cs.st-andrews.ac.uk", 
                   port=3306, 
                   user="rwr3", 
                   pass="!8Wq2!vJY8WT6v") 
#unix.sock="/Applications/MAMP/tmp/mysql/mysql.sock") # in case you are using MAMP 
tables_Twitter <- dbListTables(connect_Twitter)
Twitter_Data <- dbReadTable(conn = connect_Twitter, name = tables_Twitter)

write.csv(Bitcoin_Data, file = "Bitcoin_Data_09_04_2018.csv")
write.csv(Twitter_Data, file = "Twitter_09_04_2018.csv")

# ========== Run Files on a Unix Scheduler ========== #
#!/bin/bash
(
    /usr/lib64/R/bin/Rscript '/cs/home/rwr3/Connect_R_to_Bitcoin_DB.R' && \
    /usr/lib64/R/bin/Rscript '/cs/home/rwr3/Connect_R_to_Twitter_DB.R'
) || { echo "Failed!"; exit 1; }
echo $0 | at -M now + 1 hour

# ========== Import Data Set from CSV files for Bitcoin Data and Twitter API Feeds ========== #
options(stringsAsFactors = FALSE)

Bitcoin_Data <- read.csv("~/Dropbox/02 Semister Two/IndependentStudyModule/Final_Project_Data/Bitcoin_Data_04_04_2018.csv")
Bitcoin_Data <- Bitcoin_Data[c(29:148),c(2:4)] # Remove Prices not taken at one hour intervals
colnames(Bitcoin_Data) <- c("Date_Time", "Currency", "Price") # Update Column Names to something meaningful

Twitter_Data <- read.csv("~/Dropbox/02 Semister Two/IndependentStudyModule/Final_Project_Data/Twitter_04_04_2018.csv", 
                         comment.char="#")
Twitter_Data <- Twitter_Data[11:149020,] # Align with Bitcoin Dates

# ========== Install Packages ========== #
# Intall Packages
install.packages("stringr")
install.packages("tidytext")
install.packages("stopwords")
install.packages("tm")
install.packages("corpus")
install.packages("wordcloud")
install.packages("hunspell")
#install.packages("qdap") - This Crashes R
require("stringr")
library("tidytext")
library("ggplot2")
library("dplyr")
library("stringr")
library("tidyr")
require("stopwords")
require("tm")
#require("qdap") - This Crashes R
require("corpus")
library("wordcloud")
require("hunspell")

# ========== References ========== #
# https://rstudio-pubs-static.s3.amazonaws.com/343661_dc127bbf141845b083b2dfa2cc75c9d2.html
# ========== Clean the Twitter Data ========== #
# Encode to native for data cleansing
x <- Twitter_Data
x$tweets <- as.character(x$tweets)
x$text <- enc2native(x$tweets)

# Show which tweets have been scrapped twice from Twitter
x$duplicated <- duplicated(x$text, incomparables = FALSE)
sum(x$duplicated == "TRUE") # The total Number equals 66524/149010 - this equates to 65% of the tweets are duplicated
# this may indicate at the web scrapping interval of a hour may be to regular

# Remove Duplicated Rows (this will also remove blank rows)
x <- subset(x, duplicated != "TRUE") # Total remaining of 82486
rm(Twitter_Data)

# Extract URLs
url_pattern <- "http[s]?://(?:[a-zA-Z]|[0-9]|[$-_@.&+]|[!*\\(\\),]|(?:%[0-9a-fA-F][0-9a-fA-F]))+"
x$contentURL <- str_extract(x$text, url_pattern)

# Clean content of text
x$text <- removePunctuation(x$text) # Remove Punctuation
#x$text <- gsub("'", "%%", x$text) # Replacing apostrophes with %%
x$text <- iconv(x$text, "latin1", "ASCII", sub="") #Remove emojis and other Unicode characters
x$text <- gsub("<(.*)>", "", x$text) #Remove additional Unicode parts that may have remained like <U+A>
#x$text <- gsub("\\ \\. ", " ", x$text) # Replace orphaned fullstops with space
#x$text <- gsub("  ", " ", x$text) # Replace double space with single space
#x$text <- gsub("%%", "\'", x$text) # Changing %% back to apostrophes
x$text <- gsub("https(.*)*$", "", x$text) # remove tweet URL
x$text <- gsub("\\n", "-", x$text) # replace line breaks with
x$text <- gsub("--", "-", x$text) # remove double - from double line breaks (Remove double hyphens where there were two line breaks)
x$text <- gsub("&amp;", "&", x$text) # Fix ampersand &
x$text[x$text == " "] <- "<no text>" # Add string to empty values (when only a URL was posted)
x$text <- gsub("\\s*\\([^\\)]+\\)","",as.character(x$text)) # Remove text with brackets
#x$text <- removeNumbers(x$text) # Remove numbers - not needed due to stemming
x$text <- gsub("^[[:space:]]*","",x$text) # Remove leading whitespaces
x$text <- gsub("[[:space:]]*$","",x$text) # Remove trailing whitespaces
x$text <- gsub(" +"," ",x$text) # Remove extra whitespaces

clean_data_tokens <- text_tokens(x$text) # Token Vector 

# Create a corpus - A collection of written texts
tweets_corpus <- Corpus(VectorSource(clean_data_tokens))
tweets_corpus <- tm::tm_map(tweets_corpus, tm::removeWords, tm::stopwords("english")) # Remove Stopwords
inspect(tweets_corpus)

# Reference: https://rpubs.com/sgeletta/95577
# Create patterns to elimina special code and other patterns
# Emails
emlPat <- function(x) gsub("\\b[A-Z a-z 0-9._ - ]*[@](.*?)[.]{1,3} \\b", "", tweets_corpus)
tweets_corpus <- tm_map(tweets_corpus, emlPat)
# Twitter tags
tt<-function(x) gsub("RT |via", "", x)
tweets_corpus<- tm_map(tweets_corpus, tt)
# Twitter Usernames
tun<-function(x) gsub("[@][a - zA - Z0 - 9_]{1,15}", "", x)
tweets_corpus<- tm_map(tweets_corpus, tun)
rm(emlPat, tt, tun, url_pattern)

# Create a Term Document Matrix
# Reference: https://rpubs.com/lismajena/twitter
wordFreq <- function(corpus, word) {
  results <- lapply(tweets_corpus,
                    function(x) { grep(as.character(x), pattern=paste0("\\<",word)) }
  )
  sum(unlist(results))
}
tdm <- TermDocumentMatrix(tweets_corpus)
tdm <- rowSums(as.matrix(tdm)) 
tdm <- sort(tdm, decreasing = T) # Sort in Decreasing order 
#tdm <- removeSparseTerms(tdm, 0.7) # set an acceptable level of Sparseness
clean_tweets_tdm <- data.frame(term = names(tdm), freq = tdm)
clean_tweets_tdm$weight <- clean_tweets_tdm$freq/sum(clean_tweets_tdm$freq) # Log weights to mode clearly indicate changes
clean_tweets_tdm$ln_freq <- log(clean_tweets_tdm$freq)
clean_tweets_tdm

# Complete Stemming to create uniformity within the text
# Reference: http://www.lexiconista.com/Datasets/lemmatization-en.zip
url <- "http://www.lexiconista.com/Datasets/lemmatization-en.zip"
tmp <- tempfile()
download.file(url, tmp)
# extract the contents
con <- unz(tmp, "lemmatization-en.txt", encoding = "UTF-8") # unzip the English text file with suitable encoding format
stem_doc <- read.delim(con, header=FALSE, stringsAsFactors = FALSE) # Read the file
names(stem_doc) <- c("stem", "term") # Rename the columns
stem_doc2 <- new_stemmer(stem_doc$term, stem_doc$stem) # Make a stemmer from a set of (term, stem) pairs
clean_tweets_tdm$stemmed <- stem_doc2(as.character(clean_tweets_tdm$term))
rm(url, tmp, con, stem_doc, stem_doc2)

# Highlight Duplicated Stems 
clean_tweets_tdm$stemdup <- duplicated(as.character(clean_tweets_tdm$stemmed))
sum(clean_tweets_tdm$stemdup == TRUE)/nrow(clean_tweets_tdm) # Equals 5% of duplicates in Term Document Matrix

install.packages("dplyr")
install.packages("tables")
require("dplyr")
library("qwraps2")
require("tables")
# ========== Clean Lexicon Documents ========== #
# ========== Lexicons ========== #
# ================== Sentiment Word List ================== #
# ================== AFINN111
# AFINN111 Given on a scale of -5 to +5
AFINN.111 <- read.delim("~/Downloads/AFINN/AFINN-111.txt", header=FALSE)
colnames(AFINN.111) <- c("word", "Senti.AFINN")

AFINN.111$Pos.Senti.AFINN <- with(AFINN.111, ifelse(AFINN.111$Senti.AFINN > 0, AFINN.111$Senti.AFINN, 0)) # Separate to Positive Sentiment
AFINN.111$Neg.Senti.AFINN <- with(AFINN.111, ifelse(AFINN.111$Senti.AFINN < 0, AFINN.111$Senti.AFINN, 0)) # Separate to Negative Sentiment
colnames(AFINN.111) <- c("X", "word", "Pos.Senti.AFINN", "Neg.Senti.AFINN")

nrow(AFINN.111)

n_perc(AFINN.111$Pos.Senti.AFINN > 0) # Calculate the Percentage of Positive Words
n_perc(AFINN.111$Neg.Senti.AFINN < 0) # Calculate the Percentage of Neg Words

table(c(AFINN.111$Neg.Senti.AFINN, AFINN.111$Pos.Senti.AFINN)) # Summary table of Results
AFINN.Results.Pos <- AFINN.111$Pos.Senti.AFINN[which(AFINN.111$Pos.Senti.AFINN > 0)]
AFINN.Results.Neg <- AFINN.111$Neg.Senti.AFINN[which(AFINN.111$Neg.Senti.AFINN < 0)]
AFINN.Results <- cbind(AFINN.Results.Pos, AFINN.Results.Neg)

write.csv(t(table(c(AFINN.111$Neg.Senti.AFINN, AFINN.111$Pos.Senti.AFINN))), "AFINN.111_Stats.csv") # Export Results

# Convert to -1 and + 1
AFINN.111$New_Pos <- with(AFINN.111, ifelse(AFINN.111$Pos.Senti.AFINN > 0, 1, 0)) # Classify
AFINN.111$New_Neg <- with(AFINN.111, ifelse(AFINN.111$Neg.Senti.AFINN < 0, -1, 0)) # Classify
AFINN.111$Dictionary <- "AFINN111"
AFINN.111 <- AFINN.111[,c(1,5,6,7)]
colnames(AFINN.111) <- c("Word", "New_Pos", "New_Neg", "Dictionary")
head(AFINN.111)
# ================== UIC Resource 
pos_uic_edu <- read.csv("~/Dropbox/02 Semister Two/IndependentStudyModule/MT5990_Independent_Study_Module_Lab_Data_06.04.2018/pos_uic_edu.csv")
neg_uic_edu <- read.csv("~/Dropbox/02 Semister Two/IndependentStudyModule/MT5990_Independent_Study_Module_Lab_Data_06.04.2018/neg_uic_edu.csv")

dim(pos_uic_edu)
dim(neg_uic_edu)

colnames(pos_uic_edu) <- c("X", "Word", "Score")
colnames(neg_uic_edu) <- c("X", "Word", "Score")

UIC_EDU <- rbind(pos_uic_edu, neg_uic_edu)
UIC_EDU$Dictionary <- "UCI_EDU"
UIC_EDU$New_Pos <- ifelse(UIC_EDU$Score > 0, 1, 0) # Classify
UIC_EDU$New_Neg <- ifelse(UIC_EDU$Score < 0, -1, 0) # Classify
UIC_EDU <- UIC_EDU[, c(2,5,6,4)] # Correct order to align with other lists
head(UIC_EDU)
colnames(UIC_EDU) <- c("Word", "New_Pos", "New_Neg", "Dictionary")

# ================== Senti Word List
SentiWordNet_3.0.0_20130122 <- read.table("~/Downloads/home/swn/www/admin/dump/SentiWordNet_3.0.0_20130122.txt", quote="\"")
colnames(SentiWordNet_3.0.0_20130122) <- c("position", "id", "pos.score", "neg.score", "Word")
SentiWordNet_3.0.0_20130122$Difference <- SentiWordNet_3.0.0_20130122$pos.score - SentiWordNet_3.0.0_20130122$neg.score
SentiWordNet_3.0.0_20130122$New_Pos <- ifelse(SentiWordNet_3.0.0_20130122$Difference > 0, 1, 0)
SentiWordNet_3.0.0_20130122$New_Neg <- ifelse(SentiWordNet_3.0.0_20130122$Difference < 0, -1 ,0)
SentiWordNet_3.0.0_20130122$Dictionary <- "SentiWordNet_3.0"

table(SentiWordNet_3.0.0_20130122$Difference)
sum(SentiWordNet_3.0.0_20130122$New_Pos == 0 & SentiWordNet_3.0.0_20130122$New_Neg == 0)
sum(SentiWordNet_3.0.0_20130122$pos.score > 0 
    & SentiWordNet_3.0.0_20130122$neg.score > 0 
    & SentiWordNet_3.0.0_20130122$Difference == 0)/nrow(SentiWordNet_3.0.0_20130122)
SentiWordNet_3.0.0_20130122 <- SentiWordNet_3.0.0_20130122[!(SentiWordNet_3.0.0_20130122$New_Pos == 0 & SentiWordNet_3.0.0_20130122$New_Neg == 0),]
head(SentiWordNet_3.0.0_20130122)
write.csv(t(table(SentiWordNet_3.0.0_20130122$pos.score)), file = "Pos_Senti_Senti.csv")
write.csv(t(table(SentiWordNet_3.0.0_20130122$neg.score)), file = "Neg_Senti_Senti.csv")

SentiWordNet <- SentiWordNet_3.0.0_20130122[ , c(5,7:9)]
colnames(SentiWordNet) <- c("Word", "New_Pos", "New_Neg", "Dictionary")
head(SentiWordNet)

# ==================  LoughranMcDonald_MasterDictionary - Incorrectly coded in CSV, pos and negative values updated accordingly
LoughranMcDonald_MasterDictionary_2016 <- read.csv("~/Downloads/LoughranMcDonald_MasterDictionary_2016.csv")
LoughranMcDonald_MasterDictionary_2016$New_pos <- ifelse(LoughranMcDonald_MasterDictionary_2016$Positive > 0 ,1,0)
LoughranMcDonald_MasterDictionary_2016$New_neg <- ifelse(LoughranMcDonald_MasterDictionary_2016$Negative > 0 ,-1,0)
LoughranMcDonald_MasterDictionary_2016$Dictionary <- "LoughranMcDonald"
table(LoughranMcDonald_MasterDictionary_2016$New_pos == 0 & LoughranMcDonald_MasterDictionary_2016$New_neg == 0)
head(LoughranMcDonald_MasterDictionary_2016)
LoughranMcDonald_MasterDictionary_2016 <- LoughranMcDonald_MasterDictionary_2016[, c(1,4:6)]
colnames(LoughranMcDonald_MasterDictionary_2016) <- c("Word", "New_Pos", "New_Neg", "Dictionary")
head(LoughranMcDonald_MasterDictionary_2016)

# ================== Obtain Consolidated Lexicon List
head(AFINN.111)
head(UIC_EDU)
head(SentiWordNet)
head(LoughranMcDonald_MasterDictionary_2016)
sum(nrow(AFINN.111), nrow(UIC_EDU), nrow(SentiWordNet), nrow(LoughranMcDonald_MasterDictionary_2016))

Final_Consolidated_Lexicon <- rbind(AFINN.111, UIC_EDU, SentiWordNet, LoughranMcDonald_MasterDictionary_2016)
Final_Consolidated_Lexicon <- Final_Consolidated_Lexicon[-7491,] # Remove Multibyte String
rm(AFINN.111, UIC_EDU, SentiWordNet, SentiWordNet_3.0.0_20130122, LoughranMcDonald_MasterDictionary_2016)
rm(neg_uic_edu, pos_uic_edu, AFINN.Results, AFINN.Results.Neg, AFINN.Results.Pos)
# ================== Improve Data Quality ================== #
# ================== Reduce all to lower Case
Final_Consolidated_Lexicon$Word <- as.character(Final_Consolidated_Lexicon$Word)
Final_Consolidated_Lexicon$Word <- as.list(tolower((Final_Consolidated_Lexicon$Word))) # Lower Case

# Remove cases where both Positive and Negative Senti equals 0
sum(Final_Consolidated_Lexicon$New_Pos == 0 
    & Final_Consolidated_Lexicon$New_Neg == 0) 

sum(Final_Consolidated_Lexicon$New_Pos == 0 
    & Final_Consolidated_Lexicon$New_Neg == 0)/nrow(Final_Consolidated_Lexicon) # 82513/122341
Final_Consolidated_Lexicon <- Final_Consolidated_Lexicon[!(Final_Consolidated_Lexicon$New_Pos==0 
                                                           & Final_Consolidated_Lexicon$New_Neg ==0),] # 39827 Remain

# ================== Complete Stemming
# Complete Stemming to create uniformity within the text
# Reference: http://www.lexiconista.com/Datasets/lemmatization-en.zip
url <- "http://www.lexiconista.com/Datasets/lemmatization-en.zip"
tmp <- tempfile()
download.file(url, tmp)
# extract the contents
con <- unz(tmp, "lemmatization-en.txt", encoding = "UTF-8") # unzip the English text file with suitable encoding format
stem_doc <- read.delim(con, header=FALSE, stringsAsFactors = FALSE) # Read the file
names(stem_doc) <- c("stem", "term") # Rename the columns
stem_doc2 <- new_stemmer(stem_doc$term, stem_doc$stem) # Make a stemmer from a set of (term, stem) pairs
Final_Consolidated_Lexicon$Word <- stem_doc2(Final_Consolidated_Lexicon$Word)
rm(url, tmp, con, stem_doc, stem_doc2)

# ================== De-Duplicate Repeated Words 
Final_Consolidated_Lexicon$Duplicated <- duplicated(as.character(Final_Consolidated_Lexicon$Word))
Final_Consolidated_Lexicon_DeDup <- subset(Final_Consolidated_Lexicon, Final_Consolidated_Lexicon$Duplicated == FALSE)
head(Final_Consolidated_Lexicon_DeDup)
Final_Consolidated_Lexicon_DeDup <- Final_Consolidated_Lexicon_DeDup[,c(1:4)]
head(Final_Consolidated_Lexicon_DeDup)
dim(Final_Consolidated_Lexicon_DeDup)

## ========== Install Packages ========== #
install.packages("plyr")
install.packages("data.table")
library("dplyr")
library("plyr")
library("data.table")
library("stringr")
library("plyr")
library("lattice")
## ========== Application of Scores ========== #
head(clean_tweets_tdm)
Clean_SentiWords_DeDuped <- Final_Consolidated_Lexicon_DeDup
head(Clean_SentiWords_DeDuped)
Clean_SentiWords_DeDuped$Word <- as.character(Clean_SentiWords_DeDuped$Word) # make a word column for matching
clean_tweets_tdm$Word <- as.character(clean_tweets_tdm$stemmed)# make a word column for matching
head(clean_tweets_tdm)

tweet_scores <- left_join(Clean_SentiWords_DeDuped, clean_tweets_tdm)
tweet_scores <- tweet_scores[complete.cases(tweet_scores), ] 
head(tweet_scores)

# Create a single Column of Scores (Update the scores for pos/neg)
tweet_scores$score <- ifelse(tweet_scores$New_Pos == 1, 1, -1) # Correctly assign positive and negative score
head(tweet_scores)
dim(tweet_scores)
# Updated the frequencies based on the duplicated words to make sure they are accounted for
score_updates <- aggregate(tweet_scores$freq ~ tweet_scores$stemmed, data=tweet_scores, FUN=sum) 
colnames(score_updates) <- c("stemmed", "frequency")
tweet_scores_no_dupe <- subset(tweet_scores, tweet_scores$stemdup == FALSE) # Duplicated Stems Removed

tweet_scores_final <- left_join(tweet_scores_no_dupe, score_updates) # Apply updated frequncy for each term
head(tweet_scores_final)
write.csv(tweet_scores, file = "Tweet_Scores_Duplicates.csv")

# Re-Calculate Weights and Frequencies based on Aggregated Updates
tweet_scores_final$New_Weight <- tweet_scores_final$frequency/sum(tweet_scores_final$frequency)
head(tweet_scores_final)
write.csv(tweet_scores_final, file = "Tweet_Scores_No_Duplicates.csv")

# Create a word cloud
wordcloud:wordcloud(tweet_scores_final$stemmed, 
                    tweet_scores_final$frequency,
                    max.words = 150,
                    min.freq=3, 
                    scale=c(4,.5),
                    random.order=FALSE,
                    colors=brewer.pal(6, "Spectral"))

tweet_scores_final$sentiment <- ifelse(tweet_scores_final$score > 0, "positive", "negative")

word_analysis <- tweet_scores_final[,c(9,12:14)]
word_analysis <- word_analysis[,c(1,3,4,2)]
# Create a diagram of the most postive and negative words
word_analysis %>%
  group_by(sentiment) %>%
  top_n(n = 50, wt = frequency) %>%
  ungroup() %>%
  mutate(Word = reorder(stemmed, frequency)) %>%
  ggplot(aes(Word, frequency, fill = sentiment)) +
  geom_col(show.legend = FALSE) +
  facet_wrap(~sentiment, scales = "free_y") +
  labs(y = "Contribution to sentiment",
       x = NULL) +
  coord_flip()

# Match thes scores to the tweets for each day
sentence <- strsplit(x$text, "\\s+") # Split into individual words
positive_word_list <- subset(Clean_SentiWords_DeDuped, Clean_SentiWords_DeDuped$New_Pos==1)
positive_word_list <- positive_word_list[c(1:2)] # Positive Word List
head(positive_word_list)
neg_word_list <- subset(Clean_SentiWords_DeDuped, Clean_SentiWords_DeDuped$New_Neg==-1)
neg_word_list <- neg_word_list[c(1,3)] # Neg Word List
head(neg_word_list)

# Reference: https://stackoverflow.com/questions/28059939/retrieving-sentence-score-based-on-values-of-words-in-a-dictionary
sentencetest <- x$text
df <- data_frame(text = sentencetest)
dict <- data_frame(word = tweet_scores_final$stemmed,
                   score = tweet_scores_final$score) # Dataframe with Stemmed Words and Scores
head(dict)
scores <- df %>% mutate(score = sapply(strsplit(text, ' '), function(x) with(dict, sum(score[word %in% x]))))
head(scores)

# Final Dataset
write.csv(head(scores), file = "Example_Sentence_Score.csv")
write.csv(scores, file = "Tweets_Scored.csv") 

Final_Data_For_Modelling <- cbind(x,scores)
Final_Data_For_Modelling <- Final_Data_For_Modelling[c(4, 12, 13)] # Reduce data to only include important columns
head(Final_Data_For_Modelling)
Final_Data_For_Modelling$Classification_Score <- NA
Final_Data_For_Modelling$Classification_Score[Final_Data_For_Modelling$score > 0] <- 1 # Classify Data into Positive and Negative
Final_Data_For_Modelling$Classification_Score[Final_Data_For_Modelling$score < 0] <- -1
Final_Data_For_Modelling$Classification_Score[Final_Data_For_Modelling$score == 0] <- 0
head(Final_Data_For_Modelling)
write.csv(head(Final_Data_For_Modelling), file = "Sample_Final_Scored_Dataset.csv")
write.csv(Final_Data_For_Modelling, file = "Final_Scored_Dataset.csv")

# Ambivalent Tweets 
## Positive Scores
ambpos <- positive_word_list
ambstem <- subset(tweet_scores_final, score == 1)
colnames(ambpos) <- c("Word", "score")
AmbDict <- data_frame(word = ambstem$stemmed,
                      score = ambstem$score)
AmbPosScores <- df %>% mutate(score = sapply(strsplit(text, ' '), function(x) with(AmbDict, sum(score[word %in% x]))))
colnames(AmbPosScores) <- c("text","pos.score")
head(AmbPosScores)

## Neg Scores
ambneg <- neg_word_list
ambnegstem <- subset(tweet_scores_final, score == -1)
colnames(ambneg) <- c("Word", "score")
AmbDict <- data_frame(word = ambnegstem$stemmed,
                      score = ambnegstem$score)
AmbNegScores <- df %>% mutate(score = sapply(strsplit(text, ' '), function(x) with(AmbDict, sum(score[word %in% x]))))
colnames(AmbNegScores) <- c("text","neg.score")
head(AmbNegScores)

AmbScore <- cbind(Final_Data_For_Modelling, AmbPosScores$pos.score, AmbNegScores$neg.score)
colnames(AmbScore) <- c("created", "text", "score","Classification_Score", "pos.score", "neg.score")
head(AmbScore)
class(AmbScore)
# Percentage of Ambivalent Cases 
sum(AmbScore$Classification_Score == 0 & AmbScore$pos.score > 0 & AmbScore$neg.score < 0) 
sum(AmbScore$Classification_Score == 0 & AmbScore$pos.score > 0 & AmbScore$neg.score < 0)/nrow(AmbScore) # 7.49%

ambexamples <- subset(AmbScore, AmbScore$Classification_Score == 0 & AmbScore$pos.score > 0 & AmbScore$neg.score < 0)
write.csv(ambexamples, file = "Ambivalant_Case.csv")

rm(ambexamples, AmbScore, AmbNegScores, AmbDict, ambnegstem, ambneg, AmbPosScores, ambstem, ambpos)  # Clear down Global Environment
rm(Clean_SentiWords_DeDuped, neg_word_list, positive_word_list, score_updates, scores, tweet_scores_final, tweet_scores_no_dupe, tweet_scores, word_analysis,
   sentencetest, sentence, df)
# Spelling for testscore2 and re match
# Reference: http://www.sumsar.net/blog/2014/12/peter-norvigs-spell-checker-in-two-lines-of-r/
#testscore2 <- testscore[-complete.cases(testscore), ] # do spg on this and rerun the process
#testscore2 <- aggregate(testscore2$freq ~ testscore2$stemmed, data=testscore2, FUN=sum) 

#sorted_words <- names(sort(table(strsplit(tolower(paste(readLines("http://www.norvig.com/big.txt"), 
#                                                        collapse = " ")), 
#                                          "[^a-z]+")), decreasing = TRUE))
#correct_spg <- function(word) { 
#  c(sorted_words[adist(word, sorted_words) <= min(adist(word, sorted_words), 2)], word)[1] 
#}
#testscore2$newword <- correct_spg(testscore2$stemmed)

# ========== Modellling ==========#
install.packages("magrittr")
install.packages("dplyr")
install.packages("reshape")
install.packages("car")
install.packages("ggplot2")
install.packages("mgcv")
install.packages("boot")
install.packages("caret")
install.packages("mboost")
install.packages("ggthemes")
install.packages("visreg")
install.packages("syuzhet")
install.packages("gridExtra")
install.packages("fpp2")
install.packages("itsadug")
library("magrittr")
library("dplyr")
library("reshape")
require("gam")
library("car")
library("ggplot2")
require("plotly")
require("mgcv")
require("boot")
require("caret")
require("mboost")
require("ggthemes")
require("visreg")
require("mutate")
require("syuzhet")
require("gridExtra")
library("tidyverse")      
library("lubridate")      
library("fpp2")           
library("zoo")      
require("itsadug")

# Twitter Data
## Re-remove duplicated tweets and those classification scores of 0
Final_Data_For_Modelling$dupe <- duplicated(Final_Data_For_Modelling$text) # 21,777 Duplicates
sum(Final_Data_For_Modelling$dupe == TRUE)
Final_Data_For_Modelling_no_dupes <- subset(Final_Data_For_Modelling, Final_Data_For_Modelling$dupe == FALSE)
Final_Data_For_Modelling_no_dupes_zero <- subset(Final_Data_For_Modelling_no_dupes, 
                                                 Final_Data_For_Modelling_no_dupes$Classification_Score != 0)
Classification_Model_Data <- Final_Data_For_Modelling_no_dupes_zero
head(Classification_Model_Data)
dim(Classification_Model_Data)

write.csv(Classification_Model_Data, file = "Classification_Model_Data.csv")

table(Classification_Model_Data$created, Classification_Model_Data$Classification_Score)
write.csv(table(Classification_Model_Data$created, Classification_Model_Data$Classification_Score), "DailySummary.csv")

# Bitcoin Data
head(Bitcoin_Data)
Bitcoin_Data$Price_Update <- as.numeric(gsub('[$,]', '', Bitcoin_Data$Price)) # Convert Price from Character to Numeric
Bitcoin_Data$Price_Update <- round(Bitcoin_Data$Price_Update, 2)
Bitcoin_Data$prc_change <- (Bitcoin_Data$Price_Update/lag(Bitcoin_Data$Price_Update) - 1)*100

# Add Sentiment data to the BTC Data
BTC_Dates <- as.POSIXct(c("2018-03-30","2018-03-31","2018-04-01","2018-04-02","2018-04-03"), "UTC")
BTC_Dates <- as.vector(lapply(BTC_Dates, rep, 24))
BTC_Dates <- split(BTC_Dates, 6)
BTC_Dates <- melt(BTC_Dates, BTC_Dates = c(1:6))
Bitcoin_Data$created <- BTC_Dates$value
head(Bitcoin_Data)

# Join with Classification_Model_Data
head(Classification_Model_Data)

daily_avg_sentiment <- ddply(Classification_Model_Data, "created", summarize,avg = mean(Classification_Score))
daily_sd_sentiment <-  ddply(Classification_Model_Data, "created", summarize, variance = var(Classification_Score))

Daily_Sentiment <- cbind(daily_avg_sentiment, daily_sd_sentiment$variance)
Daily_Sentiment <- Daily_Sentiment[-c(1,7),]
colnames(Daily_Sentiment) <- c("created", "Avg_Sentiment", "Var_Sentiment")
head(Daily_Sentiment)

Bitcoin_Data$created <- as.character(as.POSIXct(Bitcoin_Data$created)) # Change to character as issue matching as Date
Daily_Sentiment$created <- as.character(as.POSIXct(Daily_Sentiment$created))
Bitcoin_Data <- left_join(Bitcoin_Data, Daily_Sentiment)
write.csv(Bitcoin_Data, file = "Bitcoin_Data.csv")

# Update the Date Column
x <- as.POSIXct("2018/03/30 00:01:00", tz="UTC")
x_update <- format(seq(x, by="hour", length.out=120), "%Y-%m-%d %H:%M:%S %Z")

Bitcoin_Data$New_Date <- x_update
Bitcoin_Data$New_Date <- as.POSIXct(Bitcoin_Data$New_Date)

## Moving Average
ma <- Bitcoin_Data %>%
  select(New_Date, srate = Avg_Sentiment) %>%
  mutate(srate_ma01 = rollmean(srate, k = 24, fill = NA, align = "right"))

ma_var <- Bitcoin_Data %>%
  select(New_Date, srate = Var_Sentiment) %>%
  mutate(srate_ma02 = rollmean(srate, k = 24, fill = NA, align = "right"))

Bitcoin_Data$Final_Senti <- ma$srate_ma01
for(i in 1:24){
  Bitcoin_Data$Final_Senti[i] <- Bitcoin_Data$Avg_Sentiment
}

Bitcoin_Data$Final_Senti_Var <- ma_var$srate_ma02
for(i in 1:24){
  Bitcoin_Data$Final_Senti_Var[i] <- Bitcoin_Data$Var_Sentiment
}


# ================ Model Using a GAM ================ #
## Model with two cont. predictors - Full Dataset
range(Bitcoin_Data$prc_change[2:120])

# Account for only taking sentiment and not other factors 
# do not ignore the time component but general trend mitigated by some sentiment
# 2 dimensional smooth and try to extract a surface

# look at fitted curves and see form GAM if any evidence of relationship - 
# expect positive repationship between x and y

# Interaction between the mean and the variance - contour plot of this
Bitcoin_Data$prc_change_bin <- ifelse(Bitcoin_Data$prc_change < 0, 0, 1)
class(Bitcoin_Data$prc_change_bin)
Bitcoin_Data$prc_change_bin <- as.factor(Bitcoin_Data$prc_change_bin)

# ================ Model One - GAM ================ #
# https://m-clark.github.io/docs/GAM.html#gam22
GAM_Bitcoin_Data <- Bitcoin_Data[2:120,]
GAM_Model <- gam(prc_change_bin ~ s(Final_Senti) + s(Final_Senti_Var), 
                 data = GAM_Bitcoin_Data, 
                 family = binomial) # Family Binominal with a Logit Link due to Predictor Variable Class
summary(GAM_Model)
GAM_Model
RSS_GAM_Model <- sum(residuals(GAM_Model)**2)
RSS_GAM_Model

summary(GAM_Model)$s.table

par(mfrow=c(1,2))
plot(GAM_Model,se = TRUE, shade = TRUE, main = "Graphical Representation of Smooth Terms")

vis.gam(GAM_Model, main = "2D Contour Plot on the Link Scale", plot.type = "contour", color = "topo", 
        contour.col = "black", lwd = 2)
vis.gam(GAM_Model, n.grid = 50, theta = 35, phi = 32, zlab = "", ticktype = "detailed", color = "topo",
        main = "Contour plot view of the \nGAM model prediction on the Link Scale")
vis.gam(GAM_Model, type='response', plot.type='contour', main = "2D Contour Plot on the Response Scale")
vis.gam(GAM_Model, type='response', plot.type='persp', phi=30, theta=30, n.grid=500, border=NA,
        main = "Contour plot view of the \nGAM model prediction on the Link Scale")

#visreg2d(GAM_Model, xvar='Final_Senti', yvar='Final_Senti_Var', scale='response')

fits_GAM_Model = predict(GAM_Model, newdata=Bitcoin_Data, type='response', se=T)
predicts_GAM_Model = data.frame(Bitcoin_Data, fits_GAM_Model) %>% 
  mutate(lower = fit - 1.96*se.fit,
         upper = fit + 1.96*se.fit) # This obtains the predicted probabilities 

# Turn predicited probabilities to binary using a threshold of 0.5
predicts_GAM_Model$pred_bin <- ifelse(predicts_GAM_Model$fit < 0.5, 0, 1)
predicts_GAM_Model$pred_bin <- as.factor(predicts_GAM_Model$pred_bin)

# ================ Confusion Matrix
GAM_Model_Prediction_Data <- c(predicts_GAM_Model$prc_change_bin, predicts_GAM_Model$pred_bin)
Results <- confusionMatrix(predicts_GAM_Model$pred_bin, predicts_GAM_Model$prc_change_bin)
Results
write.csv(Results$byClass, "Results_One_GAM.csv")
write.csv(Results$table, "Conf_Matr_GAM.csv")

# ================ Bitcoin Graphs ================ #
Data <-Bitcoin_Data
Data %>% 
  ggplot(mapping = aes(x = created, y = Final_Senti, fill = Final_Senti)) +
  geom_bar(alpha = 0.8, stat = "identity") +
  labs(y = "Sentiment Score", x = "Day", fill = "Final_Senti") +
  ggtitle("Average Sentiment Score by Day") +
  coord_flip()

Data %>% 
  ggplot(mapping = aes(x = created, y = Final_Senti_Var, fill = Final_Senti_Var)) +
  geom_bar(alpha = 0.8, stat = "identity") +
  labs(y = "Sentiment Score", x = "Day", fill = "Final_Senti_Var") +
  ggtitle("Variance Sentiment Score by Day") +
  coord_flip()

plot_1 <- Bitcoin_Data %>% 
  ggplot(aes(x=New_Date, y=Price_Update)) +
  geom_line() +
  geom_point() + labs(x="Time of Day", y="Change in Market\n Price ($)", title = "Bitcoin Asset Price", 
                      subtitle = "Between Mar 30, 2018 00:01:00 UTC - Apr 3, 2018 23:01:00 UTC")

plot_2 <- Bitcoin_Data %>% 
  ggplot(aes(x=New_Date, y=prc_change)) +
  geom_line() +
  geom_point() + labs(x="Time of Day", y="Percentage Change\n in Price (%)", 
                      title = "Bitcoin Asset Price Percentage Change", 
                      subtitle = "Between Mar 30, 2018 00:01:00 UTC - Apr 3, 2018 23:01:00 UTC")

plot_3 <- Bitcoin_Data %>% 
  ggplot(aes(x=New_Date, y=Final_Senti, group=1)) +
  geom_line() +
  geom_point() + labs(x="Time of Day", y="Average Sentiment\n Fluctuations", 
                      title = "Twitter Sentiment Average Towards Bitcoin Market Price", 
                      subtitle = "Between Mar 30, 2018 00:01:00 UTC - Apr 3, 2018 23:01:00 UTC")

plot_4 <- Bitcoin_Data %>% 
  ggplot(aes(x=New_Date, y=Final_Senti_Var, group=1)) +
  geom_line() +
  geom_point() + labs(x="Time of Day", y="Variance Sentiment\n Fluctuations", 
                      title = "Twitter Sentiment Variance Towards Bitcoin Market Price", 
                      subtitle = "Between Mar 30, 2018 00:01:00 UTC - Apr 3, 2018 23:01:00 UTC")

grid.arrange(plot_1, plot_2, plot_3, plot_4, nrow = 4)

# ================ Model Two - GAM with interaction term ================ #
# https://m-clark.github.io/docs/GAM.html#gam22
GAM_Model_Int <- gam(prc_change_bin ~ s(Final_Senti) + s(Final_Senti_Var) + Final_Senti*Final_Senti_Var,
                 data = GAM_Bitcoin_Data, 
                 family = binomial) # Family Binominal with a Logit Link due to Predictor Variable Class
summary(GAM_Model_Int)
GAM_Model_Int
RSS_GAM_Model_Int <- sum(residuals(GAM_Model_Int)**2)
RSS_GAM_Model_Int

summary(GAM_Model_Int)$s.table

fits_GAM_Model_int = predict(GAM_Model_Int, newdata=fits_GAM_Model_int, type='response', se=T)
predicts_GAM_Model_Int = data.frame(Bitcoin_Data, fits_GAM_Model) %>% 
  mutate(lower = fit - 1.96*se.fit,
         upper = fit + 1.96*se.fit) # This obtains the predicted probabilities 

# Turn predicited probabilities to binary using a threshold of 0.5
predicts_GAM_Model_Int$pred_bin <- ifelse(predicts_GAM_Model_Int$fit < 0.5, 0, 1)
predicts_GAM_Model_Int$pred_bin <- as.factor(predicts_GAM_Model_Int$pred_bin)

# ================ Confusion Matrix
GAM_Model_Int_Prediction_Data <- c(predicts_GAM_Model_Int$prc_change_bin, predicts_GAM_Model_Int$pred_bin)
Results <- confusionMatrix(predicts_GAM_Model_Int$pred_bin, predicts_GAM_Model_Int$prc_change_bin)
Results

# ================== Exploratory Data Analysis of Twitter Text ================== #
# Confirm Number of Tweets equals 1000 for each extraction
## Load Libraries
install.packages("RColorBrewer")
install.packages("ggplot2")
install.packages("tm")
install.packages("slam")
install.packages("RWeka")
install.packages("ggraph")
install.packages("ggforce")
install.packages("igraph")
install.packages("tidytext")
install.packages("broom")
library(devtools)
install_github("dgrtwo/widyr")
#install.packages("pattern.nlp")
library(RColorBrewer)
library(ggplot2)
require(tm)
require(RWeka)
library(slam)
library(ggraph)
library(ggforce)
library(igraph)
library(tidytext)
library(broom)
#require("pattern.nlp")

freq.terms <- findFreqTerms(clean_tweets_tdm, lowfreq = 100)

# Create a word cloud
wordcloud:wordcloud(clean_tweets_tdm$term, 
                    clean_tweets_tdm$ln_freq,
                    max.words = 200,
                    min.freq=3, 
                    scale=c(4,.5),
                    random.order=FALSE,
                    colors=brewer.pal(6, "Spectral"))

# Analysis of Most Frequent Words
term.freq <- rowSums(as.matrix(tdm))
term.freq <- subset(term.freq, term.freq >= 1500)
df2 <- data.frame(term = names(term.freq), freq = term.freq)
ggplot(df2, aes(x=term, y=freq)) + geom_bar(stat="identity") +xlab("Terms") + ylab("Count") + coord_flip() +theme(axis.text=element_text(size=7))

plotData <- Final_Data_For_Modelling[c(Final_Data_For_Modelling$text, Final_Data_For_Modelling$score)]
# note retweet could was originally "favoriteCount"
xLabel <- paste("Sentiment Score.  Mean sentiment: ",
                round(mean(Final_Data_For_Modelling$score), 2), sep = "")
yLabel <- paste("Number of Tweets (", nrow(Final_Data_For_Modelling),")", sep = "")
graphTitle <- paste("Twitter Sentiment Analysis of #Bitcoin")

qplot(factor(Final_Data_For_Modelling$score), data=Final_Data_For_Modelling,
      geom="bar",
      fill=factor(Final_Data_For_Modelling$score),
      xlab = xLabel,
      ylab = yLabel,
      main = graphTitle) +
  theme(legend.position="none")

## Not working quite yet
# Reference: https://www.r-bloggers.com/twitter-sentiment-analysis-with-r/
#total evaluation: positive / negative / neutral
stat <- Final_Data_For_Modelling
stat$created <- Final_Data_For_Modelling$created
stat$created <- as.Date(stat$created)
stat <- mutate(stat, tweet=ifelse(stat$score > 0, 'positive', ifelse(stat$score < 0, 'negative', 'neutral')))
by.tweet <- group_by(stat, stat$tweet, stat$created)
by.tweet <- summarise(by.tweet, number=n())
#write.csv(by.tweet, file=paste(searchterm, '_opin.csv'), row.names=TRUE)

#create chart
ggplot(by.tweet, aes(created, nrow(by.tweet))) + geom_line(aes(group=tweet, color=tweet), size=2) +
  geom_point(aes(group=tweet, color=tweet), size=4) +
  theme(text = element_text(size=18), axis.text.x = element_text(angle=90, vjust=1)) 
  #stat_summary(fun.y = 'sum', fun.ymin='sum', fun.ymax='sum', colour = 'yellow', size=2, geom = 'line') +
  #ggtitle(searchterm)

# Positive Words
# Reference: https://rpubs.com/abNY2015/90345
#ggplot(dpwords,aes(pwords,Freq))+geom_bar(stat="identity",fill="lightblue")+theme_bw()+
#  geom_text(aes(pwords,Freq,label=Freq),size=4)+
#  labs(x="Major Positive Words", y="Frequency of Occurence",title=paste("Major Positive Words and Occurence in \n '",findfd,"' twitter feeds, n =",number))+
#  geom_text(aes(1,5,label=paste("Total Positive Words :",pcount)),size=4,hjust=0)+theme(axis.text.x=element_text(angle=45))


# Scoring - Tokenizing and nGram Generation
# Tokenizing - breaking up a sequence of strings into pieces such as words, keywords, phrases, symbols and other elements called tokens. ... In the process of tokenization, some characters like punctuation marks are discarded. 
# The n-grams typically are collected from a text or speech corpus
# Reference - https://www.tidytextmining.com/twitter.html

tweets_bigrams <- scores %>%
  unnest_tokens(bigram, text, token = "ngrams", n = 2)

tweets_bigrams %>%
  count(bigram, sort = TRUE)

bigrams_separated <- tweets_bigrams %>%
  separate(bigram, c("word1", "word2"), sep = " ")

bigrams_filtered <- bigrams_separated %>%
  filter(!word1 %in% stop_words$word) %>%
  filter(!word2 %in% stop_words$word)

# new bigram counts:
bigram_counts <- bigrams_filtered %>% 
  count(word1, word2, sort = TRUE)

bigram_counts

bigrams_united <- bigrams_filtered %>%
  unite(bigram, word1, word2, sep = " ")

bigrams_united

bigrams_separated %>%
  filter(word1 == "not") %>%
  count(word1, word2, sort = TRUE)

library(igraph)

# original counts
bigram_counts

bigram_graph <- bigram_counts %>%
  filter(n > 250) %>%
  graph_from_data_frame()
bigram_graph

library(ggraph)
set.seed(2017)

ggraph(bigram_graph, layout = "fr") +
  geom_edge_link() +
  geom_node_point() +
  geom_node_text(aes(label = name), vjust = 1, hjust = 1)

set.seed(2016)

a <- grid::arrow(type = "closed", length = unit(.15, "inches"))

ggraph(bigram_graph, layout = "fr") +
  geom_edge_link(aes(edge_alpha = n), show.legend = FALSE,
                 arrow = a, end_cap = circle(.07, 'inches')) +
  geom_node_point(color = "lightblue", size = 5) +
  geom_node_text(aes(label = name), vjust = 1, hjust = 1) +
  theme_void()
