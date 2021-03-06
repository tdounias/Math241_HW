hw\_3\_tidy
================
Theodore Dounias
February 27, 2017

#### PART 1

**Exercise 5.1**

First we wrangle the data:

``` r
cubs_hr <- Teams %>%
  filter(teamID == "CHN") %>%
  select(yearID, HR, HRA) %>%
  group_by(yearID) %>%
  summarize(HR = sum(HR), HRA = sum(HRA))
```

Then we tidy the data set into narrow format:

``` r
cubs_hr <- cubs_hr %>% 
  gather(key = scored, value = number, HR, HRA)
```

And finally make our plot:

``` r
ggplot(cubs_hr, aes(x = yearID, y = number)) +
  geom_line(aes(col = scored)) +
  labs(x = "Year", y = "Number of Home Runs", title = "Chicago Cubs Home Runs 1876 to 2015")
```

![](hw_3_tidy_files/figure-markdown_github/unnamed-chunk-4-1.png)

**Exercise 5.6**

We create the table (which I couldn't find otherwise):

``` r
ds1 <- data.frame(id = c(1,2,3,1,2,3), 
                  group = c("T", "T", "T", "C", "C", "C"), 
                  vals = c(4.00, 6.00, 8.00, 5.00, 6.00, 10.00))
```

And then run the desired code:

``` r
Treat <- filter(ds1, group == "T")
Control <- filter(ds1, group == "C")
all <- mutate(Treat, diff = Treat$vals - Control$vals)
```

This can lead to two issues. First, it assumes that the treat and control groups are all already perfectly matching by id. Second, if one person's data is missing, then all the subsequent diffs are going to be wrong. A better way to do this is by using the spread function to create a single data frame and then use the mutate command.

``` r
tb <- ds1 %>%
  spread(key = group, value = vals) %>%
  mutate(diff = T - C)
```

**Exercise 5.7**

We first create the dataset:

``` r
ds57 <- data.frame(grp = c("A", "A", "B", "B"),
                   sex = c("F", "M", "F", "M"),
                   meanL = c(.22, .47, .33, .55),
                   sdL = c(.11, .33, .11, .31),
                   meanR = c(.34, .57, .40, .65),
                   sdR = c(.08, .33, .07, .27))
```

And then use gather, separate, and unite to display it in the desired format:

``` r
ds57 %>%
  gather(type, val, 3:6) %>%
  unite(type, sex, type, sep = ".") %>%
  spread(type, val)
```

    ##   grp F.meanL F.meanR F.sdL F.sdR M.meanL M.meanR M.sdL M.sdR
    ## 1   A    0.22    0.34  0.11  0.08    0.47    0.57  0.33  0.33
    ## 2   B    0.33    0.40  0.11  0.07    0.55    0.65  0.31  0.27

#### PART II

First we read the files into r, and create the voter\_sample variable. We cache this so as to not have to repeat the process:

``` r
#Read tab delineated files
cd1 <- read_tsv("C:/Users/tdounias/Desktop/Reed College/Spring 2017/MATH 241/Repositories/theodore_dounias/hw_3_tidy/data/CD1_VoterHistory_Jan2017.txt")
cd2 <- read_tsv("C:/Users/tdounias/Desktop/Reed College/Spring 2017/MATH 241/Repositories/theodore_dounias/hw_3_tidy/data/CD2_VoterHistory_Jan2017.txt")
cd3 <- read_tsv("C:/Users/tdounias/Desktop/Reed College/Spring 2017/MATH 241/Repositories/theodore_dounias/hw_3_tidy/data/CD3_VoterHistory_Jan2017.txt")
cd4 <- read_tsv("C:/Users/tdounias/Desktop/Reed College/Spring 2017/MATH 241/Repositories/theodore_dounias/hw_3_tidy/data/CD4_VoterHistory_Jan2017.txt")
cd5 <- read_tsv("C:/Users/tdounias/Desktop/Reed College/Spring 2017/MATH 241/Repositories/theodore_dounias/hw_3_tidy/data/CD5_VoterHistory_Jan2017.txt")

#Put all files into one dataframe:
ovr_file_large <- rbind(cd1, cd2, cd3, cd4, cd5)

#Delete initial variables to conserve memory and p-power:
rm(cd1, cd2, cd3, cd4, cd5)

#Cut out unnecessary columns:
ovr_file_large <- ovr_file_large %>%
  select(VOTER_ID, BIRTH_DATE, EFF_REGN_DATE, CONFIDENTIAL, 
         STATUS, PARTY_CODE, COUNTY, 41:65)

#Set seed and take voter sample:
set.seed(251981)
voter_sample <- sample_n(ovr_file_large, size = 10^6)

#Delete now unnecessary file:
rm(ovr_file_large)
```

Then we use a gather command to gather all election days into one variable, called ELECTIONS, and if the person voted into anither one called VOTE. We arrange by VOTER\_ID for readability:

``` r
voter_sample <- voter_sample %>%
  gather(key = ELECTION, value = VOTE, 8:32) %>%
  arrange(VOTER_ID)
```

In this way we have our finalized voter sample file. We could do several more steps for further tidiness. For example, we could replace the dash values in VOTE with NA values as such:

``` r
voter_sample$VOTE[voter_sample$VOTE == "-"] <- NA
```

We could also change the variables to factors, or edit the dates so they can be three different variables each. For this exercise what was required was only the above, however. If we wanted to we could use multiple as.factor() funcions to, for example, make the counties into factors.
