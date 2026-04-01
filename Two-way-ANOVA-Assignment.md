# Two-way ANOVA

This R markdown document provides an example of performing a regression
using the lm() function in R and compares the output with the
jmv::ANOVA() function in the jmv (Jamovi) package.

## Package management in R

``` r
# keep a list of the packages used in this script
packages <- c("tidyverse","rio","jmv")
```

This next code block has eval=FALSE because you don’t want to run it
when knitting the file. Installing packages when knitting an R notebook
can be problematic.

``` r
# check each of the packages in the list and install them if they're not installed already
for (i in packages){
  if(! i %in% installed.packages()){
    install.packages(i,dependencies = TRUE)
  }
  # show each package that is checked
  print(i)
}
```

``` r
# load each package into memory so it can be used in the script
for (i in packages){
  library(i,character.only=TRUE)
  # show each package that is loaded
  print(i)
}
```

    ## ── Attaching core tidyverse packages ──────────────────────── tidyverse 2.0.0 ──
    ## ✔ dplyr     1.2.0     ✔ readr     2.2.0
    ## ✔ forcats   1.0.1     ✔ stringr   1.6.0
    ## ✔ ggplot2   4.0.2     ✔ tibble    3.3.1
    ## ✔ lubridate 1.9.5     ✔ tidyr     1.3.2
    ## ✔ purrr     1.2.1     
    ## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ## ✖ dplyr::filter() masks stats::filter()
    ## ✖ dplyr::lag()    masks stats::lag()
    ## ℹ Use the conflicted package (<http://conflicted.r-lib.org/>) to force all conflicts to become errors

    ## [1] "tidyverse"
    ## [1] "rio"
    ## [1] "jmv"

## ANOVA is a linear model

The ANOVA is a type of linear model. We’re going to compare the output
from the lm() function in R with ANOVA output.To use a categorical
variable in a linear model it needs to be dummy coded. One group needs
to be coded as 0 and the other group needs to be coded as 1.If you
compare the values for F from lm() and t from the t-test you’ll see that
t^2 = F. You should also notice that the associated p values are equal.

Nice example:
<https://sites.utexas.edu/sos/guided/inferential/numeric/glm/>

## Open data file

The rio package works for importing several different types of data
files. We’re going to use it in this class. There are other packages
which can be used to open datasets in R. You can see several options by
clicking on the Import Dataset menu under the Environment tab in
RStudio. (For a csv file like we have this week we’d use either From
Text(base) or From Text (readr). Try it out to see the menu dialog.)

``` r
# Using the file.choose() command allows you to select a file to import from another folder.
dataset <- rio::import('Goggles.sav')
# This command will allow us to import a file included in our project folder.
# dataset <- rio::import("Album Sales.sav")
```

## Get R code from Jamovi output

You can get the R code for most of the analyses you do in Jamovi.

1.  Click on the three vertical dots at the top right of the Jamovi
    window.
2.  Click on the Syndax mode check box at the bottom of the Results
    section.
3.  Close the Settings window by clicking on the Hide Settings arrow at
    the top right of the settings menu.
4.  you should now see the R code for each of the analyses you just ran.

## lm() function in R

Many linear models are calculated in R using the lm() function. We’ll
look at how to perform a regression using the lm() function since it’s
so common.

#### Visualization

``` r
# plots for outcome split by groups
ggplot(dataset, aes(x = Attractiveness))+
  geom_histogram(binwidth = 1, color = "black", fill = "white")+
  facet_grid(FaceType ~ .)
```

![](Two-way-ANOVA-Assignment_files/figure-markdown_github/unnamed-chunk-5-1.png)

``` r
ggplot(dataset, aes(x = Attractiveness))+
  geom_histogram(binwidth = 1, color = "black", fill = "white")+
  facet_grid(Alcohol ~ .)
```

![](Two-way-ANOVA-Assignment_files/figure-markdown_github/unnamed-chunk-5-2.png)

``` r
# Make a factor for the box plot
dataset <- dataset %>% mutate(FaceType_f = as.factor(FaceType))
levels(dataset$FaceType_f)
```

    ## [1] "0" "1"

``` r
dataset <- dataset %>% mutate(Alcohol_f = as.factor(Alcohol))
levels(dataset$Alcohol_f)
```

    ## [1] "0" "1" "2"

``` r
ggplot(dataset, aes(x = FaceType_f, y = Attractiveness)) +
  geom_boxplot()
```

![](Two-way-ANOVA-Assignment_files/figure-markdown_github/unnamed-chunk-7-1.png)

``` r
ggplot(dataset, aes(x = Alcohol_f, y = Attractiveness)) +
  geom_boxplot()
```

![](Two-way-ANOVA-Assignment_files/figure-markdown_github/unnamed-chunk-7-2.png)

#### Dummy codes

If a categorical variable is designated as a factor in R, the lm()
function will dummy code it according to alphabetical order of the
factor levels. The reference level will be the first category when the
categories are put in alphabetical order. Since we already made factor
variables from our categorical variables, we’ll use those in the linear
model.

#### Computation

If we include independent variables in the model using the plus (+)
sign, each variable in the equation will be included in the model. If we
include independent variables in the model using the multiplication (\*)
sign, each variable will be included in the model, but interaction terms
between the variables will also be included.

``` r
model <- lm(formula = Attractiveness ~ FaceType_f*Alcohol_f, data = dataset)
model
```

    ## 
    ## Call:
    ## lm(formula = Attractiveness ~ FaceType_f * Alcohol_f, data = dataset)
    ## 
    ## Coefficients:
    ##            (Intercept)             FaceType_f1              Alcohol_f1  
    ##                  3.500                   2.875                   1.375  
    ##             Alcohol_f2  FaceType_f1:Alcohol_f1  FaceType_f1:Alcohol_f2  
    ##                  3.125                  -1.250                  -3.375

#### Model assessment

``` r
summary(model)
```

    ## 
    ## Call:
    ## lm(formula = Attractiveness ~ FaceType_f * Alcohol_f, data = dataset)
    ## 
    ## Residuals:
    ##    Min     1Q Median     3Q    Max 
    ## -2.500 -0.625 -0.125  0.625  2.500 
    ## 
    ## Coefficients:
    ##                        Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept)              3.5000     0.4137   8.461 1.29e-10 ***
    ## FaceType_f1              2.8750     0.5850   4.914 1.41e-05 ***
    ## Alcohol_f1               1.3750     0.5850   2.350 0.023531 *  
    ## Alcohol_f2               3.1250     0.5850   5.342 3.49e-06 ***
    ## FaceType_f1:Alcohol_f1  -1.2500     0.8274  -1.511 0.138319    
    ## FaceType_f1:Alcohol_f2  -3.3750     0.8274  -4.079 0.000197 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 1.17 on 42 degrees of freedom
    ## Multiple R-squared:  0.5154, Adjusted R-squared:  0.4578 
    ## F-statistic: 8.936 on 5 and 42 DF,  p-value: 7.677e-06

## function in Jamovi

Compare the output from the lm() function with the output from the
function in the jmv package.

``` r
# Two-way ANOVA
model_anova <- aov(Attractiveness ~ FaceType_f * Alcohol_f, data = dataset)
summary(model_anova)
```

    ##                      Df Sum Sq Mean Sq F value   Pr(>F)    
    ## FaceType_f            1  21.33  21.333  15.583 0.000295 ***
    ## Alcohol_f             2  16.54   8.271   6.041 0.004943 ** 
    ## FaceType_f:Alcohol_f  2  23.29  11.646   8.507 0.000791 ***
    ## Residuals            42  57.50   1.369                     
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1

``` r
# Effect sizes (omega squared)
library(effectsize)
omega_squared(model_anova)
```

    ## # Effect Size for ANOVA (Type I)
    ## 
    ## Parameter            | Omega2 (partial) |       95% CI
    ## ------------------------------------------------------
    ## FaceType_f           |             0.23 | [0.07, 1.00]
    ## Alcohol_f            |             0.17 | [0.02, 1.00]
    ## FaceType_f:Alcohol_f |             0.24 | [0.06, 1.00]
    ## 
    ## - One-sided CIs: upper bound fixed at [1.00].

``` r
# Levene's test (homogeneity of variance)
library(car)
leveneTest(Attractiveness ~ FaceType_f * Alcohol_f, data = dataset)
```

    ## Levene's Test for Homogeneity of Variance (center = median)
    ##       Df F value Pr(>F)
    ## group  5  0.7171 0.6142
    ##       42

``` r
# Normality test (Shapiro-Wilk)
shapiro.test(residuals(model_anova))
```

    ## 
    ##  Shapiro-Wilk normality test
    ## 
    ## data:  residuals(model_anova)
    ## W = 0.9873, p-value = 0.8779

``` r
# Q-Q plot
qqnorm(residuals(model_anova))
qqline(residuals(model_anova))
```

![](Two-way-ANOVA-Assignment_files/figure-markdown_github/unnamed-chunk-10-1.png)

``` r
# Post-hoc tests for main effects
TukeyHSD(model_anova)
```

    ##   Tukey multiple comparisons of means
    ##     95% family-wise confidence level
    ## 
    ## Fit: aov(formula = Attractiveness ~ FaceType_f * Alcohol_f, data = dataset)
    ## 
    ## $FaceType_f
    ##         diff       lwr      upr     p adj
    ## 1-0 1.333333 0.6516897 2.014977 0.0002952
    ## 
    ## $Alcohol_f
    ##       diff        lwr      upr     p adj
    ## 1-0 0.7500 -0.2550324 1.755032 0.1777260
    ## 2-0 1.4375  0.4324676 2.442532 0.0033704
    ## 2-1 0.6875 -0.3175324 1.692532 0.2317125
    ## 
    ## $`FaceType_f:Alcohol_f`
    ##           diff          lwr       upr     p adj
    ## 1:0-0:0  2.875  1.128535277 4.6214647 0.0001939
    ## 0:1-0:0  1.375 -0.371464723 3.1214647 0.1974738
    ## 1:1-0:0  3.000  1.253535277 4.7464647 0.0000979
    ## 0:2-0:0  3.125  1.378535277 4.8714647 0.0000491
    ## 1:2-0:0  2.625  0.878535277 4.3714647 0.0007414
    ## 0:1-1:0 -1.500 -3.246464723 0.2464647 0.1291660
    ## 1:1-1:0  0.125 -1.621464723 1.8714647 0.9999339
    ## 0:2-1:0  0.250 -1.496464723 1.9964647 0.9980556
    ## 1:2-1:0 -0.250 -1.996464723 1.4964647 0.9980556
    ## 1:1-0:1  1.625 -0.121464723 3.3714647 0.0812069
    ## 0:2-0:1  1.750  0.003535277 3.4964647 0.0492763
    ## 1:2-0:1  1.250 -0.496464723 2.9964647 0.2889504
    ## 0:2-1:1  0.125 -1.621464723 1.8714647 0.9999339
    ## 1:2-1:1 -0.375 -2.121464723 1.3714647 0.9871315
    ## 1:2-0:2 -0.500 -2.246464723 1.2464647 0.9550187

``` r
# Interaction plot (estimated marginal means)
interaction.plot(
  x.factor = dataset$Alcohol_f,
  trace.factor = dataset$FaceType_f,
  response = dataset$Attractiveness,
  type = "b",
  legend = TRUE,
  xlab = "Alcohol",
  ylab = "Attractiveness",
  trace.label = "FaceType"
)
```

![](Two-way-ANOVA-Assignment_files/figure-markdown_github/unnamed-chunk-10-2.png)
