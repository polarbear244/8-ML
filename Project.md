---
title: "Practical Machine Learning Project"
output: html_document
---

**Synopsis**

In this project, we'll use accelerometer data to develop a prediction model that will tell us how a certain user is lifting weights.

- **Class A**: Lifting weights according to specification.
- **Class B**: Throwing the elbow forward.
- **Class C**: Lifting the dumbbell halfway.
- **Class D**: Lowering the dumbbell halfway.
- **Class E**: Throwing the hips forward.

**Getting the Data**

We will use the files `pml-training.csv` for training and `pml-testing.csv` for predictions. If the files are not present, they will be downloaded.

```{r}
# Download and read the training data
if (!file.exists("pml-training.csv")) {
  download.file("https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv", "pml-training.csv", method = 'curl')
}
training_data <- read.csv("pml-training.csv", na.strings = c("NA", ""))

# Download and read the testing data
if (!file.exists("pml-testing.csv")) {
  download.file("https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv", "pml-testing.csv", method = 'curl')
}
validation_data <- read.csv("pml-testing.csv")
```

### Data Preprocessing

Load necessary libraries and set a seed for reproducibility.

```{r}
library(caret)
library(randomForest)

set.seed(17)
```

Partition the training data into training and testing sets.

```{r}
inTrain <- createDataPartition(y = training_data$classe, p = 0.7, list = FALSE)
training_set <- training_data[inTrain, ]
testing_set <- training_data[-inTrain, ]

```

Take out of the training, validation, and testing datasets any columns that have NA values or are superfluous.

```{r}
# Function to clean data
clean_data <- function(data) {
  na_columns <- sapply(data, function(x) sum(is.na(x)))
  columns_with_na <- names(na_columns[na_columns > 0])
  data <- data[, !names(data) %in% columns_with_na]
  data <- data[, !names(data) %in% c("X", "user_name", "raw_timestamp_part_1", "raw_timestamp_part_2", "cvtd_timestamp", "new_window", "num_window")]
  return(data)
}

# Clean datasets
training_set <- clean_data(training_set)
testing_set <- clean_data(testing_set)
validation_data <- clean_data(validation_data)

```

### Model Building and Evaluation

Train the Random Forest model and evaluate its performance.

```{r}
model <- randomForest(classe ~ ., data = training_set, ntree = 50)
predictions <- predict(model, testing_set)
confusion_matrix <- confusionMatrix(predictions, testing_set$classe)
model_accuracy <- confusion_matrix$overall["Accuracy"]

```

Report the model accuracy

```{r}
cat("Our model is", round(model_accuracy * 100, 2), "% accurate.\n")

```

Predict the classes for the validation set

```{r}
validation_predictions <- predict(model, validation_data)
validation_predictions

```
