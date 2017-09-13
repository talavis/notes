# Install a package
`install.packages("package name")`

# help
`?command`

# Downloading a file
```
library(downloader)
url <- "http://www.url.com/datafile.txt"
filename <- filename <- basename(url)
download(url, destfile=filename)
```

# dplyr
Extraction of data, supports pipes.
```
library(dplyr)
data <- read.csv("femaleMiceWeights.csv")
controls <- filter(data, Diet=="chow")
controls <- select(controls, Bodyweight)
controls <- unlist(controls)
controls <- filter(data, Diet=="chow") %>% select(Bodyweight) %>% unlist
```

# basic statistics
`mean()` - average
`median()` - median
`popsd()` - /n
`sd()` - /(n-1)
`var()` - variance

# object size
`ncol()` - columns
`nrow()` - rows
`NCOL()`/`NROW()` - also works with vectors
`length()` - length of vector

# matrix/tables
`cbind()` - combine e.g. vectors
```
cbind(a,a**2,a**3,a**4)
```
`matrix()` - create a matrix from the given set of values.
```
matrix(1,2,2)
     [,1] [,2]
[1,]    1    1
[2,]    1    1
matrix(c(1,2,3,4),2,2)
     [,1] [,2]
[1,]    1    3
[2,]    2    4
```
`A%*%B` - matrix multiplication


# function
```
a <- function(par1, par2){
	code
}
a(b,c,d)
```

`replicate()` - perform code in multiple replicates
```
results <- replicate(1000, function())
```

`set.seed()` - set random seed
