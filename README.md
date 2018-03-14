---
title: "Modern Data Mining - HW 2"
author: "Cheshta Dhingra"
output:
  pdf_document: default
  html_document: default
  word_document: default
---
[image_0]: ./images/a.png
[image_1]: ./images/b.png
[image_2]: ./images/c.png
[image_3]: ./images/d.png
[image_4]: ./images/e.png
[image_5]: ./images/f.png
[image_6]: ./images/g.png
[image_7]: ./images/h.png
[image_8]: ./images/i.png
[image_9]: ./images/j.png
[image_10]: ./images/k.png
[image_11]: ./images/l.png
[image_12]: ./images/m.png
[image_13]: ./images/n.png
[image_14]: ./images/o.png
[image_15]: ./images/p.png
[image_16]: ./images/q.png
[image_17]: ./images/r.png
[image_18]: ./images/s.png
[image_19]: ./images/t.png
[image_20]: ./images/u.png
[image_21]: ./images/v.png
[image_22]: ./images/w.png
[image_23]: ./images/x.png


```{r setup, include=FALSE}
knitr::opts_chunk$set(fig.height=5, fig.width=11, warning = F)

# constants for homework assignments
hw_num <- 2
hw_due_date <- "19 Febrary, 2017"
```

## Overview / Instructions

This is homework #`r paste(hw_num)` of STAT 471/571/701. It will be **due on `r paste(hw_due_date)` by 11:59 PM** on Canvas. You can directly edit this file to add your answers. Submit the Rmd file, a PDF or word or HTML version with only 1 submission per HW team.

```{r library, include=FALSE}
# add your library imports here:
rm(list=ls()) # Remove all the existing variables
dir <- "/Users/che/Documents/STAT471/Data" # my laptop
library(dplyr)
library(ISLR)
library(ggplot2)
library(leaps)
```

## Problem 0

Review the code and concepts covered during lecture: multiple regression, model selection and penalized regression through elastic net. 


## Problem 1:

Auto data from ISLR. The original data contains 408 observations about cars. It has some similarity as the data CARS that we use in our lectures. To get the data, first install the package ISLR. The data Auto should be loaded automatically. We use this case to go through methods learnt so far. 

You can access the necessary data with the following code:

```{r, eval = T}
# check if you have ISLR package, if not, install it
if(!requireNamespace('ISLR')) install.packages('ISLR') 
auto_data <- ISLR::Auto
```

Get familiar with this dataset first. You can use `?ISLR::Auto` to view a description of the dataset. 

1. Explore the data, with particular focus on pairwise plots and summary statistics. Briefly summarize your findings and any peculiarities in the data.
```{r}
# Summarize Data 
summary(auto_data)
```
![alt text][image_0]

```{r}
# Pairwise plots 
pairs(auto_data)
```

![alt text][image_1]

```{r}
name.num <- sapply(auto_data, is.numeric) # pulling out all the num. var's.
pairs(auto_data[name.num])
```

![alt text][image_2]

```{r}
cor(auto_data[name.num])
```
![alt text][image_3]

```{r}
# Plot Cylinders Distribution
ggplot(auto_data, aes(x = auto_data$cylinders)) + geom_histogram(binwidth = 1) +labs(title = "Cylinders Distribution", x = "Cylinders", y = "Count")
```
![alt text][image_4]

```{r}
# Plot Displacement Distribution 
ggplot(auto_data, aes(x = auto_data$displacement)) + geom_histogram(binwidth = 1) +labs(title = "Displacement Distribution", x = "Displacement", y = "Count")
```
![alt text][image_5]

```{r}
# Plot Horsepower Distribution
ggplot(auto_data, aes(x = auto_data$horsepower)) + geom_histogram(binwidth = 1) +labs(title = "Horsepower Distribution", x = "Horsepower", y = "Count")
```
![alt text][image_6]

```{r}
# Plot Weight Distribution
ggplot(auto_data, aes(x = auto_data$weight)) + geom_histogram(binwidth = 1) +labs(title = "Weight Distribution", x = "Weight", y = "Count")
```
![alt text][image_7]

```{r}
# Plot Acceleration Distribution
ggplot(auto_data, aes(x = auto_data$acceleration)) + geom_histogram(binwidth = 1) +labs(title = "Acceleration Distribution", x = "Acceleration", y = "Count")
```
![alt text][image_8]

```{r}
# Plot Year Distribution 
ggplot(auto_data, aes(x = auto_data$year)) + geom_bar() +labs(title = "Year Distribution", x = "Acceleration", y = "Count")
```
![alt text][image_9]

To summarize the data, we can see that the mean mpg of this sample = 23.45. The mean number of cylinders = 5.47, the mean displacement = 194.4, mean horsepower = 104.5, mean weight = 2978, mean acceleration = 15.54. Finally the mean year these cars were made = 76. We can see strong correlations by looking at the pairwise plots above. For example, mpg seems to be strongly negatively correlated with displacement, horsepower and weight, all of which seem to be positively correlated with each other. These three variables also appear to be negatively correlated with acceleration. 

2. What effect does time have on MPG?
    i. Start with a simple regression of mpg vs. year and report R's `summary` output. Is year a significant variable at the .05 level? State what effect year has on mpg, if any, according to this model. 
```{r}
reg1 <- lm(auto_data$mpg ~ auto_data$year)
summary(reg1)
```

![alt text][image_10]

Year is a significant variable at the 0.05 level (and even lower). We expect that for every additional year that passes, the MPG increases by 1.23 on average. 

    i. Add horsepower on top of the variable year. Is year still a significant variable at the .05 level? Give a precise interpretation of the year effect found here. 
```{r}
reg2 <- lm(auto_data$mpg ~ auto_data$year + auto_data$horsepower)
summary(reg2) 
```

![alt text][image_11]

Yes, year is still significant at the 0.05 level. If horsepower is held constant, the effect of an additional year on MPG is an increase of 0.657 on average. If year is held constant, the effect of an additional unit of horsepower on MPG is a decrease of 0.132 on average. 

    i. The two 95% CI's for the coefficient of year differ among i) and ii). How would you explain the difference to a non-statistician?
The reason these two CI's are different is because the variables are defined differently in the two models. With the single regressor model, we don't take any other features of the car into account and only look at the effect of year on MPG on average. However, when we add Horsepower to the model, the coefficient for year stands for the impact of year on MPG *keeping horsepower constant*. 
    i. Do a model with interaction by fitting `lm(mpg ~ year * horsepower)`. Is the interaction effect significant at .05 level? Explain the year effect (if any). 
```{r}
reg3 <- lm(auto_data$mpg ~ auto_data$year + auto_data$horsepower + auto_data$year * auto_data$horsepower)
summary(reg3) 
```
![alt text][image_12]

Yes, the interaction term is significant at the 0.05 level. This means that the effect of year on MPG is different for different values of horsepower, and vice versa. The net effect of year on the MPG is not limited to B1 but also includes B3 and the horsepower. Put another way, the slopes of the regression lines for the effect of year on MPG are different for different values of horsepower, and this difference is indicated by the interaction term coefficient, B3. Since this term is negative, the larger horsepower gets, the lower the effect of year on MPG gets. B1 by itself is now interpreted as the unique effect of year on MPG when it's horsepower = 0. Theoretically this value = 2.192 (though it's hard to interpret a situation where the car's horsepower = 0.)  

3. Remember that the same variable can play different roles! Take a quick look at the variable `cylinders`, try to use this variable in the following analyses wisely. We all agree that larger number of cylinder will lower mpg. However, we can interpret `cylinders` as either a continuous (numeric) variable or a categorical variable.
    i. Fit a model, that treats `cylinders` as a continuous/numeric variable: `lm(mpg ~ horsepower + cylinders, ISLR::Auto)`. Is `cylinders` significant at the 0.01 level? What effect does `cylinders` play in this model?
```{r}
reg4 <- lm(auto_data$mpg ~ auto_data$cylinders)
summary(reg4) 
```

![alt text][image_13]

Each additional cylinder corresponds to a decrease in MPG by 3.55 on average. This is significant at the 0.01 level. 

    ii. Fit a model that treats `cylinders` as a categorical/factor variable:  `lm(mpg ~ horsepower + as.factor(cylinders), ISLR::Auto)`. Is `cylinders` significant at the .01 level? What is the effect of `cylinders` in this model? Use `anova(fit1, fit2)` and `Anova(fit2`)` to help gauge the effect. Explain the difference between `anova()` and `Anova`.
```{r}
reg5 <- lm(auto_data$mpg ~ auto_data$horsepower + as.factor(auto_data$cylinders))
summary(reg5) 
anova(reg4, reg5)
car::Anova(reg5)
```

![alt text][image_14]

Cylinders is significant at the 0.01 level when comparing cars with 4 cylinders versus those with just 3. However, it is not significant at this level when comparing cars with 5, 6 or 8 cylinders to those with just 3. The Anova() function performs the F test for each variable. The anova() function may be used to perform a hypothesis test comparing a first model without a variable to a second model with an added variable.

    iii. What are the fundamental differences between treating `cylinders` as a numeric and or a factor models? 
When treating cylinders as a numeric entity, we find the average effect of an additional cylinder on MPG. However, when treating it as a factor we find the additional effect of 4, 5, 6 or 8 cylinders on MPG compared to the effect of 3 cylinders on MPG (which is our baseline and given by the intercept term). As.factor turns our cylinder variables into indictor variables. 

4. Final modelling question: we want to explore the effects of each feature as best as possible. <br /> You may explore interactions, feature transformations, higher order terms, or other strategies within reason. The model(s) should be as parsimonious (simple) as possible unless the gain in accuracy is significant from your point of view. Use Mallow's Cp or BIC to select the model.
  * Describe the final model and its accuracy. Include diagnostic plots with particular focus on the model residuals.
  * Summarize the effects found.
  * Predict the mpg of a car that is: built in 1983, in US, red, 180 inches long, 8 cylinders, 350 displacement, 260 as horsepower and weighs 4000 pounds. Give a 95% CI.
```{r}
data <- auto_data[1:8] # all variables except name 
fit.all <- lm(auto_data$mpg ~., data) 
summary(fit.all)
```
![alt text][image_15]

```{r}
fit.exh <- regsubsets(auto_data$mpg ~., data, nvmax=8, method="exhaustive")
```
![alt text][image_16]

```{r}
names(fit.exh)
summary(fit.exh) # List the model with the smallest RSS among each size of the model
```
![alt text][image_17]

```{r}
f.e <- summary(fit.exh)
names(f.e)
f.e$which
```
![alt text][image_18]
![alt text][image_19]

Here are the plots of cp vs number of predictors. Similarly we have the plots of BIC v.s.  number of the predictors
```{r}
par(mfrow=c(3,1))
plot(f.e$cp, xlab="Number of predictors", 
     ylab="cp", col="red", type="p", pch=16)
plot(f.e$bic, xlab="Number of predictors", 
     ylab="bic", col="blue", type="p", pch=16)
plot(f.e$adjr2, xlab="Number of predictors", 
     ylab="adjr2", col="green", type="p", pch=16)
par(mfrow=c(1,1))
```
![alt text][image_20]

In this case we may use 6 variables as our model size. 
```{r}
opt.size <- which.min(f.e$cp) # locate the optimal model size
opt.size
fit.exh.var <- f.e$which 
fit.exh.var[opt.size,]  # this gives us the optimal variables selected
fit.exh.var[6,]
```
To pull out the final model: 
```{r}
fit.exh.6 <- lm(auto_data$mpg ~., data[fit.exh.var[6,]])  
summary(fit.exh.6)  # Note: there is no guarantee that all the var's in the final model are significant.
library(car)
Anova(fit.exh.6) # Once again this gives us the test for each var at a time.
```
Forward Selection
```{r}
fit.forward <- regsubsets(auto_data$mpg ~., data, nvmax=8, method="forward")
fit.forward
f.f <- summary(fit.forward)
f.f
plot(f.f$rsq, ylab="rsq", col="red", type="p", pch=16,
     xlab="Forward Selection")
```
![alt text][image_21]

```{r}
plot(f.e$rsq, ylab="rsq", col="blue", type="p", pch=16,
   xlab="All Subset Selection")
```
![alt text][image_22]

```{r}
par(mfrow=c(1,1))
coef(fit.forward, 6) 
```
Final Model: 
```{r}
fit.final <- lm(auto_data$mpg ~ auto_data$cylinders + auto_data$displacement + auto_data$horsepower + auto_data$weight + auto_data$year + auto_data$origin)
summary(fit.final)
```

Model diagnostics
```{r}
par(mfrow=c(2,1))
plot(fit.final,1)     # Everything seems to be fine. I only use the first two plots.
plot(fit.final,2)
par(mfrow=c(1,1))
```
![alt text][image_23]

I decided to use the exhausitive  model with 6 predictors as my final model. These predictors are "cylinders", "displacement", "horsepower", "weight", "year" and "origin". I used Mallow's Cp to generate this model. Using this model, we find the LS coefficients for each predictor holding all others constant, as shown above. An additional cylinder decreases MPG by -5.067e-01 (this is not significant). An additional unit of displacement increases MPG by 1.927e-02. An additional unit of horsepower decreases MPG by -2.389e-02. An additional pound decreases MPG by -6.218e-03. With each year the MPG increases by 7.475e-01. And the effect of origin on MPG is  1.428e+00. 
```{r}
newcar <-  data[1,]  # Create a new row with same structure as in auto_data
newcar[1] <- "NA"
newcar[2] <- 8 # Assign features for the new car
newcar[3] <- 350
newcar[4] <- 260
newcar[5] <- 4000
newcar[6] <- "NA"
newcar[7] <- 83
newcar[8] <- 1
newcar
```
Get a 95% CI of the salary for the player 
```{r, eval=FALSE}
newcar.m <- predict(fit.final, newcar, interval="confidence", se.fit=TRUE) 
newcar.m  # in log scale
exp(newcar.m$fit)

newcar.p <- predict(fit.final, newcar, interval="prediction", se.fit=TRUE) 
newcar.p  # in log scale
exp(newcar.p$fit)
```
## Problem 2

Do ISLR, page 262, problem 8, and write up the answer here. This question is designed to help us understanding model selections through simulations. You may want to do this first before answering question 4 in Problem 1.

a.  Use the rnorm() function to generate a predictor X of length n = 100, as well as a noise vector ε of length n = 100.
```{r}
x <- rnorm(100)
ep <- rnorm(100)
```
b. Generate a response vector Y of length n = 100 according to the model Y = β0 +β1X +β2X2 +β3X3 +ε, where β0, β1, β2,and β3 are constants of your choice
```{r}
y = 2 + 3*x + 4*x^2+ 2*x^3+ 5*x^4 + 4*x^5 + 3*x^6 + 6*x^7 + 2*x^8 + 3*x^9 + 9*x^10 + ep
data3 <- data.frame(x, x^2, x^3, x^4, x^5, x^6, x^7, x^8, x^9, x^10, y)
```
c. 
```{r}
fit.exh2 <- regsubsets(y~., data = data3, nvmax=10, method="exhaustive")
names(fit.exh2)
summary(fit.exh2) # List the model with the smallest RSS among each size of the model
f.e <- summary(fit.exh2)
names(f.e)
f.e$which
f.e$rsq
f.e$rss
f.e$bic
f.e$cp
```

