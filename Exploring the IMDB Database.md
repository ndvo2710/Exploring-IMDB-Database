# EXPLORING IMDB DATA
## Kevin Vo


### Importing database and exploring database

* In order to work with database, we need to use **DBI** and **RSQLite** library

~~~r
library(DBI)
library("RSQLite")
~~~

* Create a connection to a DBMS of **imdb_data**

~~~r
drv = dbDriver("SQLite")
con = dbConnect(SQLite(), "~/Dropbox/Fall 2015/STAT 141/Assignment5/imdb_data")
~~~

* Let's list all the remote temporary tables under connection **con**

~~~r
> dbListTables(con)
 [1] "acted_in"        "actors"          "aka_names"       "aka_titles"
 [5] "genres"          "keywords"        "movies"          "movies_genres"
 [9] "movies_keywords" "series"          "sqlite_sequence"
~~~

* Let's list field names of all the above table:

~~~r
> dbListFields(con, "acted_in")
[1] "idacted_in"       "idmovies"         "idseries"         "idactors"
[5] "character"        "billing_position"
~~~

~~~r
> dbListFields(con, "actors")
[1] "idactors" "lname"    "fname"    "mname"    "number"   "gender"
~~~

~~~r
> dbListFields(con, "aka_names")
[1] "idaka_names" "idactors"    "name"
~~~

~~~r
> dbListFields(con, "aka_titles")
[1] "idaka_titles" "idmovies"     "title"        "location"     "year"
~~~

~~~r
> dbListFields(con, "genres")
[1] "idgenres" "genre"
~~~


~~~r
> dbListFields(con, "keywords")
[1] "idkeywords" "keyword"
~~~


~~~r
> dbListFields(con, "movies")
[1] "idmovies" "title"    "year"     "type"     "number"   "location" "language"
~~~

~~~r
> dbListFields(con, "movies_genres")
[1] "idmovies_genres" "idmovies"        "idgenres"
~~~


~~~r
> dbListFields(con, "movies_keywords")
[1] "idmovies_keywords" "idmovies"          "idseries"
[4] "idkeywords"
~~~


~~~r
> dbListFields(con, "series")
[1] "idseries" "idmovies" "name"     "season"   "number"
~~~

--------------------------------------------------------------------------------------------------

### _**Question 1:**_ How many actors are there in the database? How many movies?

* Total number of actors and actresses are **3500167**

~~~r
> dbListFields(con, "actors")
[1] "idactors" "lname"    "fname"    "mname"    "number"   "gender"
> query = "SELECT COUNT(*) FROM actors;"
> dbGetQuery(con, query)
  COUNT(*)
1  3500167
~~~


* Total number of movies is **1298737**

~~~r
> dbListFields(con, "movies")
[1] "idmovies" "title"    "year"     "type"     "number"   "location" "language"
> query = "SELECT COUNT(*) FROM movies;"
> dbGetQuery(con, query)
  COUNT(*)
1  1298737
~~~

***Recheck:***

~~~r
> query = "SELECT COUNT(DISTINCT idmovies) AS count FROM movies;"
> dbGetQuery(con, query)
    count
1 1298737
~~~



--------------------------------------------------------------------------------------------------

### _**Question 2:**_ What time period does the database cover?

~~~r
> dbListFields(con,"movies")
[1] "idmovies" "title"    "year"     "type"     "number"   "location" "language"
> dbListFields(con,"aka_titles")
[1] "idaka_titles" "idmovies"     "title"        "location"     "year"
~~~

* We see that **year** field appears in both **movies** and **aka_titles** table. Let's find the time period based on these two tables:

~~~r
> dbGetQuery(con, "SELECT MIN(year), MAX(year) FROM movies;")
  MIN(year) MAX(year)
1         1      2025
> dbGetQuery(con, "SELECT MIN(year), MAX(year) FROM aka_titles;")
  MIN(year) MAX(year)
1      1924      2018
~~~

* It seems that the **aka_titles** table generate the result much more accurate than **movies** table. However, we should take a further look at **movies** table:

~~~r
> query = "SELECT COUNT(*) FROM aka_titles;"
> dbGetQuery(con, query)
  COUNT(*)
1    27251
~~~

* The above result shows that there are *at most 27251 movies* in this table. However, the result we get from the question 1 is much larger as **1298737**. Therefore we can conclude that the **aka_titles** table cannot be used to get the time period that the database covers.

* After going back to **movies** table, we realize the problem we got is the ***min year*** as ***1***, which is impossible since the movies era are only around 19th or 20th century. There are something wrong in this data.

~~~r
> query = "SELECT COUNT(*) as lessthan100 FROM movies WHERE year < 100;"
> dbGetQuery(con, query)
  lessthan100
1         141
> query = "SELECT COUNT(*) as lessthan1850 FROM movies WHERE year < 1850;"
> dbGetQuery(con, query)
  lessthan1850
1          157
~~~

**=>** There are **141** movies which is ealier than the year of **100th** and there are **157** movies which is earlier than the year of **1850th**. As usual, we should fix the data. However, **141** is so large that we could fix the year for 1-by-1. Therefore, I will fix a few of them as a demonstration and remove the rest out of the data. 

* Fix one wrong entry which the movie title as *El sexo ataca* and idmovies as *392967*. The year was shown as *3*. And we change it with *1979* based on this following information:

<http://www.imdb.com/title/tt0073687/?ref_=fn_al_tt_1>

~~~r
> query = "SELECT * FROM movies WHERE year < 2 LIMIT 3;"
> dbGetQuery(con, query)
  idmovies                  title year type number location language
1    95492 Franklin and Friends I    1    3     NA     <NA>     <NA>
2   138079           Tout va bien    1    3     NA     <NA>     <NA>
3   392967          El sexo ataca    1    3     NA     <NA>     <NA>
> query = "UPDATE movies SET year = '1979' WHERE idmovies = '392967';"
> dbGetQuery(con, query)
> query = "SELECT * FROM movies WHERE idmovies = '392967';"
> dbGetQuery(con, query)
  idmovies         title year type number location language
1   392967 El sexo ataca 1979    3     NA     <NA>     <NA>
~~~

* Now we delete all the entries which have year smaller than 1850

~~~r
> query = "DELETE FROM movies WHERE year < 1850;"
> dbGetQuery(con, query)
> query = "SELECT COUNT(*) FROM movies;"
> # Let's confirm the number of deletion as 157
> dbGetQuery(con, query)
  COUNT(*)
1  1298581
> 1298737 - 1298581 +1 
[1] 157
~~~

* So the period that the database cover is **_from 1865 to 2025_**

~~~r
> dbGetQuery(con, "SELECT MIN(year), MAX(year) FROM movies;")
  MIN(year) MAX(year)
1      1865      2025
~~~


--------------------------------------------------------------------------------------------------





### _**Question 3:**_ What proportion of the actors are female? male?

* Let's explore the **actors** table to get the number of actors and actresses separately

~~~r
> query = "SELECT  * FROM actors LIMIT 6;"
> dbGetQuery(con, query)
  idactors         lname                  fname mname number gender
1        1     ""Steff"" Stefanie Oxmann Mcgaha  <NA>     NA     NA
2        2          <NA>               $haniqua  <NA>     NA     NA
3        3      & Ashour               Lucienne  <NA>     NA     NA
4        4     & Company     Monica Bill Barnes  <NA>     NA     NA
5        5 &Aacutelvarez               Michelle  <NA>     NA     NA
6        6          <NA>             '67 Impala  <NA>     NA     NA
~~~

**=>** After looking at 6 rows of the **actors** table, we see that the gender is all **NA**. So we need to find all the distinct value in **gender** field in **actors** table

~~~r
> query = "SELECT DISTINCT gender as gender FROM actors;"
> dbGetQuery(con, query)
  gender
1     NA
2      1
~~~

* The **gender** field contains all values in either **NA** or **1**. Let's check a few value of first name **fname** with **gender = 1** or with **gender = NA**

~~~r
> query = "SELECT fname, gender FROM actors WHERE gender = 1 LIMIT 6;"
> dbGetQuery(con, query)
    fname gender
1    Claw      1
2    Homo      1
3   Steve      1
4     Too      1
5 Bee Moe      1
6    Yung      1
~~~

~~~r
> query = "SELECT fname, gender FROM actors WHERE gender IS NULL LIMIT 6"
> dbGetQuery(con, query)
                   fname gender
1 Stefanie Oxmann Mcgaha     NA
2               $haniqua     NA
3               Lucienne     NA
4     Monica Bill Barnes     NA
5               Michelle     NA
6             '67 Impala     NA
~~~
 
**=>** Even though we are not quite sure that all of the actors with **gender = 1** are **Male** and all of the actors with **gender is NULL (or gender = NA)** are female, we are quite confident that **gender = 1** will represent for **Male/ Actor** and **gender is NULL** will represent for **Female/Actress**, based on the above two dataframes of fname and gender.

* Now, we count the number of actors grouped by **gender**

~~~r
> query = "SELECT COUNT(*) AS count, gender AS gender FROM actors GROUP BY gender;"
> gender.prop = dbGetQuery(con, query)
gender.prop$gender = c("female", "male")
> gender.prop$gender = c("female", "male")
> gender.prop$count = gender.prop$count*100/sum(gender.prop$count)
> gender.prop
     count gender
1 35.37034 female
2 64.62966   male
~~~

**=>** The _proportion of actors are female is **35.37%**_ and the _proportion of actor are male is **64.6%**_








--------------------------------------------------------------------------------------------------

### _**Question 4:**_ What proportion of the entries in the movies table are actual movies and what proportion are television series, etc.?

* To find out which entries in the movies table are actual movies and which entries are TV series, we need to take a look at table **movies** and table **series**

~~~r
> query = "SELECT * FROM movies LIMIT 3;"
> dbGetQuery(con, query)
  idmovies                                          title year type number location language
1        1                            Night of the Demons 2009    3     NA     <NA>     <NA>
2        2 The Bad Lieutenant: Port of Call - New Orleans 2009    3     NA     <NA>     <NA>
3        3                                 Please Like Me 2013    3     NA     <NA>     <NA>
> query = "SELECT * FROM series LIMIT 3;"
> dbGetQuery(con, query)
  idseries idmovies                name season number
1        1        3     All You Can Eat      1      4
2        2        3        French Toast      1      2
3        3        3 Horrible Sandwiches      1      6
~~~

* Even though **idmovies** are used as a **field** in both tables, we clearly see that there is no connection betweeen **movies.idmovies** and **series.idmovies** (By looking at both table when **idmovies = 3**). _**Question:**_ Does the intersection between **movies.title** and **series.name** identify all the TV series?

~~~r
> query = "SELECT movies.title AS movie, series.name AS series
+ FROM movies
+ INNER JOIN series
+ ON movies.title = series.name LIMIT 5;"
> dbGetQuery(con, query)
                movie              series
1 Night of the Demons Night of the Demons
2    Around the World    Around the World
3    Around the World    Around the World
4    Around the World    Around the World
5    Around the World    Around the World
~~~

**=>** The above outcome looks good to our prediction. However, there are some repeating entries in the result. One possible solution for this is using **DISTINCT**. Before doing that, let's take a look at those repeating entries

~~~r
> query = "SELECT * FROM series WHERE name = 'Night of the Demons';"
> dbGetQuery(con, query)
  idseries idmovies                name season number
1   165418   193826 Night of the Demons      1      1
> query = "SELECT * FROM series WHERE name = 'Around the World';"
> dbGetQuery(con, query)
   idseries idmovies             name season number
1    109476    94939 Around the World      6      8
2    117431   144424 Around the World      1      2
.....
20  1114877    30800 Around the World     50      5
~~~

~~~r
> query = "SELECT * FROM movies WHERE title = 'Night of the Demons';"
> dbGetQuery(con, query)
  idmovies               title year type number location language
1        1 Night of the Demons 2009    3     NA     <NA>     <NA>
2   182971 Night of the Demons 1988    3     NA     <NA>     <NA>
> query = "SELECT * FROM movies WHERE title = 'Around the World';"
> dbGetQuery(con, query)
  idmovies            title year type number location language
1        5 Around the World 1943    3     NA     <NA>     <NA>
2    74254 Around the World 1967    3     NA     <NA>     <NA>
.....
6  1051530 Around the World 1931    3     NA     <NA>     <NA>
7  1051531 Around the World 2014    3     NA     <NA>     <NA>
~~~

 * Another problem arises because we have **1 of "Night of the Demons"** in **series** table, but **2 of them** in the **movies** table; and **20 of "Around the World"** in **series** but **7 of them** in **movies**. So we should try to solve this by different approach:

 	+ **data1** include **movies.title** and **count** of each group of movie title
 	+ **data2** include **series.name** and **count** of each group of series name
 	+ return the **intersection(data1.tittle,data2.name)** and **min** value of data1.count and data2.count

 ~~~r
 > query = "SELECT SUM(min) AS TotalTVSeries FROM 
+ (SELECT data1.title AS movie, MIN(data1.count, data2.count) as min
+     FROM (SELECT title,COUNT(*) AS count FROM movies GROUP BY title) AS data1
+     INNER JOIN (SELECT name,COUNT(*) AS count FROM series GROUP BY name) AS data2
+     ON data1.title = data2.name);"
> movie.prop = dbGetQuery(con, query)[[1]]
> query = "SELECT COUNT(*) AS Total FROM movies;"
> movie.prop = c(movie.prop,dbGetQuery(con, query)[[1]])
> names(movie.prop)= c("TV","Movies")
> movie.prop[2] = movie.prop[2] - movie.prop[1]
> movie.prop = movie.prop*100/sum(movie.prop)
> movie.prop
      TV   Movies
12.04445 87.95555
 ~~~

* **Conclusion:** The proportion of the entries in movies table are actual **movies** is **87.96%**. And the proportion of the entries in movies table are **TV series** is **12.04%** 

* Run some checks:

~~~r
> query = "SELECT data1.title AS movie, MIN(data1.count, data2.count)
+     FROM (SELECT title,COUNT(*) AS count FROM movies GROUP BY title) AS data1
+     INNER JOIN (SELECT name,COUNT(*) AS count FROM series GROUP BY name) AS data2
+     ON data1.title = data2.name
+     WHERE data1.title = 'Night of the Demons';"
> dbGetQuery(con, query)
                movie MIN(data1.count, data2.count)
1 Night of the Demons                             1
~~~

~~~r
> query = "SELECT data1.title AS movie, MIN(data1.count, data2.count)
+     FROM (SELECT title,COUNT(*) AS count FROM movies GROUP BY title) AS data1
+     INNER JOIN (SELECT name,COUNT(*) AS count FROM series GROUP BY name) AS data2
+     ON data1.title = data2.name
+     WHERE data1.title = 'Around the World';"
> dbGetQuery(con, query)
                movie MIN(data1.count, data2.count)
1    Around the World                             7
~~~

--------------------------------------------------------------------------------------------------

### _**Question 5:**_ How many genres are there? What are their names/descriptions?

* We have **32 genres**.

~~~r
> query = "SELECT COUNT(DISTINCT genre) AS NumberOfGenres FROM genres;"
> dbGetQuery(con, query)
  NumberOfGenres
1             32
~~~

* They are **Documentary , Reality , Horror , Drama , Comedy , Musical , Talk , Mystery , News , Sport , Sci , Romance , Family , Short , Biography , Music , Game , Adventure , Crime , War , Fantasy , Thriller , Animation , Action , History , Adult , Western , Lifestyle , Film , Experimental , Commercial , Erotica**

~~~r
> query = "SELECT DISTINCT genre FROM genres;"
> dbGetQuery(con, query)[[1]]
 [1] "Documentary"  "Reality" "Horror"       "Drama"        "Comedy"       "Musical"
 [7] "Talk"         "Mystery" "News"         "Sport"        "Sci"          "Romance"
[13] "Family"       "Short"   "Biography"    "Music"        "Game"         "Adventure"
[19] "Crime"        "War"     "Fantasy"      "Thriller"     "Animation"    "Action"
[25] "History"      "Adult"   "Western"      "Lifestyle"    "Film"         "Experimental"
[31] "Commercial"   "Erotica"
~~~






--------------------------------------------------------------------------------------------------

### _**Question 6:**_ List the 10 most common genres of movies, showing the number of movies in each of these genres.

* 10 most common genres of movies are **Comedy, Drama, Documentary, Reality, Talk, Animation, Music, Romance, Game**

~~~r
> query = "SELECT genre, COUNT(*) FROM (SELECT movies_genres.idmovies, genres.genre
+     FROM movies_genres
+     INNER JOIN genres
+     ON movies_genres.idgenres = genres.idgenres) GROUP BY genre;"
> genre = dbGetQuery(con, query)
> genre[order(genre[,2], decreasing = TRUE),][1:10,]
         genre COUNT(*)
6       Comedy    28152
9        Drama    20149
8  Documentary    14934
20     Reality    10360
10      Family     8915
25        Talk     7949
4    Animation     6797
16       Music     5222
21     Romance     4679
12        Game     4367
~~~





--------------------------------------------------------------------------------------------------

### _**Question 7:**_ Find all movies with the keyword 'space'. How many are there? What are the years these were released? and who were the top 5 actors in each of these movies?

* There are **1786** movies containining the keyword **'space'**. We list 10 of them as a demostration

~~~r
> query = "SELECT COUNT(*) AS count FROM movies
+ WHERE title LIKE '%space%';"
> dbGetQuery(con, query)
  count
1  1786
~~~ 

~~~r
> query = "SELECT title FROM movies
+ WHERE title LIKE '%space%'
+ LIMIT 10;"
> dbGetQuery(con, query)
                                           title
1                                   Space Specks
2                                         Spaces
3        Video Classroom Lesson 1: Space and Sea
4                              The Space Between
5                                          Space
6                            Alien Space Avenger
7                     Star Trek: Deep Space Nine
8  The Last Earth Girl Went to Space to Find God
9                                      Big Space
10                                         Space
~~~

* The **year** were released for these movies

~~~r
> query = "SELECT DISTINCT year
+          FROM movies
+          WHERE title LIKE '%space%';"
> dbGetQuery(con, query)[[1]]
 [1] 2003 2013 2006 2010 1985 1989 1993 2015 2009 2001 1988 2012 2002 1999 1994
[16] 2014 2008 2005 2011 1972 1992 1998 1997 1996 2007 1968 1966 1975 1965 1958
[31] 1990 2016 2000 2004 1995   NA 1981 1956 1950 2017 1979 1987 1974 1978 1991
[46] 1959 1982 1986 1960 1926 1953 1983 1954 1984 1969 1971 1970 1955 1963 1980
[61] 1977 1957 1964 1951 1961 1967 1924 1973 1976 1962 1931 1947 2018 1911 1933
[76] 1925 1927
~~~

--------------------------------------------------------------------------------------------------

### _**Question 8:**_ Has the number of movies in each genre changed over time? Plot the overall number of movies in each year over time, and for each genre.

* Since we have **32 different genres**, we can generate 32 dataframe contatining all the year and number of movies for each genre. It requires a lot of work and the report would be too long for just one question. And the purpose of this assignment is learning and practicing SQL query in R. So, I think I will demonstrate only two genres: **Documentary and Reality**

* With **Documentary**, the structure of **SQL query** as followed:

	+ **movies_genres** table and **genres** table has 1 inner join as **idgenres**. Based on that inner join, we create a table of **movies_genres.idmovies and genres.genre**. And this table is called as **data**
	+ **data** table has inner join **idmovies** with **movies** table. Based on that inner join, we create a table of **genre, year and COUNT(\*)** GROUP BY **year** WHERE **genre is Documentary**
	
~~~r
> query = "SELECT genre,year,COUNT(*) FROM (SELECT data.genre, movies.year
+          FROM (SELECT movies_genres.idmovies, genres.genre
+                FROM movies_genres
+                INNER JOIN genres
+                ON movies_genres.idgenres = genres.idgenres) AS data
+          INNER JOIN movies
+          ON data.idmovies = movies.idmovies)
+          WHERE genre = 'Documentary'
+          GROUP BY year;"
> documentary = dbGetQuery(con, query)
> documentary = documentary[-1,]
~~~

~~~r
> documentary
         genre year COUNT(*)
2  Documentary 1931        1
3  Documentary 1937        4
4  Documentary 1938        1
....
75 Documentary 2015      596
76 Documentary 2016       35
77 Documentary 2017        4
~~~

* Since there are 77 groups of year for this dataframe. In order to see the change in number of **Documentary** movies by **year**, we need to have a plot

~~~r
> plot(x= documentary[,2], y= documentary[,3],
+     main = "Number of Documentary Movies by Year",
+     xlab = "Year", ylab= "Number of Movies")
~~~

![](https://dl.dropboxusercontent.com/u/27868566/screenshot%2066.png)

* **_Similarly_**, the change in number of **Reality** movies by year is:

~~~r
> query = "SELECT genre,year,COUNT(*) FROM (SELECT data.genre, movies.year
+          FROM (SELECT movies_genres.idmovies, genres.genre
+                FROM movies_genres
+                INNER JOIN genres
+                ON movies_genres.idgenres = genres.idgenres) AS data
+          INNER JOIN movies
+          ON data.idmovies = movies.idmovies)
+          WHERE genre = 'Reality'
+          GROUP BY year;"
> reality = dbGetQuery(con, query)
> reality = reality[-1,]
~~~

~~~r
> plot(x= reality[,2], y= reality[,3],
+     main = "Number of Reality Movies by Year",
+     xlab = "Year", ylab= "Number of Movies")
~~~

![](https://dl.dropboxusercontent.com/u/27868566/screenshot%2067.png)

* **Conclusion:** Bassed on the above two plots, we are confident to draw a conclusion that **_the number of movies of each genre has changed over time_**


* Draw 32 plots:

~~~r
> query = "SELECT DISTINCT genre FROM genres;"
> genre.name=dbGetQuery(con, query)[[1]]
> query1 = "SELECT genre,year,COUNT(*) FROM (SELECT data.genre, movies.year
FROM (SELECT movies_genres.idmovies, genres.genre FROM movies_genres
INNER JOIN genres ON movies_genres.idgenres = genres.idgenres)
AS data INNER JOIN movies ON data.idmovies = movies.idmovies) WHERE genre = '"
> query2 = "'GROUP BY year;"
> query.list = sapply(1:32, function(i)
paste0(query1,genre.name[i],query2, collapse=""))
~~~

~~~r
> par(mfrow = c(3,3))
> par(mfrow = c(3,3))
> sapply(1:9, function(i){
+ temp = dbGetQuery(con, query.list[i])
+ temp = temp[-1,]
+ plot(x= temp[,2], y= temp[,3],
+ main = paste0("Number of ",genre.name[i], " Movies by Year", collapse = ""),
+ xlab = "Year", ylab= "Number of Movies")
+ })
~~~

![](https://dl.dropboxusercontent.com/u/27868566/screenshot%2068.png)

~~~r
> par(mfrow = c(3,3))
>
> sapply(10:18, function(i){
+     temp = dbGetQuery(con, query.list[i])
+     temp = temp[-1,]
+     plot(x= temp[,2], y= temp[,3],
+     main = paste0("Number of ",genre.name[i], " Movies by Year", collapse = ""),
+     xlab = "Year", ylab= "Number of Movies")})
~~~

![](https://dl.dropboxusercontent.com/u/27868566/screenshot%2069.png)

~~~r
> par(mfrow = c(3,3))
> sapply(19:27, function(i){
+     temp = dbGetQuery(con, query.list[i])
+     temp = temp[-1,]
+     plot(x= temp[,2], y= temp[,3],
+     main = paste0("Number of ",genre.name[i], " Movies by Year", collapse = ""),
+     xlab = "Year", ylab= "Number of Movies")})
~~~

![](https://dl.dropboxusercontent.com/u/27868566/screenshot%2070.png)





--------------------------------------------------------------------------------------------------

### _**Question 9:**_ Who are the actors that have been in the most movies? List the top 20.

* **Alex Trebek** has been in the most movies( nealy 7259 movies including series). Here are the top 20 actors

~~~r
>famous_actors = dbReadTable(con,"acted_in")
>actor_names = dbReadTable(con, "actors")
>top20_actors = sort(table(famous_actors$idactors), decreasing= TRUE)[1:20]
>top20_actors= cbind(idactors = as.numeric(names(top20_actors)), top20_actors)
> top20_actors = cbind(top20_actors,full_names= sapply(1:20,function(i) 
paste0(actor_names[actor_names[,1]== top20_actors[i,1],3]," ",
actor_names[actor_names[,1]== top20_actors[i,1],2])))
~~~

~~~r
> top20_actors
        idactors  top20_actors full_names
3284305 "3284305" "7259"       "Alex Trebek"
1963709 "1963709" "7233"       "Johnny Gilbert"
1363105 "1363105" "6898"       "Bob Barker"
3007716 "3007716" "6278"       "Pat Sajak"
1191284 "1191284" "6219"       "Vanna White"
1164869 "1164869" "5740"       "Carol Vorderman"
863140  "863140"  "5540"       "Janice Pennington"
2401656 "2401656" "5352"       "Jay Leno"
~~~

~~~r
2745295 "2745295" "4971"       "Johnny Olson"
2407136 "2407136" "4740"       "David Letterman"
3413201 "3413201" "4605"       "Richard Whiteley"
2727248 "2727248" "4389"       "O'Donnell, Charlie NA"
3400320 "3400320" "4272"       "Frank Welker"
3079126 "3079126" "4215"       "Paul Shaffer"
2724638 "2724638" "4040"       "O'Brien, Conan NA"
2955614 "2955614" "4013"       "Rod Roddy"
509347  "509347"  "3994"       "Helena Isabel"
1832366 "1832366" "3914"       "Lus Esparteiro"
1569250 "1569250" "3803"       "Manuel Cavaco"
614277  "614277"  "3759"       "Katherine Kelly Lang"
~~~

--------------------------------------------------------------------------------------------------


### _**Question 10:**_ Who are the actors that have had the most number of movies with "top billing", i.e., billed as 1, 2 or 3? For each actor, also show the years these movies spanned?

* Top 5 Billing Actors are **Carol Vorderman, Richard Whiteley, Jon Stewart, Edd Hall, Jay Leno**

~~~r
>top_billing= famous_actors[which(famous_actors[,6]== 1), c(4,6)]
>top_billing= rbind(top_billing,famous_actors[which(famous_actors[,6]== 2), c(4,6)])
>top_billing= rbind(top_billing,famous_actors[which(famous_actors[,6]== 3), c(4,6)])
>top_billing= rbind(top_billing,famous_actors[which(famous_actors[,6]== 4), c(4,6)])
>top_billing= rbind(top_billing,famous_actors[which(famous_actors[,6]== 5), c(4,6)])
~~~

~~~r
> top5_billing = sort(table(top_billing[,1]),decreasing= TRUE)[1:5]
> top5_billing = cbind(top5_billing,full_names= sapply(1:5,function(i)
paste0(actor_names[actor_names[,1]== top5_billing[i,1],3]," ", 
actor_names[actor_names[,1]== top5_billing[i,1],2])))
> top5_billing
        idactors  top5_billing full_names
1164869 "1164869" "4661"       "Carol Vorderman"
3413201 "3413201" "4513"       "Richard Whiteley"
3181475 "3181475" "2607"       "Jon Stewart"
2047306 "2047306" "2239"       "Edd Hall"
2401656 "2401656" "2198"       "Jay Leno"
~~~


--------------------------------------------------------------------------------------------------

### _**Question 11:**_ Who are the 10 actors that performed in the most movies within any given year? What are their names, the year they starred in these movies and the names of the movies?




--------------------------------------------------------------------------------------------------

### _**Question 12:**_ Who are the 10 actors that have the most aliases (i.e., see the aka_names table).

* Top 10 alias actors are **Jess Franco, D'Amato Joe, Uschi Digard, Herschel Savage, Godfrey Ho, Joey Silvera, Zuzana Presova, Christoph Clark, Nathanael Len, Sandra Kalerman**

~~~r
> most_alias = dbReadTable(con,"aka_names")
> top10_alias = sort(table(most_alias[,2]),decreasing= TRUE)[1:10]
> top10_alias= cbind(idactors = as.numeric(names(top10_alias)), top10_alias)
> top10_alias = cbind(top10_alias,full_names= sapply(1:10,
function(i) paste0(actor_names[actor_names[,1]== top10_alias[i,1],3],
" ", actor_names[actor_names[,1]== top10_alias[i,1],2])))
~~~

~~~r
> top10_alias
        idactors  top10_alias full_names
1900637 "1900637" "76"        "Jess Franco"
1682347 "1682347" "69"        "D'Amato, Joe NA"
281724  "281724"  "60"        "Uschi Digard"
3035137 "3035137" "52"        "Herschel Savage"
2121005 "2121005" "49"        "Godfrey Ho"
3107669 "3107669" "41"        "Joey Silvera"
893062  "893062"  "37"        "Zuzana Presova"
1611803 "1611803" "37"        "Christoph Clark"
2414313 "2414313" "37"        "Nathanael Len"
546565  "546565"  "36"        "Sandra Kalerman"
~~~

---------------------------------------------------------------------------

### **Question 13:** Networks: Pick a (lead) actor who has been in at least 20 movies. Find all of the other actors that have appeared in a movie with that person. For each of these, find all the people they have appeared in a move with it. Use this to create a network/graph of who has appeared with who. Use the igraph or statnet packages to display this network. If you want, you can do this with individual SQL commands and the process the results in R to generate new SQL queries. In other words, don't spend too much time trying to create clever SQL queries if there is a more direct way to do this in R.



--------------------------------------------------------------------------------

### Question 14: What are the 10 television series that have the most number of movie stars appearing in the shows?

~~~r
series = dbReadTable(con,"series")
series = cbind(series,idactors= sapply(1:dim(series)[1],function(i) famous_actors[famous_actors[,2]== series[i,2],4] ))

top10_series= table(series[,1], series[,6])
top10_series = top10_series[order(series[,1])][1:10]
~~~

