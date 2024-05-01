# case_tudy-with_r_bellabeat
![alt text](https://ningjoanne.files.wordpress.com/2024/04/avg_calories-5.png?w=479)

``
merge_cal_act_type <- merge(user_avg_info_new, day_calories, by='Id') %>% 
  group_by(CustomerType) %>% summarise(AvgCal= mean(Calories))
print(merge_cal_act_type)


merge_sleep_act_type <- merge(user_avg_info_new, day_sleep, by='Id') %>% 
  group_by(CustomerType) %>% summarise(AvgSleepHr= mean((TotalMinutesAsleep/60)))
print(merge_sleep_act_type)
``
