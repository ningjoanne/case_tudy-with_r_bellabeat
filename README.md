



Case Study with R – Bellabeat
=============================

![](https://ningjoanne.files.wordpress.com/2024/04/combined_plots_cal_act_type.png?w=1024)

<div style="display:flex;">
    <img src="https://ningjoanne.files.wordpress.com/2024/04/plot_cal_step.png?w=340" style="width: 33%;">
    <img src="https://ningjoanne.files.wordpress.com/2024/04/plot_sleep_vs_sedebtary.png?w=340" style="width: 33%;">
    <img src="https://ningjoanne.files.wordpress.com/2024/04/plot_sleeplatency.png?w=340" style="width: 33%;">
</div>

* * *

Introduction
------------

Bellabeat, a high-tech manufacturer established in 2014 by Urška Sršen and Sando Mur.  The company is focusing on producing health-focused smart products especially for women to collect data on sleep, stress, activity and reproductive health allowed Bellabeat to empower women with knowledge about their own health and habits.  Through their three products, Leaf (fitbit), Time (watch) and Spring (water bottle), customers can easily track their health information through Bellabeat APP.

* * *

Business Prompt
---------------

Tasked with the marketing analyst team at Bellabeat, the executive team requires a junior data analyst to analyse smart device data. The objective is to gain insights into consumer usage patterns and provide high-level suggestions to guide marketing strategy.

* * *

Step 1. Ask Question
--------------------

**Key Task:** Based on the provided [FitBit Fitness Tracker Data](https://www.kaggle.com/datasets/arashnic/fitbit), to extract the insights to assist Bellabeat in refining its marketing strategy  
  
**Key Stakeholder:** Bellabeat’s two co-founders Urška Sršen and Sando Mur  
  
**Guiding Questions:**  
  
1\. What are some trends in smart device usage?  
2\. How could these trends apply to Bellabeat customers?  
3\. How could these trends help influence Bellabeat marketing strategy?

* * *

Step 2. Prepare Data
--------------------

#### **Evaluate the validity of the data**

In this case study, public data from the FitBit Fitness Tracker is utilized. The dataset includes personal fitness tracker information from FitBit users who have consented to share their data to be used for explore users’ habits.

*   **Reliable:** The data consists of information from 30 FitBit users who have consented to share their fitness tracker data. The dataset size meets the minimum requirements for the smallest sample size, 30.
*   **Original:**  The data is collected by third party using Amazon Mechanical Turk surveys.
*   **Comprehensive:** The data encompasses FitBit users’ daily active time, minutes of activity, sleep duration, step counts and distance covered, and more.
*   **Current:** The data spans from 4/12/2016 to 5/12/2016.
*   **Cited:** Unknown as the data is collected by a third party.

#### **Choose Datasets**

After previewing the dataset using the code below, several points are addressed below:

1.  There are 18 tables in total.
2.  The data contained in the ‘dailyCalories\_merged’, ‘dailyIntensities\_merged’, and ‘dailyIntensities\_merged’ tables is redundant within the ‘dailyActivity\_merged’ table.
3.  The ‘weightLogInfo\_merged’ is dropped since it only contains 8 users.
4.  ‘sleepDay\_merged’ database has 24 Distinct\_ID, rest of databases have 33 Distinct\_ID
5.  Categorise users for future analysis.
6.  ID and Activity Date column are combined to create a new column as a unique identifier for future analysis.
7.  Time columns need to be reformatted.
```
    # install packages
    install.packages("tidyverse")
    install.packages("ggplot2")
    install.packages("dplyr")
    install.packages("tidyr")
    install.packages("lubridate")
    install.packages("readr")
    install.packages("gridExtra")
    
    
    # load library
    library(tidyverse)
    library(ggplot2)
    library(dplyr)
    library(tidyr)
    library(lubridate)
    library(readr)
    library(gridExtra)
    
    
    # data import
    day_activity <- read.csv('dailyActivity_merged.csv')
    day_calories <- read.csv('dailyCalories_merged.csv')
    day_intensities <- read.csv('dailyIntensities_merged.csv')
    day_steps <- read.csv('dailySteps_merged.csv')
    day_sleep <- read.csv('sleepDay_merged.csv')
    weight_info <- read.csv('weightLogInfo_merged.csv')
    
    
    # data preview
    head(day_activity)
    colnames(day_activity)
    n_distinct(day_activity$Id)
    
    head(day_calories)
    colnames(day_calories)
    n_distinct(day_calories$Id)
    
    head(day_intensities)
    colnames(day_intensities)
    n_distinct(day_intensities$Id)
    
    head(day_steps)
    colnames(day_steps)
    n_distinct(day_steps$Id)
    
    head(day_sleep)
    colnames(day_sleep)
    n_distinct(day_sleep$Id)
    
    head(weight_info)
    colnames(weight_info)
    n_distinct(weight_info$Id)
    
    day_activity %>% 
      select(TotalSteps, TotalDistance, VeryActiveMinutes) %>% 
      summary()
   ```

* * *

Step 3. Process Data
--------------------

#### **Segment users**

First, I would like to categorise users based on their activity type. Quartile-based segmentation is used for this analysis.  
  
Steps and scenario show below:  
1\. Average steps, average distance, and average very activity minutes is calculated by each user.

    user_avg_info <-
      day_activity %>% 
      group_by(Id) %>% 
      drop_na() %>% 
      summarise( avg_step = mean(TotalSteps),
                 avg_distance = mean(TotalDistance),
                 avg_very_active_min = mean(VeryActiveMinutes)
                 )

  
2\. Then quartile TotalStep, TotalDistance, and VeryActiveMinutes to set up the metric for define users on the next step.

    p75_step <- quantile(day_activity$TotalSteps, probs =0.75)
    p75_distance <- quantile(day_activity$TotalDistance, probs =0.75)
    p75_very_active_min <- quantile (day_activity$VeryActiveMinutes, probs = 0.75)
    
    p25_step <- quantile(day_activity$TotalSteps, probs =0.25)
    p25_distance <- quantile(day_activity$TotalDistance, probs =0.25)
    p25_very_active_min <- quantile (day_activity$VeryActiveMinutes, probs = 0.25)
    

  
3\. Users is categorized into three groups based on below criteria:

*   High Active:  
    average steps >= 75% TotalSteps +  
    average distance >= 75% TotalDistance +  
    average very activity minutes >= 75% VeryActiveMinutes
*   Moderate Active:  
    average steps between 25 – 75% TotalSteps +  
    average distance between 25 – 75% TotalDistance +  
    average very activity minutes between 25 – 75% VeryActiveMinutes
*   Low Active:  
    average steps <= 25% TotalSteps +  
    average distance <= 25% TotalDistance +  
    average very activity minutes <= 25% VeryActiveMinutes

    ```
    # categorize customer in 3 groups
    
    customer_category<- day_activity %>%
      group_by(Id) %>%
      summarise(avg_step = mean(TotalSteps),
                avg_distance = mean(TotalDistance),
                avg_very_active_min = mean(VeryActiveMinutes)) %>%
      mutate(CustomerType = case_when(
        avg_step >= p75_step &
          avg_distance >= p75_distance &
          avg_very_active_min >= p75_very_active_min ~ 'High Active',
        avg_step <= p25_step &
          avg_distance <= p25_distance &
          avg_very_active_min <= p25_very_active_min ~ 'Low Active',
        TRUE ~ 'Moderate Active'
      ))
    
    user_avg_info <- merge(user_avg_info, customer_category, by = "Id")
    user_avg_info_new <- user_avg_info %>% 
      select(Id, avg_step.x, avg_distance.x, avg_very_active_min.x, 'CustomerType' )
    
    
    # count users by types
    
    user_avg_info_new %>%
      group_by(CustomerType) %>%
      count()
  

The outcome indicates that there are 4 users classified as High Active and 29 users classified as Moderate Active, with no users falling into the Low Active category based on the specified criteria.

![](https://ningjoanne.files.wordpress.com/2024/04/customer_aggregate.png?w=1024)

![](https://ningjoanne.files.wordpress.com/2024/04/customer_type_count.png?w=477)

#### **Reformat time**

    day_activity$ActivityDate <- as.POSIXct(day_activity$ActivityDate, format = "%m/%d/%Y")
    day_activity <- day_activity %>% 
      mutate(NewActivityDate = as.Date(ActivityDate))
    head(day_activity)
    
    
    day_sleep$SleepDay <- as.POSIXct(day_sleep$SleepDay, format = "%m/%d/%Y %H:%M:%S")
    day_sleep <- day_sleep %>%
      mutate(SleepDayNew = as.Date(SleepDay))
    head(day_sleep)

![](https://ningjoanne.files.wordpress.com/2024/04/new_activity_date.png?w=1024)

![](https://ningjoanne.files.wordpress.com/2024/04/new_sleep_date-1.png?w=1024)

#### **Create new column and merge data**

The ID and Activity Date columns are merged to form a new column ‘NewId’ serving as a unique identifier also the day\_activity table is joined with day sleep for subsequent analysis.

    # create new column
    
    day_activity_new <- day_activity %>% 
      unite('NewId', Id, NewActivityDate, sep=' ')
    head(day_activity_new)
    
    day_sleep_new <- day_sleep %>% 
      unite('NewId', Id, SleepDayNew, sep=' ')
    head(day_sleep_new)
    
    
    # merge day_active table with day_sleep table
    
    merge_act_sleep <- merge(day_activity_new, day_sleep_new, by = 'NewId', all=TRUE )
    head(merge_act_sleep)
    n_distinct(merge_act_sleep$NewId)

![](https://ningjoanne.files.wordpress.com/2024/04/unique_id.png?w=1024)

* * *

Step 4. Analyse Data
--------------------

#### **General idea**

First and foremost, let’s examine some information from the user group defined earlier in this case study.  
As indicated by the data below, High Active users tend to burn more calories, yet they sleep less than the Moderate Active users.

    merge_cal_act_type <- merge(user_avg_info_new, day_calories, by='Id') %>% 
      group_by(CustomerType) %>% summarise(AvgCal= mean(Calories))
    print(merge_cal_act_type)
    
    
    merge_sleep_act_type <- merge(user_avg_info_new, day_sleep, by='Id') %>% 
      group_by(CustomerType) %>% summarise(AvgSleepHr= mean((TotalMinutesAsleep/60)))
    print(merge_sleep_act_type)
    

![](https://ningjoanne.files.wordpress.com/2024/04/avg_calories-5.png?w=479)

![](https://ningjoanne.files.wordpress.com/2024/04/avg_sleep_new_new-1.png?w=531)

* * *

#### **Assumption**

Before diving straight into the data, I prefer to devise several hypotheses and then utilise the data to verify them.

1.  Does the duration of sleep relate to the number of hours people spend sitting?
2.  Is there a correlation between sleep latency (TimeInBed – MinutesAsleep) and the duration of time individuals spend sitting?
3.  Is there a relationship between calories burned and the duration of physical activity?
4.  What is the relationship among calories burned, steps and distance walked by individuals?
5.  How do people’s walking step counts differ across different days of the week?

* * *

#### **Verify the assumptions**

❓Assumption 1: Does the duration of sleep relate to the number of hours people spend sitting?  
  
📈Answer: The outcome indicates a negative correlation, suggesting that as the number of hours spent sitting increases, the number of hours slept tends to decrease.

    plot_sleep_vs_sedebtary <-
      ggplot(data = drop_na(merge_act_sleep), aes(x=SedentaryMinutes/60, y=TotalMinutesAsleep/60))+
      geom_point(color="#C4961A")+
      geom_smooth(size=0.5, color="#D16103")+
      labs(title = ' Asleep Hour by Sedentary Hour ', x= 'Sendentary Hour', y='Asleep Hour')+
      theme(text = element_text(family='Calibri'))
    print(plot_sleep_vs_sedebtary)

![](https://ningjoanne.files.wordpress.com/2024/04/plot_sleep_vs_sedebtary-1.png?w=1024)

* * *

❓Assumption 2: Is there a correlation between sleep latency (TimeInBed – MinutesAsleep) and the duration of time individuals spend sitting?  
  
📈Answer: There is no correlation observed between the time it takes to fall asleep and the amount of time individuals spend sitting.

    plot_sleeplatency <-
      ggplot(data = drop_na(merge_act_sleep),
      aes(x=SedentaryMinutes/60, y= (TotalTimeInBed - TotalMinutesAsleep)/60))+
      geom_point(color="#C4961A")+
      geom_smooth(size=0.5, color="#D16103")+
      labs(title = 'Sleep Latency Hour by Sedentary Hour', x= 'Sedentary Hour', y = 'Sleep Latency Hour')+
      theme(text = element_text(family = 'Calibri'))
    print(plot_sleeplatency)

![](https://ningjoanne.files.wordpress.com/2024/04/plot_sleeplatency-1.png?w=1024)

* * *

❓Assumption 3: Is there a relationship between calories burned and the duration of physical activity?  
  
📈Answer: The data illustrates a positive correlation between Very Active Hours and Calories, whereas there appears to be little to no correlation between Fairly Active Hours or Lightly Active Hours versus Calories.

    plot_cal_va <-
      ggplot(data = drop_na(merge_act_sleep), aes(x=Calories, y=VeryActiveMinutes/60))+
      geom_point(color="#52854C")+
      geom_smooth(size=0.5, color="#D16103")+
      labs(title = 'Very Active Hour by Calories', x= 'Calories', y= 'Active Hour')+
      theme(text = element_text(family = 'Calibri', size = 8))
    
    plot_cal_fa <-
      ggplot(data = drop_na(merge_act_sleep), aes(x=Calories, y=FairlyActiveMinutes/60))+
      geom_point(color="#009E73")+
      geom_smooth(size=0.5, color="#D16103")+
      labs(title = 'Fairly Active Hour by Calories', x= 'Calories', y='Active Hour' )+
      theme(text = element_text(family = 'Calibri', size = 8))
    
    
    plot_cal_la <-
    ggplot(data = drop_na(merge_act_sleep), aes(x=Calories, y=LightlyActiveMinutes/60))+
      geom_point(color="#C3D7A4")+
      geom_smooth(size=0.5, color="#D16103")+
      labs(title = 'Lightly Active Hour by Calories', x= 'Calories', y ='Active Hour')+
      theme(text = element_text(family = 'Calibri', size=8))
    
    
    combined_plots_cal_act_type <-grid.arrange(plot_cal_va, plot_cal_fa, plot_cal_la, nrow=1, ncol=3)

![](https://ningjoanne.files.wordpress.com/2024/04/combined_plots_cal_act_type-1.png?w=1024)

* * *

❓Assumption 4: What is the relationship among calories burned, steps and distance walked by individuals?  
  
📈Answer: Steps count, calories, and distance exhibit a positive correlation.

    plot_cal_step <-
      ggplot((data=merge_act_sleep), aes(x=Calories, y=TotalSteps, color=TotalDistance)) +
      geom_point()+
      geom_smooth(size=0.5, color="#D16103")+
      scale_color_gradient(low="#C3D7A4", high = "darkblue" )+
      labs(title = 'Step Count by Calories ', x= 'Calories', y= 'Step Count', color= 'Distance')+
      theme(text = element_text(family = 'Calibri'))
    
    print(plot_cal_step)

![](https://ningjoanne.files.wordpress.com/2024/04/plot_cal_step-1.png?w=1024)

* * *

❓Assumption 5: How do people’s walking step counts differ across different days of the week?  
  
📈Answer: People walk less steps on Sunday

    custom_order <- c("Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday", "Sunday")
    merge_act_sleep_new <- merge_act_sleep %>% 
      drop_na() %>% 
      mutate(DayOfWeek = reorder(DayOfWeek, match(DayOfWeek, custom_order))) %>%
      group_by(DayOfWeek) %>% 
      summarise(StepDow=sum(TotalSteps))
    
    
    plot_step_dow<-
      ggplot(data=merge_act_sleep_new, aes(x = DayOfWeek, y = StepDow)) +
      geom_bar(stat = "identity", fill = "#4E84C4")+
      labs(title = "Total Steps by Day of Week", x = "Day of Week", y = "Total Steps")+
      scale_y_continuous(labels = scales::comma)+
      annotate('text', x='Sunday', y=480000, label='people tend \nto walk \nless on sunday', size=2.5, color="#D16103", fontface='bold', angle=15)+
      theme(text = element_text(family = 'Calibri'))
    
    print(plot_step_dow)

![](https://ningjoanne.files.wordpress.com/2024/04/plot_step_dow.png?w=1024)

* * *

Save all the plots for future reference.

    plots <- list(
      plot_sleeplatency,
      plot_sleep_vs_sedebtary,
      combined_plots_cal_act_type,
      plot_cal_step,
      plot_step_dow
    )
    
    plot_names <- c("plot_sleeplatency", "plot_sleep_vs_sedebtary", "combined_plots_cal_act_type", "plot_cal_step", "plot_step_dow")
    
    for (i in seq_along(plots)) {
      ggsave(paste0(plot_names[i], '.png'), plot =plots[[i]], width = 6, height=4, dpi=300)
    }

* * *

Step 5 Share insight and Act
----------------------------

Based on the preceding five observations and insights, the marketing strategy recommendations are outlined as follows:

*   Insight 1: More sitting hours are associated with fewer hours of sleep.
    *   Suggestion: Bellabeat App could introduce a new feature allowing users to set a threshold for sitting time. Once reached, the app can prompt users to engage in physical activity to improve their sleep quality.
*   Insight 2: Very Active Hours strongly correlate with calories burned.  
    Insight 3: Steps count, calories burned, and walking distance are closely correlated.
    *   Suggestion: For users aiming to lose weight, focusing on increasing high-intensity activity time is advised.  
        Also, Bellabeat App could incorporate features like setting exercise goals based on calories burned during high-intensity activity or distance walked.
*   Insight 4: People walk less on Sundays and Mondays.
    *   Suggestion: Bellabeat App could send push notifications on Sundays and Mondays to remind users about their decreased activity levels compared to other days and motivate them to be more active.  
        

* * *

#### **Limitation**

1.  [FitBit Fitness Tracker Data](https://www.kaggle.com/datasets/arashnic/fitbit) does not contain gender information, which might introduce bias since Bellabeat primarily serves female users. Additionally, age information is absent in the data, requiring careful consideration before using the analysis from this case study.
2.  The sample size only meets the minimum requirement of 30.
3.  The [FitBit Fitness Tracker Data](https://www.kaggle.com/datasets/arashnic/fitbit) was recorded in 2016, making it outdated. Therefore, any conclusions drawn should be used cautiously, primarily for reference.

* * *

#### **Reference**

*   [https://www.ncbi.nlm.nih.gov/pmc/articles/PMC7745163/](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC7745163/)
*   [https://bellabeat.com/](https://bellabeat.com/)
*   [https://www.nhs.uk/live-well/exercise/physical-activity-guidelines-for-adults-aged-19-to-64/](https://www.nhs.uk/live-well/exercise/physical-activity-guidelines-for-adults-aged-19-to-64/)
*   [https://www.coursera.org/professional-certificates/google-advanced-data-analytics?utm\_medium=sem&utm\_source=gg&utm\_campaign=b2c\_emea\_google-advanced-data-analytics\_google\_ftcof\_professional-certificates\_arte\_feb\_24\_dr\_geo-multi\_sem\_rsa\_gads\_lg-all&campaignid=21008476799&adgroupid=158313472989&device=c&keyword=google%20data%20analytics&matchtype=p&network=g&devicemodel=&adposition=&creativeid=690404375441&hide\_mobile\_promo&gad\_source=1&gclid=CjwKCAjwt-OwBhBnEiwAgwzrUvkHQFwwuFK\_v1\_c6d9emERfTLwvNJ4itbCQNca9A9XHDNyma\_OvVBoCLocQAvD\_BwE](https://www.coursera.org/professional-certificates/google-advanced-data-analytics?utm_medium=sem&utm_source=gg&utm_campaign=b2c_emea_google-advanced-data-analytics_google_ftcof_professional-certificates_arte_feb_24_dr_geo-multi_sem_rsa_gads_lg-all&campaignid=21008476799&adgroupid=158313472989&device=c&keyword=google%20data%20analytics&matchtype=p&network=g&devicemodel=&adposition=&creativeid=690404375441&hide_mobile_promo&gad_source=1&gclid=CjwKCAjwt-OwBhBnEiwAgwzrUvkHQFwwuFK_v1_c6d9emERfTLwvNJ4itbCQNca9A9XHDNyma_OvVBoCLocQAvD_BwE)

Thank you for going through this analysis with me. Feel free to reach out if you have any questions 😊

