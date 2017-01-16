---
layout: post
title: Easy and efficient data manipulation
subtitle: Tidy data and piping
date: 2016-11-24 16:00:00
author: Sandra
meta: "Tutorials"
---
<div class="block">
	<center>
		<img src="{{ site.baseurl }}/img/tutheaderpiping.png" alt="Img">
	</center>
</div>	

### Tutorial Aims:

#### <a href="#tidy"> 1. Understand the format required for analyses in R, and how to achieve it </a>

#### <a href="#dplyr"> 2. Use efficient tools for manipulating your data </a>

#### <a href="#piping"> 3. Learn a new syntax for coding : pipes </a>

Note : all the files you need to complete this tutorial can be downloaded from <a href="GITHUBREPO">this repository</a>.


In this tutorial we will discuss the best ways to record and store data, and then learn how to format your datasets in R and to apply series of functions in an efficient way using pipes `%>%`.


<a name="tidy"></a>

## 1. Understand the format required for analyses in R, and how to achieve it

The way you record information in the field or in the lab is probably very different to the way you want your data entered into R. In the field, you want tables that you can ideally draw up ahead and fill in as you go, and you will be adding notes and all sorts of information in addition to the data you want to analyse. For instance, if you monitor the height of seedlings during a factorial experiment using warming and fertilisation treatments, you might record your data like this:

<img src="{{ site.baseurl }}/img/SAB_fig1.png" alt="Img">

Let's say you want to run a test to determine whether warming and/or fertilisation affected seedling growth. You may know how your experiment is set up, but R doesn't! At the moment, with 8 measures per row (combination of all treatments and species for one replicate, or block), you cannot run an analysis. On the contrary, 
<a href="https://www.jstatsoft.org/article/view/v059i10">tidy datasets</a> are arranged so that each **row** represents an **observation** and each **column** represents a **variable**. In our case, this would look something like this:

<img src="{{ site.baseurl }}/img/SAB_fig2.png" alt="Img">

This makes a much longer dataframe row-wise, which is why this form is often called *long format*. Now if you wanted to compare between groups, treatments, species, etc, R would be able to split the dataframe correctly, as each grouping factor has its own column. 

The `gather()` function from the `tidyr` package lets you convert a wide-format table to a tidy dataframe. 


``` r
install.packages("tidyr") # install the package
library(tidyr) # load the package

elongation <- read.csv("EmpetrumElongation.csv", sep = ";") # load the data (annual stem growth of crowberry on sand dunes)
head(elongation) # preview the data; notice how the annual measurements are in different columns even though they represent a same variable (growth)?

elongation_long <- gather(elongation, Year, Length, c(X2007, X2008, X2009, X2010, X2011, X2012)) #gather() works like this: (data, key, value, columns to gather). Here we want the lengths (value) to be gathered by year (key). Note that you are completely making up the names of the second and third arguments, unlike most functions in R.

elongation_wide <- spread(elongation_long, Year, Length) # spread() is the reverse function, allowing you to go from long to wide format.

```

However, these functions will not work on any data structure - to quote <a href="https://www.jstatsoft.org/article/view/v059i10">Hadley Wickham</a>, "every messy dataset is messy in its own way". This is why giving a bit of thought to your dataset structure *before* doing your digital entry can spare you a lot of frustration later! 


<a name="dplyr"></a>

## 2. Use efficient tools for manipulating your data 

You have a nice and tidy dataset imported in R, but there are still things you want to change? We will take a look at the package `dplyr`, which is formed of a few simple, yet powerful functions to manipulate your data. Today we will take a look at **`filter()`**, **`select()`**, **`mutate()`**, **`summarise()`**, and **`group_by()`**. 


**`filter()`** is a subsetting function that allows you to select only certain **rows** in your dataframe.


``` r
install.packages("dplyr") # install the package
library(dplyr) # load the package

germination <- read.csv("Germination.csv", sep = ";") # load the data (germination of seeds subjected to different toxic solutions)
head(elongation) # preview the data

For instance, say we only want the observations that were made for the species "SR"

germinSR <- filter(germination, Species == 'SR')

```

Once you learn the syntax of dplyr, it makes the code much more easily readable than if you had subsetted the same way using base R:

``` r
germinSR <- germination[germination$Species == 'SR', ]
# This is completely equivalent but soon becomes hard to read if you add other conditions or have long variable names

```

And you can subset for different criteria at once:

``` r
germinSR10 <- filter(germination, Species == 'SR', Nb_seeds_germin >= 10)
#What does this do?

```
The code chunk above gives you the data for the species SR, only for observations where the number of germinated seeds was greater than 10.


The equivalent of filter() for columns is **`select()`**: it will keep only the variables you specify.

``` r
germin_clean <- select(germination, Species, Treatment, Nb_seeds_germin)
# This keeps only three columns in the dataframe - useful for cleaning out large files 

```
The **`mutate()`** function lets you create a new column, which is particularly useful if you want to create a variable that is a function of other variables in the dataset. For instance, let's calculate the germination percentage using the total number of seeds and the number of seeds that germinated. (Tip: you can simply give the column a name inside the function; if you don't, it will be called 'Var1' and you will have to rename it later.)

``` r
germin_percent <- mutate(germination, Percent = Nb_seeds_germin / Nb_seeds_tot * 100)

```

Another great function is **`summarise()`**, which lets you calculate summary statistics for your data. This will always return a dataframe shorter than the initial one and it is particularly useful when used with grouping factors (more on that in a minute). Let's just calculate the overall average germination percentage for now:

``` r
germin_average <- summarise(germin_percent, Germin_average = mean(Percent))
```
That's great, but that does not tell us anything about differences in germination between species and treatments. We can use the **`group_by()`** function to tell dplyr to create different subsets of the data (e.g. for different sites, species, etc.) and to apply functions to each of these subsets. 

``` r
germin_grouped <- group_by(germin_percent, Species, Treatment) # this does not change the look of the dataframe but there is some grouping done behind the scenes
germin_summary <- summarise(germin_grouped, Average = mean(Percent))
```
What's different? Here you should end up with 6 means instead of one (one for each Species x Treatment combination). This is a simple example, but dplyr is very powerful and lets you deal with complex datasets in an intuitive way, especially when combined with piping. 


<a name="piping"></a>

## 3. Learn a new syntax for coding : pipes </a>

Piping is an efficient way of writing chains of commands in a linear way, feeding the output of the first step into the second and so on, until the desired final output. It eliminates the need to create several temporary objects just to get to what you want, and you can write the code as you think it, instead of having to "think backwards". 

The pipe operator is `%>%`. A pipe chain always starts with your initial **dataframe** (others objects are not allowed), and then you apply a suite of functions. You don't have to call the object every time (which means you can drop the first argument of the dplyr functions, or the `data = "yourdata"` argument of most functions). With pipes, the result of a function will be fed to the next command. 

Let's see how it works using the same data we worked through with dplyr, and see how we can get to that final summary more quickly.
``` r
germin_summary <- germination %>%  # this is the dataframe 
  			mutate(Percent = Nb_seeds_germin/Nb_seeds_tot * 100) %>% # we are creating the percentage variable
  			group_by(Species, Treatment) %>% # introducing the grouping levels
  			summarise(Average = mean(Percent),
           			  SD = sd(Percent)) # calculating the summary stats; you can do several at once
```
This saves the need for the temporary objects we created earlier (`germin_percent`, `germin_grouped`) and the code reads step-by-step. 

Those were simple examples to get you started, but there are plenty of excellent online resources if you want to dig further - see  <a href="#links"> links </a> below.


### Tutorial Outcomes:

#### 1. You understand the format required for analyses in R, and can use the package `tidyr` to achieve it.

#### 2. You can manipulate data using the `dplyr` package.

#### 3. You can use pipes to make your code more efficient and readable.



#### We would love to hear your feedback on the tutorial, whether you did it in the classroom or online:

[https://www.surveymonkey.co.uk/r/F5PDDHV](https://www.surveymonkey.co.uk/r/F5PDDHV)


<a name="links"></a> 
## Links:
<a href="https://www.rstudio.com/wp-content/uploads/2015/02/data-wrangling-cheatsheet.pdf"> Data Wrangling in R – Cheat sheet for tidyr and dplyr </a>

<a href="https://dynamicecology.wordpress.com/2016/08/22/ten-commandments-for-good-data-management/"> Brian McGill - Ten Commandments for Good Data Management </a>


