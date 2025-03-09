---
layout: post
title: "Providing Ratings for Amateur and Professional Athletes Based on Graded Exercise Test Results"
author: "Molly McCann & Taylor Hill"
---

## Executive Summary 

An analytical approach to understanding the body's performance during exercise has proven to be instrumental in optimizing training and recovery for athletes. Researchers at the University of Malaga have conducted Graded Exercise Tests (GETs) to gather insights into the cardio-respiratory performance of athletes from both amateur and professional backgrounds. These tests, conducted on treadmills, involved measuring key metrics such as oxygen consumption, ventilatory thresholds, and heart rate dynamics during the progression to maximal effort and subsequent recovery. While endurance and aerobic efficiency are often derived from these measures, the need for a comprehensive performance score has become increasingly important as athletes analyze their past performances and future outcomes. The following research seeks to develop a performance rating system based on these graded exercise test results, normalizing scores by age, sex, and weight, and applying clustering techniques to categorize athletes into performance tiers. Additionally, it will look into whether there are key determinants of a faster recovery after achieving maximal effort. Lastly, an analysis will be done with the results of athletes with multiple tests.

## Introduction

The initial steps of creating performance scores for GETs include selecting key physiological variables that best represent an athlete’s aerobic capacity and endurance. The data provided contains a basic number of cardio-respiratory measures; however, a series of derived variables will be beneficial to the analysis. As mentioned, the base values alone do not account for natural physiological variations due to age, gender, and weight, necessitating the normalization of these factors to ensure fair comparisons across diverse athlete populations included in the GET research. Additionally, applying clustering techniques, such as k-means, allows for the categorization of athletes into performance tiers, offering a structured framework to assess differences between amateur and professional athletes.  

Beyond performance evaluation, understanding recovery dynamics after reaching maximal effort is essential for optimizing training regimens and minimizing fatigue-related risks. Recovery is often measured using trends in heart rate reduction and ventilation after exertion. In athletics, recovery can be used as a reflection of overall fitness and health. Through regression models, it is possible to quantify recovery patterns and compare them across different athlete groups. This can help to identify what factors are key determinants of faster recovery and provide insight into the factors that contribute to an athlete’s ability to return to baseline after maximal exertion. 

By integrating these findings, this research seeks to bridge the gap between raw physiological data and practical applications, offering a framework for evaluating both performance and recovery in athletes from a wide range of ages and sporting backgrounds.

## Related Work 

Graded exercise tests (GETs) and performance tiers are well-established concepts in research on athlete performance. Previous studies have explored various applications of GET results to optimize both performance and recovery. A Journal of Sports Medicine study, “Graded Exercise Testing Protocols for the Determination of VO2 max: Historical Perspectives, Progress, and Future Considerations,” examines the relationship between GETs and physiological performance. The article highlights the impact of exercise on the "integrated cardiovascular, pulmonary, musculoskeletal, and neuropsychological systems" (Beltz et al.). Its primary focus is determining the optimal maximum oxygen consumption during exercise. While the following research takes a slightly different approach to analyzing GET results, it builds upon the fundamental concept of assessing oxygen consumption during and after exercise.

Performance tiers have historically been used by coaches and trainers to compare results from one athlete to another. It can also be used to implement fitness development plans. An article published by International Journal of Sports Physiology and Performance, “Defining Training and Performance Caliber: A Participant Classification Framework”, discusses both the benefits and negatives of classifications for athletes. It stresses the importance of taking into account a variety of factors, and wanted to expand the framework of athlete classifications by tailoring results to “different sports, athletes, and/or events” (McKay et. al). Although the article looks into the depth of sport participation as well as nationality differences, the performance tiers created using data collected by researchers at the University of Malaga will be more generalized due to lack of record for these attributes. 

## Data Description 

The GET result data used in the following analysis was collected in the Exercise Physiology and Human Performance Lab of the University of Malaga from 2008 to 2018. To obtain the data on the participants, a PowerJog J series treadmill was connected to a CPX MedGraphics gas analyzer system to collect breath-by-breath measurements while the participant wore a Mortara 12-lead ECG device for cardiovascular related metrics. There are three periods within the test: warmup, effort, and recovery. During the warmup and recovery periods participants walk at a treadmill speed of 5 km/h. For the effort stage of the test, participants were asked to go beyond exhaustion, with a cutoff time when the oxygen consumption was saturated (meaning that one cannot take in any more oxygen). 

The metrics that are recorded during the test are: heart rate (HR), oxygen consumption (VO2), CO2 production (VCO2), respiration rate (RR), and pulmonary ventilation (VE). These, in addition to the treadmill speed, time since measurement starts, participant ID, and effort test ID were provided in the test_measure.csv. There are a total of 575087 lines present in the file. It is important to note that 30 results are missing measures for VO2, VCO2, and VE. The units of measurement for the variables are outlined in Figure 1. 

![Figure 1](/images/get-figure1.png)
Figure 1: Units of measurement for variables in test_measure.csv.

Demographic information on each participant ID corresponding to the test results is provided in the _subject-info.csv_.  There are a total of 857 unique amateur and professional athletes who make up the test participants; however, the data does not specify participants’ athletic level to preserve anonymity. Included in the _subject-info.csv_ are the following measures: effort test ID, participant ID, age, weight, height, room humidity, room temperature, and gender. The participants range from age 10 to 63 years old.  

Before beginning analysis, the original structure of the data must be altered as preparation. The _subject-info.csv_ contains 992 rows; however, there are only 857 unique participants. This means some of the participants have multiple GETs recorded in the dataset. The athletes who participated in multiple tests thus have multiple entries in this csv, which are differentiated by the ID_test variable. Taking into account the complexities of the future clustering analysis, a new subset of both csv files provided was created such that each participant only has a singular test result stored. For the participants with multiple GET results, the most recent test (determined by participant age) was kept in the data subset. This new dataset will be referred to as _single_subject_info.csv_ in this paper. 

Additionally, a subset of the data was created that includes only the athletes who have taken multiple GETs and their corresponding results. This dataset adds another component to the analysis, allowing for the study of these participants’ metrics over time. This subsetted data will be referred to as _multiple_test_measure.csv_ in this paper. For this dataset, the creation of derived variables included maximum heart rate for each participant, maximum oxygen consumption, and maximum effort time. The maximum effort time for each test was created by the warmup end time from the recovery start time. In other words, it is the period in which the athlete puts in the most work. 

Derived variables for each athlete, stored in _single_subject_info.csv_, were created using data from _test_measure.csv_ and _subject-info.csv_.The first of these variables generated aimed to provide insight into the timing of when each period of the test begins and ends. This included warmup start and end time, effort start and end time, and recovery start and end time. Next, the average HR and RR during each of these time periods was calculated. Additional performance metrics such as maximum treadmill speed, speed to HR ratio, and VO2 to speed ratio were also created as derived variables for future analysis. To assess cardiovascular response in participants, peak HR, treadmill speed when peak HR is achieved, percent of time spent at peak HR, and HR variability were also computed. Ventilatory efficiency and respiratory exchange ratio variables were made to look into metabolic efficiency performance among the athletes. Finally, to analyze the recovery and fatigue of athletes the initial HR recorded (as a baseline HR), last HR recorded, HR recovery rate, and recovery HR drop percentage. Outlined in Figure 2 are the calculations for some of the generated features explained above. 

![Figure 2](/images/get-figure2.png)
Figure 2: Derived variable calculations. 

## Methods

Before beginning the k-means clustering process, an additional data cleaning step was performed on the _single_subject_info.csv_ and _test_measure.csv_. The 30 tests that do not contain records for VO2, VCO2, and VE (previously mentioned in the data description section) were removed from the datasets such that 846 rows resulted in the _single_subject_info.csv_. Before normalizing the variables, the columns containing the humidity information and temperature of the testing room were removed from the dataset. 

Using z-score scaling, the numeric columns were normalized and a new dataset called _scaled_subject_info.csv_ was created to store these scaled variables. The parameters that the k-means algorithm requires are then set for an initial cluster analysis. The x parameter is the data that will be contained in the clustering. Centers is the number of clusters the algorithm will generate. Itermax is the number of iterations to let k-means perform. Nstart is the number of starts to try for k-means. This is because k-means may not converge to the optimal solution from a single start point, thus multiple start points are tried. 

The cluster analysis was conducted using a set of physiological and performance-related variables. These variables include participant demographics (age, weight, height), time-based measures (warmup end time, recovery start time, recovery end time), heart rate (HR) metrics (peak HR, peak HR time, peak HR speed, initial HR, last HR, HR recovery rate, HR variability, and HR recovery drop percentage), speed-related variables (max speed, speed to HR ratio), and oxygen consumption measures (VO2 max, VO2 to speed ratio). Additionally, the analysis incorporated ventilatory and respiratory metrics, including the percentage of time spent at peak HR, average ventilatory efficiency, average respiratory exchange ratio, and average HR and respiratory rate (RR) during warmup, effort, and recovery periods.

In the first round of clustering, the analysis was performed using four cluster centers, with 25 nstart points and 100 iterations. This resulted in the following cluster distributions: cluster 1 contained 130 athletes, cluster 2 included 271 athletes, cluster 3 had 244 athletes, and cluster 4 consisted of 201 athletes. The variables distinguishing these clusters are illustrated in Figure 3. The figure suggests that while some features show clear differentiation, the clusters exhibit similar intensity levels across certain variables, indicating that they may not be highly distinct. To enhance cluster separation and identify potential subgroups within the initial clusters, increasing the number of clusters will be beneficial.

![Figure 3](/images/get-figure3.png)
Figure 3: Heatmap of the features most prominent in each cluster for initial round of clustering. 

Using an elbow plot, a different number of centers was selected to improve upon the initial clustering. In the second round of clustering, nine centers were chosen arbitrarily, as the elbow plot in Figure 4 was inconclusive. This resulted in clusters of the following sizes: cluster 1 with 105 participants, cluster 2 with 130, cluster 3 with 147, cluster 4 with 84, cluster 5 with 87, cluster 6 with 79, cluster 7 with 40, cluster 8 with 1, and cluster 9 with 173.

K-means clustering operates under two key assumptions: first, that clusters are spherical in a high-dimensional space, and second, that clusters are of equal size. While the first assumption is likely satisfied by the data, increasing the number of cluster centers reveals a clear violation of the second assumption. As a result, the single athlete in cluster 8 is removed from the dataset before proceeding with further analysis. 

![Figure 4](/images/get-figure4.png)
Figure 4: Elbow plot to determine the ideal number of clusters following the initial cluster analysis. 

As in the initial clustering analysis, the cardinality and magnitude of the nine clusters were examined. While these clusters offer an improvement in defining distinct participant groups, it remains essential to determine the optimal number of centers after removing the observation from cluster 8. To assess this, a silhouette plot and a gap statistic analysis were conducted to identify the ideal number of clusters. Figure 5 illustrates the optimal number of cluster centers based on the gap statistic. Since the silhouette plot suggests that only two clusters are optimal, the analysis proceeds with the 14 clusters recommended by the gap statistic.

![Figure 5](/images/get-figure5.png)
Figure 5: Mapping of the gap statistic to determine optimal number of cluster centers.

The resulting sizes of the clusters are broken down in Figure 6. These final 14 clusters are the base for the creation of the athlete performance tiers. Each cluster was then categorized based on the distinguishing variables. These categories are combined with research on fitness scores and physiological parameters in order to formulate ranks of which cluster is most to least fit. The attributes that define each cluster will be further explained in the following section, along with the details of each final performance tier created. 

![Figure 6](/images/get-figure6.png)
Figure 6: Final round of clustering size results. 

Upon the completion of the k-means cluster analysis, an examination of what factors contribute most to a faster HR recovery was conducted. When tracking HR decline after physical exertion, HR will typically decrease quickly at the start of recovery, and slow down as time increases. In other words, HR decline follows somewhat of an exponential decay curve. Thus, it was deemed fitting to use nonlinear least squares regression to model HR recovery as an exponential decay function. The physiological predictors used in this model include VO2 max, treadmill speed, respiratory exchange ratio, and ventilatory efficiency over time. The formula for multivariable exponential decay is shown in Figure 7 and will be referenced later in the results section. 

![Figure 7](/images/get-figure7.png)
Figure 7: The formula for exponential decay of multiple variables. 

## Results

Using the 14 clusters that resulted from k-means, 14 performance tiers were ranked and named based on the GET results and overall similarities. The naming and ranking of the tiers was not solely relying on relative position in terms of the 14 clusters, but rather attempted to take into consideration baseline fitness for any human. The heatmap of differentiating variables in Figure 8 was considered in the creation of the tiers, along with a heavy emphasis on VO2 max values, maximum treadmill speed, and HR recovery. Additionally, a series of visualizations for each cluster were created to assist in the discernment of the cluster rankings into tiers. 

![Figure 8](/images/get-figure8.png)
Figure 8: Heatmap of the clustering variables for all 14 total cluster groups 

The first of these visualizations is a scatter plot displaying the relationship between participant age and VO2 max, a key measure of cardiovascular fitness and aerobic capacity. The points of this scatter plot are colored based on the gender of the participant, which helps in identifying differences between males and females of different ages. The second visual is a scatter plot that displays the relationship between peak HR and HR recovery rate. It also inspects an additional variable, HR recovery drop percentage, which is represented by the size of the dots. The points are also colored by gender. This visualization helps in understanding how peak HR and recovery rate vary between males and females and how they are influenced by the recovery drop percentage. The third visualization is a scatter plot with a trend line, which displays the relationship between the speed to HR ratio and VO2 to speed ratio. These are both measures of cardiovascular efficiency and aerobic performance. Additionally, the points are colored on a gradient scale based on age. Overall, this visualization helps in identifying patterns and potential relationships in the fitness performance for each cluster. 

The 14 clusters each translated into its own performance tier, with tiers grouped into five different general categories. The first category, High Performance (Elite Athletes), contains the top three most fit and highest performing athlete tiers. Tier 1 is named VO2 Max Performer due to the elite oxygen utilization and exceptional endurance that athletes in this group (cluster 5) contain. Tier 2 is named Peak Speed Sprinter since athletes in this group (cluster 7) possess explosive power and reach the greatest top speed compared to any other cluster. The final tier in this category, tier 3, is given the name Efficient Power Performer. Athletes in this tier are well balanced and combine high aerobic and anaerobic ability. The variables that assisted in differentiating these clusters and tiers are outlined below in Figure 9.

![Figure 9](/images/get-figure9.png)
Figure 9: Performance tiers 1-3 derived from clusters & their attributes. 

The second category is referred to as Well Rounded (Strong, but Not Elite). This category is the next highest ranked tier grouping. Tier 4 (cluster 1) is named Aerobic Stamina, due to the strong endurance and efficient stamina that characterizes athletes in this cluster. The participants in cluster 6 are placed into tier 5, known as Balanced Hybrid athletes. These athletes have a mix of endurance and speed, while still maintaining a good recovery rate. Tier 6 is named Speed First Competitor due to the sprint focus and speed that athletes in this group (cluster 8) have, in combination with their average recovery. Further information on the average ranges categorizing each tier in the Well Rounded group is found in Figure 10. 

![Figure 10](/images/get-figure10.png)
Figure 10: Performance tiers 4-6 derived from clusters & their attributes. 

The next tier grouping is called Moderate Performance (Competent, but Limited). Tier 7, known as the Endurance Developing tier, is characterized by the decent endurance that athletes in cluster 4 contain, along with their slow recovery. Decreasing in terms of stamina, tier 8, the Explosive, but Limited group are strong sprinters for short periods with weak endurance. Tier 9 is named Respiratory Limited Competitor due to athletes in this cluster (cluster 11) having insufficient ventilation and struggling with aerobic work. Figure 11 further outlines each tier’s key attributes within this category.  

![Figure 11](/images/get-figure11.png)
Figure 11: Performance tiers 7-9 derived from clusters & their attributes. 

The second to last category that contains tiers 10-12 is referred to as Developing & Underperforming (Fitness Gap Present). Tier 10, Struggling with Recovery, is characterized by athletes in cluster 10 slow HR recovery and lack of endurance. Tier 11 is named Slow Adaptor due to the weak endurance and inefficient physiological responses that participants in this cluster (cluster 12) contain. Tier 12 is named Low Efficiency Performer due to their poor efficiency in all areas. Figure 12 displays the values of the variables that assisted in differentiating these clusters and tiers. It is important to note that these tiers are nearing or at the standard fitness level expected for an average human. In health, a HR reduction less than 12 bpm after 1 min of recovery is considered abnormal for a human, not just an athlete. This has been found to indicate that cardiovascular or autonomic nervous systems may not be functioning properly. 

![Figure 12](/images/get-figure12.png)
Figure 12: Performance tiers 10-12 derived from clusters & their attributes. 

The final category contains the two worst performing tiers. Tier 13 is known as the At Risk & Deconditioned group, populated by athletes in cluster 14. Athletes in this tier are the largest cluster within the data, but are one of the worst performing groups. They are significantly below athletic standards as well as basic healthy human standards. The final tier, 14, is named Physiological Concern Zone. Consisting of participants from cluster 3, this tier is characterized by having medically concerning fitness levels and should have additional evaluation into their health. It is notable that these athletes in this category are generally well below the base level for HR recovery after 1 min. This can have a variety of implications for the analysis, but may also indicate that there could be an issue with the way the data was collected and the values of metrics. 

![Figure 13](/images/get-figure13.png)
Figure 13: Performance tiers 13-14 derived from clusters & their attributes. 

After conducting the exponential decay regression, it was found that only two of the four variables had a significant effect on the speed of recovery. In the positive direction, a higher VO2 max corresponds with a faster HR recovery rate. The coefficient in this case was 0.116. Pulling recovery rate in the opposite direction was treadmill speed, which had a coefficient of -0.175. In other words, a faster treadmill speed corresponds with increased time it takes for HR to recover. Respiratory exchange rate was found to weakly contribute faster HR recovery, with a coefficient of 0.031. Ventilatory efficiency had no meaningful impact on the speed of recovery, with a coefficient of -0.002. The constant value of 0.314 represents the baseline HR recovery rate after a long period. These results correspond with basic knowledge of the human body and vitals after physical exertion. 

The final analysis conducted looked into the results of athletes with multiple GETs, specifically whether their results have changed over time. Athletes with IDs 7 and 605 were randomly selected for this analysis. Participant 7 is a 32 year old male who took his first test when he was 27. He falls into the Moderate Performance group as a member of cluster 9. Participant 605 is a 45 year old male who took both of the included tests in the same year. He falls into the Developing & Underperforming grouping as a member of cluster 12. Figure 14 shows a comparison between the maximum heart rates. Participant 7’s graph shows that the max HR decreased by 6 BPM after their most recent test, which could be attributed to improved efficiency or him getting used to the training. Participant 605 had a higher max HR and longer increase in HR over time during his most recent test. This could have something to do with his endurance or potentially fatigue. The VO2 max and RR were also examined for the two participants and are outlined below in Figures 15 and 16. Figures 17 and 18 show average heart rates and peak (maximum) heart rates for the two ends of the age spectrum. There were not any significant trends in either figure, however, in both, the older age group typically had a lower heart rate regardless of whether it was average or peak heart rate. This follows the biological fact that as one ages, cardiac health can decrease and activity levels have likely changed as well.

![Figure 14](/images/get-figure14.png)
Figure 14: Maximum Heart Rate Trends for Participant 7 and Participant 605.

![Figure 15](/images/get-figure15.png)
Figure 15: Maximum Oxygen Consumption for Participant 7 and Participant 605.

![Figure 16](/images/get-figure16.png)
Figure 16: Respiration Rate for Participant 7 and Participant 605.

![Figure 17](/images/get-figure17.png)
Figure 17: Average Heart Rates for participants between the ages of 14-24 and 43-59.

![Figure 18](/images/get-figure18.png)
Figure 18: Peak Heart Rates for participants between the ages of 14-24 and 39-59.

## Actions 

The performance tiers created based on the GET results is a beneficial tool for those working in human performance and player development. It is also a tool for athletes to have an understanding of their own health and examine bodily functions that are crucial to performance. Additionally, performance tiers are used for motivation, recognition, and progression. The tier system provides coaches, trainers, and athletes alike with an idea of where the competitor stands in relation to other athletes. It also shows a progression of skill level that can be used for improvement. The ratings also suggest ways in which the athlete can focus their training to maximize performance, workload, and training. 

The recovery analysis showed that having a higher oxygen consumption has a positive impact on faster recovery. For athletes and staff, this would be an area that could use improvement for the athletes that had low VO2 rates. Developing training plans that include aerobic and strength training, combined with proper hydration and nutrition, may improve the athlete’s endurance, and more specifically, oxygen consumption. Treadmill speed had a negative effect on the HR recovery rate, suggesting that running fast will increase the time of recovery. While these results largely follow biological consensus, it would be somewhat counteractive to suggest that athletes run slower in order to achieve a faster recovery. Instead, training in other areas to reduce recovery rate while maintaining high speeds is necessary. 

Using visualizations similar to those included in the time series portion of the analysis is an intuitive way to keep track of athlete development due to its visual nature. In training, it is important to continue to get better and track testing so that progress (or lack thereof) can be analyzed. 

## Conclusion & Future Work

To conclude this analysis, graded exercise test results can be a very useful tool in improving athletic performance and recovery. Human performance is ever-evolving, and there are many tools that can be used to understand an athlete’s performance and how they can improve. In the future, further analysis could be done on athletes who had more than one test. This could even be beneficial for teams to use, in order to compare athletes and check endurance levels from time to time. Another important aspect of the clustering analysis that could be looked into further, is why the largest and ”worst” cluster is so large. Some questions to consider include if the cluster was made up of amateurs or potentially injured athletes. All in all, the analysis was successful and would be a great resource for those interested in human performance. 

## Bibliography 

Beltz, Nicholas M., Gibson, Ann L., Janot, Jeffrey M., Kravitz, Len, Mermier, Christine M., Dalleck, Lance C., “Graded Exercise Testing Protocols for the Determination of VO2max: Historical Perspectives, Progress, and Future Considerations.” Journal of Sports Medicine, 2016. https://doi.org/10.1155/2016/3968393.

McKay, Alannah K.A., Trent Stellingwerff, Ella S. Smith, David T. Martin, Iñigo Mujika, Vicky L. Goosey-Tolfrey, Jeremy Sheppard, and Louise M. Burke. "Defining Training and Performance Caliber: A Participant Classification Framework". International Journal of Sports Physiology and Performance 17.2 (2022): 317-331. https://doi.org/10.1123/ijspp.2021-0451. Web. 22 Feb. 2025.

Mongin, D., García Romero, J., & Alvero Cruz, J. R. (2021). “Treadmill Maximal Exercise Tests from the Exercise Physiology and Human Performance Lab of the University of Malaga (version 1.0.1).” PhysioNet. https://doi.org/10.13026/7ezk-j442. 




