# Exploring Hacker News Posts
In this project, I am going to analyze two types of posts from [Hacker News](https://news.ycombinator.com/), where technology and startup related stories are voted and commented upon.                         

Two type of posts I am going to observe begin with either `'Ask HN'` or `'Show HN'`. Users of the wesbite sumbit `'Ask HN'`posts to ask the Hacker News community a specific question. For example: "How to improve my personal website?", "Am I the only one outraged by Twitter shutting down share counts?". `'Show HN'` posts are sumbitted to show the Hacker News community a project or just something interesting.

I will compare these two types of posts to indicate the following: 
- Do `'Ask HN'` or `'Show HN'` receive more comments on average? 
- Do posts created at a certain time receive more comments on average?

If you want to look on the data set, you can find it [here](https://www.kaggle.com/hacker-news/hacker-news-posts). However it is important to menthion that I reduced the data set I am going to work with from almost 300,000 rows to approximately 20,000 rows by removing all posts that didn't receive any comments. 

Let's start importing necessary libraries and analyzing the data.


```python
import csv 
open_file = open('hacker_news.csv')
hn = list(csv.reader(open_file))
hn[:5]
```




    [['id', 'title', 'url', 'num_points', 'num_comments', 'author', 'created_at'],
     ['12224879',
      'Interactive Dynamic Video',
      'http://www.interactivedynamicvideo.com/',
      '386',
      '52',
      'ne0phyte',
      '8/4/2016 11:52'],
     ['10975351',
      'How to Use Open Source and Shut the Fuck Up at the Same Time',
      'http://hueniverse.com/2016/01/26/how-to-use-open-source-and-shut-the-fuck-up-at-the-same-time/',
      '39',
      '10',
      'josep2',
      '1/26/2016 19:30'],
     ['11964716',
      "Florida DJs May Face Felony for April Fools' Water Joke",
      'http://www.thewire.com/entertainment/2013/04/florida-djs-april-fools-water-joke/63798/',
      '2',
      '1',
      'vezycash',
      '6/23/2016 22:20'],
     ['11919867',
      'Technology ventures: From Idea to Enterprise',
      'https://www.amazon.com/Technology-Ventures-Enterprise-Thomas-Byers/dp/0073523429',
      '3',
      '1',
      'hswarna',
      '6/17/2016 0:01']]



## Removing Headers from a List of Lists
This data set has first list as the column headers, and the lists after contain the data for one row. In order to analyze the data, rown containing the column headers should be removed. 


```python
headers = hn[0]
hn = hn[1:]

print(headers)
print('\n')
print(hn[:5])
```

    ['id', 'title', 'url', 'num_points', 'num_comments', 'author', 'created_at']
    
    
    [['12224879', 'Interactive Dynamic Video', 'http://www.interactivedynamicvideo.com/', '386', '52', 'ne0phyte', '8/4/2016 11:52'], ['10975351', 'How to Use Open Source and Shut the Fuck Up at the Same Time', 'http://hueniverse.com/2016/01/26/how-to-use-open-source-and-shut-the-fuck-up-at-the-same-time/', '39', '10', 'josep2', '1/26/2016 19:30'], ['11964716', "Florida DJs May Face Felony for April Fools' Water Joke", 'http://www.thewire.com/entertainment/2013/04/florida-djs-april-fools-water-joke/63798/', '2', '1', 'vezycash', '6/23/2016 22:20'], ['11919867', 'Technology ventures: From Idea to Enterprise', 'https://www.amazon.com/Technology-Ventures-Enterprise-Thomas-Byers/dp/0073523429', '3', '1', 'hswarna', '6/17/2016 0:01'], ['10301696', 'Note by Note: The Making of Steinway L1037 (2007)', 'http://www.nytimes.com/2007/11/07/movies/07stein.html?_r=0', '8', '2', 'walterbell', '9/30/2015 4:12']]
    

## Extracting Ask HN and Show HN Posts
As the headers were removed from `'hn'` variable, it is time to filter out `'Ask HN'` and `'Show HN'` by creating another list that will contain data only related to these titles.

I will use function `startswith` to find posts that begin wit either `'Ask HN'` or `'Show HN'`.


```python
ask_posts = []
show_posts = []
other_posts = []

for row in hn: 
    title = row[1]
    if title.lower().startswith('ask hn'):
        ask_posts.append(row)
    elif title.lower().startswith('show hn'): 
        show_posts.append(row)
    else: 
        other_posts.append(row)

print('There are ' + str(len(ask_posts)) + ' ' + 'ask posts in the list')
print('There are ' + str(len(show_posts)) + ' ' + 'show posts in the list')
print('There are ' + str(len(other_posts)) + ' ' + 'other posts in the list')

```

    There are 1744 ask posts in the list
    There are 1162 show posts in the list
    There are 17194 other posts in the list
    

As two types of posts are separated into different lists, I will caculate average number of comments each type of post receives. 


```python
total_ask_comments = 0
for row in ask_posts: 
    total_ask_comments += int(row[4])

    avg_ask_comments = total_ask_comments / len(ask_posts)
print('Average number of comments in ask posts:', avg_ask_comments)

total_show_comments = 0
for row in show_posts: 
    total_show_comments += int(row[4])

avg_show_comments = total_show_comments / len(show_posts)
print('Average number of comments in show posts:', avg_show_comments)
```

    Average number of comments in ask posts: 14.038417431192661
    Average number of comments in show posts: 10.31669535283993
    

On average, ask posts in this data set receive 14 comments and show posts receive around 10 comments only. As ask posts have more comments on average, my rest of the analysis will be focused on this type of posts.

## Finding the Number of Ask Posts and Comments by Hour Created
Now I will determine if ask posts created at specific *time* are more likely to attract comments. In order to accomplish this goal, I will do following steps: 
- Calculate the number of ask posts created in each hour of the day, along with the number of comments received. 
- Calculate the average number of comments ask posts received by hour created.


```python
import datetime as dt
result_list = []
for row in ask_posts: 
    result_list.append([row[6], int(row[4])])
    
print(result_list[1], '\n') # to check date and time format

counts_by_hour = {}
comments_by_hour = {}
date_format = '%m/%d/%Y %H:%M'

for row in result_list: 
    date = row[0]
    comment = row[1]
    hour = dt.datetime.strptime(date, date_format).strftime('%H')
    if hour in counts_by_hour: 
        counts_by_hour[hour] += 1
        comments_by_hour[hour] += comment
    else: 
        counts_by_hour[hour] = 1
        comments_by_hour[hour] = comment
        
print(counts_by_hour, '\n')

print(comments_by_hour)

```

    ['11/22/2015 13:43', 29] 
    
    {'09': 45, '13': 85, '10': 59, '14': 107, '16': 108, '23': 68, '12': 73, '17': 100, '15': 116, '21': 109, '20': 80, '02': 58, '18': 109, '03': 54, '05': 46, '19': 110, '01': 60, '22': 71, '08': 48, '04': 47, '00': 55, '06': 44, '07': 34, '11': 58} 
    
    {'09': 251, '13': 1253, '10': 793, '14': 1416, '16': 1814, '23': 543, '12': 687, '17': 1146, '15': 4477, '21': 1745, '20': 1722, '02': 1381, '18': 1439, '03': 421, '05': 464, '19': 1188, '01': 683, '22': 479, '08': 492, '04': 337, '00': 447, '06': 397, '07': 267, '11': 641}
    

## Calculating the Average Number of Comments for Ask HN Posts by Hour
Now, I am going to use created two dictionaries to calculate the average number of comments for posts created during each hour of the day.


```python
avg_by_hour = [] 

for row in comments_by_hour:
    avg_by_hour.append([row, comments_by_hour[row] / counts_by_hour[row]])
print(avg_by_hour)   
```

    [['09', 5.5777777777777775], ['13', 14.741176470588234], ['10', 13.440677966101696], ['14', 13.233644859813085], ['16', 16.796296296296298], ['23', 7.985294117647059], ['12', 9.41095890410959], ['17', 11.46], ['15', 38.5948275862069], ['21', 16.009174311926607], ['20', 21.525], ['02', 23.810344827586206], ['18', 13.20183486238532], ['03', 7.796296296296297], ['05', 10.08695652173913], ['19', 10.8], ['01', 11.383333333333333], ['22', 6.746478873239437], ['08', 10.25], ['04', 7.170212765957447], ['00', 8.127272727272727], ['06', 9.022727272727273], ['07', 7.852941176470588], ['11', 11.051724137931034]]
    

## Sorting and Printing Values from a List of Lists
Even if the results of the analysis are already printed out, the lack of any sorting makes it difficult to identify hours with the higest average values.


```python
swap_avg_by_hour = []
for row in avg_by_hour: 
    swap_avg_by_hour.append([row[1],row[0]])
print(swap_avg_by_hour, '\n')

sorted_final_by_hour = sorted(swap_avg_by_hour, reverse = True)
print(sorted_final_by_hour, '\n')

print('Top 5 Hours for Ask Posts Comments')
for avg, hr in sorted_final_by_hour[:5]:
    print(
        "{}: {:.2f} average comments per post".format(
            dt.datetime.strptime(hr, "%H").strftime("%H:%M"),avg
        )
    )
```

    [[5.5777777777777775, '09'], [14.741176470588234, '13'], [13.440677966101696, '10'], [13.233644859813085, '14'], [16.796296296296298, '16'], [7.985294117647059, '23'], [9.41095890410959, '12'], [11.46, '17'], [38.5948275862069, '15'], [16.009174311926607, '21'], [21.525, '20'], [23.810344827586206, '02'], [13.20183486238532, '18'], [7.796296296296297, '03'], [10.08695652173913, '05'], [10.8, '19'], [11.383333333333333, '01'], [6.746478873239437, '22'], [10.25, '08'], [7.170212765957447, '04'], [8.127272727272727, '00'], [9.022727272727273, '06'], [7.852941176470588, '07'], [11.051724137931034, '11']] 
    
    [[38.5948275862069, '15'], [23.810344827586206, '02'], [21.525, '20'], [16.796296296296298, '16'], [16.009174311926607, '21'], [14.741176470588234, '13'], [13.440677966101696, '10'], [13.233644859813085, '14'], [13.20183486238532, '18'], [11.46, '17'], [11.383333333333333, '01'], [11.051724137931034, '11'], [10.8, '19'], [10.25, '08'], [10.08695652173913, '05'], [9.41095890410959, '12'], [9.022727272727273, '06'], [8.127272727272727, '00'], [7.985294117647059, '23'], [7.852941176470588, '07'], [7.796296296296297, '03'], [7.170212765957447, '04'], [6.746478873239437, '22'], [5.5777777777777775, '09']] 
    
    Top 5 Hours for Ask Posts Comments
    15:00: 38.59 average comments per post
    02:00: 23.81 average comments per post
    20:00: 21.52 average comments per post
    16:00: 16.80 average comments per post
    21:00: 16.01 average comments per post
    

15:00 is the hour with the highest average value of comments per post - 38.59. It the most preferable hour to have a higher chances of receiving comments.

## Conclusion
In this project, I analyzed two different types of posts in Hacker News website, `'Ask HN'` and `'Show HN'`. My outcome of the analysis is that `'Ask HN'` posts receive more comments on average and from 15:00 to 16:00 (3:00 pm est - 4:00 pm est) is the most recommended time period for maximizing number of comments for a post.

However, my data analysis excluded posts that didn't receive any comments. Because of that it is more appropriate to say that *between the posts that received comments,* ask posts received more comments on average and posts created between 15:00 and 16:00 received the most comments on average.
