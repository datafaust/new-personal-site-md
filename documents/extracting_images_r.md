# Extracting Images from the Internet with R

### Why build an image database?
In a previous post I went over how to pull in video data and parse out frames for processing. That’s really useful for pulling in the data you ultimately want to screen, but at times there is also a need for having other images. You may want to classify certain objects as they appear in other videos, or build Haarcascades with specific images. Regardless you’ll need to access images. I’m going to showcase how to do this in R with imagenet.

Imagenet is a very popular image database and you can search for virtually anything. In this case I’m going to pull from the image database to construct a database from a few key word links. Ultimately, with this scenario you will easily be able to adopt the code to whatever links you want to pull.

### Define your database
Lately my focus is on building object detection for drones so in my mind I figure I will need pictures of skies, different types of drones, planes and anything else a drone might run into at higher altitudes. I went ahead and searched key word on my favorite [image website](https://image-net.org/update-mar-11-2021.php). Copy the url addresses of the images you want.


### Build the program
Repeat this process for all the keywords you are interested in. Note some pictures may overlap in categories, this is something you will probably want to filter out later when you build out a funciton for searching duplicates. Below we start out with a loop that iterates through each link:
```
#loop that iterates through each link
library(pbapply)
urls = c(#sports = "http://www.image-net.org/api/text/imagenet.synset.geturls?wnid=n00523513",
         bird = "http://www.image-net.org/api/text/imagenet.synset.geturls?wnid=n01517966",
          drones = "http://www.image-net.org/api/text/imagenet.synset.geturls?wnid=n03245889"
         ,bird = "http://www.image-net.org/api/text/imagenet.synset.geturls?wnid=n01517966"
        ,people = "http://www.image-net.org/api/text/imagenet.synset.geturls?wnid=n07942152"
        ,planes = "http://www.image-net.org/api/text/imagenet.synset.geturls?wnid=n02690373"
        ,sky = "http://www.image-net.org/api/text/imagenet.synset.geturls?wnid=n09436708")
pblapply(1:length(urls),function(x){
  print(urls[x])
})
##                                                                       bird 
## "http://www.image-net.org/api/text/imagenet.synset.geturls?wnid=n01517966" 
##                                                                     drones 
## "http://www.image-net.org/api/text/imagenet.synset.geturls?wnid=n03245889" 
##                                                                       bird 
## "http://www.image-net.org/api/text/imagenet.synset.geturls?wnid=n01517966" 
##                                                                     people 
## "http://www.image-net.org/api/text/imagenet.synset.geturls?wnid=n07942152" 
##                                                                     planes 
## "http://www.image-net.org/api/text/imagenet.synset.geturls?wnid=n02690373" 
##                                                                        sky 
## "http://www.image-net.org/api/text/imagenet.synset.geturls?wnid=n09436708"
## [[1]]
##                                                                       bird 
## "http://www.image-net.org/api/text/imagenet.synset.geturls?wnid=n01517966" 
## 
## [[2]]
##                                                                     drones 
## "http://www.image-net.org/api/text/imagenet.synset.geturls?wnid=n03245889" 
## 
## [[3]]
##                                                                       bird 
## "http://www.image-net.org/api/text/imagenet.synset.geturls?wnid=n01517966" 
## 
## [[4]]
##                                                                     people 
## "http://www.image-net.org/api/text/imagenet.synset.geturls?wnid=n07942152" 
## 
## [[5]]
##                                                                     planes 
## "http://www.image-net.org/api/text/imagenet.synset.geturls?wnid=n02690373" 
## 
## [[6]]
##                                                                        sky 
## "http://www.image-net.org/api/text/imagenet.synset.geturls?wnid=n09436708"
```

If we loop through the urls one at a time it could take a while. In my work I often parallelize code in order to be more efficient so it’s applied here using the parallel package in R. The general write up looks like this:

```
cl = makeCluster(detectCores()-1) #this function detects yours cores 
clusterExport(cl,c("libs","urls")) #you will want to export the functions and objects to each core
clusterEvalQ(cl,lapply(libs,require,character.only = T)) #you will want to pass your libraries as well
pblapply(1:length(urls),function(x){
  print(urls[x])
},cl = cl)
stopCluster(cl)
rm(gcl)
gc()
```

You can choose to omit the parellizing if you wish, as you can see, it’s quite simple to remove or add on. Regardless now that we are successfully looping through the links we can nest another loop inside that will run through each of the links and extract the pictures. It then labels the picture by sequence and saves it into a directory. Note the trycatch function is used to pass through the errors and keep going:
```
#to download negative people and sky images from imagenet
#author: fausto
#libraries
libs = c('data.table', 'lubridate', 'fasttime'
         , 'pbapply', 'dplyr', 'parallel','fst'
         ,'RCurl')
lapply(libs, require, character.only = T)
urls = c(#sports = "http://www.image-net.org/api/text/imagenet.synset.geturls?wnid=n00523513",
         bird = "http://www.image-net.org/api/text/imagenet.synset.geturls?wnid=n01517966",
          drones = "http://www.image-net.org/api/text/imagenet.synset.geturls?wnid=n03245889"
         ,bird = "http://www.image-net.org/api/text/imagenet.synset.geturls?wnid=n01517966"
        ,people = "http://www.image-net.org/api/text/imagenet.synset.geturls?wnid=n07942152"
        ,planes = "http://www.image-net.org/api/text/imagenet.synset.geturls?wnid=n02690373"
        ,sky = "http://www.image-net.org/api/text/imagenet.synset.geturls?wnid=n09436708")
#functional loop for entire data set pull----------------------------------
#lets speed up the process with parallelizing it
#stopCluster(cl)
cl = makeCluster(detectCores()-3)
clusterExport(cl,c("libs","urls"))
clusterEvalQ(cl,lapply(libs,require,character.only = T))
pblapply(1:length(urls[1])
         ,function(x){
  
  #print the directory            
  print(paste0(getwd(),"/",names(urls)[x]))
           
  #create a directory for it
  dir.create(paste0(getwd(),"/",names(urls)[x]))
           
  #set that directory
  new_direc = paste0(getwd(),"/",names(urls)[x])
  setwd(new_direc)
  print(new_direc)
  
  #pull the lines from the site
  foto = readLines(urls[x])
  
    pblapply(1:length(foto), function(g){
               
      #(g)
      
      #loop will continue no matter what
      tryCatch({              
        
        #download the data
        download.file(
          foto[g]
          ,destfile = paste0(new_direc
                             ,"/",g,".jpg")
          ,mode = "wb")
        }, error=function(i){
                 
                 print("error keep going")
                 
                 })
      
      
    })
 
 #reset direc
 setwd("my output directory")  
  
},cl = cl
)
stopCluster(cl)
rm(cl)
gc()
```

Now check you directory for the jpgs to make sure it has the files:

```
setwd("my output directory")
length(list.files(pattern = ".jpg"))

## [1] 1053
```

And read a file in to see what it looks like:

```
library(EBImage)
setwd(direc)
plot(
  readImage(
    list.files(pattern = ".png")[4]
    )
)
```

And that’s it, you now have an image database at your disposal with seperate directories which you can manipulate to your heart’s desire. In a later post I’ll cover how to clean the bad images and have everything tidy.