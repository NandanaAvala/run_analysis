run_analysis
============
#download data
startdir <- "/Volumes/DATA/Documents/Big Data/DataScience/Coursera/Data Science All 9 courses/3--Getting and Cleaning Data/"
setwd(startdir)

fileUrl <- "https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip"

if(!file.exists("./data.zip")){
download.file(fileUrl,destfile = "./data.zip",method = "curl")
}

datadir <- "projectdata"
unzip("data.zip",exdir = paste0("./",datadir,"/"))

setwd(paste0("./",datadir))

extracteddir <- (list.dirs(recursive = FALSE))[1]
setwd(paste0("./",extracteddir))
print("----Reached here in code-----")
#Create a combined directory

testdirlist <- list.dirs("./test",full.names = TRUE, recursive = TRUE)
combinedirlist <- gsub("test","combine",testdirlist)
combinedirlist
for(i in seq_along(testdirlist)){
if(!file.exists(combinedirlist[i])){dir.create(combinedirlist[i])}
}
testlist <- list.files("./test",pattern = ".txt",full.names = TRUE, recursive = TRUE)
testlist <- sort(testlist,decreasing = TRUE)
atest<-data.frame(testlist,tofile = gsub("test","combine",testlist),stringsAsFactors = FALSE)

trainlist <- list.files("./train",pattern = ".txt",full.names = TRUE, recursive = TRUE)
atrain<-data.frame(trainlist,tofile = gsub("train","combine",trainlist),stringsAsFactors = FALSE)
acombine <- merge(atest,atrain)


for(i in seq_along(acombine[,1]))
{
	if(!file.exists(acombine[i,1])){file.create(acombine[i,1])}
	df2 <- read.table(acombine[i,2])
	df3 <- read.table(acombine[i,3])
	df1 <- rbind(df2,df3)
	write.table(df1,acombine[i,1],col.names = FALSE, row.names = FALSE)
}
getwd()



dfcol_subject <- read.table("./combine/subject_combine.txt", stringsAsFactors = FALSE, col.names = "subject")
dfcol_activity <- read.table("./combine/y_combine.txt", stringsAsFactors = FALSE,col.names = "activity")

df_activitylabels <- read.table("./activity_labels.txt", stringsAsFactors = FALSE, col.names = c("activity","activitylabel"))

# CREATE COLUMNS FOR DESCRIPTIVE ACTIVITIES
dfcol_activitylabels <- merge(dfcol_activity, df_activitylabels)
# head(dfcol_activitylabels)

# CREATE DATATABLE TO GET VARIABLE NAMES
df_features <- read.table("./features.txt", col.names = c("varnum","varname"), stringsAsFactors = FALSE)

# ADD DESCRIPTIVE VARIABLE NAMES TO RAW DATA
df_rawdata <- read.table("./combine/X_combine.txt", stringsAsFactors = FALSE , col.names = df_features$varname)


# SELECT ONLY MEAN AND STANDARD DEVIATION VARIABLES
# ASSUMPTION : THOSE WHOSE NAMES CONTAIN "mean" or "std" are considered to satisfy the condition
xx <- grep("mean",tolower(df_features$varname))
xy <- grep("std",tolower(df_features$varname))
xselected <- df_features$varname[unique(c(xx,xy))]

df_selected <- df_rawdata[,df_features$varname %in% xselected]


# CREATE A DATA TABLE WITH DESCRIPTIVE ACTIVITY LABELS
df_final <- cbind(dfcol_subject, dfcol_activitylabels,df_selected)

library(dplyr)
tbl_final <- tbl_df(df_final)
tbl_final2 <- group_by(tbl_final,subject,activitylabel)
#tbl_tidy <- summarise(tbl_final2,mean(varnum))


Performing analysis
