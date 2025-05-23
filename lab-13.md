### Load packages and data

    library(DT)
    library(tidyverse) 
    library(MASS)
    library(ggplot2)

### Exercise 1 Simulating Our Colonists

The distribution of age resembles a normal distribution, because rnorm
function is to extract data from a normal distribution.

    set.seed(123)
    age <- rnorm(100, mean = 30, sd = 5)

    df_colonists <- data.frame(id = 1:100, # this is our id column
                                age = age ) # add the age column here


    ggplot(df_colonists, aes(x = age)) +
      geom_histogram(binwidth = 5, fill = "skyblue", color = "white") +
      labs(title = "Age Distribution of Colonists",
           x = "Age",
           y = "Count") +
      theme_minimal()

![](lab-13_files/figure-markdown_strict/unnamed-chunk-1-1.png)

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

    ## Hey Mason, I am not understanding how the codes above work, so I just did things below, which seems to achieve the same goal we intended?


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

\#creating variables that have specific correlations

    # Simulate technical skills based on age

    # Set the seed for reproducibility
    set.seed(123)

    # Create a variable for technical skills
    df_colonists$technical_skills <- 2 * df_colonists$age + rnorm(100, mean = 0, sd = 1)

    # visualize the relationship

    ggplot(df_colonists, aes(x = age, y = technical_skills)) +
      geom_jitter(color = "purple", size = 1.5) +
      geom_smooth(method = "lm", color = "black") +
      labs(title = "Scatter Plot of Colonist Age and technical skills",
           x = "Age",
           y = "Skills") +
      theme_minimal()

    ## `geom_smooth()` using formula = 'y ~ x'

![](lab-13_files/figure-markdown_strict/unnamed-chunk-3-1.png) Engineer
probably have the highest problem solving abilities, since solving
problem is really a large part of their job… Then scientist and medic
have second and third highest problem solving abilities, followed by
workers, who need more instructions to get work done. I assume the sd
across all 4 roles are the same, as the variance within each role
population should probably be the same.

    ##since I have 4 levels in role...

    set.seed(123)
    df_colonists$problem_solving[df_colonists$role == "Engineer"]  <- rnorm(sum(df_colonists$role == "Engineer"),  mean = 100, sd = 10)
    df_colonists$problem_solving[df_colonists$role == "Scientist"] <- rnorm(sum(df_colonists$role == "Scientist"), mean = 80,  sd = 10)
    df_colonists$problem_solving[df_colonists$role == "Medic"]     <- rnorm(sum(df_colonists$role == "Medic"),     mean = 70,  sd = 10)
    df_colonists$problem_solving[df_colonists$role == "Worker"]    <- rnorm(sum(df_colonists$role == "Worker"),    mean = 60,  sd = 10)

### Exercise 3

\#stimulate data using multivariate rnorm =&gt; mvrnorm (MASS)

    ?mvrnorm

    mean_traits <- c(50, 50)
    cov_matrix <- matrix(c(100, 50, 
                           50, 100), ncol = 2)

    # Generate correlated data

    traits_data <- mvrnorm(n = 100, mu = mean_traits, Sigma = cov_matrix, empirical = FALSE)

    colnames(traits_data) <- c("resilience", "agreeableness")

    df_colonists <- cbind(df_colonists, traits_data)

# stimulating Big Five

      cor_matrix_bigfive <- matrix(c(
      1.0000, 0.2599, 0.1972, 0.1860, 0.2949,
      0.2599, 1.0000, 0.1576, 0.2306, 0.0720,
      0.1972, 0.1576, 1.0000, 0.2866, 0.1951,
      0.1860, 0.2306, 0.2866, 1.0000, 0.1574,
      0.2949, 0.0720, 0.1951, 0.1574, 1.0000
      ), nrow=5, ncol=5, byrow=TRUE,
      dimnames=list(c("EX", "ES", "AG", "CO", "OP"), 
            c("EX", "ES", "AG", "CO", "OP")))

    # Generate correlated data
    mu_bigfive <- rep(0, 5)

    big_five_data <- mvrnorm(n = 100, mu = mu_bigfive, Sigma = cor_matrix_bigfive, empirical = FALSE)

    colnames(big_five_data) <- c("openness", "agreeableness", "conscientiousness", "extraversion", "neuroticism") ## this agreeableness here will be agreeableness.1, since we had another agreeableness before

    df_colonists <- cbind(df_colonists, big_five_data)

### Excercise 4

\#replicate the colonies

    set.seed(12)

    # simulate 100 planets
    colonies_list <- replicate(
      n = 100,
      expr = {
        scores <- mvrnorm(n = 100, mu = mu_bigfive, Sigma = cor_matrix_bigfive)
        df <- as.data.frame(scores)
        colnames(df) <- c("Extraversion", "Neurotism", 
                          "Agreeableness", "Conscientiousness", "Openness")
        return(df)
      },
      simplify = FALSE
    )

    df_all <- bind_rows(
      lapply(1:100, function(i) {
        colonies_list[[i]] %>%
          mutate(planet = i, colonist_id = 1:100)
      })
    )

    ## extract sd and mean
    summary_df <- df_all %>%
      group_by(planet) %>%
      summarise(
        mean_extraversion = mean(Extraversion),
        sd_extraversion = sd(Extraversion),
        correlation_with_openness = cor(Extraversion, Openness)
      )

    print(summary_df)

    ## # A tibble: 100 × 4
    ##    planet mean_extraversion sd_extraversion correlation_with_openness
    ##     <int>             <dbl>           <dbl>                     <dbl>
    ##  1      1           -0.0593           0.951                     0.320
    ##  2      2           -0.0578           0.946                     0.112
    ##  3      3            0.0520           1.07                      0.127
    ##  4      4           -0.0299           1.04                      0.357
    ##  5      5            0.0416           1.02                      0.249
    ##  6      6            0.126            0.925                     0.283
    ##  7      7           -0.107            0.926                     0.216
    ##  8      8            0.135            0.868                     0.191
    ##  9      9            0.0780           1.09                      0.371
    ## 10     10            0.154            1.05                      0.247
    ## # ℹ 90 more rows

The distribution of the r between extraversion and openness is
approaching a normal distribution, because we stimulate the data with a
given correlation of 0.2949, and the more sample we randomly draw from
the population, the more this distribution is going to resemble a normal
distribution.

    ggplot(summary_df, aes(x = correlation_with_openness)) +
      geom_histogram(binwidth = 0.02, fill = "skyblue", color = "white") +
      labs(
        title = "Distribution of Correlations Between Extraversion and Openness",
        x = "Correlation Coefficient (r)",
        y = "Number of Colonies"
      ) +
      theme_minimal()

![](lab-13_files/figure-markdown_strict/unnamed-chunk-8-1.png)
