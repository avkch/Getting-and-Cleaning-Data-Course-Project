Getting and Cleaning Data Course Project
================
avkch
16 October 2016

#### Preparing the data

Download the data from the web and saves it in the working directory as **HARdataset.zip**

``` r
fileURL <- "https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip"
download.file(fileURL, destfile = "HARdataset.zip")
```

unzip the downloaded data set

``` r
unzip("HARdataset.zip")
```

Reading the subject information from **subject\_train.txt** and **subject\_test.txt** files and assigning them to variables `subjtrain` and `subjtest`. Both variables are data.frame objects with one column holding the ID nubers of the subjects participating in the study.

``` r
subjtrain <- read.table("UCI HAR Dataset/train/subject_train.txt", col.names = "subject")
subjtest <- read.table("UCI HAR Dataset/test/subject_test.txt", col.names = "subject")
```

Reading the activity information from **y\_train.txt** and **y\_test.txt** files and assigning them to variables `activitytrain` and `activitytest`. Both variables are data.frame objects with one column holding the index number of the activities.

``` r
activitytrain <- read.table("UCI HAR Dataset/train/y_train.txt", col.names = "activity")
activitytest <- read.table("UCI HAR Dataset/test/y_test.txt", col.names = "activity")
```

Reading the activity information from **X\_train.txt** and **X\_test.txt** files and assigning them to variables `datatrain` and `datatest`. Both are data.frame objects with 561 columns holding the measurments data.

``` r
datatrain <- read.table("UCI HAR Dataset/train/X_train.txt")
datatest <- read.table("UCI HAR Dataset/test/X_test.txt")
```

#### 1. Merges the training and the test sets to create one data set.

Binding both datatrain and datatest dataframes in one dataframe called alldata.

``` r
alldata <- rbind(datatrain, datatest)
```

Rading the **features.txt** file which contains information about the measurements taken (names of the measurements) and stores the data in `features` data.frame with has 2 columns the first is the number of the measurement and the second the name of the measurement.

``` r
features <- read.table("UCI HAR Dataset/features.txt")
```

Assigns the measurement names from `features` as column names of `alldata`.

``` r
names(alldata) <- features$V2
```

#### 2. Extracts only the measurements on the mean and standard deviation for each measurement.

Extracts only the measurements on the mean and standard deviation for each measurement from `alldata`. Searches with grep in the names of the columns tor "mean(" and "std(" the "(" is spetial symbos and should be escaped with two \\ in front. Assigns the result to `extrmeansd` vector which contains the indexes of the columns selectedfirst are the mean indexes followed by the std indexes.

``` r
extrmeansd <- c(grep("mean\\(",names(alldata)),grep("std\\(", names(alldata)))
```

Reorders `extrmeansd` vector so the column indexes extracted are in the same order as in `alldata`.

``` r
extrmeansd <- extrmeansd[order(extrmeansd)]
```

Binding the columns from `alldata` with mean and std in their names (their indecis are in `extrmeansd` vector) together in new data.frame `extracteddata`.

``` r
extracteddata <- alldata[,extrmeansd]
```

#### 3. Uses descriptive activity names to name the activities in the data set

Binding both `subjtrain` and `subjtest` dataframes in one data.frame called `allsubjects`.

``` r
allsubjects <- rbind(subjtrain, subjtest)
```

Binding both `activitytrain` and `activitytest` dataframes in one data.frame called `allactivity`.

``` r
allactivity <- rbind(activitytrain, activitytest)
```

Binds `allsubjects` and `allactivity`to `extracteddata` as last two columns.

``` r
extracteddata <- cbind(alldata[,extrmeansd], allsubjects, allactivity)
```

Rading data from **activity\_labels.txt** and assigning the data to `actlabels` data.frame which is with 2 columns first is the activity index used in activity data and the second is the corresponding activity name.

``` r
actlabels <- read.table("UCI HAR Dataset/activity_labels.txt")
```

Using `factor()` function substitutes the activity indexes with their corresponding names and assigns them back to activity column in `extracteddata` dataframe.

``` r
extracteddata$activity <- factor(extracteddata$activity, labels = actlabels$V2)
```

#### 4. Appropriately labels the data set with descriptive variable names.

The variables are already labeled in step 1.

#### 5. From the data set in step 4, creates a second, independent tidy data set with the average of each variable for each activity and each subject.

Load `data.table` package

``` r
library(data.table)
```

Converts `extracteddata` to data.table with name `dtalldata`.

``` r
dtalldata <- data.table(extracteddata)
```

Makes new data.table called `tidydata` by averaging of the each variable for each activity and each subject. Is not subsetting the data (no "i"), is calculating mean for all the columns (.SD = all columns exept ones used in "by" argument) using `lapply` in "j", calculating by two groups(columns activity and subject) in "by" using .()

``` r
tidydata <- dtalldata[,lapply(.SD,mean), by = .(activity, subject)]
```

Writing the tidydata set in **titydata.txt** file.

``` r
write.table(tidydata, file = "tidydata.txt", row.names = FALSE)
```
