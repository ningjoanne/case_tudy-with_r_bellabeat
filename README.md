# case_tudy-with_r_bellabeat
Step 4. Analyse Data
General idea
First and foremost, let’s examine some information from the user group defined earlier in this case study.
As indicated by the data below, High Active users tend to burn more calories, yet they sleep less than the Moderate Active users.

merge_cal_act_type <- merge(user_avg_info_new, day_calories, by='Id') %>% 
  group_by(CustomerType) %>% summarise(AvgCal= mean(Calories))
print(merge_cal_act_type)


merge_sleep_act_type <- merge(user_avg_info_new, day_sleep, by='Id') %>% 
  group_by(CustomerType) %>% summarise(AvgSleepHr= mean((TotalMinutesAsleep/60)))
print(merge_sleep_act_type)

