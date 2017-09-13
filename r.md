# Install a package
install.packages("package name")

# Downloading a file
'''
library(downloader)
url <- "http://www.url.com/datafile.txt"
filename <- filename <- basename(url)
download(url, destfile=filename)
'''

# dplyr
Extraction of data, supports pipes.
'''
library(dplyr)
data <- read.csv("femaleMiceWeights.csv")
controls <- filter(data, Diet=="chow")
controls <- select(controls, Bodyweight)
controls <- unlist(controls)
controls <- filter(data, Diet=="chow") %>% select(Bodyweight) %>% unlist
'''

# standard deviation
popsd() - /n
sd() - /(n-1)
