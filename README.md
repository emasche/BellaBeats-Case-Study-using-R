---
title: "Bella Beats case study"
author: "Emma Schenegg"
date: "2025-07-19"
output:
  html_document: default
  word_document: default
---
```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```
Introduction

Business Context

  Bellabeat is a wellness technology company focused on creating smart products designed specifically for women. Their devices track physical       activity, sleep, stress, and reproductive health to promote holistic wellbeing. Bellabeat emphasizes combining stylish design with health         insights to support a balanced lifestyle. Their products serve as lifestyle partners, helping women monitor both mental and physical health. The   brand is known for female-centric features and empowering users through personalized wellness data. Sršen, co-founder of Bellabeat, has           requested an analysis of smart device usage data to gain insights into how consumers—especially women—use non-Bellabeat tracking devices. The     objective is to understand key user behaviors, identify trending features, and explore opportunities to improve Bellabeat’s product offerings     and marketing strategy.

Guiding Problem

  The primary problem identifying which trends are most relevant to Bellabeat’s target audience.
  The goal is to reduce internal bias by examining the behaviours of users from competing products and using those findings to:
  •	Improve Bellabeat’s existing product features
  •	Tailor marketing campaigns based on real consumer behaviour
  •	Provide stakeholders with evidence-based recommendations for product development
  Stakeholders
  •	Bellabeat co-founders
  •	The marketing team
  •	Product development and design teams

Datasets

  The data used in this analysis comes from the publicly available FitBit Fitness Tracker Data provided by Möbius on Kaggle. It includes            anonymized activity, sleep, heart rate, and weight data collected from 33 to 35 users over a period of 31 to 60 days between March and May 2016.   Four key datasets were utilized: dailyActivity_merged.csv (tracking total steps, activity intensity levels, sedentary minutes, and calories       burned), heartrate_seconds_merged.csv (second-by-second heart rate data in BPM), sleepDay_merged.csv (sleep duration, number of sleep records,    and time spent in bed), and weightLogInfo_merged.csv (user weight in kilograms, BMI, and manual/automatic entry type). The activity, heart rate,   and weight data were originally split across two separate datasets covering different time periods—12/03/2016 to 11/04/2016 and 12/04/2016 to     12/05/2016—and this analysis will later combine them using the rbind() function to ensure a continuous timeline for analysis. The sleep data is   already consolidated and did not require merging.
  
  
Business Task Statement

  To gain insights into trends in women’s behavior with competitor smart tracking devices in order to guide Bellabeat’s marketing strategy and      product development, helping the brand better connect with existing and potential customers.


# Install packages

```{r install-packages, eval=FALSE}
options(repos = c(CRAN = "https://cloud.r-project.org"))

install.packages("tidyverse") # (ggplot2, dplyr,tidyr,readr, stringr etc...)
install.packages("lubridate") # Manipulation of date/time
install.packages("corrplot")  # Correlation heat map
```

# Load packages

```{r load-packages}
library(tidyverse)
library(lubridate)
library(corrplot)
```

# Bind and load data sets

```{r Link-data-sets}
# Using rbind function to link the matching data from the 12/03/16-11/04/16 (dd/mm/yyyy) period to the 12/04/16-12/05/2016 period since they are located in different data sets.

Activity <- rbind(read.csv("C:/Users/emma3/Downloads/Fitabase Data 4.12.16-5.12.16/dailyActivity_merged2.csv"),read.csv("C:/Users/emma3/Downloads/mturkfitbit_export_3.12.16-4.11.16/dailyActivity_merged.csv"))

Weight <- rbind(read.csv("C:/Users/emma3/Downloads/Fitabase Data 4.12.16-5.12.16/weightLogInfo_merged.csv"),read.csv("C:/Users/emma3/Downloads/mturkfitbit_export_3.12.16-4.11.16/weightLogInfo_merged1.csv"))

Heartrate <- rbind(read.csv("C:/Users/emma3/Downloads/Fitabase Data 4.12.16-5.12.16/heartrate_seconds_merged.csv"), read.csv("C:/Users/emma3/Downloads/mturkfitbit_export_3.12.16-4.11.16/heartrate_seconds_merged1.csv"))

Sleep <- read.csv("C:/Users/emma3/Downloads/Fitabase Data 4.12.16-5.12.16/sleepDay_merged.csv")
 #There is only one Data set containing sleep data, so there is no need to use rbind function.
```

# Visualise Data

```{r head}
# Using head function to check columns and have a visuals on the data.

head(Activity)
head(Weight)
head(Heartrate)
head(Sleep)
```

# Inspecting Data Formats

```{r str}
# Using the str function to see if the formatting of columns are consistent and correct. 

str(Activity)
str(Weight)
str(Heartrate)
str(sleep) 

#Date and time are in characters rather than date format, time indicate AM/PM rather than 23:59:59 format. Date is in US style (mm/dd/yyyy) rather than in uk style (dd/mm/yyyy). Activity does not present time but only date, Sleep presents time but all indicate 00:00:00.
```

# Adjusting Dates

### Dates format
```{r Adjust-date-type}
# Using POSIXct to indicate the curing character string format in date format.
# Using Sis.Timezone to match the timezone data times are actually in 
# Formatting time in a UK format dd/mm/yyy HH:MM:SS for easier understanding and avoid any future confusion and saving it as "FormattedTime".

Activity$ActivityDate <- as.POSIXct(Activity$ActivityDate, format="%m/%d/%Y", tz= Sys.timezone())
Activity$FormattedTime <- format(Activity$ActivityDate, format = "%d/%m/%Y")
```


```{r Date-ampm}
# Using format %I/%M/%S %p because 12h format (AM/PM) for those data sets.

Weight$Date <- as.POSIXct(Weight$Date, format="%m/%d/%Y %I:%M:%S %p", tz= Sys.timezone())
Weight$FormattedTime <- format(Weight$Date, format = "%d/%m/%Y %H:%M:%S")

Heartrate$Time <- as.POSIXct(Heartrate$Time, format="%m/%d/%Y %I:%M:%S %p", tz= Sys.timezone())
Heartrate$FormattedTime <- format(Heartrate$Time, format = "%d/%m/%Y %H:%M:%S")

Sleep$SleepDay <- as.POSIXct(Sleep$SleepDay, format="%m/%d/%Y %H:%M:%S", tz= Sys.timezone())
Sleep$FormattedTime <- format(Sleep$SleepDay, format = "%d/%m/%Y") # Since Sleep Data set time are all 00:00:00, only the date is relevant and kept.
```

### Remove previous date variable

```{r Remove-previous-date}
# Remove previous US Date/time columns.

Activity <- Activity %>% select(-ActivityDate)
Weight <- Weight %>%  select(-Date)
Heartrate <- Heartrate %>% select(-Time)
Sleep <- Sleep %>% select(-SleepDay)
```

# Distinct values

```{r ndistinct}
# Using n_distinct function to summarise the number of unique participants (Id) in each data set.

n_distinct(Activity$Id)          #35
n_distinct(Weight$Id)            #13
n_distinct(Heartrate$Id)         #15
n_distinct(Sleep$Id)             #24
```

# Mean steps

```{r mean-steps}
# Check if users meets in average the recommended 10 000 steps a day (Tudor-Locke & Bassett, 2004).

mean(Activity$TotalSteps, na.rm = TRUE)

# Average steps = 7281, the mean doesn't reach daily steps recommendation
```

# Creating New Variables

### Total Active Minutes

```{r total-active-minutes}
# Create a column for total daily active minutes to group Lightly Active, Fairly Active and Very Active minutes.

Activity <- Activity %>%
  mutate(TotalActiveMinutes = LightlyActiveMinutes + FairlyActiveMinutes + VeryActiveMinutes)
```

### Awake Sedentary Time
```{r awake-sedentary-time}
# Create a column for sedentary time that doesn't take into account sleeping time and where sleeping time higher than sedentary time is not taken into account (After checking Sedentary awake time values some are negative and cannot be accounted as not accurate since total minutes asleep cannot be more than sedentary time). 

Activity$FormattedTime <- dmy(Activity$FormattedTime) # parse the datetime columns
Sleep$FormattedTime <- dmy(Sleep$FormattedTime)

Activity %>% count(Id, FormattedTime) %>% filter(n > 1) # check if id/formatted time have unique values
Sleep %>% count(Id, FormattedTime) %>% filter(n > 1)

Sleep_unique <- Sleep %>%
  distinct(Id, FormattedTime, .keep_all = TRUE) # remove duplicates

Activity_unique <-Activity %>% 
  distinct(Id, FormattedTime, .keep_all = TRUE)

Activity <- Activity_unique %>%   # join Sleeping time and Sedentary time that belong to two different data frames
  full_join(
    Sleep_unique[, c("Id", "FormattedTime", "TotalMinutesAsleep")],
    by = c("Id", "FormattedTime") # group new column by Id and date
  ) %>%
  arrange(Id, FormattedTime) %>%
  mutate(
    SedentaryAwakeTime = ifelse(
      SedentaryMinutes > TotalMinutesAsleep,
      SedentaryMinutes - TotalMinutesAsleep, NA)) # only keep values where Sedentary > Sleep, else NA
# value_if_true, value_if_false (NA)
```

### Time In Bed Awake

```{r time-in-bed-awake}
# Create column for time spent in bed awake by subtracting Sleeping time to Time in bed.

Sleep$TimeInBedAwake <- Sleep$TotalTimeInBed - Sleep$TotalMinutesAsleep
```


# Means

### Sleep

```{r Sleep}
Sleep %>% 
  select(TimeInBedAwake, TotalMinutesAsleep, TotalSleepRecords) %>% 
  summary()
```

### Activity 

```{r Activity}

Activity %>% 
  select(TotalSteps, VeryActiveMinutes, FairlyActiveMinutes, LightlyActiveMinutes, TotalActiveMinutes, SedentaryMinutes) %>% 
  summary() 
```

### Weight

```{r Weight}

Weight %>% 
  select(BMI, WeightKg) %>% 
  summary() 
```

### Heart rate

```{r Heart-rate}

Heartrate %>% 
  select(Value) %>% 
  summary()
```


# Weekend vs Weekdays breakdown

### Activity

```{r Activity-days}


Activity$weekday <- weekdays(Activity$FormattedTime)
Activity$day_type <- ifelse(Activity$weekday %in% c("Saturday", "Sunday"), "Weekend", "Weekday")

Activity %>% 
  group_by(day_type) %>% 
  summarise(
    avg_steps = mean(TotalSteps, na.rm = TRUE),
    avg_sedentary = mean(SedentaryAwakeTime, na.rm = TRUE),
    avg_TotalActiveMinutes = mean(TotalActiveMinutes, na.rm = TRUE),
    avg_Calories = mean(Calories, na.rm = TRUE)
  )

# Slightly more active and less sedentary on the weekend but not a big difference.
```

### Sleep

```{r Sleep-days}

Sleep$weekday <- weekdays(Sleep$FormattedTime)
Sleep$day_type <- ifelse(Sleep$weekday %in% c("Saturday", "Sunday"), "Weekend", "Weekday")

Sleep %>% 
  group_by(day_type) %>% 
  summarise(
    avg_TotalMinutesAsleep = mean(TotalMinutesAsleep, na.rm = TRUE),
    avg_TimeInBedAwake = mean(TimeInBedAwake, na.rm = TRUE)
  )

# People spend more time sleeping on the weekend.
```

### Heart Rate

```{r Heart-rate-days}

Heartrate$FormattedTime <- dmy_hms(Heartrate$FormattedTime)
Heartrate$weekday <- weekdays(Heartrate$FormattedTime)
Heartrate$day_type <- ifelse(Heartrate$weekday %in% c("Saturday", "Sunday"), "Weekend", "Weekday")

Heartrate %>% 
  group_by(day_type) %>% 
  summarise(
    avg_Value = mean(Value, na.rm = TRUE)
  )

# Heart Rate is very slightly higher on the weekend
```


### Weight
```{r Weight-days}
Weight$FormattedTime <- dmy_hms(Weight$FormattedTime)
Weight$weekday <- weekdays(Weight$FormattedTime)
Weight$day_type <- ifelse(Weight$weekday %in% c("Saturday", "Sunday"), "Weekend", "Weekday")

Weight %>% 
  group_by(day_type) %>% 
  summarise(
    avg_WeightKg = mean(WeightKg, na.rm = TRUE),
    avg_BMI = mean(BMI, na.rn= TRUE)
  )

# Weight and BMI higher on Weekends.
```


# Daily Activity Breakdown

### Activity

```{r Activity-specific-days}
summary_weekday <- Activity %>%
  group_by(weekday) %>%
  summarise(
    avg_steps = mean(TotalSteps, na.rm = TRUE),
    avg_SedentaryAwakeTime = mean(SedentaryAwakeTime, na.rm = TRUE), 
    avg_TotalActiveMinutes = mean(TotalActiveMinutes, na.rm = TRUE),
    avg_Calories = mean(Calories, na.rm = TRUE)
  ) %>%
  mutate(weekday = factor(weekday,levels = c("Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday", "Sunday"))) # order weekdays Monday to Sunday order rather than alphabetically (mutate).


plot_data <- summary_weekday %>%
  pivot_longer(cols = starts_with("avg_"),
               names_to = "Metric",
               values_to = "Value")

ggplot(plot_data, aes(x = weekday, y = Value, fill = Metric)) +
  geom_col(position = "dodge") +
  facet_wrap(~ Metric, scales = "free_y") +  # separate y-axis scales per metric
  labs(
    title = "Average Activity by Day of the Week",
    x = "Day of the Week",
    y = "Average Value",
    fill = "Metric"
  ) +
  theme_minimal() +
  theme(axis.text.x = element_text(angle = 45, hjust = 1))

https://raw.githubusercontent.com/emasche/BellaBeats-Case-Study-using-R/main/Plots/Average-Activity-by-Day-of-the-Week.PNG

# People are less active on Friday and Sunday but most active on Saturday.
```


### Sleep
```{r sleep-specific-days}
summary_sleep <- Sleep %>%
  group_by(weekday) %>%
  summarise(
    avg_timeAsleep = mean(TotalMinutesAsleep, na.rm = TRUE),
    avg_timeInBed = mean(TotalTimeInBed, na.rm = TRUE),
  ) %>%
  mutate(weekday = factor(weekday,levels = c("Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday", "Sunday")))

plot_sleep <- summary_sleep %>%
  pivot_longer(cols = starts_with("avg_"),
               names_to = "Metric",
               values_to = "Value")

ggplot(plot_sleep, aes(x = weekday, y = Value, fill = Metric)) +
  geom_col(position = "dodge",
           color = "black",
           size = 0.1          
  ) +
  labs(
    title = "Average Time in Bed vs Time Asleep by Weekday",
    x = "Day of the Week",
    y = "Time (minutes)",
    fill = "Metric"
  ) +
  theme_minimal() +
  theme(axis.text.x = element_text(angle = 45, hjust = 1))

# People spend most time asleep and in bed on Sunday and Wednesday and less time both asleep and in bed on Thursday, Tuesday and Friday.
```


### Heart Rate

```{r heart-rate-specific-days}
summary_heartrate <- Heartrate %>%
  group_by(weekday) %>%
  summarise(
    avg_bpm = mean(Value, na.rm = TRUE),
  ) %>%
  mutate(weekday = factor(weekday,levels = c("Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday", "Sunday")))

plot_heartrate <- summary_heartrate %>%
  pivot_longer(cols = starts_with("avg_"),
               names_to = "Metric",
               values_to = "Value")

ggplot(plot_heartrate, aes(x = weekday, y = Value, fill = Metric)) +
  geom_line(size = 1.2)+
  geom_point(size = 2)
labs(
  title = "Average Heart rate by Weekday",
  x = "Day of the Week",
  y = "Heartrate (BPM)",
  fill = "Metric"
) +
  theme_minimal() +
  theme(axis.text.x = element_text(angle = 45, hjust = 1))

# BPM is very slightly higher on Saturday and lower on Monday, Tuesday and Wednesday.
```


### Weight

```{r weight-specific-days}
summary_weight <- Weight%>%
  group_by(weekday) %>%
  summarise(
    avg_weight = mean(WeightKg, na.rm = TRUE),
    avg_BMI = mean(BMI, na.rm = TRUE),
  ) %>%
  mutate(weekday = factor(weekday,levels = c("Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday", "Sunday")))

plot_weight <- summary_weight %>%
  pivot_longer(cols = starts_with("avg_"),
               names_to = "Metric",
               values_to = "Value")

ggplot(plot_weight, aes(x = weekday, y = Value, fill = Metric)) +
  geom_col() +
  facet_wrap(~ Metric, scales = "free_y") +
  labs(
    title = "Average Weight by Day of the Week",
    x = "Day of the Week",
    y = "Weight (Kg)",
    fill = "Metric"
  ) +
  theme_minimal() +
  theme(axis.text.x = element_text(angle = 45, hjust = 1))

# Both weight and BMI are higher on Wednesday and Sunday and slightly lower on Thursday.
```


# Correlations

```{r date}
# Create a common column name "Date" for each data frame that only use d/m/y and no h:m:s in order to use it for correlation within all data frame as some data frames use h/m/s and other don't.

Activity$Date <- as.Date(Activity$FormattedTime, format = "%d/%m/%Y") 
Sleep$Date <- as.Date (Sleep$FormattedTime, format = "%d/%m/%Y")
Heartrate$Date <- as.Date(Heartrate$FormattedTime, format = "%d/%m/%Y %H:%M:%S") 
Weight$Date <- as.Date(Weight$FormattedTime, format = "%d/%m/%Y %H:%M:%S") 
```


```{r Average-daily-heart-rate}
# There is too many Heart rate data as data have been recorded every 5minutes of each date. Next step will aggregate the Heart Rate data by computing the daily average, in order to reduce multiple values per day and match them with the corresponding single daily record from other data frames.

Heartrate_daily <- Heartrate %>% 
  group_by(Id, Date) %>% 
  summarize(AvgHeartRate = mean(Value, na.rm = TRUE)) 
```


## Multi correlation matrix

```{r Multi-correlation-Matrix}
# Creating a data frame that merges all 4 main data frames by Id and Date.

Allmerged <- list(Sleep, Weight, Activity, Heartrate_daily) %>% 
  reduce(full_join, by = c("Id", "Date"))

# Before to run correlation matrix. Remove unneeded variable for more clarity.

Allmerged <- Allmerged %>% select(-Id, -Fat, -TotalDistance, -TotalSleepRecords, -TotalMinutesAsleep.y, -WeightPounds, -TotalTimeInBed, - LogId, -LoggedActivitiesDistance, -TrackerDistance, -VeryActiveDistance, -ModeratelyActiveDistance, -LightActiveDistance, -SedentaryActiveDistance)


# Select only the numeric columns to avoid errors.

numeric_data <- Allmerged %>% select(where(is.numeric))


# To avoid error message, checks each numeric column to see if its standard deviation is 0 (all values are the same) and stores the result as a logical vector.

zero_var_cols <- sapply(numeric_data, function(x) sd(x, na.rm = TRUE) == 0) 

# Only keeps the columns in numeric_data that do not have zero variance.

numeric_data_filtered <- numeric_data[, !zero_var_cols]

# Create a correlation matrix of all the relevant numeric variables

cor_matrix <- cor(numeric_data, use = "pairwise.complete.obs") 
print(cor_matrix)

# Create a plot of the correlation matrix for clearer results.

corrplot(cor_matrix, 
         method = "color",       # Use colored tiles instead of numbers or circles
         type = "upper",         # Show only the upper triangle (to avoid repetition)
         tl.cex = 0.8,           # Text label size for variable names
         number.cex = 0.5,       # Size of the numbers (correlation values)
         addCoef.col = "black",  # Color of coefficients
         tl.srt      = 55,       # 
         tl.col = "darkred")     # rotate labels 45 degrees
```


## 1- Sedentary time correlate with total sleeping time (without accounting for sedentary time spend sleeping).

```{r Sedentary-Sleep}

cor.test(Activity$SedentaryAwakeTime, Activity$TotalMinutesAsleep, use = "complete.obs")

# r(342) = -0.89, p < .001 (Very strong negative correlation/Very significant).

plot(Activity$SedentaryAwakeTime, Activity$TotalMinutesAsleep,
     main = "Sedentary Minutes vs Total Sleep",
     xlab = "Sedentary Minutes (awake)",
     ylab = "Total Minutes Asleep",
     pch = 19, col = "darkblue")
abline(lm(TotalMinutesAsleep ~ SedentaryAwakeTime, data = Activity), col = "red", lwd = 2) # adding a trend line (linear regression)

## As sedentary minutes increase, sleep minutes tend to decrease and vice versa.
```


## 2- Heart rate correlate with sleeping time

```{r Heart -rate-sleep}
Heartrate_daily <- Heartrate %>% 
  group_by(Id, Date) %>% 
  summarize(AvgHeartRate = mean(Value, na.rm = TRUE)) 
# Aggregate the Heart Rate data by computing the daily average, in order to reduce multiple values per day and match them with the corresponding single daily Sleep record.

HeartbeatSleep <- merge(Heartrate_daily, Activity, by= c("Id", "Date")) 

cor.test(HeartbeatSleep$AvgHeartRate, HeartbeatSleep$TotalMinutesAsleep)

# r(179)= -0.26, p<.001 (Weak negative correlation, very significant).


plot(HeartbeatSleep$AvgHeartRate, HeartbeatSleep$TotalMinutesAsleep,
     main = "Heart Rate vs Total Sleep",
     xlab = "Heart Rate",
     ylab = "Total Minutes Asleep",
     pch = 19, col = "darkblue")
abline(lm(TotalMinutesAsleep ~ AvgHeartRate, data = HeartbeatSleep), col = "red", lwd = 2) # adding a trend line (linear regression)

# The more people spend time sleeping, the lower is their heart rate and vice versa.
```


## 3- Heart Rate correlates with Sedentary time

```{r Heart-rate-sedentary-time}
HeartrateActivity <- merge(Heartrate_daily, Activity, by= c("Id", "Date")) # using awake sedentary time rather than sedentary time (including sleeping time) as Sleep naturally lowers heart rate (due to parasympathetic activity)

cor.test(HeartrateActivity$AvgHeartRate, HeartrateActivity$SedentaryAwakeTime)

# r(156)= 0.20, p<.05 (Weak positive correlation, statistically significant)

ggplot(HeartrateActivity, aes(x = SedentaryAwakeTime, y = AvgHeartRate)) +
  geom_point(alpha = 0.6) +
  geom_smooth(method = "lm", se = FALSE, color = "blue") +
  labs(title = "Avg Heart Rate vs Sedentary Awake Time",
       x = "Sedentary Awake Time (mins)",
       y = "Average Heart Rate") +
  theme_minimal()

# People with more sedentary time tend to have slightly higher heart rates and vice versa.
```
 

## 4- Total Active Minutes and Steps

```{r active-minutes-steps}
cor.test(Activity$TotalActiveMinutes, Activity$TotalSteps)

# r(1371)= 0.77, p<.001 (Very strong positive correlation, very significant)


ggplot(Activity, aes(x= TotalActiveMinutes, y= Activity$TotalSteps))+
  geom_point(alpha= 0.6)+
  geom_smooth(method = "lm", se= FALSE, Color="blue")+
  labs(title= "Total Active Time vs Steps",
       x= "Total Daily Active Time (min)",
       y= "Daily Steps")+
  theme_minimal()

# The correlation between Total Active Minutes and Total Steps is very strong (r = 0.77, p < .001), indicating a significant and positive linear relationship. This suggests that individuals who spend more time being active tend to take more steps throughout the day. The strong correlation also implies that a large portion of daily physical activity is likely made up of walking or stepping-based movement, reinforcing the role of step count as a key component of overall activity levels.
```

## 5- Calories and Very Active minutes

```{r Calories-Very-active-minutes}
cor.test(Activity$Calories, Activity$VeryActiveMinutes)

# r(1371) =0.59p<.001 (positive correlation, Very significant)

ggplot(Activity, aes(x=Calories, y=VeryActiveMinutes))+
  geom_point(alpha=0.6)+ #60% opaque
  geom_smooth(method = "lm", se= FALSE, Color="blue")+ # linear model, False = hides shaded confidence interval
  labs(title= "Calories vs Very Active Minutes",
       x= "Calories",
       y="Very Active Minutes")+
  theme_minimal()

# As number of Very Active minutes increase, burnt Calories increases
```

## 6- Calories and Steps

```{r Calories-steps}
cor.test(Activity$Calories, Activity$TotalSteps)

# r(1371)=0.58, p<.001 (positive correlation, very significant)

ggplot(Activity, aes(x=Calories, y=TotalSteps))+
  geom_point(alpha=0.6)+
  geom_smooth(method = "lm", se= FALSE, Color="blue")+
  labs(title = "Calories vs Steps",
       x="Daily Caloric expenditure",
       y="Daily Steps")+
  theme_minimal()

# There is a positive relationship between daily step count and calorie burn, indicating that individuals who walk more steps tend to expend more calories throughout the day.
```

## 7- Time in Bed (Awake) and Fairly Active minutes

```{r Awake-time-bed-fairly-active}
cor.test(Allmerged$TimeInBedAwake, Allmerged$FairlyActiveMinutes)
# r(412)=0.32, p<.001 (weak positive correlation, very significant)


ggplot(Allmerged, aes(x=TimeInBedAwake, y=FairlyActiveMinutes))+
  geom_point(alpha=0.6)+
  geom_smooth(method="lm", se=FALSE, Color="blue")+
  labs(title="Time in Bed Awake vs Daily Fairly Active Minutes",
       x="Time in Bed Awake (min)",
       y="Daily Fairly Active Minutes")+
  coord_cartesian(xlim = c(0, 200), ylim = c(0, 200)) +  # zoom to focus area without dropping data
  theme_minimal()

#The more time Daily fairly active time is spent the more time in bed is spent before and after sleeping and vice versa. Because correlation does not imply causation, this association highlights a connection but doesn’t clarify the direction of influence. Those results could indicate that spending time in bed helps relaxation and increases motivation for the rest of the day resulting in higher fairly active time but it could also indicate that longer fairly active time leads to the need of more time to relax once in bed.
```


Summary of Analysis

Key findings from the data analysis include:

Activity Trends: The average daily step count was 7,281—well below the recommended 10,000. Users were more active on Saturdays and   least active on Fridays and Sundays. Weekends showed slightly more active minutes and lower sedentary behavior.

Sleep Patterns: Users sleep longer on weekends, especially on         Sundays. A significant portion of time in bed is spent awake,         particularly on weeknights.

Heart Rate Insights: Slightly elevated average heart rate on          weekends. Weak negative correlation between heart rate and sleep       duration.

Weight and BMI: Slight increase in weight and BMI over weekends,   peaking on Sundays and Wednesdays. Correlation Highlights: Strong positive correlation between total active minutes and step count (r = 0.77). Moderate positive correlation between very active minutes and calories burned. Strong negative correlation between sedentary awake time and total sleep (r = -0.89). Slight positive correlation between sedentary awake time and heart rate.

Key Insight Summaries: Less activity correlates with reduced sleep and slightly higher heart rate. Most users do not meet daily activity guidelines. User behaviors differ significantly between weekdays and weekends, affecting sleep, heart rate, and calorie burn.


Based on the analysis, the following strategic recommendations are proposed for Bellabeat:

1. Introduce Personalized Activity Nudges

Insight: Most users do not meet the 10,000-step daily              recommendation; activity is lowest on weekdays,especially Fridays.
Action: Use app notifications to encourage short bursts of movement    during sedentary periods, especially during workdays.
Example: “Take 1,000 steps by 3 PM to hit your daily goal!”


2. Emphasize Weekend Wellness Features

Insight: Users sleep more and are slightly more active on weekends.
Action: Launch weekend wellness campaigns featuring mindfulness,   recovery, hydration tracking, and sleep rituals. “Recharge Sundays”:Guided breathing + hydration reminders + sleep prep content.
    
    
3. Create Smart Sleep Coaching

Insight: There's a strong negative correlation between sedentary   time and sleep; many users spend time in bed awake. Action: Offer in-app sleep coaching to help users improve sleep efficiency (not just duration), including: Pre-bed routines: Light/stretch reminders     before sleep. Content to reduce "awake in bed" time (e.g., sleep stories, relaxation sounds)
    
4. Launch Heart Health Awareness Tools

Insight: Higher sedentary time is weakly correlated with higher    heart rate. Action: Introduce heart rate monitoring insights that detect patterns and offer tips (e.g., breathing exercises, movement alerts).“We noticed your heart rate is elevated—want to take a 3-minute breathing break?”
    
5. Add Adaptive Daily Goals

Insight: Activity and sleep levels vary by day of the week.
Action: Use machine learning to adapt daily step or sleep goals    based on user trends (e.g., higher weekend activity). Adjusted    expectations may improve user satisfaction and consistency.

6. Implement “Lifestyle Snapshot” Dashboards

Insight: Correlation analysis showed meaningful connections between activity, heart rate, sleep, and calories. Action: Provide users   with a weekly health summary combining: Steps, Heart rate trends, Sleep efficiency, Calories burned. Example: “This week you were most active on Saturday. Sleep improved on days with less sedentary    time.”

7. Gamify Movement & Sleep

Insight: Users fall short on physical activity and may benefit from motivation. Action: Add achievements, badges, and streaks for:     Meeting step goals, Improving sleep efficiency, Lowering resting  heart rate over time.


Area for future research

1. Audience Segmentation

Further research could explore how different age groups of female  users engage with wearable technology. Specifically: Do younger   women (ages 20–35) prioritize fitness tracking, step goals, and the visual/aesthetic appeal of devices? Do older women (ages 35+) place more value on features that support wellness, heart health, sleep monitoring, and stress management?
    
2. Competitive Differentiation

Further exploration could assess whether Bellabeat can create a    stronger market position by focusing on holistic wellness rather than traditional fitness tracking alone. Specifically: Do users         respond more positively to devices that support both physical and      mental wellbeing, such as stress reduction, sleep quality,  mindfulness, and emotional balance?
Would framing Bellabeat as a lifestyle partner—rather than a step      counter—better align with the needs and expectations of its core       audience?
    
  
3. Female-Specific Health Features
    
Further development could explore how Bellabeat can better support    women’s unique health needs by integrating features tailored to       different life stages. Specifically: Do users benefit from tracking tools related to menstrual cycles, fertility windows, and hormonal  fluctuations? Could women entering perimenopause and menopause find   value in tools focused on stress management, sleep support, and     holistic wellness guidance?
    
Understanding these preferences could help Bellabeat tailor its product features and marketing messages to better meet the distinct needs of these segments.

About
Bella Beats case study, Google Data Analytics milestone project

file:///C:/Users/emma3/OneDrive/Documents/BellaBeats/Bella%20Beats%20case%20study.html
Topics
smart-devices fitness-tracker health-tracker
Resources
 Readme
 Activity
Stars
 0 stars
Watchers
 0 watching
Forks
 0 forks
Releases
No releases published
Create a new release
Packages
No packages published
Publish your first package
Footer
