Lab 13 - Colonizing Mars
================
Cynthia Jiao
3/31/2025

### Load packages and data

``` r
library(DT)
library(tidyverse) 
library(MASS)
library(ggplot2)
```

### Exercise 1 Simulating Our Colonists

The distribution of age resembles a normal distribution, because rnorm
function is to extract data from a normal distribution.
\`\`\`{r-excercise-1.1}

set.seed(123) age \<- rnorm(100, mean = 30, sd = 5)

df_colonists \<- data.frame(id = 1:100, \# this is our id column age =
age ) \# add the age column here

ggplot(df_colonists, aes(x = age)) + geom_histogram(binwidth = 5, fill =
“skyblue”, color = “white”) + labs(title = “Age Distribution of
Colonists”, x = “Age”, y = “Count”) + theme_minimal()



    ```{r-excercise-1.2}


    set.seed(1)
    roleA <- rep(c("engineer", "scientist", "medic"), 
                 length.out = 100)
    # this works if we want to set a specific number of each role
    roleB <- rep(c("engineer", "scientist", "medic"), 
                 each = 34, 
                 length.out = 100)

    roleC <- rep(c("engineer", "scientist", "medic"), 
                 times = c(33, 33, 33), 
                 length.out = 100)
                 
    # if you want to use sampling weights
    roleD <- sample(c("engineer", "scientist", "medic"), 
                    replace = TRUE, 
                    size = 100, prob = c(1,1,1))

    # if you want to randomly shuffle

    roleE <- sample(roleB, size = 100, replace = FALSE)

    ## Hey Mason, I am not understanding how the codes above work, so I just did the below, which seems to achieve the same goal we intended?

    ## adding randomized role variable to df_colonist
    roles <- c("Scientist", "Farmer", "Medic", "Engineer", "Cook")

    # Randomly assign one role to each colonist
    set.seed(123)  # for reproducibility
    df_colonists$role <- sample(roles, size = nrow(df_colonists), replace = TRUE)


    ## in addition to engineer, scientist, and medic, a colony probably needs workers to get the work done, so adding workers to the role variable...

    set.seed(42)

    # create roles (25 of each for 100 people)
    roles <- rep(c("Engineer", "Worker", "Medic", "Scientist"), each = 25)

    # randomly shuffle
    df_colonists$role <- sample(roles)


    ## create the MARSGAR variable

    set.seed(123)

    df_colonists$marsgar <- runif(100, min = 0, max = 100)

### Exercise 2
