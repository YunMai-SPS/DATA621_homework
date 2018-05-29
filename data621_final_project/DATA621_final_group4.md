DATA621 Final Project\_Group4
================
Yun Mai, Gurpreet Singh, Chirag Vithalani
May 2, 2018

Purpose
=======

The motion picture industry is growing at a rapid growth rate, likely due to the acceleration of online and mobile distribution, lower admission prices, and government policy initiatives. This industry is also rich in data, thus making it extremely exciting for statisticians. The movie industry, which used to rely on traditional conventional wisdom and simple rules of thumb to predict box office outcomes, is slowly seeking new "analytical" approaches.

More and more analytical models will play a greater role in the motion picture industry by contributing towards superior marketing strategies that better predict the overall success of each movie.

Introduction
============

Data preparation
================

``` r
suppressMessages(suppressWarnings(library(data.table)))
suppressMessages(suppressWarnings(library(knitr)))
suppressMessages(suppressWarnings(library(stringr)))
suppressMessages(suppressWarnings(library(pastecs)))
suppressMessages(suppressWarnings(library(psych)))
suppressMessages(suppressWarnings(library(Hmisc)))
suppressMessages(suppressWarnings(library(reshape)))
suppressMessages(suppressWarnings(library(corrplot)))
suppressMessages(suppressMessages(library(MASS)))
suppressMessages(suppressWarnings(library(dplyr)))
suppressMessages(suppressWarnings(library(ggplot2)))
suppressMessages(suppressWarnings(library(ggthemes)))
suppressMessages(suppressWarnings(library(mice)))
suppressMessages(suppressWarnings(library(VIM)))
suppressMessages(suppressWarnings(library(caret)))

suppressMessages(suppressWarnings(library(Amelia)))
suppressMessages(suppressWarnings(library(car)))
suppressMessages(suppressWarnings(library(Amelia)))


suppressMessages(suppressWarnings(library(readr)))

#options(stringsAsFactors = FALSE)
#movie <- readr::read_csv('https://raw.githubusercontent.com/chirag-vithlani/Business_Analytics_and_Data_Mining_DATA_621/master/Movie_data_analysis_project/data/movie_metadata.csv', locale = locale(encoding = "UTF-8"))

movie <- readr::read_csv('https://raw.githubusercontent.com/YunMai-SPS/DATA621_homework/master/data621_final_project/movie_add_missing.csv', locale = locale(encoding = "UTF-8"))
```

    ## Parsed with column specification:
    ## cols(
    ##   .default = col_integer(),
    ##   movie_title = col_character(),
    ##   imdb_score = col_double(),
    ##   content_rating = col_character(),
    ##   color = col_character(),
    ##   aspect_ratio = col_double(),
    ##   language = col_character(),
    ##   country = col_character(),
    ##   director_name = col_character(),
    ##   actor_1_name = col_character(),
    ##   actor_2_name = col_character(),
    ##   actor_3_name = col_character(),
    ##   genres = col_character(),
    ##   plot_keywords = col_character(),
    ##   movie_imdb_link = col_character()
    ## )

    ## See spec(...) for full column specifications.

    ## Warning in rbind(names(probs), probs_f): number of columns of result is not
    ## a multiple of vector length (arg 1)

    ## Warning: 3 parsing failures.
    ## row # A tibble: 3 x 5 col     row col   expected               actual     file                       expected   <int> <chr> <chr>                  <chr>      <chr>                      actual 1  1457 gross an integer             2710796031 'https://raw.githubuserco~ file 2  1727 gross no trailing characters s          'https://raw.githubuserco~ row 3  2128 gross no trailing characters s          'https://raw.githubuserco~

``` r
movie <- as.data.frame(movie)
movie$Index <- seq(1,nrow(movie))
movie <-movie[,c(length(movie),1:(length(movie)-1))]
#readr::write_csv(movie, "E:/YM_work/CUNY_DAMS/CUNY_621/DATA621_finalproject/movie_original.csv")
#temp <- movie <- readr::read_csv('E:/YM_work/CUNY_DAMS/CUNY_621/DATA621_finalproject/movie_original.csv', locale = locale(encoding = "UTF-8"))
```

Data preparation for linear regression
--------------------------------------

Clean up

``` r
# correct some incorrect aspect ratio 
movie$aspect_ratio[which(movie$aspect_ratio == 16)] <- 1.78
movie$aspect_ratio[which(movie$aspect_ratio == 4)] <- 1.33

# fill empty cell with NA if any
for(i in 1:length(movie)){
 movie[,i][which(movie[,i]=="")]<-NA
}

#replace NAs as string 'NA' 
for(i in 1:length(movie)){
  if (class(movie[,i]) == 'character'){
    movie[,i][which(is.na(movie[,i]))] <- 'NA'
  }
}

#replace 'Not Rated' in content_rating with 'Unrated' as they are the same
movie$'content_rating' <- str_replace_all(movie$'content_rating','Not Rated','Unrated')
 
#unique(movie$'content_rating')
```

### check the missing data in movie

``` r
pMiss <- function(x){(sum(is.null(x))+sum(is.na(x)))/length(x)[1]*100}
pMiss_count <- function(x){sum(is.null(x))+sum(is.na(x))}
missfeature_percent <- as.data.frame(apply(movie[,-1],2,pMiss))
missfeature_count <- as.data.frame(apply(movie[,-1],2,pMiss_count))
feature_mode <- unlist(t(lapply(movie[,-1],class)))
miss_feature <- cbind(missfeature_count,feature_mode,missfeature_percent)
miss_feature <- miss_feature[,c(2,1,3)]
colnames(miss_feature) <- c('feature_mode','missing value','missing value(%)')
miss_feature <- miss_feature[order(miss_feature$'missing value', decreasing =TRUE ),]
miss_feature 
```

    ##                           feature_mode missing value missing value(%)
    ## gross                          integer           788      15.65044687
    ## budget                         integer           456       9.05660377
    ## aspect_ratio                   numeric           309       6.13704071
    ## director_facebook_likes        integer           101       2.00595829
    ## num_critic_for_reviews         integer            49       0.97318769
    ## actor_3_facebook_likes         integer            32       0.63555114
    ## actor_2_facebook_likes         integer            22       0.43694141
    ## num_user_for_reviews           integer            20       0.39721946
    ## actor_1_facebook_likes         integer            16       0.31777557
    ## cast_total_facebook_likes      integer            10       0.19860973
    ## movie_facebook_likes           integer             9       0.17874876
    ## duration                       integer             1       0.01986097
    ## movie_title                  character             0       0.00000000
    ## imdb_score                     numeric             0       0.00000000
    ## num_voted_users                integer             0       0.00000000
    ## facenumber_in_poster           integer             0       0.00000000
    ## content_rating               character             0       0.00000000
    ## color                        character             0       0.00000000
    ## title_year                     integer             0       0.00000000
    ## language                     character             0       0.00000000
    ## country                      character             0       0.00000000
    ## director_name                character             0       0.00000000
    ## actor_1_name                 character             0       0.00000000
    ## actor_2_name                 character             0       0.00000000
    ## actor_3_name                 character             0       0.00000000
    ## genres                       character             0       0.00000000
    ## plot_keywords                character             0       0.00000000
    ## movie_imdb_link              character             0       0.00000000

``` r
miss_record <- data.frame(cbind(movie[,1],apply(movie,1,pMiss)))
colnames(miss_record )<-c('Index','missing value(%)')
nrow(miss_record[miss_record[,2]!=0,])
```

    ## [1] 1143

``` r
nrow(miss_record[miss_record[,2]>5,])
```

    ## [1] 405

``` r
cat('There are', nrow(miss_record[miss_record[,2]!=0,]),'records have missing values.')
```

    ## There are 1143 records have missing values.

``` r
cat("\n\n")
```

``` r
cat('There are', nrow(miss_record[miss_record[,2]>5,]),' records missed more than 5% data points.')
```

    ## There are 405  records missed more than 5% data points.

``` r
cat("\n\n")
```

``` r
missmap(movie, main = "Missing values vs observed")
```

![](DATA621_final_group4_v2_files/figure-markdown_github/unnamed-chunk-3-1.png)

separate the numeric and the categorical variables
--------------------------------------------------

``` r
# Because of too many levels, variables "movie_title", "director_name","actor_1_name", "actor_2_name", "actor_3_name", "plot_keywords" and "movie_imdb_link" will not be used in the model.

# separate the numeric and the categorical variables
num_var <- dplyr::select_if(movie[,-1], is.numeric)

ctg_var <- movie[,-which(names(movie) %in% names(num_var))][,-1]

#temp <- movie

#temp <- movie[,-which(names(movie) %in% c("director_name","actor_1_name", "actor_2_name", "actor_3_name", "plot_keywords" ,"movie_imdb_link"))]

#for( i in 3: length(temp)){
#  if(class(temp[,i])=='character'){
#    temp[,i]<-as.factor(temp[,i])
#  }
#}
```

Visualization of the dependent variable
---------------------------------------

``` r
ggplot(movie[!is.na(movie$gross),], aes(x = gross,fill = cut(gross, 100))) +
  geom_histogram(show.legend = FALSE,cex.lab = 1.8,cex.axis = 1.5) + 
  theme_few(base_size = 20)
```

    ## Warning: Ignoring unknown parameters: cex.lab, cex.axis

    ## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.

![](DATA621_final_group4_v2_files/figure-markdown_github/unnamed-chunk-5-1.png)

``` r
scatterplot(x=movie$title_year,y=movie$imdb_score)
```

![](DATA621_final_group4_v2_files/figure-markdown_github/unnamed-chunk-6-1.png)

``` r
scatterplot(x=movie$title_year,y=log(movie$gross),cex.lab = 1.8,cex.axis = 1.5) 
```

![](DATA621_final_group4_v2_files/figure-markdown_github/unnamed-chunk-6-2.png)

imputation
----------

``` r
#memory.limit()
#memory.limit(size=20000)

# impute 5 data sets for the missing values in the dataset
impData <- mice(num_var, m=5, printFlag=FALSE, meth="sample", maxit=50,seed=500) 

#compare the distributions of original and imputed data
densityplot(impData)
```

![](DATA621_final_group4_v2_files/figure-markdown_github/unnamed-chunk-7-1.png)

``` r
# get the completed dataset where the missing values have been replaced with the imputed values in the first of the five datasets
movie_im <- complete(impData,1)
movie_im <- cbind(movie$Index,movie_im)
colnames(movie_im)[1] <- 'Index'

#write.csv(movie_im,'impute.csv')

#delete the imputeded Y
not_na <- movie[!is.na(movie$gross),][,'Index']

num_mid <- movie_im[which(movie_im[,'Index'] %in% not_na),] 
```

remove records with NA in gross. completed dataset 'movie\_mid' is genreated.
-----------------------------------------------------------------------------

``` r
#remove NA from dicrector and actors columns
ctg_var <- cbind(movie$Index,ctg_var)
colnames(ctg_var)[1] <- 'Index'
ctg_mid <- ctg_var[which(ctg_var[,'Index'] %in% not_na),] 

movie_mid <- cbind(num_mid,ctg_mid[,-1])

#anyNA(movie_mid) #[1] FALSE
```

calculate the averger gross of each dicrector and actor made in the previous movies
-----------------------------------------------------------------------------------

``` r
# the compuation will be based on the whole dataset
# the records with missing gross values will be removed before calculation

#"director_name","actor_1_name", "actor_2_name", "actor_3_name",

####################################################
#convert director_name to dummay variable
####################################################
dummy_director <- as.data.frame(model.matrix(~director_name-1, data = movie_mid))
gross_director <- cbind(movie_mid[,'gross'],dummy_director)
colnames(gross_director)[1] <- c('gross')
director_sum <- as.matrix(unlist(lapply(dummy_director,sum)))
gross_director <- as.matrix(t(as.matrix(gross_director$gross)) %*% as.matrix(gross_director[,-1]))
director_ave_gross <- gross_director/t(director_sum)
hist(director_ave_gross)
```

![](DATA621_final_group4_v2_files/figure-markdown_github/unnamed-chunk-9-1.png)

``` r
#Aaron Schneider


#director_sum <- lapply(dummy_director,sum)
#for (i in 1:length(dummy_director)){
#  dummy_director[,i] <- ifelse(dummy_director[i]==1,director_sum[[i]],0)
#}

####################################################
#convert actor_1_name to dummay variable
####################################################
dummy_actor_1 <- as.data.frame(model.matrix(~actor_1_name-1, data = movie_mid))
gross_actor_1 <- cbind(movie_mid[,'gross'],dummy_actor_1)
colnames(gross_actor_1)[1] <- c('gross')
actor_1_sum <- as.matrix(unlist(lapply(dummy_actor_1,sum)))
gross_actor_1 <- as.matrix(t(as.matrix(gross_actor_1$gross)) %*% as.matrix(gross_actor_1[,-1]))
actor_1_ave_gross <- gross_actor_1/t(actor_1_sum)
hist(actor_1_ave_gross)
```

![](DATA621_final_group4_v2_files/figure-markdown_github/unnamed-chunk-9-2.png)

``` r
####################################################
#convert actor_2_name to dummay variable
####################################################
dummy_actor_2 <- as.data.frame(model.matrix(~actor_2_name-1, data = movie_mid))
gross_actor_2 <- cbind(movie_mid[,'gross'],dummy_actor_2)
colnames(gross_actor_2)[1] <- c('gross')
actor_2_sum <- as.matrix(unlist(lapply(dummy_actor_2,sum)))
gross_actor_2 <- as.matrix(t(as.matrix(gross_actor_2$gross)) %*% as.matrix(gross_actor_2[,-1]))
actor_2_ave_gross <- gross_actor_2/t(actor_2_sum)
hist(actor_2_ave_gross)
```

![](DATA621_final_group4_v2_files/figure-markdown_github/unnamed-chunk-9-3.png)

``` r
####################################################
#convert actor_3_name to dummay variable
####################################################
dummy_actor_3 <- as.data.frame(model.matrix(~actor_3_name-1, data = movie_mid))
gross_actor_3 <- cbind(movie_mid[,'gross'],dummy_actor_3)
colnames(gross_actor_3)[1] <- c('gross')
actor_3_sum <- as.matrix(unlist(lapply(dummy_actor_3,sum)))
gross_actor_3 <- as.matrix(t(as.matrix(gross_actor_3$gross)) %*% as.matrix(gross_actor_3[,-1]))
actor_3_ave_gross <- gross_actor_3/t(actor_3_sum)
hist(actor_3_ave_gross)
```

![](DATA621_final_group4_v2_files/figure-markdown_github/unnamed-chunk-9-4.png)

``` r
par(mfrow=c(2,2))
hist(director_ave_gross)
hist(actor_1_ave_gross)
hist(actor_1_ave_gross)
hist(actor_1_ave_gross)
```

![](DATA621_final_group4_v2_files/figure-markdown_github/unnamed-chunk-9-5.png)

``` r
colnames(director_ave_gross) <- str_replace_all(colnames(director_ave_gross),"director_name","")
colnames(actor_1_ave_gross) <- str_replace_all(colnames(actor_1_ave_gross),"actor_1_name","")
colnames(actor_2_ave_gross) <- str_replace_all(colnames(actor_2_ave_gross),"actor_2_name","")
colnames(actor_3_ave_gross) <- str_replace_all(colnames(actor_3_ave_gross),"actor_3_name","")

director_ave_gross <- as.data.frame(t(director_ave_gross))
director_ave_gross$director <- rownames(director_ave_gross)

actor_1_ave_gross <- as.data.frame(t(actor_1_ave_gross))
actor_1_ave_gross$actor_1 <- rownames(actor_1_ave_gross)

actor_2_ave_gross <- as.data.frame(t(actor_2_ave_gross))
actor_2_ave_gross$actor_2 <- rownames(actor_2_ave_gross)

actor_3_ave_gross <- as.data.frame(t(actor_3_ave_gross))
actor_3_ave_gross$actor_3 <- rownames(actor_3_ave_gross)

movie_mid$director_ave_gross <- unlist(lapply(movie_mid$director_name, function(x) director_ave_gross$V1[match(x, director_ave_gross$director)]))

movie_mid$actor_1_ave_gross <- unlist(lapply(movie_mid$actor_1_name, function(x) actor_1_ave_gross$V1[match(x, actor_1_ave_gross$actor_1)]))

movie_mid$actor_2_ave_gross <- unlist( lapply(movie_mid$actor_2_name, function(x) actor_2_ave_gross$V1[match(x, actor_2_ave_gross$actor_2)]))

movie_mid$actor_3_ave_gross <- unlist( lapply(movie_mid$actor_3_name, function(x) actor_3_ave_gross$V1[match(x, actor_3_ave_gross$actor_3)]))

anyNA(movie_mid)
```

    ## [1] FALSE

calculate how many movies of each dicrector and actor has been involved previously
----------------------------------------------------------------------------------

``` r
rownames(director_sum) <- str_replace_all(rownames(director_sum),"director_name","")
rownames(actor_1_sum) <- str_replace_all(rownames(actor_1_sum),"actor_1_name","")
rownames(actor_2_sum) <- str_replace_all(rownames(actor_2_sum),"actor_2_name","")
rownames(actor_3_sum) <- str_replace_all(rownames(actor_3_sum),"actor_3_name","")

director_sum <- as.data.frame(director_sum)
director_sum$director <- rownames(director_sum)

actor_1_sum <- as.data.frame(actor_1_sum)
actor_1_sum$actor_1 <- rownames(actor_1_sum)

actor_2_sum <- as.data.frame(actor_2_sum)
actor_2_sum$actor_2 <- rownames(actor_2_sum)

actor_3_sum <- as.data.frame(actor_3_sum)
actor_3_sum$actor_3 <- rownames(actor_3_sum)

movie_mid$director_num <- unlist(lapply(movie_mid$director_name, function(x) director_sum$V1[match(x, director_sum$director)]))

movie_mid$actor_1_num <- unlist(lapply(movie_mid$actor_1_name, function(x) actor_1_sum$V1[match(x, actor_1_sum$actor_1)]))

movie_mid$actor_2_num <- unlist( lapply(movie_mid$actor_2_name, function(x) actor_2_sum$V1[match(x, actor_2_sum$actor_2)]))

movie_mid$actor_3_num <- unlist( lapply(movie_mid$actor_3_name, function(x) actor_3_sum$V1[match(x, actor_3_sum$actor_3)]))


anyNA(movie_mid)
```

    ## [1] FALSE

replace hyphen to underscore in 'genres' and 'content\_rating'
--------------------------------------------------------------

``` r
movie_mid$genres <- str_replace_all(movie_mid$genres,"-","_")
movie_mid$content_rating <- str_replace_all(movie_mid$content_rating,"-","_")
```

### convert some categorical variables to factor

``` r
movie_mid$content_rating <- as.factor(movie_mid$content_rating)
movie_mid$color <- as.factor(movie_mid$color)
movie_mid$language <- as.factor(movie_mid$language)
movie_mid$country <- as.factor(movie_mid$language)
movie_mid$genres <- as.factor(movie_mid$genres)
movie_mid$aspect_ratio <- as.factor(movie_mid$aspect_ratio)
```

explore the correlation
-----------------------

### early blind guess

``` r
#qplot(x=log(budget), y = log(gross), data=movie) +
  #geom_smooth(method = "glm", formula = y~x, family = gaussian(link = 'log'))

#qplot(x=log(imdb_score), y = log(gross), data=movie) +
  #geom_smooth(method = "glm", formula = y~x, family = gaussian(link = 'log'))

#qplot(x=log(director_facebook_likes), y = log(gross), data=movie) +
  #geom_smooth(method = "glm", formula = y~x, family = gaussian(link = 'log'))

#qplot(x=log(actor_1_facebook_likes), y = log(gross), data=movie) +
  #geom_smooth(method = "glm", formula = y~x, family = gaussian(link = 'log'))


ggplot(movie, aes(x = log(budget), y = log(gross))) + geom_point() +
    stat_smooth(method = 'lm', aes(colour = 'linear'), se = FALSE,size=1.5) +
    stat_smooth(method = 'lm', formula = y ~ poly(x,2), aes(colour = 'polynomial'), se= FALSE,size=1.5) +
    stat_smooth(method = 'nls', formula = y ~ a * log(x) +b, aes(colour = 'logarithmic'), se = FALSE, start = list(a=1,b=1),size=1.5) +
    stat_smooth(method = 'nls', formula = y ~ a*exp(b *x), aes(colour = 'Exponential'), se = FALSE, start = list(a=1,b=1),size=1.5)+ theme_grey(base_size = 16.5)
```

    ## Warning: Ignoring unknown parameters: start

    ## Warning: Ignoring unknown parameters: start

    ## Warning: Removed 1047 rows containing non-finite values (stat_smooth).

    ## Warning: Removed 1047 rows containing non-finite values (stat_smooth).

    ## Warning: Removed 1047 rows containing non-finite values (stat_smooth).

    ## Warning in (function (formula, data = parent.frame(), start, control = nls.control(), : No starting values specified for some parameters.
    ## Initializing 'a', 'b' to '1.'.
    ## Consider specifying 'start' or using a selfStart model

    ## Warning: Removed 1047 rows containing non-finite values (stat_smooth).

    ## Warning in (function (formula, data = parent.frame(), start, control = nls.control(), : No starting values specified for some parameters.
    ## Initializing 'a', 'b' to '1.'.
    ## Consider specifying 'start' or using a selfStart model

    ## Warning: Computation failed in `stat_smooth()`:
    ## number of iterations exceeded maximum of 50

    ## Warning: Removed 1047 rows containing missing values (geom_point).

![](DATA621_final_group4_v2_files/figure-markdown_github/unnamed-chunk-13-1.png)

``` r
ggplot(movie, aes(x = imdb_score, y = log(gross))) + geom_point() +
    stat_smooth(method = 'lm', aes(colour = 'linear'), se = FALSE,size=1.5) +
    stat_smooth(method = 'lm', formula = y ~ poly(x,2), aes(colour = 'polynomial'), se= FALSE,size=1.5) +
    stat_smooth(method = 'nls', formula = y ~ a * log(x) +b, aes(colour = 'logarithmic'), se = FALSE, start = list(a=1,b=1),size=1.5) +
    stat_smooth(method = 'nls', formula = y ~ a*exp(b *x), aes(colour = 'Exponential'), se = FALSE, start = list(a=1,b=1),size=1.5)+ theme_grey(base_size = 16.5)
```

    ## Warning: Ignoring unknown parameters: start

    ## Warning: Ignoring unknown parameters: start

    ## Warning: Removed 788 rows containing non-finite values (stat_smooth).

    ## Warning: Removed 788 rows containing non-finite values (stat_smooth).

    ## Warning: Removed 788 rows containing non-finite values (stat_smooth).

    ## Warning in (function (formula, data = parent.frame(), start, control = nls.control(), : No starting values specified for some parameters.
    ## Initializing 'a', 'b' to '1.'.
    ## Consider specifying 'start' or using a selfStart model

    ## Warning: Removed 788 rows containing non-finite values (stat_smooth).

    ## Warning in (function (formula, data = parent.frame(), start, control = nls.control(), : No starting values specified for some parameters.
    ## Initializing 'a', 'b' to '1.'.
    ## Consider specifying 'start' or using a selfStart model

    ## Warning: Removed 788 rows containing missing values (geom_point).

![](DATA621_final_group4_v2_files/figure-markdown_github/unnamed-chunk-13-2.png)

``` r
ggplot(movie, aes(x = log(director_facebook_likes), y = log(gross))) + geom_point() +
    stat_smooth(method = 'lm', aes(colour = 'linear'), se = FALSE,size=1.5) +
    stat_smooth(method = 'lm', formula = y ~ poly(x,2), aes(colour = 'polynomial'), se= FALSE,size=1.5) +
    stat_smooth(method = 'nls', formula = y ~ a * log(x) +b, aes(colour = 'logarithmic'), se = FALSE, start = list(a=1,b=1),size=1.5) +
    stat_smooth(method = 'nls', formula = y ~ a*exp(b *x), aes(colour = 'Exponential'), se = FALSE, start = list(a=1,b=1),size=1.5)+ theme_grey(base_size = 16.5)
```

    ## Warning: Ignoring unknown parameters: start

    ## Warning: Ignoring unknown parameters: start

    ## Warning: Removed 1532 rows containing non-finite values (stat_smooth).

    ## Warning: Removed 1532 rows containing non-finite values (stat_smooth).

    ## Warning: Removed 1532 rows containing non-finite values (stat_smooth).

    ## Warning in (function (formula, data = parent.frame(), start, control = nls.control(), : No starting values specified for some parameters.
    ## Initializing 'a', 'b' to '1.'.
    ## Consider specifying 'start' or using a selfStart model

    ## Warning: Removed 1532 rows containing non-finite values (stat_smooth).

    ## Warning in (function (formula, data = parent.frame(), start, control = nls.control(), : No starting values specified for some parameters.
    ## Initializing 'a', 'b' to '1.'.
    ## Consider specifying 'start' or using a selfStart model

    ## Warning: Removed 799 rows containing missing values (geom_point).

![](DATA621_final_group4_v2_files/figure-markdown_github/unnamed-chunk-13-3.png)

``` r
ggplot(movie, aes(x = log(actor_1_facebook_likes), y = log(gross))) + geom_point() +
    stat_smooth(method = 'lm', aes(colour = 'linear'), se = FALSE,size=1.5) +
    stat_smooth(method = 'lm', formula = y ~ poly(x,2), aes(colour = 'polynomial'), se= FALSE,size=1.5) +
    stat_smooth(method = 'nls', formula = y ~ a * log(x) +b, aes(colour = 'logarithmic'), se = FALSE, start = list(a=1,b=1),size=1.5) +
    stat_smooth(method = 'nls', formula = y ~ a*exp(b *x), aes(colour = 'Exponential'), se = FALSE, start = list(a=1,b=1),size=1.5)+ theme_grey(base_size = 16.5)
```

    ## Warning: Ignoring unknown parameters: start

    ## Warning: Ignoring unknown parameters: start

    ## Warning: Removed 811 rows containing non-finite values (stat_smooth).

    ## Warning: Removed 811 rows containing non-finite values (stat_smooth).

    ## Warning: Removed 811 rows containing non-finite values (stat_smooth).

    ## Warning in (function (formula, data = parent.frame(), start, control = nls.control(), : No starting values specified for some parameters.
    ## Initializing 'a', 'b' to '1.'.
    ## Consider specifying 'start' or using a selfStart model

    ## Warning: Removed 811 rows containing non-finite values (stat_smooth).

    ## Warning in (function (formula, data = parent.frame(), start, control = nls.control(), : No starting values specified for some parameters.
    ## Initializing 'a', 'b' to '1.'.
    ## Consider specifying 'start' or using a selfStart model

    ## Warning: Removed 800 rows containing missing values (geom_point).

![](DATA621_final_group4_v2_files/figure-markdown_github/unnamed-chunk-13-4.png)

### correlation matrix

``` r
pairs.panels( movie_mid[,c(2:6,9)], pch=21,lm=TRUE,cex.lab = 1.8,cex.axis = 1.5,cex.sub=1.3)
```

![](DATA621_final_group4_v2_files/figure-markdown_github/unnamed-chunk-14-1.png)

``` r
pairs.panels( movie_mid[,c(7:8,10:12,9)], pch=21,lm=TRUE,cex.lab = 1.8,cex.axis = 1.5,cex.sub=1.3)
```

![](DATA621_final_group4_v2_files/figure-markdown_github/unnamed-chunk-14-2.png)

``` r
pairs.panels( movie_mid[,c(13:17,9)], pch=21,lm=TRUE,cex.lab = 1.8,cex.axis = 1.5,cex.sub=1.3)
```

![](DATA621_final_group4_v2_files/figure-markdown_github/unnamed-chunk-14-3.png)

``` r
pairs.panels( movie_mid[,c(18:22,9)], pch=21,lm=TRUE,cex.lab = 1.8,cex.axis = 1.5,cex.sub=1.3)
```

![](DATA621_final_group4_v2_files/figure-markdown_github/unnamed-chunk-14-4.png)

``` r
pairs.panels( movie_mid[,c(23:27,9)], pch=21,lm=TRUE,cex.lab = 1.8,cex.axis = 1.5,cex.sub=1.3)
```

![](DATA621_final_group4_v2_files/figure-markdown_github/unnamed-chunk-14-5.png)

``` r
pairs.panels( movie_mid[,c(28:32,9)], pch=21,lm=TRUE,cex.lab = 1.8,cex.axis = 1.5,cex.sub=1.3)
```

![](DATA621_final_group4_v2_files/figure-markdown_github/unnamed-chunk-14-6.png)

``` r
pairs.panels( movie_mid[,c(33:37,9)], pch=21,lm=TRUE,cex.lab = 1.8,cex.axis = 1.5,cex.sub=1.3)
```

![](DATA621_final_group4_v2_files/figure-markdown_github/unnamed-chunk-14-7.png)

``` r
#pairs.panels( movie_mid[,-1], pch=21,lm=TRUE,cex.lab = 1.8,cex.axis = 1.5,cex.sub=1.3)
```

correlation between the possible predictors
-------------------------------------------

Num\_user\_for\_reviews, Num\_voted\_users, Director\_ave\_gross, Actor\_1\_ave\_gross,Actor\_2\_ave\_gross, Actor\_3\_ave\_gross has a moderate or strong positive correlation with gross.

``` r
ggplot(movie_mid, aes(x = log(num_user_for_reviews), y = log(gross))) + geom_point() +
    stat_smooth(method = 'lm', aes(colour = 'linear'), se = FALSE,size=1.5) +
    stat_smooth(method = 'lm', formula = y ~ poly(x,2), aes(colour = 'polynomial'), se= FALSE,size=1.5) +
    stat_smooth(method = 'nls', formula = y ~ a * log(x) +b, aes(colour = 'logarithmic'), se = FALSE, start = list(a=1,b=1),size=1.5) +
    stat_smooth(method = 'nls', formula = y ~ a*exp(b *x), aes(colour = 'Exponential'), se = FALSE, start = list(a=1,b=1),size=1.5)+ theme_few(base_size = 27)
```

    ## Warning: Ignoring unknown parameters: start

    ## Warning: Ignoring unknown parameters: start

    ## Warning in (function (formula, data = parent.frame(), start, control = nls.control(), : No starting values specified for some parameters.
    ## Initializing 'a', 'b' to '1.'.
    ## Consider specifying 'start' or using a selfStart model

    ## Warning: Computation failed in `stat_smooth()`:
    ## Missing value or an infinity produced when evaluating the model

    ## Warning in (function (formula, data = parent.frame(), start, control = nls.control(), : No starting values specified for some parameters.
    ## Initializing 'a', 'b' to '1.'.
    ## Consider specifying 'start' or using a selfStart model

![](DATA621_final_group4_v2_files/figure-markdown_github/unnamed-chunk-15-1.png)

``` r
ggplot(movie_mid, aes(x = log(num_voted_users), y = log(gross))) + geom_point() +
    stat_smooth(method = 'lm', aes(colour = 'linear'), se = FALSE,size=1.5) +
    stat_smooth(method = 'lm', formula = y ~ poly(x,2), aes(colour = 'polynomial'), se= FALSE,size=1.5) +
    stat_smooth(method = 'nls', formula = y ~ a * log(x) +b, aes(colour = 'logarithmic'), se = FALSE, start = list(a=1,b=1),size=1.5) +
    stat_smooth(method = 'nls', formula = y ~ a*exp(b *x), aes(colour = 'Exponential'), se = FALSE, start = list(a=1,b=1),size=1.5)+ theme_few(base_size = 27)
```

    ## Warning: Ignoring unknown parameters: start

    ## Warning: Ignoring unknown parameters: start

    ## Warning in (function (formula, data = parent.frame(), start, control = nls.control(), : No starting values specified for some parameters.
    ## Initializing 'a', 'b' to '1.'.
    ## Consider specifying 'start' or using a selfStart model

    ## Warning in (function (formula, data = parent.frame(), start, control = nls.control(), : No starting values specified for some parameters.
    ## Initializing 'a', 'b' to '1.'.
    ## Consider specifying 'start' or using a selfStart model

![](DATA621_final_group4_v2_files/figure-markdown_github/unnamed-chunk-15-2.png)

``` r
ggplot(movie_mid, aes(x = log(director_ave_gross), y = log(gross))) + geom_point() +
    stat_smooth(method = 'lm', aes(colour = 'linear'), se = FALSE,size=1.5) +
    stat_smooth(method = 'lm', formula = y ~ poly(x,2), aes(colour = 'polynomial'), se= FALSE,size=1.5) +
    stat_smooth(method = 'nls', formula = y ~ a * log(x) +b, aes(colour = 'logarithmic'), se = FALSE, start = list(a=1,b=1),size=1.5) +
    stat_smooth(method = 'nls', formula = y ~ a*exp(b *x), aes(colour = 'Exponential'), se = FALSE, start = list(a=1,b=1),size=1.5)+ theme_few(base_size = 27)
```

    ## Warning: Ignoring unknown parameters: start

    ## Warning: Ignoring unknown parameters: start

    ## Warning in (function (formula, data = parent.frame(), start, control = nls.control(), : No starting values specified for some parameters.
    ## Initializing 'a', 'b' to '1.'.
    ## Consider specifying 'start' or using a selfStart model

    ## Warning in (function (formula, data = parent.frame(), start, control = nls.control(), : No starting values specified for some parameters.
    ## Initializing 'a', 'b' to '1.'.
    ## Consider specifying 'start' or using a selfStart model

    ## Warning: Computation failed in `stat_smooth()`:
    ## number of iterations exceeded maximum of 50

![](DATA621_final_group4_v2_files/figure-markdown_github/unnamed-chunk-15-3.png)

``` r
ggplot(movie_mid, aes(x = log(actor_1_ave_gross), y = log(gross))) + geom_point() +
    stat_smooth(method = 'lm', aes(colour = 'linear'), se = FALSE,size=1.5) +
    stat_smooth(method = 'lm', formula = y ~ poly(x,2), aes(colour = 'polynomial'), se= FALSE,size=1.5) +
    stat_smooth(method = 'nls', formula = y ~ a * log(x) +b, aes(colour = 'logarithmic'), se = FALSE, start = list(a=1,b=1),size=1.5) +
    stat_smooth(method = 'nls', formula = y ~ a*exp(b *x), aes(colour = 'Exponential'), se = FALSE, start = list(a=1,b=1),size=1.5)+ theme_few(base_size = 27)
```

    ## Warning: Ignoring unknown parameters: start

    ## Warning: Ignoring unknown parameters: start

    ## Warning in (function (formula, data = parent.frame(), start, control = nls.control(), : No starting values specified for some parameters.
    ## Initializing 'a', 'b' to '1.'.
    ## Consider specifying 'start' or using a selfStart model

    ## Warning in (function (formula, data = parent.frame(), start, control = nls.control(), : No starting values specified for some parameters.
    ## Initializing 'a', 'b' to '1.'.
    ## Consider specifying 'start' or using a selfStart model

    ## Warning: Computation failed in `stat_smooth()`:
    ## number of iterations exceeded maximum of 50

![](DATA621_final_group4_v2_files/figure-markdown_github/unnamed-chunk-15-4.png)

``` r
ggplot(movie_mid, aes(x = log(actor_2_ave_gross), y = log(gross))) + geom_point() +
    stat_smooth(method = 'lm', aes(colour = 'linear'), se = FALSE,size=1.5) +
    stat_smooth(method = 'lm', formula = y ~ poly(x,2), aes(colour = 'polynomial'), se= FALSE,size=1.5) +
    stat_smooth(method = 'nls', formula = y ~ a * log(x) +b, aes(colour = 'logarithmic'), se = FALSE, start = list(a=1,b=1),size=1.5) +
    stat_smooth(method = 'nls', formula = y ~ a*exp(b *x), aes(colour = 'Exponential'), se = FALSE, start = list(a=1,b=1),size=1.5)+ theme_few(base_size = 27)
```

    ## Warning: Ignoring unknown parameters: start

    ## Warning: Ignoring unknown parameters: start

    ## Warning in (function (formula, data = parent.frame(), start, control = nls.control(), : No starting values specified for some parameters.
    ## Initializing 'a', 'b' to '1.'.
    ## Consider specifying 'start' or using a selfStart model

    ## Warning in (function (formula, data = parent.frame(), start, control = nls.control(), : No starting values specified for some parameters.
    ## Initializing 'a', 'b' to '1.'.
    ## Consider specifying 'start' or using a selfStart model

    ## Warning: Computation failed in `stat_smooth()`:
    ## number of iterations exceeded maximum of 50

![](DATA621_final_group4_v2_files/figure-markdown_github/unnamed-chunk-15-5.png)

``` r
ggplot(movie_mid, aes(x = log(actor_3_ave_gross), y = log(gross))) + geom_point() +
    stat_smooth(method = 'lm', aes(colour = 'linear'), se = FALSE,size=1.5) +
    stat_smooth(method = 'lm', formula = y ~ poly(x,2), aes(colour = 'polynomial'), se= FALSE,size=1.5) +
    stat_smooth(method = 'nls', formula = y ~ a * log(x) +b, aes(colour = 'logarithmic'), se = FALSE, start = list(a=1,b=1),size=1.5) +
    stat_smooth(method = 'nls', formula = y ~ a*exp(b *x), aes(colour = 'Exponential'), se = FALSE, start = list(a=1,b=1),size=1.5)+ theme_few(base_size = 27)
```

    ## Warning: Ignoring unknown parameters: start

    ## Warning: Ignoring unknown parameters: start

    ## Warning in (function (formula, data = parent.frame(), start, control = nls.control(), : No starting values specified for some parameters.
    ## Initializing 'a', 'b' to '1.'.
    ## Consider specifying 'start' or using a selfStart model

    ## Warning in (function (formula, data = parent.frame(), start, control = nls.control(), : No starting values specified for some parameters.
    ## Initializing 'a', 'b' to '1.'.
    ## Consider specifying 'start' or using a selfStart model

    ## Warning: Computation failed in `stat_smooth()`:
    ## number of iterations exceeded maximum of 50

![](DATA621_final_group4_v2_files/figure-markdown_github/unnamed-chunk-15-6.png)

split the data into train and evaluation datasets
-------------------------------------------------

``` r
#movie$Index <- seq(1,nrow(movie))
# <- movie[,c(29,1:28)]  

# fill the empty cell with NA
#movie [movie ==""] <- NA

#split the data for trainning and evaluation:
set.seed(123)
indexes <- sample(1:nrow(movie_mid), size=0.2*nrow(movie_mid))
 
# Split data into two part, train for building model and evaluation for 
evaluation <- movie_mid[indexes,]
train <- movie_mid[-indexes,]

#train.rows <- createDataPartition(y=movie$Index, p=0.8, list = FALSE)   # caret package
#train <- movie[train.rows,]  
#evaluation <-movie[-train.rows,]  
anyNA(train)
```

    ## [1] FALSE

multillinearity for dataset I
-----------------------------

``` r
suppressMessages(suppressWarnings(library(usdm)))

vif_test <- vif(train[,-which(names(train) %in% c("Index","gross","aspect_ratio","movie_title" , "content_rating" , "color", "language" ,"country" , "director_name" , "actor_1_name" ,"actor_2_name" , "actor_3_name" , "genres" , "plot_keywords",  "movie_imdb_link"))])
vif_test
```

    ##                    Variables       VIF
    ## 1       num_user_for_reviews  3.010793
    ## 2     num_critic_for_reviews  3.272572
    ## 3                 imdb_score  1.490539
    ## 4            num_voted_users  4.000220
    ## 5       facenumber_in_poster  1.055334
    ## 6                   duration  1.347857
    ## 7                     budget  1.269068
    ## 8                 title_year  1.406830
    ## 9    director_facebook_likes  1.551842
    ## 10    actor_1_facebook_likes 16.964478
    ## 11    actor_2_facebook_likes  1.505497
    ## 12    actor_3_facebook_likes  2.815392
    ## 13 cast_total_facebook_likes 21.577501
    ## 14      movie_facebook_likes  2.229626
    ## 15        director_ave_gross  2.368349
    ## 16         actor_1_ave_gross  1.876262
    ## 17         actor_2_ave_gross  2.616391
    ## 18         actor_3_ave_gross  2.812728
    ## 19              director_num  1.604498
    ## 20               actor_1_num  1.725136
    ## 21               actor_2_num  1.328993
    ## 22               actor_3_num  1.101177

``` r
vif_final <- vif(train[,-which(names(train) %in% c("Index","aspect_ratio","gross","movie_title" , "content_rating" , "color", "language" ,"country" , "director_name" , "actor_1_name" ,"actor_2_name" , "actor_3_name" , "genres" , "plot_keywords",  "movie_imdb_link","cast_total_facebook_likes"))])
vif_final
```

    ##                  Variables      VIF
    ## 1     num_user_for_reviews 3.010056
    ## 2   num_critic_for_reviews 3.271651
    ## 3               imdb_score 1.489755
    ## 4          num_voted_users 4.000124
    ## 5     facenumber_in_poster 1.055173
    ## 6                 duration 1.347187
    ## 7                   budget 1.268979
    ## 8               title_year 1.406757
    ## 9  director_facebook_likes 1.551396
    ## 10  actor_1_facebook_likes 1.523236
    ## 11  actor_2_facebook_likes 1.184523
    ## 12  actor_3_facebook_likes 1.278070
    ## 13    movie_facebook_likes 2.226994
    ## 14      director_ave_gross 2.368348
    ## 15       actor_1_ave_gross 1.871656
    ## 16       actor_2_ave_gross 2.565049
    ## 17       actor_3_ave_gross 2.812658
    ## 18            director_num 1.604312
    ## 19             actor_1_num 1.724564
    ## 20             actor_2_num 1.259875
    ## 21             actor_3_num 1.100775

dataset II - convert categorical variables to dummy variables
-------------------------------------------------------------

convert "content\_rating", "color", "language", "country","genres" to dummy variables

``` r
movie_mid_dummy <- movie_mid

####################################################
#convert genre to dummay variable
####################################################
genre <- paste(unique(movie_mid$genres),collapse=",")
genre <- str_replace_all(genre,"\\|",",")
genre <-as.list(strsplit(genre, ",")[[1]])
genre <- str_replace_all(genre," ","")
genre <- unique(genre)

#set the last levle of genre "Film-Noir" as control
for(i in 1: (length(genre)-1)){
  movie_mid_dummy[,paste('genre.',genre[i],sep='')] <- ifelse(grepl(genre[i], movie_mid_dummy$genres),1,0)
}


####################################################
#convert color to dummay variable
####################################################
color <- unique(movie_mid$color)

#set the last levle "Black and White" as control
movie_mid_dummy[,'color.Color'] <- ifelse(grepl('Color', movie_mid_dummy$color),1,0)


####################################################
#convert content rating (MMAP rating) to dummay variable
####################################################
#set the "NA" as control
content_rating <- relevel(movie_mid_dummy$content_rating, "NA")
dummy_mmap <- as.data.frame(model.matrix(~content_rating)[,-1])

####################################################
#convert language to dummay variable
####################################################
#set the "None" as control
language <- relevel(movie_mid_dummy$language, "None")
dummy_language <- as.data.frame(model.matrix(~language)[,-1])

####################################################
#convert country to dummay variable
####################################################
#set the "Zulu" as control
country <- relevel(movie_mid_dummy$country, "Zulu")
dummy_country <- as.data.frame(model.matrix(~country)[,-1])

####################################################
#convert aspect_ratio to dummay variable
####################################################
#set the "1.18" as control
aspect <- relevel(movie_mid_dummy$aspect_ratio, "1.18")
dummy_aspect_ratio <- as.data.frame(model.matrix(~aspect)[,-1])


movie_mid_dummy <- cbind(movie_mid_dummy,dummy_mmap,dummy_language,dummy_country,dummy_aspect_ratio)

# remove the variables converted to dummy
# movie_mid_dummy <- movie_mid_dummy[,-which(names(movie_mid_dummy) %in% c("content_rating",  "color", "language",  "country","genres","director_name","actor_1_name","actor_2_name","actor_3_name"))]

anyNA(movie_mid_dummy)
```

    ## [1] FALSE

split the data into train and evaluation datasets
-------------------------------------------------

``` r
#movie$Index <- seq(1,nrow(movie))
# <- movie[,c(29,1:28)]  

# fill the empty cell with NA
#movie [movie ==""] <- NA

#split the data for trainning and evaluation:
set.seed(123)
indexes <- sample(1:nrow(movie_mid_dummy), size=0.2*nrow(movie_mid_dummy))
 
# Split data into two part, train for building model and evaluation for 
evaluation_dummy <- movie_mid_dummy[indexes,]
train_dummy <- movie_mid_dummy[-indexes,]

#train.rows <- createDataPartition(y=movie$Index, p=0.8, list = FALSE)   # caret package
#train <- movie[train.rows,]  
#evaluation <-movie[-train.rows,]  
anyNA(train_dummy)
```

    ## [1] FALSE

Multiple Linear Regression I - based on dataset I
-------------------------------------------------

cast\_total\_facebook\_likes will not be used.

``` r
train$content_rating<- relevel(train$content_rating, "NA")
train$country <- relevel(train$country, "Zulu")
train$language <- relevel(train$language, "Zulu")
train$aspect_ratio<- relevel(train$aspect_ratio, "1.18")

fit_lm_1 <- lm(gross~.,
                  train[,-which(names(train) %in% c("Index", "movie_title","director_name" , "actor_1_name" ,"actor_2_name" , "actor_3_name" , "plot_keywords",  "movie_imdb_link","cast_total_facebook_likes"))])

fit_lm_1.sum <- summary(fit_lm_1)


data.frame(fit_lm_1.sum$coef[fit_lm_1.sum$coef[,4] <= .05, ])
```

    ##                                                           Estimate
    ## num_voted_users                                       7.047237e+01
    ## aspect_ratio1.5                                      -9.401695e+07
    ## aspect_ratio2                                         4.756703e+07
    ## budget                                                9.622645e-02
    ## director_facebook_likes                              -9.082822e+02
    ## actor_1_facebook_likes                               -1.111174e+02
    ## actor_3_facebook_likes                               -1.371376e+03
    ## content_ratingR                                      -1.252972e+07
    ## colorColor                                            1.027690e+07
    ## genresAction|Adventure|Comedy|Family|Fantasy          7.569083e+07
    ## genresAction|Adventure|Comedy|Family|Fantasy|Sci_Fi  -5.770279e+07
    ## genresAction|Adventure|Crime|Drama|Mystery|Thriller   6.197611e+07
    ## genresAction|Adventure|Drama|History|War             -4.011643e+07
    ## genresAction|Adventure|Drama|Sci_Fi|Thriller         -8.963541e+07
    ## genresAction|Adventure|Family|Sci_Fi                 -4.833490e+07
    ## genresAction|Adventure|Fantasy|Romance                9.761742e+07
    ## genresAction|Adventure|Fantasy|Sci_Fi                 4.723316e+07
    ## genresAction|Adventure|Horror|Sci_Fi                 -4.007143e+07
    ## genresAction|Adventure|Romance|Sci_Fi|Thriller       -8.916736e+07
    ## genresAction|Animation|Comedy|Family|Sci_Fi           7.336906e+07
    ## genresAction|Animation|Sci_Fi                        -1.084037e+08
    ## genresAction|Biography|Drama|History|Romance|War     -6.003502e+07
    ## genresAction|Comedy|Romance|Thriller                 -4.724267e+07
    ## genresAction|Drama|Mystery|Sci_Fi                    -5.917653e+07
    ## genresAction|Western                                 -7.392241e+07
    ## genresAdventure|Animation|Comedy|Family|Sport         5.720151e+07
    ## genresAdventure|Animation|Drama|Family|Fantasy       -1.032687e+08
    ## genresAdventure|Animation|Drama|Family|Musical        1.004262e+08
    ## genresAdventure|Comedy|Family|Mystery|Sci_Fi          9.729601e+07
    ## genresAdventure|Comedy|Fantasy                       -4.734159e+07
    ## genresAdventure|Comedy|Fantasy|Sci_Fi                -8.938838e+07
    ## genresAdventure|Drama                                -3.205044e+07
    ## genresAdventure|Drama|Family|Fantasy                  6.390156e+07
    ## genresAdventure|Drama|Fantasy|Romance                 4.439784e+07
    ## genresAdventure|Fantasy|Mystery                      -6.249960e+07
    ## genresAnimation|Comedy|Family|Fantasy|Music           4.827757e+07
    ## genresAnimation|Drama|Family|Fantasy|Musical|Romance -6.195227e+07
    ## genresBiography|Comedy|Crime|Drama                   -4.688229e+07
    ## genresBiography|Crime|Drama|History|Western          -6.257742e+07
    ## genresBiography|Crime|Drama|Romance|Thriller         -6.464115e+07
    ## genresBiography|Drama|History|Romance                -3.671502e+07
    ## genresComedy|Crime|Drama|Mystery                     -9.196786e+07
    ## genresComedy|Horror|Sci_Fi                           -6.388903e+07
    ## genresDrama | Fantasy | Thriller                     -1.592910e+08
    ## genresDrama|Fantasy|Thriller                         -5.373648e+07
    ## genresDrama|Musical                                  -4.300540e+07
    ## genresFamily|Sci_Fi                                   2.231789e+08
    ## genresFantasy | Horror | Mystery                     -9.581513e+07
    ## director_ave_gross                                    2.425076e-01
    ## actor_1_ave_gross                                     1.498446e-01
    ## actor_2_ave_gross                                     3.194067e-01
    ## actor_3_ave_gross                                     4.075478e-01
    ## actor_1_num                                          -1.882114e+05
    ##                                                        Std..Error
    ## num_voted_users                                      7.399735e+00
    ## aspect_ratio1.5                                      3.778090e+07
    ## aspect_ratio2                                        1.616566e+07
    ## budget                                               1.450376e-02
    ## director_facebook_likes                              1.991234e+02
    ## actor_1_facebook_likes                               4.782201e+01
    ## actor_3_facebook_likes                               3.173784e+02
    ## content_ratingR                                      5.640698e+06
    ## colorColor                                           3.031957e+06
    ## genresAction|Adventure|Comedy|Family|Fantasy         2.879614e+07
    ## genresAction|Adventure|Comedy|Family|Fantasy|Sci_Fi  1.954507e+07
    ## genresAction|Adventure|Crime|Drama|Mystery|Thriller  2.876825e+07
    ## genresAction|Adventure|Drama|History|War             1.627860e+07
    ## genresAction|Adventure|Drama|Sci_Fi|Thriller         3.364262e+07
    ## genresAction|Adventure|Family|Sci_Fi                 2.223793e+07
    ## genresAction|Adventure|Fantasy|Romance               1.951739e+07
    ## genresAction|Adventure|Fantasy|Sci_Fi                1.508717e+07
    ## genresAction|Adventure|Horror|Sci_Fi                 1.789570e+07
    ## genresAction|Adventure|Romance|Sci_Fi|Thriller       2.220761e+07
    ## genresAction|Animation|Comedy|Family|Sci_Fi          2.246493e+07
    ## genresAction|Animation|Sci_Fi                        3.428372e+07
    ## genresAction|Biography|Drama|History|Romance|War     2.878471e+07
    ## genresAction|Comedy|Romance|Thriller                 2.237128e+07
    ## genresAction|Drama|Mystery|Sci_Fi                    2.887617e+07
    ## genresAction|Western                                 2.867803e+07
    ## genresAdventure|Animation|Comedy|Family|Sport        2.283301e+07
    ## genresAdventure|Animation|Drama|Family|Fantasy       2.894023e+07
    ## genresAdventure|Animation|Drama|Family|Musical       3.005945e+07
    ## genresAdventure|Comedy|Family|Mystery|Sci_Fi         2.879525e+07
    ## genresAdventure|Comedy|Fantasy                       2.215450e+07
    ## genresAdventure|Comedy|Fantasy|Sci_Fi                2.895855e+07
    ## genresAdventure|Drama                                1.632394e+07
    ## genresAdventure|Drama|Family|Fantasy                 1.712222e+07
    ## genresAdventure|Drama|Fantasy|Romance                1.980703e+07
    ## genresAdventure|Fantasy|Mystery                      2.869722e+07
    ## genresAnimation|Comedy|Family|Fantasy|Music          2.230437e+07
    ## genresAnimation|Drama|Family|Fantasy|Musical|Romance 2.894124e+07
    ## genresBiography|Comedy|Crime|Drama                   1.944522e+07
    ## genresBiography|Crime|Drama|History|Western          2.884068e+07
    ## genresBiography|Crime|Drama|Romance|Thriller         2.878163e+07
    ## genresBiography|Drama|History|Romance                1.589788e+07
    ## genresComedy|Crime|Drama|Mystery                     3.227069e+07
    ## genresComedy|Horror|Sci_Fi                           2.879038e+07
    ## genresDrama | Fantasy | Thriller                     3.168131e+07
    ## genresDrama|Fantasy|Thriller                         1.946535e+07
    ## genresDrama|Musical                                  2.037422e+07
    ## genresFamily|Sci_Fi                                  2.901797e+07
    ## genresFantasy | Horror | Mystery                     3.175918e+07
    ## director_ave_gross                                   1.619826e-02
    ## actor_1_ave_gross                                    1.657780e-02
    ## actor_2_ave_gross                                    1.548762e-02
    ## actor_3_ave_gross                                    1.537190e-02
    ## actor_1_num                                          6.830025e+04
    ##                                                        t.value
    ## num_voted_users                                       9.523635
    ## aspect_ratio1.5                                      -2.488478
    ## aspect_ratio2                                         2.942474
    ## budget                                                6.634587
    ## director_facebook_likes                              -4.561404
    ## actor_1_facebook_likes                               -2.323561
    ## actor_3_facebook_likes                               -4.320951
    ## content_ratingR                                      -2.221306
    ## colorColor                                            3.389527
    ## genresAction|Adventure|Comedy|Family|Fantasy          2.628506
    ## genresAction|Adventure|Comedy|Family|Fantasy|Sci_Fi  -2.952294
    ## genresAction|Adventure|Crime|Drama|Mystery|Thriller   2.154323
    ## genresAction|Adventure|Drama|History|War             -2.464365
    ## genresAction|Adventure|Drama|Sci_Fi|Thriller         -2.664341
    ## genresAction|Adventure|Family|Sci_Fi                 -2.173535
    ## genresAction|Adventure|Fantasy|Romance                5.001562
    ## genresAction|Adventure|Fantasy|Sci_Fi                 3.130684
    ## genresAction|Adventure|Horror|Sci_Fi                 -2.239165
    ## genresAction|Adventure|Romance|Sci_Fi|Thriller       -4.015172
    ## genresAction|Animation|Comedy|Family|Sci_Fi           3.265938
    ## genresAction|Animation|Sci_Fi                        -3.161958
    ## genresAction|Biography|Drama|History|Romance|War     -2.085657
    ## genresAction|Comedy|Romance|Thriller                 -2.111755
    ## genresAction|Drama|Mystery|Sci_Fi                    -2.049320
    ## genresAction|Western                                 -2.577667
    ## genresAdventure|Animation|Comedy|Family|Sport         2.505211
    ## genresAdventure|Animation|Drama|Family|Fantasy       -3.568343
    ## genresAdventure|Animation|Drama|Family|Musical        3.340918
    ## genresAdventure|Comedy|Family|Mystery|Sci_Fi          3.378891
    ## genresAdventure|Comedy|Fantasy                       -2.136883
    ## genresAdventure|Comedy|Fantasy|Sci_Fi                -3.086770
    ## genresAdventure|Drama                                -1.963401
    ## genresAdventure|Drama|Family|Fantasy                  3.732083
    ## genresAdventure|Drama|Fantasy|Romance                 2.241520
    ## genresAdventure|Fantasy|Mystery                      -2.177897
    ## genresAnimation|Comedy|Family|Fantasy|Music           2.164489
    ## genresAnimation|Drama|Family|Fantasy|Musical|Romance -2.140622
    ## genresBiography|Comedy|Crime|Drama                   -2.410993
    ## genresBiography|Crime|Drama|History|Western          -2.169762
    ## genresBiography|Crime|Drama|Romance|Thriller         -2.245917
    ## genresBiography|Drama|History|Romance                -2.309428
    ## genresComedy|Crime|Drama|Mystery                     -2.849889
    ## genresComedy|Horror|Sci_Fi                           -2.219110
    ## genresDrama | Fantasy | Thriller                     -5.027915
    ## genresDrama|Fantasy|Thriller                         -2.760623
    ## genresDrama|Musical                                  -2.110775
    ## genresFamily|Sci_Fi                                   7.691057
    ## genresFantasy | Horror | Mystery                     -3.016928
    ## director_ave_gross                                   14.971215
    ## actor_1_ave_gross                                     9.038870
    ## actor_2_ave_gross                                    20.623352
    ## actor_3_ave_gross                                    26.512519
    ## actor_1_num                                          -2.755647
    ##                                                           Pr...t..
    ## num_voted_users                                       3.695013e-21
    ## aspect_ratio1.5                                       1.289147e-02
    ## aspect_ratio2                                         3.284916e-03
    ## budget                                                3.947674e-11
    ## director_facebook_likes                               5.317691e-06
    ## actor_1_facebook_likes                                2.022603e-02
    ## actor_3_facebook_likes                                1.612140e-05
    ## content_ratingR                                       2.641634e-02
    ## colorColor                                            7.106124e-04
    ## genresAction|Adventure|Comedy|Family|Fantasy          8.626719e-03
    ## genresAction|Adventure|Comedy|Family|Fantasy|Sci_Fi   3.182563e-03
    ## genresAction|Adventure|Crime|Drama|Mystery|Thriller   3.130677e-02
    ## genresAction|Adventure|Drama|History|War              1.379008e-02
    ## genresAction|Adventure|Drama|Sci_Fi|Thriller          7.761741e-03
    ## genresAction|Adventure|Family|Sci_Fi                  2.983039e-02
    ## genresAction|Adventure|Fantasy|Romance                6.067031e-07
    ## genresAction|Adventure|Fantasy|Sci_Fi                 1.763434e-03
    ## genresAction|Adventure|Horror|Sci_Fi                  2.522971e-02
    ## genresAction|Adventure|Romance|Sci_Fi|Thriller        6.108999e-05
    ## genresAction|Animation|Comedy|Family|Sci_Fi           1.105239e-03
    ## genresAction|Animation|Sci_Fi                         1.585239e-03
    ## genresAction|Biography|Drama|History|Romance|War      3.710736e-02
    ## genresAction|Comedy|Romance|Thriller                  3.480298e-02
    ## genresAction|Drama|Mystery|Sci_Fi                     4.053131e-02
    ## genresAction|Western                                  1.000177e-02
    ## genresAdventure|Animation|Comedy|Family|Sport         1.229875e-02
    ## genresAdventure|Animation|Drama|Family|Fantasy        3.657653e-04
    ## genresAdventure|Animation|Drama|Family|Musical        8.468584e-04
    ## genresAdventure|Comedy|Family|Mystery|Sci_Fi          7.385537e-04
    ## genresAdventure|Comedy|Fantasy                        3.270087e-02
    ## genresAdventure|Comedy|Fantasy|Sci_Fi                 2.044846e-03
    ## genresAdventure|Drama                                 4.970641e-02
    ## genresAdventure|Drama|Family|Fantasy                  1.939859e-04
    ## genresAdventure|Drama|Fantasy|Romance                 2.507678e-02
    ## genresAdventure|Fantasy|Mystery                       2.950358e-02
    ## genresAnimation|Comedy|Family|Fantasy|Music           3.051789e-02
    ## genresAnimation|Drama|Family|Fantasy|Musical|Romance  3.239758e-02
    ## genresBiography|Comedy|Crime|Drama                    1.597832e-02
    ## genresBiography|Crime|Drama|History|Western           3.011546e-02
    ## genresBiography|Crime|Drama|Romance|Thriller          2.479330e-02
    ## genresBiography|Drama|History|Romance                 2.099811e-02
    ## genresComedy|Crime|Drama|Mystery                      4.408010e-03
    ## genresComedy|Horror|Sci_Fi                            2.656555e-02
    ## genresDrama | Fantasy | Thriller                      5.296954e-07
    ## genresDrama|Fantasy|Thriller                          5.809758e-03
    ## genresDrama|Musical                                   3.488721e-02
    ## genresFamily|Sci_Fi                                   2.057892e-14
    ## genresFantasy | Horror | Mystery                      2.578344e-03
    ## director_ave_gross                                    1.156310e-48
    ## actor_1_ave_gross                                     3.021005e-19
    ## actor_2_ave_gross                                     1.267887e-87
    ## actor_3_ave_gross                                    3.025650e-137
    ## actor_1_num                                           5.898603e-03

### outlier test

``` r
outlierTest(fit_lm_1)
```

    ##       rstudent unadjusted p-value Bonferonni p
    ## 1    14.522917         5.3036e-46   1.6245e-42
    ## 1231 -9.433231         8.5375e-21   2.6150e-17
    ## 30    7.550723         5.9611e-14   1.8259e-10
    ## 52   -5.979760         2.5425e-09   7.7878e-06
    ## 1395  5.530090         3.5209e-08   1.0785e-04
    ## 365  -5.354678         9.3239e-08   2.8559e-04
    ## 437   5.231786         1.8132e-07   5.5537e-04
    ## 4     5.134522         3.0386e-07   9.3074e-04
    ## 198   5.078471         4.0753e-07   1.2483e-03
    ## 562  -5.078471         4.0753e-07   1.2483e-03

``` r
influencePlot(fit_lm_1,col='red',id.n=2)
```

![](DATA621_final_group4_v2_files/figure-markdown_github/unnamed-chunk-21-1.png)

    ##         StudRes       Hat      CookD
    ## 1    14.5229173 0.1555584 0.04453745
    ## 59          NaN 1.0000000        NaN
    ## 62          NaN 1.0000000        NaN
    ## 176   0.9842565 0.9996467 3.39644034
    ## 198   5.0784714 0.6131591 0.05017618
    ## 1231 -9.4332308 0.1736975 0.02241811

### update the model by removing the outliers and influential points

``` r
fit_lm_1_mdf <- update(fit_lm_1,subset=c(-1-1231-30-52-1395-365-437-4-198-562-176))
#compareCoefs(fit_lm_1, fit_lm_1_mdf)
```

``` r
fit_lm_1_mdf.sum <- summary(fit_lm_1_mdf)
data.frame(fit_lm_1_mdf.sum$coef[fit_lm_1_mdf.sum$coef[,4] <= .05, ])
```

    ##                                                           Estimate
    ## num_voted_users                                       7.047237e+01
    ## aspect_ratio1.5                                      -9.401695e+07
    ## aspect_ratio2                                         4.756703e+07
    ## budget                                                9.622645e-02
    ## director_facebook_likes                              -9.082822e+02
    ## actor_1_facebook_likes                               -1.111174e+02
    ## actor_3_facebook_likes                               -1.371376e+03
    ## content_ratingR                                      -1.252972e+07
    ## colorColor                                            1.027690e+07
    ## genresAction|Adventure|Comedy|Family|Fantasy          7.569083e+07
    ## genresAction|Adventure|Comedy|Family|Fantasy|Sci_Fi  -5.770279e+07
    ## genresAction|Adventure|Crime|Drama|Mystery|Thriller   6.197611e+07
    ## genresAction|Adventure|Drama|History|War             -4.011643e+07
    ## genresAction|Adventure|Drama|Sci_Fi|Thriller         -8.963541e+07
    ## genresAction|Adventure|Family|Sci_Fi                 -4.833490e+07
    ## genresAction|Adventure|Fantasy|Romance                9.761742e+07
    ## genresAction|Adventure|Fantasy|Sci_Fi                 4.723316e+07
    ## genresAction|Adventure|Horror|Sci_Fi                 -4.007143e+07
    ## genresAction|Adventure|Romance|Sci_Fi|Thriller       -8.916736e+07
    ## genresAction|Animation|Comedy|Family|Sci_Fi           7.336906e+07
    ## genresAction|Animation|Sci_Fi                        -1.084037e+08
    ## genresAction|Biography|Drama|History|Romance|War     -6.003502e+07
    ## genresAction|Comedy|Romance|Thriller                 -4.724267e+07
    ## genresAction|Drama|Mystery|Sci_Fi                    -5.917653e+07
    ## genresAction|Western                                 -7.392241e+07
    ## genresAdventure|Animation|Comedy|Family|Sport         5.720151e+07
    ## genresAdventure|Animation|Drama|Family|Fantasy       -1.032687e+08
    ## genresAdventure|Animation|Drama|Family|Musical        1.004262e+08
    ## genresAdventure|Comedy|Family|Mystery|Sci_Fi          9.729601e+07
    ## genresAdventure|Comedy|Fantasy                       -4.734159e+07
    ## genresAdventure|Comedy|Fantasy|Sci_Fi                -8.938838e+07
    ## genresAdventure|Drama                                -3.205044e+07
    ## genresAdventure|Drama|Family|Fantasy                  6.390156e+07
    ## genresAdventure|Drama|Fantasy|Romance                 4.439784e+07
    ## genresAdventure|Fantasy|Mystery                      -6.249960e+07
    ## genresAnimation|Comedy|Family|Fantasy|Music           4.827757e+07
    ## genresAnimation|Drama|Family|Fantasy|Musical|Romance -6.195227e+07
    ## genresBiography|Comedy|Crime|Drama                   -4.688229e+07
    ## genresBiography|Crime|Drama|History|Western          -6.257742e+07
    ## genresBiography|Crime|Drama|Romance|Thriller         -6.464115e+07
    ## genresBiography|Drama|History|Romance                -3.671502e+07
    ## genresComedy|Crime|Drama|Mystery                     -9.196786e+07
    ## genresComedy|Horror|Sci_Fi                           -6.388903e+07
    ## genresDrama | Fantasy | Thriller                     -1.592910e+08
    ## genresDrama|Fantasy|Thriller                         -5.373648e+07
    ## genresDrama|Musical                                  -4.300540e+07
    ## genresFamily|Sci_Fi                                   2.231789e+08
    ## genresFantasy | Horror | Mystery                     -9.581513e+07
    ## director_ave_gross                                    2.425076e-01
    ## actor_1_ave_gross                                     1.498446e-01
    ## actor_2_ave_gross                                     3.194067e-01
    ## actor_3_ave_gross                                     4.075478e-01
    ## actor_1_num                                          -1.882114e+05
    ##                                                        Std..Error
    ## num_voted_users                                      7.399735e+00
    ## aspect_ratio1.5                                      3.778090e+07
    ## aspect_ratio2                                        1.616566e+07
    ## budget                                               1.450376e-02
    ## director_facebook_likes                              1.991234e+02
    ## actor_1_facebook_likes                               4.782201e+01
    ## actor_3_facebook_likes                               3.173784e+02
    ## content_ratingR                                      5.640698e+06
    ## colorColor                                           3.031957e+06
    ## genresAction|Adventure|Comedy|Family|Fantasy         2.879614e+07
    ## genresAction|Adventure|Comedy|Family|Fantasy|Sci_Fi  1.954507e+07
    ## genresAction|Adventure|Crime|Drama|Mystery|Thriller  2.876825e+07
    ## genresAction|Adventure|Drama|History|War             1.627860e+07
    ## genresAction|Adventure|Drama|Sci_Fi|Thriller         3.364262e+07
    ## genresAction|Adventure|Family|Sci_Fi                 2.223793e+07
    ## genresAction|Adventure|Fantasy|Romance               1.951739e+07
    ## genresAction|Adventure|Fantasy|Sci_Fi                1.508717e+07
    ## genresAction|Adventure|Horror|Sci_Fi                 1.789570e+07
    ## genresAction|Adventure|Romance|Sci_Fi|Thriller       2.220761e+07
    ## genresAction|Animation|Comedy|Family|Sci_Fi          2.246493e+07
    ## genresAction|Animation|Sci_Fi                        3.428372e+07
    ## genresAction|Biography|Drama|History|Romance|War     2.878471e+07
    ## genresAction|Comedy|Romance|Thriller                 2.237128e+07
    ## genresAction|Drama|Mystery|Sci_Fi                    2.887617e+07
    ## genresAction|Western                                 2.867803e+07
    ## genresAdventure|Animation|Comedy|Family|Sport        2.283301e+07
    ## genresAdventure|Animation|Drama|Family|Fantasy       2.894023e+07
    ## genresAdventure|Animation|Drama|Family|Musical       3.005945e+07
    ## genresAdventure|Comedy|Family|Mystery|Sci_Fi         2.879525e+07
    ## genresAdventure|Comedy|Fantasy                       2.215450e+07
    ## genresAdventure|Comedy|Fantasy|Sci_Fi                2.895855e+07
    ## genresAdventure|Drama                                1.632394e+07
    ## genresAdventure|Drama|Family|Fantasy                 1.712222e+07
    ## genresAdventure|Drama|Fantasy|Romance                1.980703e+07
    ## genresAdventure|Fantasy|Mystery                      2.869722e+07
    ## genresAnimation|Comedy|Family|Fantasy|Music          2.230437e+07
    ## genresAnimation|Drama|Family|Fantasy|Musical|Romance 2.894124e+07
    ## genresBiography|Comedy|Crime|Drama                   1.944522e+07
    ## genresBiography|Crime|Drama|History|Western          2.884068e+07
    ## genresBiography|Crime|Drama|Romance|Thriller         2.878163e+07
    ## genresBiography|Drama|History|Romance                1.589788e+07
    ## genresComedy|Crime|Drama|Mystery                     3.227069e+07
    ## genresComedy|Horror|Sci_Fi                           2.879038e+07
    ## genresDrama | Fantasy | Thriller                     3.168131e+07
    ## genresDrama|Fantasy|Thriller                         1.946535e+07
    ## genresDrama|Musical                                  2.037422e+07
    ## genresFamily|Sci_Fi                                  2.901797e+07
    ## genresFantasy | Horror | Mystery                     3.175918e+07
    ## director_ave_gross                                   1.619826e-02
    ## actor_1_ave_gross                                    1.657780e-02
    ## actor_2_ave_gross                                    1.548762e-02
    ## actor_3_ave_gross                                    1.537190e-02
    ## actor_1_num                                          6.830025e+04
    ##                                                        t.value
    ## num_voted_users                                       9.523635
    ## aspect_ratio1.5                                      -2.488478
    ## aspect_ratio2                                         2.942474
    ## budget                                                6.634587
    ## director_facebook_likes                              -4.561404
    ## actor_1_facebook_likes                               -2.323561
    ## actor_3_facebook_likes                               -4.320951
    ## content_ratingR                                      -2.221306
    ## colorColor                                            3.389527
    ## genresAction|Adventure|Comedy|Family|Fantasy          2.628506
    ## genresAction|Adventure|Comedy|Family|Fantasy|Sci_Fi  -2.952294
    ## genresAction|Adventure|Crime|Drama|Mystery|Thriller   2.154323
    ## genresAction|Adventure|Drama|History|War             -2.464365
    ## genresAction|Adventure|Drama|Sci_Fi|Thriller         -2.664341
    ## genresAction|Adventure|Family|Sci_Fi                 -2.173535
    ## genresAction|Adventure|Fantasy|Romance                5.001562
    ## genresAction|Adventure|Fantasy|Sci_Fi                 3.130684
    ## genresAction|Adventure|Horror|Sci_Fi                 -2.239165
    ## genresAction|Adventure|Romance|Sci_Fi|Thriller       -4.015172
    ## genresAction|Animation|Comedy|Family|Sci_Fi           3.265938
    ## genresAction|Animation|Sci_Fi                        -3.161958
    ## genresAction|Biography|Drama|History|Romance|War     -2.085657
    ## genresAction|Comedy|Romance|Thriller                 -2.111755
    ## genresAction|Drama|Mystery|Sci_Fi                    -2.049320
    ## genresAction|Western                                 -2.577667
    ## genresAdventure|Animation|Comedy|Family|Sport         2.505211
    ## genresAdventure|Animation|Drama|Family|Fantasy       -3.568343
    ## genresAdventure|Animation|Drama|Family|Musical        3.340918
    ## genresAdventure|Comedy|Family|Mystery|Sci_Fi          3.378891
    ## genresAdventure|Comedy|Fantasy                       -2.136883
    ## genresAdventure|Comedy|Fantasy|Sci_Fi                -3.086770
    ## genresAdventure|Drama                                -1.963401
    ## genresAdventure|Drama|Family|Fantasy                  3.732083
    ## genresAdventure|Drama|Fantasy|Romance                 2.241520
    ## genresAdventure|Fantasy|Mystery                      -2.177897
    ## genresAnimation|Comedy|Family|Fantasy|Music           2.164489
    ## genresAnimation|Drama|Family|Fantasy|Musical|Romance -2.140622
    ## genresBiography|Comedy|Crime|Drama                   -2.410993
    ## genresBiography|Crime|Drama|History|Western          -2.169762
    ## genresBiography|Crime|Drama|Romance|Thriller         -2.245917
    ## genresBiography|Drama|History|Romance                -2.309428
    ## genresComedy|Crime|Drama|Mystery                     -2.849889
    ## genresComedy|Horror|Sci_Fi                           -2.219110
    ## genresDrama | Fantasy | Thriller                     -5.027915
    ## genresDrama|Fantasy|Thriller                         -2.760623
    ## genresDrama|Musical                                  -2.110775
    ## genresFamily|Sci_Fi                                   7.691057
    ## genresFantasy | Horror | Mystery                     -3.016928
    ## director_ave_gross                                   14.971215
    ## actor_1_ave_gross                                     9.038870
    ## actor_2_ave_gross                                    20.623352
    ## actor_3_ave_gross                                    26.512519
    ## actor_1_num                                          -2.755647
    ##                                                           Pr...t..
    ## num_voted_users                                       3.695013e-21
    ## aspect_ratio1.5                                       1.289147e-02
    ## aspect_ratio2                                         3.284916e-03
    ## budget                                                3.947674e-11
    ## director_facebook_likes                               5.317691e-06
    ## actor_1_facebook_likes                                2.022603e-02
    ## actor_3_facebook_likes                                1.612140e-05
    ## content_ratingR                                       2.641634e-02
    ## colorColor                                            7.106124e-04
    ## genresAction|Adventure|Comedy|Family|Fantasy          8.626719e-03
    ## genresAction|Adventure|Comedy|Family|Fantasy|Sci_Fi   3.182563e-03
    ## genresAction|Adventure|Crime|Drama|Mystery|Thriller   3.130677e-02
    ## genresAction|Adventure|Drama|History|War              1.379008e-02
    ## genresAction|Adventure|Drama|Sci_Fi|Thriller          7.761741e-03
    ## genresAction|Adventure|Family|Sci_Fi                  2.983039e-02
    ## genresAction|Adventure|Fantasy|Romance                6.067031e-07
    ## genresAction|Adventure|Fantasy|Sci_Fi                 1.763434e-03
    ## genresAction|Adventure|Horror|Sci_Fi                  2.522971e-02
    ## genresAction|Adventure|Romance|Sci_Fi|Thriller        6.108999e-05
    ## genresAction|Animation|Comedy|Family|Sci_Fi           1.105239e-03
    ## genresAction|Animation|Sci_Fi                         1.585239e-03
    ## genresAction|Biography|Drama|History|Romance|War      3.710736e-02
    ## genresAction|Comedy|Romance|Thriller                  3.480298e-02
    ## genresAction|Drama|Mystery|Sci_Fi                     4.053131e-02
    ## genresAction|Western                                  1.000177e-02
    ## genresAdventure|Animation|Comedy|Family|Sport         1.229875e-02
    ## genresAdventure|Animation|Drama|Family|Fantasy        3.657653e-04
    ## genresAdventure|Animation|Drama|Family|Musical        8.468584e-04
    ## genresAdventure|Comedy|Family|Mystery|Sci_Fi          7.385537e-04
    ## genresAdventure|Comedy|Fantasy                        3.270087e-02
    ## genresAdventure|Comedy|Fantasy|Sci_Fi                 2.044846e-03
    ## genresAdventure|Drama                                 4.970641e-02
    ## genresAdventure|Drama|Family|Fantasy                  1.939859e-04
    ## genresAdventure|Drama|Fantasy|Romance                 2.507678e-02
    ## genresAdventure|Fantasy|Mystery                       2.950358e-02
    ## genresAnimation|Comedy|Family|Fantasy|Music           3.051789e-02
    ## genresAnimation|Drama|Family|Fantasy|Musical|Romance  3.239758e-02
    ## genresBiography|Comedy|Crime|Drama                    1.597832e-02
    ## genresBiography|Crime|Drama|History|Western           3.011546e-02
    ## genresBiography|Crime|Drama|Romance|Thriller          2.479330e-02
    ## genresBiography|Drama|History|Romance                 2.099811e-02
    ## genresComedy|Crime|Drama|Mystery                      4.408010e-03
    ## genresComedy|Horror|Sci_Fi                            2.656555e-02
    ## genresDrama | Fantasy | Thriller                      5.296954e-07
    ## genresDrama|Fantasy|Thriller                          5.809758e-03
    ## genresDrama|Musical                                   3.488721e-02
    ## genresFamily|Sci_Fi                                   2.057892e-14
    ## genresFantasy | Horror | Mystery                      2.578344e-03
    ## director_ave_gross                                    1.156310e-48
    ## actor_1_ave_gross                                     3.021005e-19
    ## actor_2_ave_gross                                     1.267887e-87
    ## actor_3_ave_gross                                    3.025650e-137
    ## actor_1_num                                           5.898603e-03

check multillinearity for model I
---------------------------------

``` r
# extract the variable with significant coefficient
library(broom)
lm_model_coefficients <- tidy(fit_lm_1)
lm_model_coefficients[lm_model_coefficients$p.value<0.05,1]
```

    ##  [1] "num_voted_users"                                     
    ##  [2] "aspect_ratio1.5"                                     
    ##  [3] "aspect_ratio2"                                       
    ##  [4] "budget"                                              
    ##  [5] "director_facebook_likes"                             
    ##  [6] "actor_1_facebook_likes"                              
    ##  [7] "actor_3_facebook_likes"                              
    ##  [8] "content_ratingR"                                     
    ##  [9] "colorColor"                                          
    ## [10] "genresAction|Adventure|Comedy|Family|Fantasy"        
    ## [11] "genresAction|Adventure|Comedy|Family|Fantasy|Sci_Fi" 
    ## [12] "genresAction|Adventure|Crime|Drama|Mystery|Thriller" 
    ## [13] "genresAction|Adventure|Drama|History|War"            
    ## [14] "genresAction|Adventure|Drama|Sci_Fi|Thriller"        
    ## [15] "genresAction|Adventure|Family|Sci_Fi"                
    ## [16] "genresAction|Adventure|Fantasy|Romance"              
    ## [17] "genresAction|Adventure|Fantasy|Sci_Fi"               
    ## [18] "genresAction|Adventure|Horror|Sci_Fi"                
    ## [19] "genresAction|Adventure|Romance|Sci_Fi|Thriller"      
    ## [20] "genresAction|Animation|Comedy|Family|Sci_Fi"         
    ## [21] "genresAction|Animation|Sci_Fi"                       
    ## [22] "genresAction|Biography|Drama|History|Romance|War"    
    ## [23] "genresAction|Comedy|Romance|Thriller"                
    ## [24] "genresAction|Drama|Mystery|Sci_Fi"                   
    ## [25] "genresAction|Western"                                
    ## [26] "genresAdventure|Animation|Comedy|Family|Sport"       
    ## [27] "genresAdventure|Animation|Drama|Family|Fantasy"      
    ## [28] "genresAdventure|Animation|Drama|Family|Musical"      
    ## [29] "genresAdventure|Comedy|Family|Mystery|Sci_Fi"        
    ## [30] "genresAdventure|Comedy|Fantasy"                      
    ## [31] "genresAdventure|Comedy|Fantasy|Sci_Fi"               
    ## [32] "genresAdventure|Drama"                               
    ## [33] "genresAdventure|Drama|Family|Fantasy"                
    ## [34] "genresAdventure|Drama|Fantasy|Romance"               
    ## [35] "genresAdventure|Fantasy|Mystery"                     
    ## [36] "genresAnimation|Comedy|Family|Fantasy|Music"         
    ## [37] "genresAnimation|Drama|Family|Fantasy|Musical|Romance"
    ## [38] "genresBiography|Comedy|Crime|Drama"                  
    ## [39] "genresBiography|Crime|Drama|History|Western"         
    ## [40] "genresBiography|Crime|Drama|Romance|Thriller"        
    ## [41] "genresBiography|Drama|History|Romance"               
    ## [42] "genresComedy|Crime|Drama|Mystery"                    
    ## [43] "genresComedy|Horror|Sci_Fi"                          
    ## [44] "genresDrama | Fantasy | Thriller"                    
    ## [45] "genresDrama|Fantasy|Thriller"                        
    ## [46] "genresDrama|Musical"                                 
    ## [47] "genresFamily|Sci_Fi"                                 
    ## [48] "genresFantasy | Horror | Mystery"                    
    ## [49] "director_ave_gross"                                  
    ## [50] "actor_1_ave_gross"                                   
    ## [51] "actor_2_ave_gross"                                   
    ## [52] "actor_3_ave_gross"                                   
    ## [53] "actor_1_num"

``` r
suppressMessages(suppressWarnings(library(usdm)))

vif(train[,which(names(train) %in%c("num_voted_users", "aspect_ratio1.5" ,"aspect_ratio2", "budget", "director_facebook_likes","actor_1_facebook_likes", "actor_3_facebook_likes","content_ratingR", "color","genres", "director_ave_gross","actor_1_ave_gross", "actor_2_ave_gross", "actor_3_ave_gross","actor_1_num"   ))])
```

    ## Warning in model.response(mf, "numeric"): using type = "numeric" with a
    ## factor response will be ignored

    ## Warning in Ops.factor(y, z$residuals): '-' not meaningful for factors

    ## Warning in Ops.factor(r, 2): '^' not meaningful for factors

    ## Warning in model.response(mf, "numeric"): using type = "numeric" with a
    ## factor response will be ignored

    ## Warning in Ops.factor(y, z$residuals): '-' not meaningful for factors

    ## Warning in Ops.factor(r, 2): '^' not meaningful for factors

    ##                  Variables      VIF
    ## 1          num_voted_users 2.369049
    ## 2                   budget 3.557834
    ## 3  director_facebook_likes 1.344705
    ## 4   actor_1_facebook_likes 1.725678
    ## 5   actor_3_facebook_likes 1.540898
    ## 6                    color       NA
    ## 7                   genres       NA
    ## 8       director_ave_gross 3.225087
    ## 9        actor_1_ave_gross 2.372722
    ## 10       actor_2_ave_gross 3.455505
    ## 11       actor_3_ave_gross 3.738799
    ## 12             actor_1_num 2.018039

### Pseudo R-squared, LogLikelihood, AIC

``` r
lm_1_null <- lm(gross~1,
                  train[,-which(names(train) %in% c("Index", "movie_title","director_name" , "actor_1_name" ,"actor_2_name" , "actor_3_name" , "plot_keywords",  "movie_imdb_link","cast_total_facebook_likes"))])

# McFadden's pseudo-R^2
(p_R2_1 <- 1 - deviance(fit_lm_1_mdf) / deviance(lm_1_null))
```

    ## [1] 0.8893085

``` r
# Log likelihood
(logLik_1 <-logLik(fit_lm_1_mdf))
```

    ## 'log Lik.' -62353.04 (df=808)

``` r
#or
#(pR2 <- 1- logLik(logitfit.1)/logLik(fit_lm_null))

# AIC
(AIC_1 <- AIC(fit_lm_1_mdf))
```

    ## [1] 126322.1

### prediction

``` r
id <- which(!(evaluation$aspect_ratio %in% levels(train$aspect_ratio)))
evaluation$aspect_ratio[id] <- NA
#pred_1 <- predict(fit_lm_1_mdf,newdata=evaluation[,-which(names(train) %in% c("Index", "movie_title","director_name" , "actor_1_name" ,"actor_2_name" , "actor_3_name" , "plot_keywords",  "movie_imdb_link","cast_total_facebook_likes"))] )

evaluation.new <- evaluation
evaluation.new[which(evaluation.new$aspect_ratio=='1.18'),]$aspect_ratio <- NA
evaluation.new[which(evaluation.new$language %in% c('Dzongkha', 'Filipino')),]$language <- NA
evaluation.new[which(evaluation.new$country %in% c('Dzongkha', 'Filipino')),]$country <- NA
evaluation.new[-which(evaluation.new$genres %in% c("Adventure|Animation|Comedy|Family|Sport"    
, "Biography|Drama|History|Romance"              
, "Adventure|Drama"                              
, "Action|Adventure|Drama|Sci_Fi|Thriller"       
, "Action|Animation|Comedy|Family|Sci_Fi"        
, "Comedy|Horror|Sci_Fi"                         
,"Adventure|Drama|Fantasy|Romance"              
, "Action|Adventure|Fantasy|Sci_Fi"              
, "Action|Adventure|Comedy|Family|Fantasy|Sci_Fi"
, "Adventure|Comedy|Fantasy"                     
, "Action|Adventure|Romance|Sci_Fi|Thriller")),]$genres <- NA

pred_1 <- predict(fit_lm_1_mdf,evaluation.new[,-which(names(evaluation.new) %in% c("Index","gross", "movie_title","director_name" , "actor_1_name" ,"actor_2_name" , "actor_3_name" , "plot_keywords",  "movie_imdb_link","cast_total_facebook_likes"))])
```

    ## Warning in predict.lm(fit_lm_1_mdf, evaluation.new[, -
    ## which(names(evaluation.new) %in% : prediction from a rank-deficient fit may
    ## be misleading

``` r
clsdf_1<-data.frame(cbind(evaluation$Index,evaluation$gross,pred_1))
colnames(clsdf_1)<-c('Index','gross','pred')
clsdf_1[!is.na(clsdf_1$pred),]
```

    ##      Index     gross      pred
    ## 179    179  83024900 139376071
    ## 2605  2605 138795342  92594019
    ## 1158  1158  57750000   9976874
    ## 1065  1065  92001027 -18190851
    ## 694    694  19548064 104536308
    ## 4955  4955    110536 -77859658
    ## 3156  3156  11703287 -26328934
    ## 924    924 296623634 301865703
    ## 2597  2597   7774730 -32432633
    ## 235    235 310675583 407445950
    ## 2133  2133  42660000  -9079887
    ## 4788  4788   1229197  -2636808
    ## 379    379  61355436 -22623777
    ## 2805  2805   2474000 -30706632
    ## 1356  1356  10300000  -1046228
    ## 16      16 291021565 275958106
    ## 1000  1000  26616999 -57427000

``` r
length(clsdf_1[!is.na(clsdf_1$pred),])/849
```

    ## [1] 0.003533569

Multiple Linear Regression II - based on dataset II
---------------------------------------------------

### VIF test faild for the dummy dataset

``` r
suppressMessages(suppressWarnings(library(usdm)))

vif(train_dummy[,-which(names(train_dummy) %in% c(names(ctg_mid),"gross"))])
```

``` r
fit_lm_2 <- lm(gross~.,
                  train_dummy[,-which(names(train_dummy) %in% 
                        c("Index","movie_title","plot_keywords","movie_imdb_link","content_rating",  "color", "language","country","genres","aspect_ratio","director_name","actor_1_name","actor_2_name","actor_3_name"))])


fit_lm_2.sum <- summary(fit_lm_2)

data.frame(fit_lm_2.sum$coef[fit_lm_2.sum$coef[,4] <= .05, ])
```

    ##                              Estimate   Std..Error   t.value      Pr...t..
    ## (Intercept)              2.742317e+08 1.382791e+08  1.983176  4.743100e-02
    ## num_voted_users          7.220259e+01 6.729021e+00 10.730029  2.007113e-26
    ## budget                   4.380149e-02 8.841787e-03  4.953918  7.640960e-07
    ## title_year              -1.337406e+05 6.683453e+04 -2.001071  4.546697e-02
    ## director_facebook_likes -8.061407e+02 1.882712e+02 -4.281806  1.906911e-05
    ## actor_3_facebook_likes  -1.205453e+03 4.318249e+02 -2.791531  5.276271e-03
    ## director_ave_gross       2.638018e-01 1.427532e-02 18.479564  1.286648e-72
    ## actor_1_ave_gross        1.593852e-01 1.509264e-02 10.560457  1.166655e-25
    ## actor_2_ave_gross        3.188490e-01 1.391196e-02 22.919064 5.816603e-108
    ## actor_3_ave_gross        4.386336e-01 1.373677e-02 31.931347 4.399935e-195
    ## actor_1_num             -1.999891e+05 6.276820e+04 -3.186154  1.455301e-03
    ## actor_2_num             -4.271554e+05 2.043329e+05 -2.090488  3.665075e-02
    ## genre.Fantasy           -3.511051e+06 1.567339e+06 -2.240135  2.514883e-02
    ## genre.Family             8.278381e+06 2.457703e+06  3.368341  7.649363e-04
    ## genre.Drama             -2.657662e+06 1.241260e+06 -2.141100  3.233943e-02
    ## genre.History           -5.715232e+06 2.762160e+06 -2.069117  3.861320e-02
    ## color.Color              6.575856e+06 2.693947e+06  2.440974  1.470020e-02
    ## content_ratingApproved  -2.043656e+07 9.450724e+06 -2.162434  3.065675e-02
    ## content_ratingG         -1.202543e+07 6.015377e+06 -1.999114  4.567831e-02
    ## content_ratingR         -1.331142e+07 4.919119e+06 -2.706057  6.843944e-03
    ## aspect1.5               -7.576946e+07 3.019944e+07 -2.508969  1.215622e-02

### outlier test

``` r
outlierTest(fit_lm_2)
```

    ##        rstudent unadjusted p-value Bonferonni p
    ## 1     15.253009         8.3487e-51   2.8202e-47
    ## 1231 -10.126491         9.3809e-24   3.1689e-20
    ## 3077   8.269197         1.9360e-16   6.5399e-13
    ## 160    6.471950         1.1115e-10   3.7547e-07
    ## 30     6.322765         2.9164e-10   9.8517e-07
    ## 79     6.302385         3.3218e-10   1.1221e-06
    ## 1803   6.301958         3.3309e-10   1.1252e-06
    ## 32     5.782128         8.0674e-09   2.7252e-05
    ## 37     5.413641         6.6196e-08   2.2361e-04
    ## 94     5.351600         9.3165e-08   3.1471e-04

``` r
influencePlot(fit_lm_2,col='red',id.n=2)
```

![](DATA621_final_group4_v2_files/figure-markdown_github/unnamed-chunk-30-1.png)

    ##         StudRes        Hat        CookD
    ## 1     15.253009 0.05692752 1.140719e-01
    ## 30     6.322765 0.32274146 1.637159e-01
    ## 124         NaN 1.00000000          NaN
    ## 176    3.461311 0.99995432 2.272967e+03
    ## 464         NaN 1.00000000          NaN
    ## 1231 -10.126491 0.08362767 7.893482e-02

### update the model by removing the outliers and influential points

``` r
fit_lm_2_mdf <- update(fit_lm_2,subset=c(-1-1231-3077-160-30-79-1803-32-37-94-176))
#compareCoefs(fit_lm_2, fit_lm_2_mdf)
```

``` r
fit_lm_2_mdf.sum <- summary(fit_lm_2_mdf)
data.frame(fit_lm_2_mdf.sum$coef[fit_lm_2_mdf.sum$coef[,4] <= .05, ])
```

    ##                              Estimate   Std..Error   t.value      Pr...t..
    ## (Intercept)              2.742317e+08 1.382791e+08  1.983176  4.743100e-02
    ## num_voted_users          7.220259e+01 6.729021e+00 10.730029  2.007113e-26
    ## budget                   4.380149e-02 8.841787e-03  4.953918  7.640960e-07
    ## title_year              -1.337406e+05 6.683453e+04 -2.001071  4.546697e-02
    ## director_facebook_likes -8.061407e+02 1.882712e+02 -4.281806  1.906911e-05
    ## actor_3_facebook_likes  -1.205453e+03 4.318249e+02 -2.791531  5.276271e-03
    ## director_ave_gross       2.638018e-01 1.427532e-02 18.479564  1.286648e-72
    ## actor_1_ave_gross        1.593852e-01 1.509264e-02 10.560457  1.166655e-25
    ## actor_2_ave_gross        3.188490e-01 1.391196e-02 22.919064 5.816603e-108
    ## actor_3_ave_gross        4.386336e-01 1.373677e-02 31.931347 4.399935e-195
    ## actor_1_num             -1.999891e+05 6.276820e+04 -3.186154  1.455301e-03
    ## actor_2_num             -4.271554e+05 2.043329e+05 -2.090488  3.665075e-02
    ## genre.Fantasy           -3.511051e+06 1.567339e+06 -2.240135  2.514883e-02
    ## genre.Family             8.278381e+06 2.457703e+06  3.368341  7.649363e-04
    ## genre.Drama             -2.657662e+06 1.241260e+06 -2.141100  3.233943e-02
    ## genre.History           -5.715232e+06 2.762160e+06 -2.069117  3.861320e-02
    ## color.Color              6.575856e+06 2.693947e+06  2.440974  1.470020e-02
    ## content_ratingApproved  -2.043656e+07 9.450724e+06 -2.162434  3.065675e-02
    ## content_ratingG         -1.202543e+07 6.015377e+06 -1.999114  4.567831e-02
    ## content_ratingR         -1.331142e+07 4.919119e+06 -2.706057  6.843944e-03
    ## aspect1.5               -7.576946e+07 3.019944e+07 -2.508969  1.215622e-02

check multillinearity for model II
----------------------------------

``` r
# extract the variable with significant coefficient
library(broom)
lm_model_coefficients <- tidy(fit_lm_2)
lm_model_coefficients[lm_model_coefficients$p.value<0.05,1]
```

    ##  [1] "(Intercept)"             "num_voted_users"        
    ##  [3] "budget"                  "title_year"             
    ##  [5] "director_facebook_likes" "actor_3_facebook_likes" 
    ##  [7] "director_ave_gross"      "actor_1_ave_gross"      
    ##  [9] "actor_2_ave_gross"       "actor_3_ave_gross"      
    ## [11] "actor_1_num"             "actor_2_num"            
    ## [13] "genre.Fantasy"           "genre.Family"           
    ## [15] "genre.Drama"             "genre.History"          
    ## [17] "color.Color"             "content_ratingApproved" 
    ## [19] "content_ratingG"         "content_ratingR"        
    ## [21] "aspect1.5"

``` r
suppressMessages(suppressWarnings(library(usdm)))

vif(train[,which(names(train) %in%c("num_voted_users", "aspect_ratio1.5" ,"aspect_ratio2", "budget", "director_facebook_likes","actor_1_facebook_likes", "actor_3_facebook_likes","content_ratingR", "color","genres", "director_ave_gross","actor_1_ave_gross", "actor_2_ave_gross", "actor_3_ave_gross","actor_1_num"   ))])
```

    ## Warning in model.response(mf, "numeric"): using type = "numeric" with a
    ## factor response will be ignored

    ## Warning in Ops.factor(y, z$residuals): '-' not meaningful for factors

    ## Warning in Ops.factor(r, 2): '^' not meaningful for factors

    ## Warning in model.response(mf, "numeric"): using type = "numeric" with a
    ## factor response will be ignored

    ## Warning in Ops.factor(y, z$residuals): '-' not meaningful for factors

    ## Warning in Ops.factor(r, 2): '^' not meaningful for factors

    ##                  Variables      VIF
    ## 1          num_voted_users 2.369049
    ## 2                   budget 3.557834
    ## 3  director_facebook_likes 1.344705
    ## 4   actor_1_facebook_likes 1.725678
    ## 5   actor_3_facebook_likes 1.540898
    ## 6                    color       NA
    ## 7                   genres       NA
    ## 8       director_ave_gross 3.225087
    ## 9        actor_1_ave_gross 2.372722
    ## 10       actor_2_ave_gross 3.455505
    ## 11       actor_3_ave_gross 3.738799
    ## 12             actor_1_num 2.018039

### Pseudo R-squared, LogLikelihood, AIC

``` r
lm_2_null <- lm(gross~1,
                  train_dummy[,-which(names(train_dummy) %in% 
                        c("Index","movie_title","plot_keywords","movie_imdb_link","content_rating",  "color", "language","country","genres","aspect_ratio","director_name","actor_1_name","actor_2_name","actor_3_name"))])

# McFadden's pseudo-R^2
(p_R2_2 <- 1 - deviance(fit_lm_2) / deviance(lm_2_null))
```

    ## [1] 0.850665

``` r
# Log likelihood
(logLik_2 <-logLik(fit_lm_2))
```

    ## 'log Lik.' -62861.8 (df=116)

``` r
#or
#(pR2 <- 1- logLik(logitfit.1)/logLik(fit_lm_null))

# AIC
(AIC_2 <- AIC(fit_lm_2))
```

    ## [1] 125955.6

### prediction

``` r
pred_2 <- predict(fit_lm_2_mdf,evaluation_dummy[,-which(names(evaluation_dummy) %in% 
                        c("Index","gross","movie_title","plot_keywords","movie_imdb_link","content_rating",  "color", "language","country","genres","aspect_ratio","director_name","actor_1_name","actor_2_name","actor_3_name"))])
```

    ## Warning in predict.lm(fit_lm_2_mdf, evaluation_dummy[, -
    ## which(names(evaluation_dummy) %in% : prediction from a rank-deficient fit
    ## may be misleading

``` r
clsdf_2<-data.frame(cbind(evaluation_dummy$Index,evaluation_dummy$gross,pred_2))
colnames(clsdf_2)<-c('Index','gross','pred')

ggplot(clsdf_2, aes(x=log(clsdf_2$gross), y = log(clsdf_2$pred))) + 
  geom_point() +
  xlab('log(gross)')+
  ylab('log(pred)')+
  stat_smooth(method = 'lm', aes(colour = 'linear'), se = FALSE,size=1.5) +
  stat_smooth(method = 'lm', formula = y ~ poly(x,2), aes(colour = 'polynomial'), se= FALSE,size=1.5) +
  stat_smooth(method = 'nls', formula = y ~ a * log(x) +b, aes(colour = 'logarithmic'), se = FALSE, start = list(a=1,b=1),size=1.5) +
  stat_smooth(method = 'nls', formula = y ~ a*exp(b *x), aes(colour = 'Exponential'), se = FALSE, start = list(a=1,b=1),size=1.5)+ theme_few(base_size = 20)
```

    ## Warning: Ignoring unknown parameters: start

    ## Warning: Ignoring unknown parameters: start

    ## Warning in log(clsdf_2$pred): NaNs produced

    ## Warning in log(clsdf_2$pred): NaNs produced

    ## Warning in log(clsdf_2$pred): NaNs produced

    ## Warning in log(clsdf_2$pred): NaNs produced

    ## Warning in log(clsdf_2$pred): NaNs produced

    ## Warning in log(clsdf_2$pred): NaNs produced

    ## Warning: Removed 135 rows containing non-finite values (stat_smooth).

    ## Warning: Removed 135 rows containing non-finite values (stat_smooth).

    ## Warning: Removed 135 rows containing non-finite values (stat_smooth).

    ## Warning in (function (formula, data = parent.frame(), start, control = nls.control(), : No starting values specified for some parameters.
    ## Initializing 'a', 'b' to '1.'.
    ## Consider specifying 'start' or using a selfStart model

    ## Warning: Removed 135 rows containing non-finite values (stat_smooth).

    ## Warning in (function (formula, data = parent.frame(), start, control = nls.control(), : No starting values specified for some parameters.
    ## Initializing 'a', 'b' to '1.'.
    ## Consider specifying 'start' or using a selfStart model

    ## Warning: Removed 135 rows containing missing values (geom_point).

![](DATA621_final_group4_v2_files/figure-markdown_github/unnamed-chunk-35-1.png)

Ordinal Logistic Regression (OLR) - Model III
---------------------------------------------

### bin the dependent variable to 10 levels

``` r
#hist(movie_mid$gross)

movie_bin<-movie_mid
movie_bin$Performance <- factor(ifelse(movie_mid$gross > 5*10^8, "10", ifelse(movie_mid$gross > 3*10^8, "9", ifelse(movie_mid$gross > 1*10^8, "8",ifelse(movie_mid$gross > 0.5*10^8, "7",ifelse(movie_mid$gross > 0.25*10^8, "6",ifelse(movie_mid$gross > 0.125*10^8, "5",ifelse(movie_mid$gross > 0.625*10^7,"4",ifelse(movie_mid$gross > 0.3*10^7,"3",ifelse(movie_mid$gross > 0.1*10^7,"2","1"))))))))))
movie_bin$Performance <- ordered(movie_bin$Performance,levels = c("1", "2", "3","4","5","6", "7", "8","9","10"))

#hist(movie_mid[which(movie_mid$gross<0.1*10^7),]$gross)

##########################################################################
# movie_bin_2: bin the DV to 10 bucketsmovie_mid_dummy (without language and country)
##########################################################################

#names(movie_mid_dummy)
#dummy code genre
movie_bin_2 <- movie_bin[,-which(names(movie_bin)%in% c('genres',"aspect_ratio","content_rating","color"))]
movie_bin_2 <- cbind(movie_bin,movie_mid_dummy[,c(38:60)]) #genre 
movie_bin_2 <- cbind(movie_bin_2,movie_mid_dummy[,61]) #color 
movie_bin_2 <- cbind(movie_bin_2,movie_mid_dummy[,c(62:74)]) #content-rating
movie_bin_2 <- cbind(movie_bin_2,movie_mid_dummy[,c(157:173)]) #aspect-ratio
colnames(movie_bin_2)[which(colnames(movie_bin_2)=='movie_mid_dummy[, 61]')] <-'color.Color'

##########################################################################
# movie_bin_3: bin the DV to 10 bucketsmovie_mid_dummy(with language and country)
##########################################################################
movie_bin_3 <- movie_bin_2[,-which(names(movie_bin_2)%in% c("language","country"))]
movie_bin_3 <- cbind(movie_bin_2,movie_mid_dummy[,c(75:115)])  #language
movie_bin_3 <- cbind(movie_bin_3,movie_mid_dummy[,c(116:156)]) #country


sta_buket<- data.frame(table(movie_bin$Performance))
colnames(sta_buket)[1] <- 'Levels'
sta_buket[c(1,3:10,2),]
```

    ##    Levels Freq
    ## 1       1  593
    ## 3       3  297
    ## 4       4  359
    ## 5       5  598
    ## 6       6  819
    ## 7       7  710
    ## 8       8  545
    ## 9       9   58
    ## 10     10    6
    ## 2       2  262

``` r
anyNA(movie_bin_2)
```

    ## [1] FALSE

``` r
anyNA(movie_bin_3)
```

    ## [1] FALSE

split the data
--------------

``` r
####################################################
# movie_bin
####################################################

#split the data for trainning and evaluation:
set.seed(125)
indexes <- sample(1:nrow(movie_bin), size=0.2*nrow(movie_bin))
 
# Split data into two part, train for building model and evaluation for 
evaluation_bin <- movie_bin[indexes,]
train_bin <- movie_bin[-indexes,]

#train.rows <- createDataPartition(y=movie$Index, p=0.8, list = FALSE)   # caret package
#train <- movie[train.rows,]  
#evaluation <-movie[-train.rows,]  
anyNA(train_bin)
```

    ## [1] FALSE

``` r
####################################################
# movie_bin_2
####################################################

set.seed(125)
indexes <- sample(1:nrow(movie_bin), size=0.2*nrow(movie_bin))
 
# Split data into two part, train for building model and evaluation for 
evaluation_bin_2 <- movie_bin_2[indexes,]
train_bin_2 <- movie_bin_2[-indexes,]

#train.rows <- createDataPartition(y=movie$Index, p=0.8, list = FALSE)   # caret package
#train <- movie[train.rows,]  
#evaluation <-movie[-train.rows,]  
anyNA(train_bin_2)
```

    ## [1] FALSE

``` r
####################################################
# movie_bin_3
####################################################
set.seed(125)
indexes <- sample(1:nrow(movie_bin), size=0.2*nrow(movie_bin))
 
# Split data into two part, train for building model and evaluation for 
evaluation_bin_3 <- movie_bin_3[indexes,]
train_bin_3 <- movie_bin_3[-indexes,]

#train.rows <- createDataPartition(y=movie$Index, p=0.8, list = FALSE)   # caret package
#train <- movie[train.rows,]  
#evaluation <-movie[-train.rows,]  
anyNA(train_bin_3)
```

    ## [1] FALSE

``` r
buskets <- as.data.frame(cbind(table(evaluation_bin$Performance),'Class'=seq(1,10),'Range'= c("0~1m", "1m~3m", "3m~6.25m","6.25m~12.5m","12.5m~25m","25m~50m","50m~100m", "100m~300m","300m~500m", "500m+")))
colnames(buskets)[1] <- c('Frequancy')
(buskets<-buskets[,c(2,3,1)])
```

    ##    Class       Range Frequancy
    ## 1      1        0~1m       114
    ## 2      2       1m~3m        56
    ## 3      3    3m~6.25m        63
    ## 4      4 6.25m~12.5m        76
    ## 5      5   12.5m~25m       128
    ## 6      6     25m~50m       141
    ## 7      7    50m~100m       143
    ## 8      8   100m~300m       110
    ## 9      9   300m~500m        15
    ## 10    10       500m+         3

``` r
sum(as.numeric(as.character(buskets[,3])))
```

    ## [1] 849

``` r
suppressMessages(suppressWarnings(library(VGAM)))


dif <- setdiff(names(train_bin_2) , strsplit("Index+movie_title+director_name+actor_1_name+actor_2_name+actor_3_name+plot_keywords+movie_imdb_link+gross+imdb_score+language+country+genres+aspect_ratio+content_rating+genre.News+genre.Short+content_ratingTV_MA+content_ratingTV_PG+content_ratingGP+content_ratingM+content_ratingPassed+content_ratingx+content_ratingpassed+content_ratingNC_17+aspect1.44+aspect1.5+aspect1.75+aspect1.77+aspect2.24+aspect2.4+aspect2.55+aspect2.76+duration+color","\\+")[[1]])

sub_ml <-train_bin_2[,which(names(train_bin_2) %in% c(dif))]
sub_eva_ml <- evaluation_bin_2[,which(names(evaluation_bin_2) %in%c(dif))]


## With nonproportional odds assumptions
fit_vglm <-
 vglm(Performance ~  ., sub_ml, family = cumulative(link = "logit", parallel = FALSE, reverse = TRUE))
```

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in checkwz(wz, M = M, trace = trace, wzepsilon = control
    ## $wzepsilon): 52 diagonal elements of the working weights variable 'wz' have
    ## been replaced by 1.819e-12

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in vglm.fitter(x = x, y = y, w = w, offset = offset, Xm2 = Xm2, :
    ## iterations terminated because half-step sizes are very small

    ## Warning in vglm.fitter(x = x, y = y, w = w, offset = offset, Xm2 =
    ## Xm2, : some quantities such as z, residuals, SEs may be inaccurate due to
    ## convergence at a half-step

    ## Warning in log(prob): NaNs produced

``` r
## With proportional odds assumptions
fit_vglm_p <-
 vglm(Performance ~  ., sub_ml, family = cumulative(link = "logit", parallel = TRUE, reverse = TRUE))
```

    ## Warning in checkwz(wz, M = M, trace = trace, wzepsilon = control
    ## $wzepsilon): 2 diagonal elements of the working weights variable 'wz' have
    ## been replaced by 1.819e-12

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y = y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in checkwz(wz, M = M, trace = trace, wzepsilon = control
    ## $wzepsilon): 10 diagonal elements of the working weights variable 'wz' have
    ## been replaced by 1.819e-12

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y = y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in checkwz(wz, M = M, trace = trace, wzepsilon = control
    ## $wzepsilon): 30 diagonal elements of the working weights variable 'wz' have
    ## been replaced by 1.819e-12

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y = y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in checkwz(wz, M = M, trace = trace, wzepsilon = control
    ## $wzepsilon): 106 diagonal elements of the working weights variable 'wz'
    ## have been replaced by 1.819e-12

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y = y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in checkwz(wz, M = M, trace = trace, wzepsilon = control
    ## $wzepsilon): 139 diagonal elements of the working weights variable 'wz'
    ## have been replaced by 1.819e-12

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y = y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in checkwz(wz, M = M, trace = trace, wzepsilon = control
    ## $wzepsilon): 139 diagonal elements of the working weights variable 'wz'
    ## have been replaced by 1.819e-12

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y = y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in checkwz(wz, M = M, trace = trace, wzepsilon = control
    ## $wzepsilon): 139 diagonal elements of the working weights variable 'wz'
    ## have been replaced by 1.819e-12

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y = y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in checkwz(wz, M = M, trace = trace, wzepsilon = control
    ## $wzepsilon): 139 diagonal elements of the working weights variable 'wz'
    ## have been replaced by 1.819e-12

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y = y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in checkwz(wz, M = M, trace = trace, wzepsilon = control
    ## $wzepsilon): 139 diagonal elements of the working weights variable 'wz'
    ## have been replaced by 1.819e-12

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y = y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in checkwz(wz, M = M, trace = trace, wzepsilon = control
    ## $wzepsilon): 139 diagonal elements of the working weights variable 'wz'
    ## have been replaced by 1.819e-12

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y = y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in checkwz(wz, M = M, trace = trace, wzepsilon = control
    ## $wzepsilon): 139 diagonal elements of the working weights variable 'wz'
    ## have been replaced by 1.819e-12

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y = y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in Deviance.categorical.data.vgam(mu = mu, y = y, w = w, residuals
    ## = residuals, : fitted values close to 0 or 1

    ## Warning in slot(family, "validparams")(eta, y, extra = extra): It seems
    ## that the nonparallelism assumption has resulted in intersecting linear/
    ## additive predictors. Try propodds() or fitting a partial nonproportional
    ## odds model or choosing some other link function, etc.

    ## Warning in vglm.fitter(x = x, y = y, w = w, offset = offset, Xm2 = Xm2, :
    ## iterations terminated because half-step sizes are very small

    ## Warning in vglm.fitter(x = x, y = y, w = w, offset = offset, Xm2 =
    ## Xm2, : some quantities such as z, residuals, SEs may be inaccurate due to
    ## convergence at a half-step

``` r
#depvar(fit_vglm)
#coef(fit_vglm, matrix = TRUE)


#xfit_vglm <- vglm(Performance ~  num_user_for_reviews+budget+director_facebook_likes+actor_1_facebook_likes+actor_2_facebook_likes+actor_3_facebook_likes+cast_total_facebook_likes+movie_facebook_likes+color+director_ave_gross+actor_1_ave_gross+actor_2_ave_gross+actor_3_ave_gross+director_num+actor_1_num+actor_2_num+actor_3_num+genre.Action+genre.Adventure+genre.Fantasy+genre.Thriller+genre.Romance+genre.Animation+genre.Comedy+genre.Family+genre.Musical+genre.Mystery+genre.Western+genre.Drama+genre.History+genre.Sport+genre.Crime+genre.Horror+genre.War+genre.Biography+genre.Music+genre.Documentary+content_ratingApproved+content_ratingG+content_ratingPG+content_ratingR+aspect1.33+aspect1.37+aspect1.44+aspect1.5+aspect1.66+aspect1.85+aspect2+aspect2.2+aspect2.35+aspect2.76, sub_ml, family = cumulative(link = "logit", parallel = FALSE, reverse = TRUE))


#fit_vglm <- vglm(Performance ~  num_voted_users+budget+title_year+director_facebook_likes+actor_1_facebook_likes+actor_2_facebook_likes+actor_3_facebook_likes+cast_total_facebook_likes+movie_facebook_likes+color+director_ave_gross+actor_1_ave_gross+actor_2_ave_gross+actor_3_ave_gross+director_num+actor_1_num+actor_2_num+actor_3_num+genre.Action+genre.Adventure+genre.Fantasy+genre.Sci_Fi+genre.Thriller+genre.Romance+genre.Animation+genre.Comedy+genre.Family+genre.Musical+genre.Mystery+genre.Western+genre.Drama+genre.History+genre.Sport+genre.Crime+genre.Horror+genre.War+genre.Biography+genre.Music+genre.Documentary+genre.News+genre.Short+content_ratingApproved+content_ratingG+content_ratingGP+content_ratingM+content_ratingNC_17+content_ratingPassed+content_ratingPG+content_ratingPG_13+content_ratingR+content_ratingTV_MA+content_ratingTV_PG+content_ratingUnrated+content_ratingX+aspect1.33+aspect1.37+aspect1.44+aspect1.5+aspect1.66+aspect1.75+aspect1.77+aspect1.78+aspect1.85+aspect2+aspect2.2+aspect2.24+aspect2.35+aspect2.39+aspect2.4+aspect2.55+aspect2.76, train_bin_2, family = cumulative(link = "logit", parallel = FALSE, reverse = TRUE))

#fit_vglm <- vglm(Performance ~  num_user_for_reviews+num_critic_for_reviews+imdb_score+num_voted_users+facenumber_in_poster+budget+title_year+director_facebook_likes+duration+actor_1_facebook_likes+actor_2_facebook_likes+actor_3_facebook_likes+cast_total_facebook_likes+movie_facebook_likes+director_ave_gross+actor_1_ave_gross+actor_2_ave_gross+actor_3_ave_gross+director_num+actor_1_num+actor_2_num+actor_3_num+genre.Action+genre.Adventure+genre.Fantasy+genre.Sci_Fi+genre.Thriller+genre.Romance+genre.Animation+genre.Comedy+genre.Family+genre.Musical+genre.Mystery+genre.Western+genre.Drama+genre.History+genre.Sport+genre.Crime+genre.Horror+genre.War+genre.Biography+genre.Music+genre.Documentary+color.Color+content_ratingApproved+content_ratingG+content_ratingPG+content_ratingUnrated+content_ratingPG_13+content_ratingR+content_ratingX+content_ratingNC_17+aspect1.33+aspect1.37+aspect1.66+aspect1.78+aspect1.85+aspect2+aspect2.2+aspect2.35+aspect2.39, train_bin_2, family = cumulative(link = "logit", parallel = FALSE, reverse = TRUE))

#num_coef_vglm <- coef(summary(fit_vglm))[coef(summary(fit_vglm))[,4]<0.05,4]
```

### Check the proportional odds assumption with a LRT

``` r
lrtest(fit_vglm, fit_vglm_p)
```

    ## Likelihood ratio test
    ## 
    ## Model 1: Performance ~ .
    ## Model 2: Performance ~ .
    ##     #Df LogLik  Df Chisq Pr(>Chisq)
    ## 1 30051                            
    ## 2 30515        464

### diagnostic

### Pseudo R-squared, LogLikelihood, AIC

``` r
vglm_null <- vglm(Performance ~  1, sub_ml, family = cumulative(link = "logit", parallel = FALSE, reverse = TRUE))

# Log likelihood
logLik.vglm <- function(x) -x@criterion$deviance/2
logLik_3 <- logLik.vglm(fit_vglm)
logLik_3_null <- logLik.vglm(vglm_null)

# McFadden's pseudo-R^2
(p_R2_3 <- 1 - deviance(fit_vglm) / deviance(vglm_null))
```

    ## [1] -12.94482

``` r
#or
p_R2_3 <- 1- logLik_3/logLik_3_null
```

### predict the gross in test data

``` r
######################################################
#predic with non-proportional odds model
######################################################
pred_3 <- predictvglm(fit_vglm, newdata = sub_eva_ml[,-which(names(sub_eva_ml) %in% 
                                                               c("Performance"))],
            type = "response",
            se.fit = FALSE, deriv = 0, dispersion = NULL,
            untransform = FALSE,
            type.fitted = NULL, percentiles = NULL)

clsdf_3 <- data.frame(cbind('Index'=evaluation_bin_2$Index,'Pred.prob'=apply(pred_3,1,max),'Pred'=colnames(pred_3)[max.col(pred_3,ties.method="first")], sub_eva_ml$Performance))
colnames(clsdf_3)[4]<-'Performance'
clsdf_3$Performance <- ordered(clsdf_3$Performance,levels=c("1", "2", "3","4","5","6", "7", "8","9","10"))

#or 'Pred'=apply(fitted(fit_vglm), 1, which.max) 

suppressMessages(suppressWarnings(library(caret)))
cfmx_3 <- confusionMatrix(data = clsdf_3$Pred, reference = clsdf_3$Performance, positive = "1")
```

    ## Warning in confusionMatrix.default(data = clsdf_3$Pred, reference =
    ## clsdf_3$Performance, : Levels are not in the same order for reference and
    ## data. Refactoring data to match.

``` r
cfmx_3
```

    ## Confusion Matrix and Statistics
    ## 
    ##           Reference
    ## Prediction   1   2   3   4   5   6   7   8   9  10
    ##         1  101  37  29  37  31  19   6   0   0   0
    ##         2    2   0   2   0   0   3   0   0   0   0
    ##         3    0   1   2   2   0   0   0   0   0   0
    ##         4    0   2   2   2   0   1   0   0   0   0
    ##         5    1   4   4   1  15   8   3   0   0   0
    ##         6    6   9  20  21  62  55  31   0   0   0
    ##         7    0   0   0   4  11  27  20   5   0   0
    ##         8    0   0   0   1   1   6  39  23   0   0
    ##         9    0   2   1   0   0   2  17  25   2   0
    ##         10   4   1   3   8   8  20  27  57  13   3
    ## 
    ## Overall Statistics
    ##                                           
    ##                Accuracy : 0.2627          
    ##                  95% CI : (0.2333, 0.2936)
    ##     No Information Rate : 0.1684          
    ##     P-Value [Acc > NIR] : 3.275e-12       
    ##                                           
    ##                   Kappa : 0.1671          
    ##  Mcnemar's Test P-Value : NA              
    ## 
    ## Statistics by Class:
    ## 
    ##                      Class: 1 Class: 2 Class: 3 Class: 4 Class: 5 Class: 6
    ## Sensitivity            0.8860 0.000000 0.031746 0.026316  0.11719  0.39007
    ## Specificity            0.7837 0.991173 0.996183 0.993532  0.97087  0.78955
    ## Pos Pred Value         0.3885 0.000000 0.400000 0.285714  0.41667  0.26961
    ## Neg Pred Value         0.9779 0.933492 0.927725 0.912114  0.86101  0.86667
    ## Prevalence             0.1343 0.065960 0.074205 0.089517  0.15077  0.16608
    ## Detection Rate         0.1190 0.000000 0.002356 0.002356  0.01767  0.06478
    ## Detection Prevalence   0.3062 0.008245 0.005889 0.008245  0.04240  0.24028
    ## Balanced Accuracy      0.8348 0.495586 0.513965 0.509924  0.54403  0.58981
    ##                      Class: 7 Class: 8 Class: 9 Class: 10
    ## Sensitivity           0.13986  0.20909 0.133333  1.000000
    ## Specificity           0.93343  0.93640 0.943645  0.833333
    ## Pos Pred Value        0.29851  0.32857 0.040816  0.020833
    ## Neg Pred Value        0.84271  0.88832 0.983750  1.000000
    ## Prevalence            0.16843  0.12956 0.017668  0.003534
    ## Detection Rate        0.02356  0.02709 0.002356  0.003534
    ## Detection Prevalence  0.07892  0.08245 0.057715  0.169611
    ## Balanced Accuracy     0.53664  0.57275 0.538489  0.916667

``` r
(kappa_3 <- cfmx_3$overall['Kappa'])
```

    ##     Kappa 
    ## 0.1671018

``` r
(acrcy_3 <- cfmx_3$overall['Accuracy'])
```

    ## Accuracy 
    ## 0.262662

``` r
(err_rate_3 <- 1-cfmx_3$overall['Accuracy'])
```

    ## Accuracy 
    ## 0.737338

``` r
data.frame(t(cfmx_3$byClass))
```

    ##                       Class..1    Class..2    Class..3    Class..4
    ## Sensitivity          0.8859649 0.000000000 0.031746032 0.026315789
    ## Specificity          0.7836735 0.991172762 0.996183206 0.993531695
    ## Pos Pred Value       0.3884615 0.000000000 0.400000000 0.285714286
    ## Neg Pred Value       0.9779287 0.933491686 0.927725118 0.912114014
    ## Precision            0.3884615 0.000000000 0.400000000 0.285714286
    ## Recall               0.8859649 0.000000000 0.031746032 0.026315789
    ## F1                   0.5401070         NaN 0.058823529 0.048192771
    ## Prevalence           0.1342756 0.065959953 0.074204947 0.089517079
    ## Detection Rate       0.1189635 0.000000000 0.002355713 0.002355713
    ## Detection Prevalence 0.3062426 0.008244994 0.005889282 0.008244994
    ## Balanced Accuracy    0.8348192 0.495586381 0.513964619 0.509923742
    ##                        Class..5  Class..6   Class..7   Class..8
    ## Sensitivity          0.11718750 0.3900709 0.13986014 0.20909091
    ## Specificity          0.97087379 0.7895480 0.93342776 0.93640054
    ## Pos Pred Value       0.41666667 0.2696078 0.29850746 0.32857143
    ## Neg Pred Value       0.86100861 0.8666667 0.84271100 0.88831836
    ## Precision            0.41666667 0.2696078 0.29850746 0.32857143
    ## Recall               0.11718750 0.3900709 0.13986014 0.20909091
    ## F1                   0.18292683 0.3188406 0.19047619 0.25555556
    ## Prevalence           0.15076561 0.1660777 0.16843345 0.12956419
    ## Detection Rate       0.01766784 0.0647821 0.02355713 0.02709069
    ## Detection Prevalence 0.04240283 0.2402827 0.07891637 0.08244994
    ## Balanced Accuracy    0.54403064 0.5898095 0.53664395 0.57274573
    ##                         Class..9   Class..10
    ## Sensitivity          0.133333333 1.000000000
    ## Specificity          0.943645084 0.833333333
    ## Pos Pred Value       0.040816327 0.020833333
    ## Neg Pred Value       0.983750000 1.000000000
    ## Precision            0.040816327 0.020833333
    ## Recall               0.133333333 1.000000000
    ## F1                   0.062500000 0.040816327
    ## Prevalence           0.017667845 0.003533569
    ## Detection Rate       0.002355713 0.003533569
    ## Detection Prevalence 0.057714959 0.169611307
    ## Balanced Accuracy    0.538489209 0.916666667

Random Forest Model - Model IV
------------------------------

``` r
library(randomForest)
```

    ## Warning: package 'randomForest' was built under R version 3.4.4

    ## randomForest 4.6-14

    ## Type rfNews() to see new features/changes/bug fixes.

    ## 
    ## Attaching package: 'randomForest'

    ## The following object is masked from 'package:dplyr':
    ## 
    ##     combine

    ## The following object is masked from 'package:ggplot2':
    ## 
    ##     margin

    ## The following object is masked from 'package:psych':
    ## 
    ##     outlier

``` r
# randomForest can not handle categorical predictors with more than 53 categories.

########################################################################
#RF model-1
########################################################################
# for "aspect_ratio", "content_rating" , "color",  and "genres",  only use dummy variable
# drop factors with to many levels:"movie_title","director_name","actor_1_name", "actor_2_name","actor_3_name", "plot_keywords" , "movie_imdb_link",  "language" ,"country"
set.seed(100)
fit_rf <- randomForest(Performance ~ . , data = train_bin_2[,-which(names(train_bin_2)%in%c("Index","gross","aspect_ratio", "movie_title","content_rating" , "color", "language" ,"country" , "director_name","actor_1_name", "actor_2_name","actor_3_name",  "genres" , "plot_keywords" , "movie_imdb_link"))] , importance=TRUE, ntree=2000)

varImpPlot(fit_rf)
```

![](DATA621_final_group4_v2_files/figure-markdown_github/unnamed-chunk-44-1.png)

``` r
########################################################################
#RF model-2
########################################################################
# keep "aspect_ratio", "content_rating" , "color", but not use dummy variable
# keep "genres","language" and "country" but in dummy format
# drop factors with to many levels:"movie_title","director_name","actor_1_name", "actor_2_name","actor_3_name", "plot_keywords" , "movie_imdb_link" 
set.seed(101)
sub_rf <- train_bin_3[,-c(62:92)]
fit_rf_2 <- randomForest(Performance ~ . , data = sub_rf[,-which(names(sub_rf)%in%c("Index","gross","movie_title","language", "country" ,"director_name","actor_1_name", "actor_2_name","actor_3_name",  "genres", "plot_keywords" , "movie_imdb_link"))] , importance=TRUE, ntree=2000)

varImpPlot(fit_rf_2)
```

![](DATA621_final_group4_v2_files/figure-markdown_github/unnamed-chunk-44-2.png)

``` r
pred_4 <- predict(fit_rf,evaluation_bin_2[,-which(names(evaluation_bin_2)%in%c("Performance","Index","gross","aspect_ratio", "movie_title","content_rating"  ,"color", "language" ,"country" , "director_name","actor_1_name", "actor_2_name","actor_3_name",      "genres" , "plot_keywords" , "movie_imdb_link"))] , type="response", norm.votes=TRUE, predict.all=FALSE, proximity=FALSE, nodes=FALSE)


clsdf_4 <- data.frame(cbind('Index'=evaluation_bin_2$Index,'Pred'=pred_4, evaluation_bin_2$Performance))
colnames(clsdf_4)[3]<-'Performance'

cfmx_4 <- confusionMatrix(data = clsdf_4$Pred, reference = clsdf_4$Performance, positive = "1")

cfmx_4
```

    ## Confusion Matrix and Statistics
    ## 
    ##           Reference
    ## Prediction   1   2   3   4   5   6   7   8   9  10
    ##         1  103   4   1   1   0   0   0   0   0   0
    ##         2    1  38   2   0   0   0   0   0   0   0
    ##         3    0   3  50   2   0   0   0   0   0   0
    ##         4    0   2   3  36   3   0   0   0   0   0
    ##         5    4   7   4  26  93   5   0   0   0   0
    ##         6    6   2   3   9  26 122  19   0   0   0
    ##         7    0   0   0   2   6  14 112  15   0   0
    ##         8    0   0   0   0   0   0  12  95   8   1
    ##         9    0   0   0   0   0   0   0   0   7   1
    ##         10   0   0   0   0   0   0   0   0   0   1
    ## 
    ## Overall Statistics
    ##                                           
    ##                Accuracy : 0.7739          
    ##                  95% CI : (0.7442, 0.8016)
    ##     No Information Rate : 0.1684          
    ##     P-Value [Acc > NIR] : < 2.2e-16       
    ##                                           
    ##                   Kappa : 0.7375          
    ##  Mcnemar's Test P-Value : NA              
    ## 
    ## Statistics by Class:
    ## 
    ##                      Class: 1 Class: 2 Class: 3 Class: 4 Class: 5 Class: 6
    ## Sensitivity            0.9035  0.67857  0.79365  0.47368   0.7266   0.8652
    ## Specificity            0.9918  0.99622  0.99364  0.98965   0.9362   0.9082
    ## Pos Pred Value         0.9450  0.92683  0.90909  0.81818   0.6691   0.6524
    ## Neg Pred Value         0.9851  0.97772  0.98363  0.95031   0.9507   0.9713
    ## Prevalence             0.1343  0.06596  0.07420  0.08952   0.1508   0.1661
    ## Detection Rate         0.1213  0.04476  0.05889  0.04240   0.1095   0.1437
    ## Detection Prevalence   0.1284  0.04829  0.06478  0.05183   0.1637   0.2203
    ## Balanced Accuracy      0.9477  0.83739  0.89364  0.73167   0.8314   0.8867
    ##                      Class: 7 Class: 8 Class: 9 Class: 10
    ## Sensitivity            0.7832   0.8636 0.466667  0.333333
    ## Specificity            0.9476   0.9716 0.998801  1.000000
    ## Pos Pred Value         0.7517   0.8190 0.875000  1.000000
    ## Neg Pred Value         0.9557   0.9795 0.990488  0.997642
    ## Prevalence             0.1684   0.1296 0.017668  0.003534
    ## Detection Rate         0.1319   0.1119 0.008245  0.001178
    ## Detection Prevalence   0.1755   0.1366 0.009423  0.001178
    ## Balanced Accuracy      0.8654   0.9176 0.732734  0.666667

``` r
(kappa_4 <- cfmx_4$overall['Kappa'])
```

    ##     Kappa 
    ## 0.7374738

``` r
(acrcy_4 <- cfmx_4$overall['Accuracy'])
```

    ##  Accuracy 
    ## 0.7738516

``` r
(err_rate_4 <- 1-cfmx_4$overall['Accuracy'])
```

    ##  Accuracy 
    ## 0.2261484

``` r
data.frame(t(cfmx_4$byClass))
```

    ##                       Class..1   Class..2   Class..3   Class..4  Class..5
    ## Sensitivity          0.9035088 0.67857143 0.79365079 0.47368421 0.7265625
    ## Specificity          0.9918367 0.99621690 0.99363868 0.98965071 0.9361997
    ## Pos Pred Value       0.9449541 0.92682927 0.90909091 0.81818182 0.6690647
    ## Neg Pred Value       0.9851351 0.97772277 0.98362720 0.95031056 0.9507042
    ## Precision            0.9449541 0.92682927 0.90909091 0.81818182 0.6690647
    ## Recall               0.9035088 0.67857143 0.79365079 0.47368421 0.7265625
    ## F1                   0.9237668 0.78350515 0.84745763 0.60000000 0.6966292
    ## Prevalence           0.1342756 0.06595995 0.07420495 0.08951708 0.1507656
    ## Detection Rate       0.1213192 0.04475854 0.05889282 0.04240283 0.1095406
    ## Detection Prevalence 0.1283863 0.04829211 0.06478210 0.05182568 0.1637220
    ## Balanced Accuracy    0.9476728 0.83739416 0.89364474 0.73166746 0.8313811
    ##                       Class..6  Class..7  Class..8    Class..9   Class..10
    ## Sensitivity          0.8652482 0.7832168 0.8636364 0.466666667 0.333333333
    ## Specificity          0.9081921 0.9475921 0.9715832 0.998800959 1.000000000
    ## Pos Pred Value       0.6524064 0.7516779 0.8189655 0.875000000 1.000000000
    ## Neg Pred Value       0.9712991 0.9557143 0.9795362 0.990487515 0.997641509
    ## Precision            0.6524064 0.7516779 0.8189655 0.875000000 1.000000000
    ## Recall               0.8652482 0.7832168 0.8636364 0.466666667 0.333333333
    ## F1                   0.7439024 0.7671233 0.8407080 0.608695652 0.500000000
    ## Prevalence           0.1660777 0.1684335 0.1295642 0.017667845 0.003533569
    ## Detection Rate       0.1436985 0.1319199 0.1118963 0.008244994 0.001177856
    ## Detection Prevalence 0.2202591 0.1755006 0.1366313 0.009422850 0.001177856
    ## Balanced Accuracy    0.8867202 0.8654044 0.9176098 0.732733813 0.666666667

``` r
########################################################################
#RF model-2
########################################################################
sub_rf <- evaluation_bin_3[,-c(62:92)]
pred_rf2 <- predict(fit_rf_2,sub_rf[,-which(names(sub_rf)%in%c("Performance","Index","gross","movie_title", "language", "country" ,"director_name","actor_1_name", "actor_2_name","actor_3_name",  "genres", "plot_keywords" , "movie_imdb_link"))] , type="response", norm.votes=TRUE, predict.all=FALSE, proximity=FALSE, nodes=FALSE)

clsdf_rf2 <- data.frame(cbind('Index'=evaluation_bin_2$Index,'Pred'=pred_rf2, evaluation_bin_2$Performance))
colnames(clsdf_rf2)[3]<-'Performance'

cfmx_rf2 <- confusionMatrix(data = clsdf_rf2$Pred, reference = clsdf_rf2$Performance, positive = "1")

cfmx_rf2
```

    ## Confusion Matrix and Statistics
    ## 
    ##           Reference
    ## Prediction   1   2   3   4   5   6   7   8   9  10
    ##         1  103   5   1   1   0   0   0   0   0   0
    ##         2    1  36   2   0   0   0   0   0   0   0
    ##         3    0   3  48   1   0   0   0   0   0   0
    ##         4    0   1   3  34   2   0   0   0   0   0
    ##         5    4   8   6  26  89   5   0   0   0   0
    ##         6    5   3   3  11  32 122  22   0   0   0
    ##         7    1   0   0   3   5  14 107  16   0   0
    ##         8    0   0   0   0   0   0  14  94  11   1
    ##         9    0   0   0   0   0   0   0   0   4   1
    ##         10   0   0   0   0   0   0   0   0   0   1
    ## 
    ## Overall Statistics
    ##                                          
    ##                Accuracy : 0.7515         
    ##                  95% CI : (0.721, 0.7802)
    ##     No Information Rate : 0.1684         
    ##     P-Value [Acc > NIR] : < 2.2e-16      
    ##                                          
    ##                   Kappa : 0.7111         
    ##  Mcnemar's Test P-Value : NA             
    ## 
    ## Statistics by Class:
    ## 
    ##                      Class: 1 Class: 2 Class: 3 Class: 4 Class: 5 Class: 6
    ## Sensitivity            0.9035  0.64286  0.76190  0.44737   0.6953   0.8652
    ## Specificity            0.9905  0.99622  0.99491  0.99224   0.9320   0.8927
    ## Pos Pred Value         0.9364  0.92308  0.92308  0.85000   0.6449   0.6162
    ## Neg Pred Value         0.9851  0.97531  0.98118  0.94808   0.9451   0.9708
    ## Prevalence             0.1343  0.06596  0.07420  0.08952   0.1508   0.1661
    ## Detection Rate         0.1213  0.04240  0.05654  0.04005   0.1048   0.1437
    ## Detection Prevalence   0.1296  0.04594  0.06125  0.04711   0.1625   0.2332
    ## Balanced Accuracy      0.9470  0.81954  0.87841  0.71980   0.8137   0.8790
    ##                      Class: 7 Class: 8 Class: 9 Class: 10
    ## Sensitivity            0.7483   0.8545 0.266667  0.333333
    ## Specificity            0.9448   0.9648 0.998801  1.000000
    ## Pos Pred Value         0.7329   0.7833 0.800000  1.000000
    ## Neg Pred Value         0.9488   0.9781 0.986967  0.997642
    ## Prevalence             0.1684   0.1296 0.017668  0.003534
    ## Detection Rate         0.1260   0.1107 0.004711  0.001178
    ## Detection Prevalence   0.1720   0.1413 0.005889  0.001178
    ## Balanced Accuracy      0.8465   0.9097 0.632734  0.666667

``` r
(kappa_rf2 <- cfmx_rf2$overall['Kappa'])
```

    ##     Kappa 
    ## 0.7110768

``` r
(acrcy_rf2 <- cfmx_rf2$overall['Accuracy'])
```

    ##  Accuracy 
    ## 0.7514723

``` r
(err_rate_rf2 <- 1-cfmx_rf2$overall['Accuracy'])
```

    ##  Accuracy 
    ## 0.2485277

``` r
data.frame(t(cfmx_rf2$byClass))
```

    ##                       Class..1   Class..2   Class..3   Class..4  Class..5
    ## Sensitivity          0.9035088 0.64285714 0.76190476 0.44736842 0.6953125
    ## Specificity          0.9904762 0.99621690 0.99491094 0.99223803 0.9320388
    ## Pos Pred Value       0.9363636 0.92307692 0.92307692 0.85000000 0.6449275
    ## Neg Pred Value       0.9851150 0.97530864 0.98117942 0.94808405 0.9451477
    ## Precision            0.9363636 0.92307692 0.92307692 0.85000000 0.6449275
    ## Recall               0.9035088 0.64285714 0.76190476 0.44736842 0.6953125
    ## F1                   0.9196429 0.75789474 0.83478261 0.58620690 0.6691729
    ## Prevalence           0.1342756 0.06595995 0.07420495 0.08951708 0.1507656
    ## Detection Rate       0.1213192 0.04240283 0.05653710 0.04004711 0.1048292
    ## Detection Prevalence 0.1295642 0.04593640 0.06124853 0.04711425 0.1625442
    ## Balanced Accuracy    0.9469925 0.81953702 0.87840785 0.71980323 0.8136757
    ##                       Class..6  Class..7  Class..8    Class..9   Class..10
    ## Sensitivity          0.8652482 0.7482517 0.8545455 0.266666667 0.333333333
    ## Specificity          0.8926554 0.9447592 0.9648173 0.998800959 1.000000000
    ## Pos Pred Value       0.6161616 0.7328767 0.7833333 0.800000000 1.000000000
    ## Neg Pred Value       0.9708141 0.9487909 0.9780521 0.986966825 0.997641509
    ## Precision            0.6161616 0.7328767 0.7833333 0.800000000 1.000000000
    ## Recall               0.8652482 0.7482517 0.8545455 0.266666667 0.333333333
    ## F1                   0.7197640 0.7404844 0.8173913 0.400000000 0.500000000
    ## Prevalence           0.1660777 0.1684335 0.1295642 0.017667845 0.003533569
    ## Detection Rate       0.1436985 0.1260306 0.1107185 0.004711425 0.001177856
    ## Detection Prevalence 0.2332155 0.1719670 0.1413428 0.005889282 0.001177856
    ## Balanced Accuracy    0.8789518 0.8465055 0.9096814 0.632733813 0.666666667

Generalized Linear Models with L1 Penalty - Model V
---------------------------------------------------

``` r
sub <- train_bin_2[,-which(names(train_bin_2) %in% c("Index","gross","movie_title","director_name", "actor_1_name","actor_2_name", "actor_3_name", "plot_keywords","movie_imdb_link", "color", "language", "country","genres","content_rating","aspect_ratio"))]

library(glmpathcr)
```

    ## Warning: package 'glmpathcr' was built under R version 3.4.4

    ## Loading required package: glmpath

    ## Warning: package 'glmpath' was built under R version 3.4.4

``` r
x <- sub[, -which(names(sub)%in% c("Performance"))]
y <- sub$Performance

fit_glmpath <- glmpathcr(x,y)

BIC.step <- model.select(fit_glmpath, which = "BIC")
AIC.step <- model.select(fit_glmpath, which = "AIC")
coefficients<-coef(fit_glmpath, s=AIC.step)
nonzero.coef(fit_glmpath, s=AIC.step)
```

    ##                 Intercept      num_user_for_reviews 
    ##              5.326597e+01              2.197342e-04 
    ##    num_critic_for_reviews                imdb_score 
    ##              4.331548e-03             -1.276490e-01 
    ##           num_voted_users      facenumber_in_poster 
    ##              3.234294e-06              1.088422e-02 
    ##                  duration                    budget 
    ##              7.540256e-03              5.526017e-11 
    ##                title_year   director_facebook_likes 
    ##             -3.228145e-02             -4.931502e-05 
    ##    actor_1_facebook_likes    actor_2_facebook_likes 
    ##              6.774942e-06              2.898404e-07 
    ##    actor_3_facebook_likes cast_total_facebook_likes 
    ##             -4.046599e-05             -1.322444e-05 
    ##      movie_facebook_likes        director_ave_gross 
    ##             -8.644571e-06              1.570583e-08 
    ##         actor_1_ave_gross         actor_2_ave_gross 
    ##              8.360939e-09              1.571561e-08 
    ##         actor_3_ave_gross              director_num 
    ##              2.452833e-08              9.009791e-03 
    ##               actor_2_num               actor_3_num 
    ##             -4.644004e-02             -9.806932e-02 
    ##              genre.Action           genre.Adventure 
    ##              2.890595e-01             -5.995831e-02 
    ##             genre.Fantasy              genre.Sci_Fi 
    ##             -2.362776e-01             -2.433909e-01 
    ##            genre.Thriller             genre.Romance 
    ##              1.575178e-01              1.734597e-01 
    ##           genre.Animation              genre.Comedy 
    ##              1.286542e-01              5.892585e-02 
    ##              genre.Family             genre.Musical 
    ##              6.751232e-01             -2.952141e-01 
    ##             genre.Mystery             genre.Western 
    ##              1.975672e-01             -1.280554e-01 
    ##               genre.Drama             genre.History 
    ##             -2.565208e-01             -2.585655e-01 
    ##               genre.Sport               genre.Crime 
    ##              3.403801e-01             -9.160139e-02 
    ##              genre.Horror                 genre.War 
    ##              1.874011e-01             -7.105685e-03 
    ##           genre.Biography               genre.Music 
    ##             -1.998613e-02              4.617875e-01 
    ##         genre.Documentary                genre.News 
    ##              4.538491e-01              8.167509e-01 
    ##               genre.Short               color.Color 
    ##             -2.648930e+00              6.697075e-01 
    ##    content_ratingApproved           content_ratingG 
    ##              2.952433e-01              9.220431e-01 
    ##           content_ratingM      content_ratingPassed 
    ##              1.396728e-01             -5.794635e-01 
    ##          content_ratingPG       content_ratingPG_13 
    ##              1.040213e+00              1.221243e+00 
    ##           content_ratingR       content_ratingTV_MA 
    ##              6.461700e-01             -2.322000e+00 
    ##       content_ratingTV_PG     content_ratingUnrated 
    ##              2.197310e+00             -2.575508e-01 
    ##           content_ratingX                aspect1.33 
    ##             -1.826001e-01             -2.445470e-01 
    ##                aspect1.37                aspect1.44 
    ##             -3.386284e-01              1.081537e+00 
    ##                 aspect1.5                aspect1.66 
    ##             -2.445761e+00              1.271932e-01 
    ##                aspect1.75                aspect1.77 
    ##             -4.018608e+00             -3.152183e-01 
    ##                aspect1.78                aspect1.85 
    ##              2.017220e-02              8.448194e-02 
    ##                   aspect2                 aspect2.2 
    ##             -7.898016e-02             -1.018742e+00 
    ##                aspect2.39                 aspect2.4 
    ##             -2.268378e-01              1.085280e+00 
    ##                aspect2.55                aspect2.76 
    ##             -1.700711e+00             -3.018783e-01 
    ##                       cp1                       cp2 
    ##              8.322670e+00              7.779721e+00 
    ##                       cp3                       cp4 
    ##              7.345358e+00              7.169964e+00 
    ##                       cp5                       cp6 
    ##              6.475499e+00              4.882875e+00 
    ##                       cp7                       cp8 
    ##              2.517785e+00             -6.236148e+00 
    ##                       cp9 
    ##             -1.655555e+01

``` r
pred <- predict(fit_glmpath,which = "AIC", type = "class")
confusion <- table(pred, y)

accuracy_glmpath <- data.frame('class'=rownames(table(pred, y)),'accuracy'=rep(0,length(unique(pred))))
for(i in 1:length(unique(pred))){
  ac <- table(pred, y)[i,which(rownames(table(pred, y))[i]==colnames(table(pred, y)))]/sum(table(pred, y)[i,])
  accuracy_glmpath[i,2]<-ac
}
accuracy_glmpath
```

    ##   class  accuracy
    ## 1     1 0.4880096
    ## 2    10 0.6666667
    ## 3     5 0.3676056
    ## 4     6 0.5065359
    ## 5     7 0.5813953
    ## 6     8 0.8042328
    ## 7     9 0.5897436

``` r
sub_ev <- evaluation_bin_2[,-which(names(evaluation_bin_2) %in% c("Index","gross","movie_title","director_name", "actor_1_name","actor_2_name", "actor_3_name", "plot_keywords","movie_imdb_link", "color", "language", "country","genres","content_rating","aspect_ratio"))]
```

### prediction

``` r
pred_5 <- predict(fit_glmpath,newx = sub_ev[,-which(names(sub_ev)=='Performance')], which = "AIC", type = "class")
clsdf_5 <- as.data.frame(cbind(evaluation_bin_2$Index,sub_ev$Performance,pred_5))
colnames(clsdf_5) <- c('Index','Performance',"Pred" )
clsdf_5$Pred <- ordered(clsdf_5$Pred,levels(clsdf_5$Pred)[c(1,3:7,2)])
clsdf_5$Performance <- ordered(clsdf_5$Performance,levels=c("1", "2", "3","4","5","6", "7", "8","9","10"))

cfmx_5_v1 <- table(pred_5, sub_ev$Performance)
#or
cfmx_5 <- confusionMatrix(data = clsdf_5$Pred, reference = clsdf_5$Performance, positive = "1")
```

    ## Warning in levels(reference) != levels(data): longer object length is not a
    ## multiple of shorter object length

    ## Warning in confusionMatrix.default(data = clsdf_5$Pred, reference =
    ## clsdf_5$Performance, : Levels are not in the same order for reference and
    ## data. Refactoring data to match.

``` r
cfmx_5
```

    ## Confusion Matrix and Statistics
    ## 
    ##           Reference
    ## Prediction  1  2  3  4  5  6  7  8  9 10
    ##         1  95 39 35 25 10  1  0  0  0  0
    ##         2   0  0  0  0  0  0  0  0  0  0
    ##         3   0  0  0  0  0  0  0  0  0  0
    ##         4   0  0  0  0  0  0  0  0  0  0
    ##         5  15 14 23 33 63 23  3  0  0  0
    ##         6   3  2  5 16 50 96 47  4  0  0
    ##         7   1  1  0  1  5 15 78 28  0  0
    ##         8   0  0  0  1  0  6 15 77  4  0
    ##         9   0  0  0  0  0  0  0  1 10  1
    ##         10  0  0  0  0  0  0  0  0  1  2
    ## 
    ## Overall Statistics
    ##                                           
    ##                Accuracy : 0.4959          
    ##                  95% CI : (0.4617, 0.5301)
    ##     No Information Rate : 0.1684          
    ##     P-Value [Acc > NIR] : < 2.2e-16       
    ##                                           
    ##                   Kappa : 0.4079          
    ##  Mcnemar's Test P-Value : NA              
    ## 
    ## Statistics by Class:
    ## 
    ##                      Class: 1 Class: 2 Class: 3 Class: 4 Class: 5 Class: 6
    ## Sensitivity            0.8333  0.00000   0.0000  0.00000   0.4922   0.6809
    ## Specificity            0.8503  1.00000   1.0000  1.00000   0.8460   0.8206
    ## Pos Pred Value         0.4634      NaN      NaN      NaN   0.3621   0.4305
    ## Neg Pred Value         0.9705  0.93404   0.9258  0.91048   0.9037   0.9281
    ## Prevalence             0.1343  0.06596   0.0742  0.08952   0.1508   0.1661
    ## Detection Rate         0.1119  0.00000   0.0000  0.00000   0.0742   0.1131
    ## Detection Prevalence   0.2415  0.00000   0.0000  0.00000   0.2049   0.2627
    ## Balanced Accuracy      0.8418  0.50000   0.5000  0.50000   0.6691   0.7507
    ##                      Class: 7 Class: 8 Class: 9 Class: 10
    ## Sensitivity           0.54545  0.70000  0.66667  0.666667
    ## Specificity           0.92776  0.96482  0.99760  0.998818
    ## Pos Pred Value        0.60465  0.74757  0.83333  0.666667
    ## Neg Pred Value        0.90972  0.95576  0.99403  0.998818
    ## Prevalence            0.16843  0.12956  0.01767  0.003534
    ## Detection Rate        0.09187  0.09069  0.01178  0.002356
    ## Detection Prevalence  0.15194  0.12132  0.01413  0.003534
    ## Balanced Accuracy     0.73661  0.83241  0.83213  0.832742

``` r
(kappa_5 <- cfmx_5$overall['Kappa'])
```

    ##     Kappa 
    ## 0.4079479

``` r
(acrcy_5 <- cfmx_5$overall['Accuracy'])
```

    ##  Accuracy 
    ## 0.4958775

``` r
(err_rate_5 <- 1-cfmx_5$overall['Accuracy'])
```

    ##  Accuracy 
    ## 0.5041225

``` r
data.frame(t(cfmx_5$byClass))
```

    ##                       Class..1   Class..2   Class..3   Class..4   Class..5
    ## Sensitivity          0.8333333 0.00000000 0.00000000 0.00000000 0.49218750
    ## Specificity          0.8503401 1.00000000 1.00000000 1.00000000 0.84604716
    ## Pos Pred Value       0.4634146        NaN        NaN        NaN 0.36206897
    ## Neg Pred Value       0.9704969 0.93404005 0.92579505 0.91048292 0.90370370
    ## Precision            0.4634146         NA         NA         NA 0.36206897
    ## Recall               0.8333333 0.00000000 0.00000000 0.00000000 0.49218750
    ## F1                   0.5956113         NA         NA         NA 0.41721854
    ## Prevalence           0.1342756 0.06595995 0.07420495 0.08951708 0.15076561
    ## Detection Rate       0.1118963 0.00000000 0.00000000 0.00000000 0.07420495
    ## Detection Prevalence 0.2414605 0.00000000 0.00000000 0.00000000 0.20494700
    ## Balanced Accuracy    0.8418367 0.50000000 0.50000000 0.50000000 0.66911733
    ##                       Class..6   Class..7   Class..8   Class..9
    ## Sensitivity          0.6808511 0.54545455 0.70000000 0.66666667
    ## Specificity          0.8206215 0.92776204 0.96481732 0.99760192
    ## Pos Pred Value       0.4304933 0.60465116 0.74757282 0.83333333
    ## Neg Pred Value       0.9281150 0.90972222 0.95576408 0.99402628
    ## Precision            0.4304933 0.60465116 0.74757282 0.83333333
    ## Recall               0.6808511 0.54545455 0.70000000 0.66666667
    ## F1                   0.5274725 0.57352941 0.72300469 0.74074074
    ## Prevalence           0.1660777 0.16843345 0.12956419 0.01766784
    ## Detection Rate       0.1130742 0.09187279 0.09069494 0.01177856
    ## Detection Prevalence 0.2626620 0.15194346 0.12131920 0.01413428
    ## Balanced Accuracy    0.7507363 0.73660829 0.83240866 0.83213429
    ##                        Class..10
    ## Sensitivity          0.666666667
    ## Specificity          0.998817967
    ## Pos Pred Value       0.666666667
    ## Neg Pred Value       0.998817967
    ## Precision            0.666666667
    ## Recall               0.666666667
    ## F1                   0.666666667
    ## Prevalence           0.003533569
    ## Detection Rate       0.002355713
    ## Detection Prevalence 0.003533569
    ## Balanced Accuracy    0.832742317

``` r
accuracy_glmpath_ev <- data.frame('class'=rownames(table(pred_5, sub_ev$Performance)),'accuracy'=rep(0,length(unique(pred_5))))
for(i in 1:length(unique(pred_5))){
  ac <- cfmx_5_v1[i,which(rownames(cfmx_5_v1)[i]==colnames(cfmx_5_v1))]/sum(cfmx_5_v1[i,])
  accuracy_glmpath_ev[i,2]<-ac
}

accuracy_glmpath_ev
```

    ##   class  accuracy
    ## 1     1 0.4634146
    ## 2    10 0.6666667
    ## 3     5 0.3620690
    ## 4     6 0.4304933
    ## 5     7 0.6046512
    ## 6     8 0.7475728
    ## 7     9 0.8333333

Classification Trees - Model VI
-------------------------------

``` r
library(rpartScore)
```

    ## Warning: package 'rpartScore' was built under R version 3.4.4

    ## Loading required package: rpart

``` r
#use the same subset as random forest model 1
train_bin_2_rp <- train_bin_2
train_bin_2_rp$Performance <- as.numeric(as.character(train_bin_2_rp$Performance))
fit_rpartScore <- rpartScore(Performance ~ ., data = train_bin_2_rp[,-which(names(train_bin_2_rp)%in%c("Index","gross","aspect_ratio", "movie_title","content_rating"  ,"color", "language" ,"country" , "director_name","actor_1_name", "actor_2_name","actor_3_name",      "genres" , "plot_keywords" , "movie_imdb_link"))],split = "quad", prune = "mr")

#fit_rpart <- rpart (fit_rpartScore)

plotcp(fit_rpartScore)
```

![](DATA621_final_group4_v2_files/figure-markdown_github/unnamed-chunk-48-1.png)

``` r
#fit_rpartScore.pruned<-prune(fit_rpartScore,cp=0.02)
#par(mar=c(0.5,0,0.5,0))
#plot(fit_rpartScore.pruned)
#text(fit_rpartScore.pruned,cex=0.8,srt = 0)

#T.quad.mr <- rpartScore(Performance ~ ., data = train_bin_2_rp[,-which(names(train_bin_2_rp)%in%c("Index","gross","aspect_ratio", "movie_title","content_rating" , "color", "language" ,"country" , "director_name","actor_1_name", "actor_2_name","actor_3_name",  "genres" , "plot_keywords" , "movie_imdb_link"))], split = "quad", prune = "mr")

xerror.min.pos <- which.min(fit_rpartScore$cptable[, 4])
th.1std.rule <- fit_rpartScore$cptable[xerror.min.pos, 4] +
fit_rpartScore$cptable[xerror.min.pos, 5]
best.1std.rule <- which.max(fit_rpartScore$cptable[, 4] < th.1std.rule)
best.1std.rule.cp <- fit_rpartScore$cptable[best.1std.rule, 1]
fit_rpartScore.best <- prune(fit_rpartScore, cp = best.1std.rule.cp)

par(mar=c(0.5,0,0.5,0))
plot(fit_rpartScore.best)
text(fit_rpartScore.best,cex=0.8,srt = 0)
```

![](DATA621_final_group4_v2_files/figure-markdown_github/unnamed-chunk-48-2.png)

### prediction

``` r
pred_6  <- predict(fit_rpartScore.best, newdata = evaluation_bin_2[,-which(names(evaluation_bin_2)%in%c("Performance","Index","gross","aspect_ratio", "movie_title","content_rating"  ,"color", "language" ,"country" , "director_name","actor_1_name", "actor_2_name","actor_3_name", "genres" , "plot_keywords" , "movie_imdb_link"))])

#pred_7  <- predict(fit_rpart , newdata = evaluation_bin_2[,-which(names(evaluation_bin_2)%in%c("Performance","Index","gross","aspect_ratio", "movie_title","content_rating"  ,"color", "language" ,"country" , "director_name","actor_1_name", "actor_2_name","actor_3_name", "genres" , "plot_keywords" , "movie_imdb_link"))],type='class')

clsdf_6 <- as.data.frame(cbind(evaluation_bin_2$Index,evaluation_bin_2$Performance,pred_6))
colnames(clsdf_6) <- c('Index','Performance',"Pred" )
cfmx_6_v1 <- table(pred_6, sub_ev$Performance)
#or
cfmx_6 <- confusionMatrix(data = clsdf_6$Pred, reference = clsdf_6$Performance, positive = "1")
```

    ## Warning in levels(reference) != levels(data): longer object length is not a
    ## multiple of shorter object length

    ## Warning in confusionMatrix.default(data = clsdf_6$Pred, reference =
    ## clsdf_6$Performance, : Levels are not in the same order for reference and
    ## data. Refactoring data to match.

``` r
cfmx_6
```

    ## Confusion Matrix and Statistics
    ## 
    ##           Reference
    ## Prediction   1   2   3   4   5   6   7   8   9  10
    ##         1  104  10   2   1   2   0   0   0   0   0
    ##         2    1  34   1   0   0   0   0   0   0   0
    ##         3    0   3  45   4   1   0   0   0   0   0
    ##         4    0   2   3  45   9   0   0   0   0   0
    ##         5    5   4   9  12  68  31   4   0   0   0
    ##         6    4   3   2   9  38  90  35   3   0   0
    ##         7    0   0   1   5  10  20  96  52   0   0
    ##         8    0   0   0   0   0   0   8  55  15   3
    ##         9    0   0   0   0   0   0   0   0   0   0
    ##         10   0   0   0   0   0   0   0   0   0   0
    ## 
    ## Overall Statistics
    ##                                          
    ##                Accuracy : 0.6325         
    ##                  95% CI : (0.5991, 0.665)
    ##     No Information Rate : 0.1684         
    ##     P-Value [Acc > NIR] : < 2.2e-16      
    ##                                          
    ##                   Kappa : 0.5722         
    ##  Mcnemar's Test P-Value : NA             
    ## 
    ## Statistics by Class:
    ## 
    ##                      Class: 1 Class: 2 Class: 3 Class: 4 Class: 5 Class: 6
    ## Sensitivity            0.9123  0.60714  0.71429  0.59211  0.53125   0.6383
    ## Specificity            0.9796  0.99748  0.98982  0.98189  0.90985   0.8672
    ## Pos Pred Value         0.8739  0.94444  0.84906  0.76271  0.51128   0.4891
    ## Neg Pred Value         0.9863  0.97294  0.97739  0.96076  0.91620   0.9233
    ## Prevalence             0.1343  0.06596  0.07420  0.08952  0.15077   0.1661
    ## Detection Rate         0.1225  0.04005  0.05300  0.05300  0.08009   0.1060
    ## Detection Prevalence   0.1402  0.04240  0.06243  0.06949  0.15665   0.2167
    ## Balanced Accuracy      0.9459  0.80231  0.85205  0.78700  0.72055   0.7528
    ##                      Class: 7 Class: 8 Class: 9 Class: 10
    ## Sensitivity            0.6713  0.50000  0.00000  0.000000
    ## Specificity            0.8754  0.96482  1.00000  1.000000
    ## Pos Pred Value         0.5217  0.67901      NaN       NaN
    ## Neg Pred Value         0.9293  0.92839  0.98233  0.996466
    ## Prevalence             0.1684  0.12956  0.01767  0.003534
    ## Detection Rate         0.1131  0.06478  0.00000  0.000000
    ## Detection Prevalence   0.2167  0.09541  0.00000  0.000000
    ## Balanced Accuracy      0.7733  0.73241  0.50000  0.500000

``` r
(kappa_6 <- cfmx_6$overall['Kappa'])
```

    ##     Kappa 
    ## 0.5722134

``` r
(acrcy_6 <- cfmx_6$overall['Accuracy'])
```

    ##  Accuracy 
    ## 0.6325088

``` r
(err_rate_6 <- 1-cfmx_6$overall['Accuracy'])
```

    ##  Accuracy 
    ## 0.3674912

``` r
data.frame(t(cfmx_6$byClass))
```

    ##                       Class..1   Class..2   Class..3   Class..4   Class..5
    ## Sensitivity          0.9122807 0.60714286 0.71428571 0.59210526 0.53125000
    ## Specificity          0.9795918 0.99747793 0.98982188 0.98188875 0.90984743
    ## Pos Pred Value       0.8739496 0.94444444 0.84905660 0.76271186 0.51127820
    ## Neg Pred Value       0.9863014 0.97293973 0.97738693 0.96075949 0.91620112
    ## Precision            0.8739496 0.94444444 0.84905660 0.76271186 0.51127820
    ## Recall               0.9122807 0.60714286 0.71428571 0.59210526 0.53125000
    ## F1                   0.8927039 0.73913043 0.77586207 0.66666667 0.52107280
    ## Prevalence           0.1342756 0.06595995 0.07420495 0.08951708 0.15076561
    ## Detection Rate       0.1224971 0.04004711 0.05300353 0.05300353 0.08009423
    ## Detection Prevalence 0.1401649 0.04240283 0.06242638 0.06949352 0.15665489
    ## Balanced Accuracy    0.9459363 0.80231039 0.85205380 0.78699700 0.72054872
    ##                       Class..6  Class..7   Class..8   Class..9   Class..10
    ## Sensitivity          0.6382979 0.6713287 0.50000000 0.00000000 0.000000000
    ## Specificity          0.8672316 0.8753541 0.96481732 1.00000000 1.000000000
    ## Pos Pred Value       0.4891304 0.5217391 0.67901235        NaN         NaN
    ## Neg Pred Value       0.9233083 0.9293233 0.92838542 0.98233216 0.996466431
    ## Precision            0.4891304 0.5217391 0.67901235         NA          NA
    ## Recall               0.6382979 0.6713287 0.50000000 0.00000000 0.000000000
    ## F1                   0.5538462 0.5871560 0.57591623         NA          NA
    ## Prevalence           0.1660777 0.1684335 0.12956419 0.01766784 0.003533569
    ## Detection Rate       0.1060071 0.1130742 0.06478210 0.00000000 0.000000000
    ## Detection Prevalence 0.2167256 0.2167256 0.09540636 0.00000000 0.000000000
    ## Balanced Accuracy    0.7527648 0.7733414 0.73240866 0.50000000 0.500000000

``` r
library("ggfortify")


#library(caret)
#plot(cfmx_4$table)
#plot(cfmx_5$table)

# Generated palette with rich rainbow and dark (12, s = 0.6, v = 0.75)
richcolor <- c("#000041", "#FF3300", "#0081FF",  "#80FE1A", "#FDEE02", "#FFAB00" )
clsdf_2 <- as.data.frame(clsdf_2)
qplot(x=log(clsdf_2$gross), y = log(clsdf_2$pred),main='Multiple Linear Regression', data=clsdf_2,xlab='log(gross)',ylab='log(pred)') +
  geom_smooth(method = "glm", formula = y~x, family = gaussian(link = 'log'))+
  theme_few(base_size = 20)
```

    ## Warning: Ignoring unknown parameters: family

    ## Warning in log(clsdf_2$pred): NaNs produced

    ## Warning in log(clsdf_2$pred): NaNs produced

    ## Warning in log(clsdf_2$pred): NaNs produced

    ## Warning: Removed 135 rows containing non-finite values (stat_smooth).

    ## Warning: Removed 135 rows containing missing values (geom_point).

![](DATA621_final_group4_v2_files/figure-markdown_github/unnamed-chunk-50-1.png)

``` r
clsdf_3_plot <- clsdf_3
par(cex.axis=1, cex.lab=2, cex.main=3, cex.sub=3)
qplot( Performance,  Pred, data=clsdf_3_plot,  colour= Performance, geom = c("boxplot", "jitter"), main = "Ordinal Logistic Regression", xlab = "Observed Classe", ylab = "Predicted Classe",scale_color_manual(values = richcolor)) + theme_grey(base_size = 16.5) 
```

    ## Warning: Ignoring unknown parameters: NA

    ## Warning: Ignoring unknown parameters: NA

![](DATA621_final_group4_v2_files/figure-markdown_github/unnamed-chunk-50-2.png)

``` r
clsdf_4_plot <- clsdf_4
clsdf_4_plot[,2] <- ordered(clsdf_4_plot[,2],levels=c("1", "2", "3","4","5","6", "7", "8","9","10"))
clsdf_4_plot[,3] <- ordered(clsdf_4_plot[,3],levels=c("1", "2", "3","4","5","6", "7", "8","9","10"))
qplot( Performance,  Pred, data=clsdf_4_plot,  colour= Performance, geom = c("boxplot", "jitter"), main = "Random Forest", xlab = "Observed Classe", ylab = "Predicted Classe",scale_color_manual(values = richcolor))+ theme_grey(base_size = 16.5)
```

    ## Warning: Ignoring unknown parameters: NA

    ## Warning: Ignoring unknown parameters: NA

![](DATA621_final_group4_v2_files/figure-markdown_github/unnamed-chunk-50-3.png)

``` r
clsdf_rf2_plot <- clsdf_rf2
clsdf_rf2_plot[,2] <- ordered(clsdf_rf2_plot[,2],levels=c("1", "2", "3","4","5","6", "7", "8","9","10"))
clsdf_rf2_plot[,3] <- ordered(clsdf_rf2_plot[,3],levels=c("1", "2", "3","4","5","6", "7", "8","9","10"))
qplot( Performance,  Pred, data=clsdf_rf2_plot,  colour= Performance, geom = c("boxplot", "jitter"), main = "Random Forest_based on dummy variable", xlab = "Observed Classe", ylab = "Predicted Classe",scale_color_manual(values = richcolor))+ theme_grey(base_size = 16.5)
```

    ## Warning: Ignoring unknown parameters: NA

    ## Warning: Ignoring unknown parameters: NA

![](DATA621_final_group4_v2_files/figure-markdown_github/unnamed-chunk-50-4.png)

``` r
clsdf_5_plot <- clsdf_5
qplot( Performance,  Pred, data=clsdf_5_plot,  colour= Performance, geom = c("boxplot", "jitter"), main = "Generalized Linear Models with L1 Penalty", xlab = "Observed Classe", ylab = "Predicted Classe",scale_color_manual(values = richcolor))+ theme_grey(base_size = 16.5)
```

    ## Warning: Ignoring unknown parameters: NA

    ## Warning: Ignoring unknown parameters: NA

![](DATA621_final_group4_v2_files/figure-markdown_github/unnamed-chunk-50-5.png)

``` r
clsdf_6_plot <- clsdf_6
clsdf_6_plot[,2] <- ordered(clsdf_6_plot[,2],levels=c("1", "2", "3","4","5","6", "7", "8","9","10"))
clsdf_6_plot[,3] <- ordered(clsdf_6_plot[,3],levels=c("1", "2", "3","4","5","6", "7", "8","9","10"))
qplot( Performance,  Pred, data=clsdf_6_plot,  colour= Performance, geom = c("boxplot", "jitter"), main = "Classification Trees", xlab = "Observed Classe", ylab = "Predicted Classe",scale_color_manual(values = richcolor))+ theme_grey(base_size = 16.5)
```

    ## Warning: Ignoring unknown parameters: NA

    ## Warning: Ignoring unknown parameters: NA

![](DATA621_final_group4_v2_files/figure-markdown_github/unnamed-chunk-50-6.png)

``` r
table(clsdf_3$Performance)
```

    ## 
    ##   1   2   3   4   5   6   7   8   9  10 
    ## 114  56  63  76 128 141 143 110  15   3

``` r
# or
#ggplot(as.data.frame(cfmx_4$table)) +
#  geom_tile(aes(x=Prediction, y=Reference, fill=Freq)) + scale_x_discrete(name="Actual Class") + scale_y_discrete(name="Predicted Class") + scale_fill_gradient(breaks=seq(from=-.5, to=4, by=.2)) + labs(fill="Normalized\nFrequency") 
```

metrics
-------

``` r
R2 <- format(c(p_R2_1,p_R2_2))
Loglik <- format(c(logLik_1,logLik_2))
AICs<-format(c(AIC_1,AIC_2))
data.frame(cbind('Model'=c('Multiple Linear Regression','Multiple Linear Regression - dummy encode predictor'),'R_squared'=R2, 'LogLike'=Loglik,'AIC'=AICs,'variable'=c(53,21)))
```

    ##                                                 Model R_squared   LogLike
    ## 1                          Multiple Linear Regression 0.8893085 -62353.04
    ## 2 Multiple Linear Regression - dummy encode predictor 0.8506650 -62861.80
    ##        AIC variable
    ## 1 126322.1       53
    ## 2 125955.6       21

``` r
acc <- format(c(acrcy_3,acrcy_4,acrcy_5,acrcy_6))
err <- format(c(err_rate_3,err_rate_4,err_rate_5,err_rate_6))
kapa <- format(c(kappa_3,kappa_4,kappa_5,kappa_6))
data.frame(cbind('Ordinal Classification Model'=c('Ordinal Logistic Regression (OLR)','Random Forest','Generalized Linear Models with L1 Penalty',' Classification Trees'),'Accuracy'=acc, 'Error.Rate'=err,'Kappa'=kapa))
```

    ## Warning in data.row.names(row.names, rowsi, i): some row.names duplicated:
    ## 2,3,4 --> row.names NOT used

    ##                Ordinal.Classification.Model  Accuracy Error.Rate     Kappa
    ## 1         Ordinal Logistic Regression (OLR) 0.2626620  0.7373380 0.1671018
    ## 2                             Random Forest 0.7738516  0.2261484 0.7374738
    ## 3 Generalized Linear Models with L1 Penalty 0.4958775  0.5041225 0.4079479
    ## 4                      Classification Trees 0.6325088  0.3674912 0.5722134

``` r
length(pred_1[!is.na(pred_1)])
```

    ## [1] 17
