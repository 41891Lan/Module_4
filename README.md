# Module_4
7316 Assignment 4
41891 Lan Yao



library(rio)
library(tidyverse)
getwd()
setwd("E:/SSE_semester2/7316 Intro to R/Assignment/A4_41891/Module_4")

#import the data sets *basic.dta* and *genold108.dta*
basic<-import("E:/SSE_semester2/7316 Intro to R/Assignment/A4_41891/Module_4/basic.dta")
genold108<-import("E:/SSE_semester2/7316 Intro to R/Assignment/A4_41891/Module_4/genold108.dta")


#create a subset of the 108th congress from the *basic* dataset

subset108<-subset(basic,basic$congress=="108")

#join this subset with the *genold* dataset

joined_data<-merge(subset108,genold108,by ="name")


# Data preparation

#check table 1 in the appendix of the paper and decide which variables are necessary for the analysis (check the footnote for control variables)
#drop all other variables.
analysis<-select(joined_data,anygirls,rgroup,region,female,party,white,demvote,age,srvlng,ngirls,totchi,genold)


#Recode *genold* such that gender is a factor variable and missing values are coded as NAs.
#Recode *party* as a factor with 3 levels (D, R, I)
#Recode *rgroup* and *region* as factors.

analysis <- analysis %>% mutate(genold = ifelse(genold == "", NA, genold)) %>% mutate(genold = as.factor(genold)) 
analysis$female<-as.factor(analysis$female)
is.na(analysis)

party_levels <- c("D", "R", "I")
analysis$party<-as.factor(analysis$party)
levels(analysis$party) <- c("D", "R", "I")
analysis$rgroup<-as.factor(analysis$rgroup)
analysis$region<-as.factor(analysis$region)

#generate variables for age squared and service length squared
#create an additional variable of the number of children as factor variable

analysis <- mutate(analysis,age_square=age^2,srvling_square=srvlng^2)
analysis <- mutate(analysis,nochildren=totchi)
analysis$nochildren<-as.factor(analysis$nochildren)

# Replicationg Table 1 from the Appendix

#We haven't covered regressions in R yet. Use the function *lm()*. The function takes the regression model (formula) and the data as an input. The model is written as $y \sim x$, where $x$ stands for any linear combination of regressors (e.g. $y \sim x_1 + x_2 + female$). Use the help file to understand the function.

fit <- lm(totchi ~ genold+female, data=analysis)
summary(fit)


#Run the regression $total.children = \beta_0 + \beta_1 gender.oldest + \gamma'X$ where $\gamma$ stands for a vector of coefficients and $X$ is a matrix that contains all columns that are control variables.\footnote{This is just a short notation instead of writing the full model with all control variables $totchi = \beta_0 + \beta_1 genold + \gamma_1 age + \gamma_2 age^2 + \gamma_3 Democrat + ... + \epsilon$ which quickly gets out of hand for large models.}
#Save the main coefficient of interest ($\beta_1$)

fit2 <- lm(totchi ~ genold+female+region+female+party+white+demvote+age+srvlng+age_square+srvling_square, data=analysis)
summary(fit2)
summary(fit2)$coefficients
nobs_main<-nobs(fit2)
beta_1_main<-summary(fit2)$coefficients[2,1]
standard_main<-summary(fit2)$coefficients[2,2]

fit2_girls <- lm(ngirls ~ genold+female+region+female+party+white+demvote+age+srvlng+age_square+srvling_square, data=analysis)
summary(fit2_girls)
summary(fit2_girls)$coefficients
nobs_girls_main<-nobs(fit2_girls)
beta_1_girls_main<-summary(fit2_girls)$coefficients[2,1]
standard_girls_main<-summary(fit2_girls)$coefficients[2,2]


#Run the same regression separately for Democrats and Republicans (assign the independent to one of the parties). Save the coefficient and standard error of *genold*

fit3_Dem <- lm(totchi ~ genold+female+region+female+party+white+demvote+age+srvlng+age_square+srvling_square, data=subset(analysis,party="D"))
summary(fit3_Dem)
summary(fit3_Dem)$coefficients
nobs_Dem<-nobs(fit3_Dem)
beta_1_Dem<-summary(fit3_Dem)$coefficients[2,1]
standard_Dem<-summary(fit3_Dem)$coefficients[2,2]

fit3_girls_Dem <- lm(ngirls ~ genold+female+region+female+party+white+demvote+age+srvlng+age_square+srvling_square, data=subset(analysis,party="D"))
summary(fit3_girls_Dem)
summary(fit3_girls_Dem)$coefficients
nobs_girls_Dem<-nobs(fit3_girls_Dem)
beta_1_girls_Dem<-summary(fit3_girls_Dem)$coefficients[2,1]
standard_girls_Dem<-summary(fit3_girls_Dem)$coefficients[2,2]

fit4_Rep <- lm(totchi ~ genold+female+region+female+party+white+demvote+age+srvlng+age_square+srvling_square, data=subset(analysis,party="R"))
summary(fit4_Rep)
summary(fit4_Rep)$coefficients
nobs_Rep<-nobs(fit4_Rep)
beta_1_Rep<-summary(fit4_Rep)$coefficients[2,1]
standard_Rep<-summary(fit4_Rep)$coefficients[2,2]

fit4_girls_Rep <- lm(ngirls ~ genold+female+region+female+party+white+demvote+age+srvlng+age_square+srvling_square, data=subset(analysis,party="R"))
summary(fit4_girls_Rep)
summary(fit4_girls_Rep)$coefficients
nobs_Rep<-nobs(fit4_girls_Rep)
beta_1_girls_Rep<-summary(fit4_girls_Rep)$coefficients[2,1]
standard_girls_Rep<-summary(fit4_girls_Rep)$coefficients[2,2]

#Collect all the *genold* coefficients from the six regressions, including their standard errors and arrange them in a table as in the paper.
#print the table

table<- matrix(c(beta_1_girls_main,beta_1_main,beta_1_girls_Dem,beta_1_Dem,beta_1_girls_Rep,beta_1_Rep,standard_girls_main,standard_main,standard_girls_Dem,standard_Dem,standard_girls_Rep,standard_Rep,nobs_main,nobs_main,nobs_Dem,nobs_Dem,nobs_Rep,nobs_Rep),ncol=6,byrow=TRUE)
colnames(table) <- c("Number of daughters","Number of Children","Number of daughters","Number of Children","Number of daughters","Number of Children")
rownames(table) <- c("first child","female","No")
table <- as.table(table)
print(table)