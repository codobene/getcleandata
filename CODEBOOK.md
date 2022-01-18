The run_analysis.R script performs all steps necessary to download the raw data and to process them in order to obtain a clean, or tidy, dataset.


# 0. Loading dependencies

library(data.table)


# 1. Downloading the raw data

> fileurl = 'https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip'
> if (!file.exists('./UCI HAR Dataset.zip')){
>   download.file(fileurl,'./UCI HAR Dataset.zip', mode = 'wb')
>   unzip("UCI HAR Dataset.zip", exdir = getwd())
> }


# 2. Reading and converting the data

> 
> features <- read.csv('./UCI HAR Dataset/features.txt', header = FALSE, sep = ' ')
> features <- as.character(features[,2])
> 
> data.train.x <- read.table('./UCI HAR Dataset/train/X_train.txt')
> data.train.activity <- read.csv('./UCI HAR Dataset/train/y_train.txt', header = FALSE, sep = ' ')
> data.train.subject <- read.csv('./UCI HAR Dataset/train/subject_train.txt',header = FALSE, sep = ' ')
> 
> data.train <-  data.frame(data.train.subject, data.train.activity, data.train.x)
> names(data.train) <- c(c('subject', 'activity'), features)
> 
> data.test.x <- read.table('./UCI HAR Dataset/test/X_test.txt')
> data.test.activity <- read.csv('./UCI HAR Dataset/test/y_test.txt', header = FALSE, sep = ' ')
> data.test.subject <- read.csv('./UCI HAR Dataset/test/subject_test.txt', header = FALSE, sep = ' ')
> 
> data.test <-  data.frame(data.test.subject, data.test.activity, data.test.x)
> names(data.test) <- c(c('subject', 'activity'), features)
> 
> data.all <- rbind(data.train, data.test)
> mean_std.select <- grep('mean|std', features)
> data.sub <- data.all[,c(1,2,mean_std.select + 2)]
> 

# 3. Reading the labels from the activity_labels.txt file

> 
> activity.labels <- read.table('./UCI HAR Dataset/activity_labels.txt', header = FALSE)
> activity.labels <- as.character(activity.labels[,2])
> data.sub$activity <- activity.labels[data.sub$activity]
> 

# 4. Replacing the names in data set with names from activity labels

> 
> name.new <- names(data.sub)
> name.new <- gsub("[(][)]", "", name.new)
> name.new <- gsub("^t", "TimeDomain_", name.new)
> name.new <- gsub("^f", "FrequencyDomain_", name.new)
> name.new <- gsub("Acc", "Accelerometer", name.new)
> name.new <- gsub("Gyro", "Gyroscope", name.new)
> name.new <- gsub("Mag", "Magnitude", name.new)
> name.new <- gsub("-mean-", "_Mean_", name.new)
> name.new <- gsub("-std-", "_StandardDeviation_", name.new)
> name.new <- gsub("-", "_", name.new)
> names(data.sub) <- name.new
> 

# 5. Tidy data = output as data_tidy.txt file

> 
> data.tidy <- aggregate(data.sub[,3:81], by = list(activity = data.sub$activity, subject = data.sub$subject),FUN = mean)
> write.table(x = data.tidy, file = "data_tidy.txt", row.names = FALSE)
> 