**Description**  
Additional information about the variables, data and transformations used in the course project for the Johns Hopkins Getting and Cleaning Data course.

**Source Data**  
Data + Description can be found here http://archive.ics.uci.edu/ml/datasets/Human+Activity+Recognition+Using+Smartphones

**Data Set Information**  
The experiments have been carried out with a group of 30 volunteers within an age bracket of 19-48 years. Each person performed six activities (WALKING, WALKING_UPSTAIRS, WALKING_DOWNSTAIRS, SITTING, STANDING, LAYING) wearing a smartphone (Samsung Galaxy S II) on the waist. Using its embedded accelerometer and gyroscope, we captured 3-axial linear acceleration and 3-axial angular velocity at a constant rate of 50Hz. The experiments have been video-recorded to label the data manually. The obtained dataset has been randomly partitioned into two sets, where 70% of the volunteers was selected for generating the training data and 30% the test data.

The sensor signals (accelerometer and gyroscope) were pre-processed by applying noise filters and then sampled in fixed-width sliding windows of 2.56 sec and 50% overlap (128 readings/window). The sensor acceleration signal, which has gravitational and body motion components, was separated using a Butterworth low-pass filter into body acceleration and gravity. The gravitational force is assumed to have only low frequency components, therefore a filter with 0.3 Hz cutoff frequency was used. From each window, a vector of features was obtained by calculating variables from the time and frequency domain.

**Attribute Information**
For each record in the dataset it is provided:  
  - Triaxial acceleration from the accelerometer (total acceleration) and the estimated body acceleration.  
  - Triaxial Angular velocity from the gyroscope.
  - A 561-feature vector with time and frequency domain variables.  
  - Its activity label.  
  - An identifier of the subject who carried out the experiment.  
  
**Attribute Information**
For each record in the dataset it is provided:  
  - Triaxial acceleration from the accelerometer (total acceleration) and the estimated body acceleration.  
  - Triaxial Angular velocity from the gyroscope.
  - A 561-feature vector with time and frequency domain variables.  
  - Its activity label.  
  - An identifier of the subject who carried out the experiment. 
  
**The variables, the data**  
  *libraries*
  - "data.table", "reshape2"  
sapply(packages, require, character.only=TRUE, quietly=TRUE)  
path <- getwd()  
  *downloading data*
  - url <- "https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip"  
download.file(url, file.path(path, "dataFiles.zip"))  
unzip(zipfile = "dataFiles.zip")  

  *activity labels + features*  
  - activityLabels <- fread(file.path(path, "UCI HAR Dataset/activity_labels.txt"), col.names = c("classLabels", "activityName"))  
  - features <- fread(file.path(path, "UCI HAR Dataset/features.txt"), col.names = c("index", "featureNames"))  
  - featuresWanted <- grep("(mean|std)\\(\\)", features[, featureNames])  
  - measurements <- features[featuresWanted, featureNames]  
  - measurements <- gsub('[()]', '', measurements)  
  
  *train datasets*  
  - train <- fread(file.path(path, "UCI HAR Dataset/train/X_train.txt"))[, featuresWanted, with = FALSE]  
  - data.table::setnames(train, colnames(train), measurements)  
  - trainActivities <- fread(file.path(path, "UCI HAR Dataset/train/Y_train.txt"), col.names = c("Activity"))  
  - trainSubjects <- fread(file.path(path, "UCI HAR Dataset/train/subject_train.txt")                  , col.names = c("SubjectNum"))  
  - train <- cbind(trainSubjects, trainActivities, train)  

  *test datasets*  
  - test <- fread(file.path(path, "UCI HAR Dataset/test/X_test.txt"))[, featuresWanted, with = FALSE]  
  - data.table::setnames(test, colnames(test), measurements)  
  - testActivities <- fread(file.path(path, "UCI HAR Dataset/test/Y_test.txt")                    , col.names = c("Activity"))  
  - testSubjects <- fread(file.path(path, "UCI HAR Dataset/test/subject_test.txt")                , col.names = c("SubjectNum"))  
  - test <- cbind(testSubjects, testActivities, test)  

  *merge datasets*  
  - combined <- rbind(train, test)  
  
**Convert classLabels to activityName basically.**  
combined[["Activity"]] <- factor(combined[, Activity], levels = activityLabels[["classLabels"]], labels = activityLabels[["activityName"]])  
combined[["SubjectNum"]] <- as.factor(combined[, SubjectNum])  
combined <- reshape2::melt(data = combined, id = c("SubjectNum", "Activity"))  
combined <- reshape2::dcast(data = combined, SubjectNum + Activity ~ variable, fun.aggregate = mean)  

*writing final result in file tidyData.txt*  
data.table::fwrite(x = combined, file = "tidyData.txt", quote = FALSE)  
