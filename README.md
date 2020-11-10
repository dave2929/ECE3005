# What makes a YouTube video trend?

## About Me

Hello, I am Yuqi(David) He, a third year Computer Engineering student at Georgia Tech, specializing in low-level programming in C/C++, computer architecture, and operating system. My career goal is to learn from my experience and my interaction with other people and leave a positive impact in this world through my work.

## Past Experience

I have completed machine learning and AI cmyses and have decent understanding in machine learning. I have done many machine learning projects including a well-trained Discord chat bot. I am familiar with both supervised and unsupervised machine learning approaches. There is my [resume](https://github.gatech.edu/yhe374/ECE3005/blob/master/David%20He's%20Resume.pdf).

## Work Purpose & Scope

YouTube is an online video-sharing platform with billions of users, posting and watching content for entertainment and education. YouTube __monetizes popular videos__ for the number of views as it increases the use and popularity of the platform itself. With my project, I hope to provide insights on how to make money on YouTube by analyzing trending video features and predicting the popularity of a video given certain features. This article is for people who are interested in machine learning and YouTube content creators.

## Work Introduction & Motivations

Why do I want to do machine learning analysis on YouTube videos?

I want to help people **monetize their content more effectively** on YouTube by increasing their popularity on the platform! To that end, I carry out the following analyses:

1. **PCA**: to **reduce dimension of features** through capturing variation and visualize correlation between different features <br/>
2. **DBSCAN**: to find the **popular time published** and **optimal length, number of capital letters, and punctuation in video title** <br/>
3. **Linear Regression**: predict the views based upon any other correlated features <br/>
4. **Multiple Regression**: to predict the popularity of a video solely based on the **title's characteristics** (length, capital letters, and puctuation)<br/>

## Dataset

-> https://www.kaggle.com/datasnaek/youtube-new <br/>
This dataset includes several months of data on daily trending YouTube videos. Data includes the video title, channel title, publish time, tags, views, likes and dislikes, description, and comment count for up to 200 trending videos per day for several regions. More information like the channel’s age, channel's video count, and subscriber count have been added using the YouTube API. From this dataset, I will only be using the USA's, Canada's, and Great Britain's trending video data.

### YouTube API

I also augment the available data with my own data scavenged from YouTube using its APIs. I wrote a script to fetch the top 200 trending videos from USA, Canada and Great Britain everyday for roughly a month. In addition, the channel information was also fetched and compiled with the data from Kaggle.

## Cleaning up the data

1. **Languages**: Youtube fosters content from all around the world in numerous languages. I discarded video entries in the dataset that do not have English titles to facilitate the analysis of the data.
2. **Handling duplicates**: Several videos are in trending charts for multiple days. Therefore, I retained only one copy of each video, the version with the highest number of views.
3. **Null entries**: videos with ratings disabled and comments disabled were removed.
4. **Handling dates and time**: I parsed the dates and times in YouTube's native format to formats suitable for machine learning algorithms (floats and integets)
5. **Handling outliers**: To gain insights for the majority of the data, any data points that were outside +/-2 standard deviations from the mean were eliminated.

### Data Format

The csv format of final file that contains both Old and New data:
| | | |
| -------------------- | ---------------------- | ----------------------------- |
|1. regionTrending |12. videoDislikes |23. thumbnail_link |
|2. trendingRank |13. videoCommentCount |24. comments_disabled |
|3. timeFetched |14. videoDescription |25. ratings_disabled |
|4. videoId |15. videoLicenced |26. video_error_or_removed |
|5. videoTitle |16. channelTitle |27. publishDateCorrectFormat |
|6. videoCategoryId |17. channelId |28. trendingDateCorrectFormat |
|7. videoPublishTime |18. channelDescription |29. dayDifference |
|8. videoDuration |19. channelPublishedAt |30. publishedZTime |
|9. videoTags |20. channelViewCount |31. publishedZTimeFloat |
|10. videoViews |21. channelSubsCount |32. publishedDayOfWeek |
|11. videoLikes |22. channelVideoCount |33. newOrOldData |

## Data Analysis

**Correlation**\
First, an analysis of the correlation between different variables was performed. Looking at the first row, it is seen that only likes, dislikes and comment count are correlated with the number of views of a video.

<p align="center">
  <img src="https://raw.githubusercontent.com/shyam100v/cs4641Project/master/image/Correlation%20table.PNG">
	<br>
	  Figure 1: Correlation Table
</p>

**Statistical Summary**\
This analysis is useful to potentially identify any outliers in the data.

<p align="center">
  <img src="https://raw.githubusercontent.com/shyam100v/cs4641Project/master/image/max-min-std.PNG">
	<br>
	  Figure 2: Statistical Summary
</p>

**Category ID Analysis**\
The category ID represents the content of the video.

- **Categories with high number of views**: "Film & Animation", "Music", "News and Politics", and "Entertainment"
- **Categories with least number of views**: "Nonprofits & Activism" and "Shorts"

<p align="center">
  <img src="https://raw.githubusercontent.com/shyam100v/cs4641Project/master/image/CategoryID.PNG">
	<br>
	  Figure 3: Views versus Category ID
</p>

## Principal Component Analysis(PCA)

I selected some properties from the original video dataset as features of videos, including trending rank, video category, number of views, likes, and dislikes, number of comments, publish time, and video channel related features. For features like the duration of video and the publish time, I preprocessed my data such that they are represented in the same unit (seconds) and in a 24-hour time scale.

I combined all 12 features into a training dataset and apply PCA. PCA was used to reduce the dimension of features through capturing variation. Here is the cumulative explained variance plot.

<p align="center">
  <img src="https://raw.githubusercontent.com/shyam100v/cs4641Project/master/image/pca_variance.PNG">
	<br>
	  Figure 4: PCA variance plot
</p>

From the plot, I can see that at 6 components, I will get a desired cumulative explained variance(0.9). I also made two component PCA scatter plots to give us some visualizations.

<p align="center">
  <img src="https://raw.githubusercontent.com/shyam100v/cs4641Project/master/image/pca_views.PNG">
	<br>
	  Figure 5: PCA number of views plot
</p>

<p align="center">
  <img src="https://raw.githubusercontent.com/shyam100v/cs4641Project/master/image/pca_likes.PNG">
	<br>
	  Figure 6: PCA number of likes plot
</p>

<p align="center">
  <img src="https://raw.githubusercontent.com/shyam100v/cs4641Project/master/image/pca_dislikes.PNG">
	<br>
	  Figure 7: PCA number of dislikes plot
</p>

From the scatter plots, I can see that the number of views, likes, and dislikes are correlated. Low number of views, low number of likes, and low number of dislikes are all clustered at the middle-left part of the graph.

## DBSCAN

**Publishing Times**\
Using DBSCAN clustering on the video views and publishing time features, I can see that the optimal time frame to publish videos on YouTube is from about **1:00 pm to 7:00 pm GMT** with the peak time between **5:00 pm - 7:00 pm**; however, I did find many noise points and the clusters found were quite low in view count. (see [code)](https://raw.githubusercontent.com/shyam100v/cs4641Project/master/DBSCAN_publishingHour.ipynb)

<p align="center">
  <img src="https://raw.githubusercontent.com/shyam100v/cs4641Project/master/image/dbscan_clusters.PNG">
	<br>
	  Figure 8.1: DBSCAN Hour Published
</p>

**Video Titles**\
After using DBSCAN clustering on three different characteristics of a video title (length, number of capital letters,and number of exclamation/question marks) I can see that the results in the graphs below are different than I originally expected. (see [code](https://aw.githubusercontent.com/shyam100v/cs4641Project/master/title_analysis.py))

<p align="center">
  <img src="https://raw.githubusercontent.com/shyam100v/cs4641Project/master/image/Title_Length_DBSCAN.png">
	<br>
	  Figure 8.2: DBSCAN Title Length
</p>

<p align="center">
  <img src="https://raw.githubusercontent.com/shyam100v/cs4641Project/master/image/Num_Caps_DBSCAN.png">
	<br>
	  Figure 8.3: DBSCAN Capital Letters
</p>

<p align="center">
  <img src="https://raw.githubusercontent.com/shyam100v/cs4641Project/master/image/Num_Puncs_DBSCAN.png">
	<br>
	  Figure 8.4: DBSCAN Punctuation Marks
</p>

The clusters for majority of the graphs are evenly distributed along the x axis, showing very little correlation to the number of views. But, there are a couple clusters higher up on the graph that show that titles with lengths around 40 characters, minimal capital letters, and minimal punctuation marks are the videos accumulating more views. As a conclusion, there is little relation seen between the factors, but there is some evidence pointing to certain title formats.

## Linear Regression

**Histograms**\
A basic histogram of the number of views, likes and dislikes was plotted. As it is appreciated on the graphs below, the three graphs are heavily skewed which is understandable — most common YouTubers probably won’t have that many views, likes and dislikes. Ideally, the data should resemble a Gaussian distribution. Luckily, a log tranformation can be applied to the number of views, likes and dislikes to achieve that.

![Number of views without logs](https://raw.githubusercontent.com/shyam100v/cs4641Project/master/image/Number%20of%20views%20without%20logs.PNG)
![Number of dislikes without logs](https://raw.githubusercontent.com/shyam100v/cs4641Project/master/image/Number%20of%20dislikes%20without%20logs.PNG)
![Number of likes without logs](https://raw.githubusercontent.com/shyam100v/cs4641Project/master/image/Number%20of%20likes%20without%20logs.PNG)
![Log number of dislikes](https://raw.githubusercontent.com/shyam100v/cs4641Project/master/image/Log%20number%20of%20dislikes.PNG)
![Log number of likes](https://raw.githubusercontent.com/shyam100v/cs4641Project/master/image/Log%20number%20of%20likes.PNG)
![Log number of views](https://raw.githubusercontent.com/shyam100v/cs4641Project/master/image/Log%20number%20of%20views.PNG)

**Linear Regression**\
Using this 3 histograms above, a linear regression of the log number of views versus the log number of likes and dislikes can be plotted (see [code](https://github.com/shyam100v/cs4641Project/blob/master/Linear%20Regression.py)). As expected, both correlations show an R^2 greater than 0.6, showing a big correlation between both the number of likes and dislikes and the number of views of a video.

<p align="center">
  <img src="https://raw.githubusercontent.com/shyam100v/cs4641Project/master/image/Linear%20regression%20of%20log%20views%20versus%20log%20dislikes.PNG">
	<br>
	  Figure 9: Linear regression of log views versus log dislikes
</p>
<p align="center">
  <img src="https://raw.githubusercontent.com/shyam100v/cs4641Project/master/image/Linear%20regression%20of%20log%20views%20versus%20log%20likes.PNG">
	<br>
	  Figure 10: Linear regression of log views versus log likes
</p>

This analysis was only performed on these 2 variables since no other variables appear to correlate with the number of views of a video. For example, when performing the linear regression of views versus the title length, an R^2 close to 0 is obtained.

<p align="center">
  <img src="https://raw.githubusercontent.com/shyam100v/cs4641Project/master/image/Views%20versus%20Title%20Length.PNG">
	<br>
	  Figure 11: Linear regression of views versus title length
</p>

## Multiple Regression

**Data Modifications**\
Data regarding number of views was originally very skewed, so I removed any outliers and took the log of the number of views. This resulted in a Gaussian distribution for the number of views, which is ideal for any modeling. The below graphs show the data distribution before and after applying the log function.

<p align="center">
  <img src="https://raw.githubusercontent.com/shyam100v/cs4641Project/master/image/MultReg_viewskew.png">
	<br>
	  Figure 12: Multiple Regression Skewed Data
</p>

<p align="center">
  <img src="https://raw.githubusercontent.com/shyam100v/cs4641Project/master/image/MultReg_viewlog.png">
	<br>
	  Figure 13: Multiple Regression Modified Data
</p>

**Multiple Regression Analysis**\
I used three parameters versus the number of views on a video (title length, number of capital letters, number of punctuations marks). This would have resulted in a 3-dimensional regression model in a 4-dimensional space. As it is difficult to understand a 4-dimensional model, the following graphs show the regression models of just two parameters each versus the log of the number of views. (see [code](<https://raw.githubusercontent.com/shyam100v/cs4641Project/master/Multiple%20Regression%20(Video%20Title%20Analysis).ipynb>))

<p align="center">
  <img src="https://raw.githubusercontent.com/shyam100v/cs4641Project/master/image/MultReg_lengthcapsreg.png">
	<br>
	  Figure 14: Multiple Regression Length vs Number of Capital Letters
</p>

<p align="center">
  <img src="https://raw.githubusercontent.com/shyam100v/cs4641Project/master/image/MultReg_lengthpuncsreg.png">
	<br>
	  Figure 15: Multiple Regression Length vs Number of Punctuation Marks
</p>

<p align="center">
  <img src="https://raw.githubusercontent.com/shyam100v/cs4641Project/master/image/MultReg_capspuncsreg.png">
	<br>
	  Figure 16: Multiple Number of Capital Letters vs Number of Punctuation Marks
</p>

As can be seen above, the grey plane in each of the graphs is the regression model that fits the set of data points the best. Since my data points are fairly spread out and do not show much correlation to the number of views, the fitted planes are relatively flat, and center around the area where most of the data points are in order to at least fit for a majority of videos.

**Result Analysis**\
As a representation of the accuracy of my model, the below graph shows a comparison of the actual number views versus the predicted number of views for a random set of data points.

</p>

<p align="center">
  <img src="https://raw.githubusercontent.com/shyam100v/cs4641Project/master/image/MultReg_predvsact.png">
	<br>
	  Figure 17: Predicted Views vs Actual Views
</p>

In the below graph, where it shows the error in prediction based on the number of actual views, it can be seen that the lower the actual number of views a video has, the more accurate this model will be in its predictions.

<p align="center">
  <img src="https://raw.githubusercontent.com/shyam100v/cs4641Project/master/image/MultReg_accuracy.png">
	<br>
	  Figure 18: Multiple Regression Model Accuracy
</p>

In conclusion, I see that the characteristics of a video title actually have a much smaller effect on the popularity of a video than I originally believed and has an incredibly low correlation with the number of views, especially on videos with extremely high view count.

## Conclusion

From the analyses I carried out, following are the key insights and results:

1. **Categories with high number of views**: Film & Animation, Music, Entertainment and News and Politics.
2. **Categories with least number of views**: Nonprofits & Activism and Shorts
3. **1:00 pm to 7:00 pm GMT** is a popular time frame to publish videos so they trend, especially between 5:00 pm-7:00 pm. There is **no optimal title length. Fewer (or none) capital letters and punctuation** in video title is optimal.
4. There is a relationship between number of dislikes, number of likes, and number of views.
5. The video title plays a **minor role** in the popularity of the most popular videos. But, for the less exposed or advertised videos, keeping to minimal capital letters and punctuation can help boost views slightly.
6. The **channel's popularity** plays a major role in determining the popularity of a video. Particularly, the **channel subscriber and view count, age of channel and channel video count** are dominant factors.

## Contact Information

If you want to reach out to me or ask any questions, please email yhe374@gatech.edu.

## References:

[1] Gill, Phillipa et al. "Youtube traffic characterization: a view from the edge." Proceedings of the 7th ACM SIGCOMM conference on Internet measurement. 2007. <br/>
[2] F Figueiredoet al "The tube over time: characterizing popularity growth of youtube videos." Proceedings of the fourth ACM international conference on Web search and data mining. 2011.<br/>
[3] G. Chatzopoulou et al, "A First Step Towards Understanding Popularity in YouTube," INFOCOM IEEE Conference on Computer Communications Workshops, CA, 2010 <br/>
[4] Coding, Sigma. “How to Build a Linear Regression Model in Python | Part 1.” Youtube, Apr. 2019, www.youtube.com/watch?v=MRm5sBfdBBQ.<br/>
[5] "DBSCAN Clustering Easily Explained with Implementation." Youtube, https://www.youtube.com/watch?v=C3r7tGRe2eI&t=942s <br/>
[6] "Multiple Regression Analysis in Python | Part 1." Youtube Apr 27, 2019 https://www.youtube.com/watch?v=M32ghIt1c88 <br/>
[7] "Youtube Views Predictor." Aravind Srinivasan, December 12, 2017 https://towardsdatascience.com/youtube-views-predictor-9ec573090acb <br/>
