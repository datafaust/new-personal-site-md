# Accessing Clickhouse Data in R & Python
Dec 13, 2018

### Why use Python or R?
I think a lot of people wonder why we would use R or python when there are behemoths like Spark and Clickhouse able to crunch data quickly, but the reality is that these database languages weren’t necesarrily set up to do a lot of the more specific data munging work that R and python can do. A lot of data sceintists know that database languages and scripting lanuageges compliment each other.

In some of my previous posts I set up a clickhouse database under the wsl system in windows, a bit of a tricky setup but for the purpose of testing locally as well as a potential prototype for the company. In thise post I’m going to focus on accessing the data via R and Python.

### R Access
The leading package for clickhouse access in R at the moment is RClickHouse, which comes with dplyr integration for those who enjoy piping. I’m more of a datatable guy, but I appreciate flexibiity. You’ll want to install the package and connect to the database; if you followed my previous post, it will be at the localhost address:

```
#install.packages('RClickhouse')
library(RClickhouse)
library(DBI)
library(dplyr)
con = dbConnect(RClickhouse::clickhouse(), host="localhost")
```
Once you’re connected you can begin querying. We will start off with some basic sql:

```
dbGetQuery(con, "SELECT cab_type, count(*)
FROM trips
           GROUP BY cab_type")
## Warning in evalq((function (..., call. = TRUE, immediate. = FALSE,
## noBreaks. = FALSE, : column count() converted from UInt64 to Numeric
##   cab_type   count()
## 1   yellow 759083667
## 2    green  69145084
```

If you’re a fan of dplyr you can pull that way as well:
```
trips = 
tbl(con, "trips") %>% 
  group_by(cab_type) %>%
  summarise(trips=sum(tip_amount, na.rm = T))
trips
## # Source:   lazy query [?? x 2]
## # Database: clickhouse 19.1.6 [default@localhost:9000/default; uptime: 0
## #   days ]
##   cab_type       trips
##   <chr>          <dbl>
## 1 yellow   1242908568.
## 2 green      80399851.
```

And when you’re done with the code you can disconnect from the database:
```
# Close the connection
dbDisconnect(con)
```

### Python Access
Python access is very straightforward. You’ll want to reference the documentation. Simply install the driver, I use anaconda for most of my python operations so easily pulled up the terminal and ran the pip install. Next you’ll want to run the following code to enter clickhouse through python and see existing tables:

```
from clickhouse_driver import Client
client = Client('localhost')
client.execute('SHOW TABLES')
client.execute('SELECT sum(rain) FROM trips')
```

And from there you are able to access all the tables you want.

### Conclusions
This is a great way to access large data with cutting edge database technology throough your favorite scripting language on a windows machines still running the open source software you love. I see it as a good opportunity for teams working within a windows enviornment but needing access to linux software. In one of my next posts I’ll build a quick shiny gui to access the data through a custom front end.