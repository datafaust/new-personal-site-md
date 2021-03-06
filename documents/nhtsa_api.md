# Making an R package to Call NHTSA API

### Why use the NHTSA API?
Recently my team has had to work with matching vehicle information to taxi data, specifically make and model information we didn’t have. It reminded me of a while back, mid 2016 during the beginnings of our driver income study when we needed vin information to calculate msrp and look at fuel consumption. At the time I had not been able to locate a package in R to call NHTSA (National Highway Transportation Authority). They have a slim but useful [api](https://vpic.nhtsa.dot.gov/api/).

I hadn’t built a package before then, so I thought it would be a good chance to collect some interesting data and learn how to make a package. I reference Hilary Park’s [blog](https://hilaryparker.com/2014/04/29/writing-an-r-package-from-scratch/) at the time and created my first R package which I called caRshop. It’s nothing more than a basic wrapper but it ended up being very useful for our study as well as our enforcement unit which used the information to validate their inspection data.

### Example
By now there are some powerful vin decoding packages out there but if you wish to give caRshop a chance you can install it from the repo:
```
devtools::install_github('datafaust/caRshop')
```
Once it’s installed you can leverage functions to extract data for a vin. For instance you can pull for basic vin information with our favorite fast and furious character:
```
library(caRshop)
vin_diesel("1G2HX54K724118697", sec = 1, tidyup = T)
"https://vpic.nhtsa.dot.gov/api/vehicles/DecodeVinValues/1G2HX54K724118697?format=json"
## No encoding supplied: defaulting to UTF-8.
##   Count                       Message            SearchCriteria
## 1   131 Results returned successfully VIN(s): 1G2HX54K724118697
...
```

You’ll see that this prints out the entire record response fromt the json, with a few parameters to go ahead and convert the response to a dataframe as well as not overload the api with too many calls too quickly.

In our case we needed to loop over thousands of vehicles. As an example of what a loop for that might look like here we run a few (I’m a huge fan of the pbapply package):

```
library(pbapply)
library(dplyr)
library(data.table)
my_cars = c('1G2HX54K724118697', '5UXZV4C58D0E07160', '1NXBR18E1XZ184142')

#run loop
my_data = 
my_cars %>% pblapply(function(x){
  vin_diesel(x, sec = 1, tidyup = T)
}) %>% rbindlist()
## [1] "https://vpic.nhtsa.dot.gov/api/vehicles/DecodeVinValues/1G2HX54K724118697?format=json"
## [1] "https://vpic.nhtsa.dot.gov/api/vehicles/DecodeVinValues/5UXZV4C58D0E07160?format=json"
## [1] "https://vpic.nhtsa.dot.gov/api/vehicles/DecodeVinValues/1NXBR18E1XZ184142?format=json"
print(my_data[,.(Results.Make, Results.Model)])
##    Results.Make Results.Model
## 1:      PONTIAC    Bonneville
## 2:          BMW            X5
## 3:       TOYOTA       Corolla

```

I went ahead and pulled only a few columns, but you get the idea. Running in Python is pretty straightforward, but I’ll go over that another time. This was a fun experience and a great lookback to a time when I was still learning some of the more basic functions in R.