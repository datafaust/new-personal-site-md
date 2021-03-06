# Downloading & Extracting Video Frames in R & Python

### Video Analytics
I had a recent project proposal in which I was to do something very interesting with data and rather than download some data set to work with I thought it be fun to work on something new. For some time I have been interested in getting into computer vision and now that I build and fly drones merging the two just seems inevitable. So I spent the last weekend learning how to work with this, most of it in python and then a little bit of it in R for fun. The ultimate goal: to run object detection on drones.

Another note: a lot of this work as usual comes from learning from great people who help the community. Credits to Harrison at https://pythonprogramming.net/ who hosts an entire youtube tutorial series on opencv not to mention teaches everything for Python. I recommend starting off with this:

https://www.youtube.com/watch?v=Z78zbnLlPUA&list=PLQVvvaa0QuDdttJXlLtAJxJetJcqmqlQq

### Computer Vision in R? What? Why?
R is my bread and butter, and I love the language; for those who aren’t as familiar with python this could be a gentle transition into the computer vision world (albeit there is still so much to learn!) and so why not?

### What do I need?
So before we jump into the code let’s go over the dependencies we will need because there are quite a few. A note of caution, setups differ and I am by no means an expert in installation procedures but I will give you my setup:

****Running Windows 10 Surface

### Python

-for python, you’ll have to decide which version to install, 2.7 o3 3.xx, some people will tell you to install Anaconda. Personally I have all three, but for windows video (one link is Harrison’s run of the install):

https://www.python.org/downloads/release/python-352/ https://www.youtube.com/watch?v=ulJdZn0qBCQ

### OpenCV

-if you follow the youtube video above it should be installed with python 3.5.2

### R

R is a little easier, install the most recent R version (google it). Install devtools and rtools (just google it, very simple) and then install the ROpenCVlite and Rvision:

https://github.com/swarm-lab/ROpenCVLite https://github.com/swarm-lab/Rvision

ROpenCVLite is just going to reinstall OpenCV in the right location for Rvision and future R computer vision libraries to access.

Can we start writing some Code?
Now on to the fun stuff. We can start with R; first lets pull the video file we need (drone video from a detection data set I’m working with):

```
#directory
setwd("C:/Users/0000/Documents/original_web_posts/video_prc_r_py")
                                                                                        
#the name of the file
my_path =paste0(getwd(),"/","Video_1.avi")
#download the video
download.file(url = "https://drive.switch.ch/index.php/s/3b3bdbd6f8fb61e05d8b0560667ea992/download?path=%2Fvideos%2Fdrones&files=Video_1.avi"
,destfile = my_path
,mode = "wb"
)
```

Once you’ve downloaded the video into a directory we can begin to parse through it; we can check the number of frames in the video and then extract one to plot:

```
library(Rvision)
#rvision wants us to create a video object
the_vid = Rvision::video("C:/Users/0000/Documents/original_web_posts/video_prc_r_py/Video_1.avi")
nframes(the_vid)#shows us the number of frames
## [1] 393
my_frame = readFrame(the_vid,3)
plot(my_frame)
Release(the_vid) #i believe like closing an odbc connection this is good practice
```

## Video released successfully.
Now that we have the video in place and know how to extract one frame we can simply loop through the frames and save them as images to train our models:

```
library(pbapply)
                                                                                        
#rvision wants us to create a video object
the_vid = video("C:/Users/0000/Documents/original_web_posts/video_prc_r_py/Video_1.avi")
                                                                                        
#then we loop through that video object and extract the frame
pblapply(1:10, function(x){
z = readFrame(the_vid,x)
setwd("C:/Users/0000/Documents/original_web_posts/video_prc_r_py")
write.Image(z,paste0(x,"_","frame",".jpg"))
})
## [[1]]
## NULL
## 
## [[2]]
## NULL
## 
## [[3]]
## NULL
## 
## [[4]]
## NULL
## 
## [[5]]
## NULL
## 
## [[6]]
## NULL
## 
## [[7]]
## NULL
## 
## [[8]]
## NULL
## 
## [[9]]
## NULL
## 
## [[10]]
## NULL
#for the newbies who perfer the for loops syntax:
                                                                                        
# for (i in 1:10) {
#   the_vid = video("C:\\Users\\fausto\\Documents\\incubator_img_recog\\plane_classifier\\drone_proj\\videos\\drones\\Video_1.avi")
#   z = readFrame(the_vid,i)
#   plot(z)
# }
                                                                                        
setwd("C:/Users/0000/Documents/original_web_posts/video_prc_r_py")
list.files() 
##  [1] "1_frame.jpg"         "10_frame.jpg"        "2_frame.jpg"        
##  [4] "3_frame.jpg"         "4_frame.jpg"         "5_frame.jpg"        
##  [7] "6_frame.jpg"         "7_frame.jpg"         "8_frame.jpg"        
## [10] "9_frame.jpg"         "vid_frames_r_py.Rmd" "Video_1.avi"
```

And that’s about it for downloading video data in R. Insert grayscaling functions into the loop, or other analyses with the EBI or imageR package.

### How about Python?
Since we already have the video downloaded in R I won’t bring it in again python. Instead I will run through how to pull the frames and save them. You can run the following:

```
import cv2
import numpy as np
import os
#set directory
os.chdir("C:/Users/0000/Documents/original_web_posts/video_prc_r_py")
#pull in video
cap = cv2.VideoCapture("Video_1.avi")
count = 0
#success =T
#loop through video and pull frames for saving
while True:
    
    ret, img = cap.read()
    print 'Read a new frame: ', ret
    gray = cv2.cvtColor(img,cv2.COLOR_BGR2GRAY)
    os.chdir("C:/Users/0000/Documents/original_web_posts/video_prc_r_py")
    cv2.imwrite("frame%d.jpg" % count, img)     # save frame as JPEG file
    count += 1
```

The code will kick and error at the end as it runs out of frames, you can adjust to get rid of that. And there it is! A few lines of code and you can already extract frames from videos. In one of my next posts I’ll go over how to use some of that picture data to train a basic Haarcascade so we can detect specific objects in videos.