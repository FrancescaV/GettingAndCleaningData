# Step by step description of the script run_analysis.R

Note, Rmarkdown would have been much better for this.....

The R commands used in the script are in italic.


### 1. Merge the training and the test sets to create one data set.

I want to Combine the \*.txt files in test/ and train/ folders (not in Inertial Signals/)

- loading the data into table frames.

*test_sub <-read.table("test/subject_test.txt", header = FALSE)*

*test_X <- read.table("test/X_test.txt", header = FALSE)*

*test_Y <- read.table("test/Y_test.txt", header = FALSE)*

*train_sub <- read.table("train/subject_train.txt", header = FALSE)*

*train_X <- read.table("train/X_train.txt", header = FALSE)*

*train_Y <- read.table("train/Y_train.txt", header = FALSE)*

- Changing the name of the columns to subject_\*.txt and Y_\*.txt.

*colnames(test_sub) <- c("subject")*

*colnames(train_sub) <- c("subject")*

*colnames(test_Y) <- c("activity")*

*colnames(train_Y) <- c("activity")*


- Binding the test/ and train/ data by columns separately

*dat_test <- cbind(test_X,test_Y)*

*dat_test <- cbind(dat_test,test_sub)*

*dat_train <- cbind(train_X,train_Y)*

*dat_train <- cbind(dat_train,train_sub)*


- Adding a "type" column to denote test and train data (not required in Homework), but still good practice.

*library(dplyr)*

*dat_test <- mutate(dat_test, type = "test")*

*dat_train <- mutate(dat_train, type = "train")*

- Binding the train/ and test/ datasets by row

*dat <- rbind(dat_test, dat_train)*

###    2. Extracts only the measurements on the mean and standard deviation for each measurement.

-Loading the "features.txt" file into a data frame.

*features <- read.table("features.txt", header = FALSE,stringsAsFactors=FALSE)*

-Looking for the features containing mean() and std() in the second column. The escape "\\" is needed for the parenthesis "()"

*feat_mean <- features[grep("mean\\(", features$V2),]*

*feat_std <- features[grep("std\\(", features$V2),]*

- binding them together by row and arranging them in increasing V1

*feat_meanStd <- rbind(feat_mean, feat_std)*

*feat_meanStd <- arrange(feat_meanStd,V1)*

- sub-setting the data containing columns with mean and std PLUS also the columns with "activity", "subject", and "type" (columns 562,563,564).

*dat_meadStd <- dat[, c(feat_meanStd$V1,562,563,564)]*

###    3. Uses descriptive activity names to name the activities in the data set

- storing "activity_labels.txt" into a data.frame

*act_labels <- read.table("activity_labels.txt", header = FALSE,stringsAsFactors=FALSE)*


- substituting the labels with names in my data-set:

*for (i in act_labels$V1){*

*dat_meadStd <- within(dat_meadStd,activity[activity==i] <- act_labels$V2[i])*

*}*

###    4. Appropriately labels the data set with descriptive variable names.

- Here I use the data frame "feat_meanStd" created above plut the last 3 columns which I add by hand

*colnames(dat_meadStd) <- c(feat_meanStd$V2, "activity", "subject", "type")*

- Get rid of "-" and "()". For this I use grep to substitute characters

*names(dat_meadStd) <- gsub("-m", "M", names(dat_meadStd))*

*names(dat_meadStd) <- gsub("-s", "S", names(dat_meadStd))*

*names(dat_meadStd) <- gsub("\\(\\)", "", names(dat_meadStd)) #this gets rid of parenthesis*

*names(dat_meadStd) <- gsub("-", "", names(dat_meadStd))*

*names(dat_meadStd)*

###    5. From the data set in step 4, creates a second, independent tidy data set with the average of each variable for each activity and each subject.

- Use aggregate() to aggregate the data by "subject", "activity", and "type" (test or train).

- I then compute the mean for each column in the data set, but the last 3 which contain "subject", "activity", and "type".

*new_dataSet <- aggregate(dat_meadStd[,1:(ncol(dat_meadStd)-3)], by=list(subject = dat_meadStd$subject, activity = dat_meadStd$activity, type = dat_meadStd$type), mean)*

*new_dataSet[,4:ncol(new_dataSet)] <- signif(new_dataSet[,4:ncol(new_dataSet)], digits=5)*

- Save the dataset in a text file:

*write.table(new_dataSet, "myTidyDataSet_progAssignment.txt", sep=",", row.name=FALSE )*

- spring cleaning

*rm(list=ls())*
