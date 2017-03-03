## Splunk Search
```
index=behavior source=suiyue_behavior sourcetype=suiyue_behavior behavior_event=app_on_open | fields channel udid |dedup 1 channel udid | timechart span=1d count by channel
```

## 解析从pgsql导出的数据，数据中部分字段为json格式，将这样的字段解析成json
```
index=behavior source=suiyue_behavior sourcetype=suiyue_behavior behavior_event=app_on_open | rex "geo_info=\"(?<geo_info>.+)\", header_info=" | spath input=geo_info | table geo_info
```

## 城市地理位置解析
```
index=behavior source=suiyue_behavior sourcetype=suiyue_behavior behavior_event=app_on_open | dedup 1 channel udid | rex "geo_info=\"(?<geo_info>.+)\", header_info=" | fields geo_info |eval _raw=geo_info | spath input=geo_info | timechart span=1d count by geoip_city

index=behavior source=suiyue_behavior sourcetype=suiyue_behavior behavior_event=app_on_open | dedup 1 channel udid | rex "geo_info=\"(?<geo_info>.+)\", header_info=" | fields geo_info |eval _raw=geo_info | spath input=geo_info | geostats latfield=geoip_latitude longfield=geoip_longitude count
```

## feed_click report
```
index=behavior source=suiyue_behavior sourcetype=suiyue_behavior behavior_event=feed_on_entry referer != '' |eval feed_type=case(
referer == "feed/comment", "乐评",
referer == "feed/hot", "精选",
referer == "feed/listen_top", "收听榜",
referer == "feed/new", "最新",
referer == "feed/question", "问答",
referer == "feed/ranklist", "榜单",
referer == "feed/ranklist-new", "新榜单",
referer == "feed/recommend", "推荐",
referer == "feed/sale", "销量榜",
referer == "feed/star", "明星",
referer == "feed/topic", "话题") | timechart span=1d count by feed_type
```

## 统计去留
```
index=behavior source=suiyue_behavior sourcetype=suiyue_behavior behavior_event=app_on_open earliest=-4d@d latest=-3d@d | dedup 1 channel udid |fields udid | eval zero=1 | join type=outer udid [search index=behavior source=suiyue_behavior sourcetype=suiyue_behavior behavior_event=app_on_open earliest=-3d@d latest=-2d@d | dedup 1 channel udid | eval first=1 | table udid first] | stats sum(zero) as zero_sum, sum(first) as first_sum | eval first_day_ratio = first_sum/zero_sum*100 | table *


index=behavior source=suiyue_behavior sourcetype=suiyue_behavior behavior_event=app_on_open earliest=-31d latest=-30d | dedup 1 channel udid |fields udid | eval zero=1 
| join type=outer udid [search index=behavior source=suiyue_behavior sourcetype=suiyue_behavior behavior_event=app_on_open earliest=-30d latest=-29d | dedup 1 channel udid | eval first=1 | table udid first] 
| join type=outer udid [search index=behavior source=suiyue_behavior sourcetype=suiyue_behavior behavior_event=app_on_open earliest=-29d latest=-28d | dedup 1 channel udid | eval second=1 | table udid second] 
| stats sum(zero) as zero_sum, sum(first) as first_sum, sum(second) as second_sum 
| eval first_ratio = first_sum/zero_sum*100, second_raio=second_sum/zero_sum*100
```


convert mktime(_time) AS ms_time | table ms_time

| convert mktime(_time) AS s_time | where s_time > 1484236800 and s_time < 1484323199| table _time

convert mktime(_time) AS s_time | where s_time > 1484236800 and s_time < 1484323199| table _time

convert timeformat='%Y-%m-%d' mktime('2017-01-13') AS ms_time | table ms_time

index=behavior source=suiyue_behavior sourcetype=suiyue_behavior behavior_event=app_on_open platform="ios" | dedup 1 channel udid | fields udid  | convert mktime(_time) AS s_time | where s_time >= 1484236800 and s_time <= 1484323199 | table _time



```
source="/Users/rick/service/splunk/splunk_data/feed/last_rank_listen_count.csv" host="ricks-MacBook-Pro.local" index="last_rank_listen_count" sourcetype="csv" earliest=-3h latest=-2h | join works_id [ | search source="/Users/rick/service/splunk/splunk_data/feed/last_rank_listen_count.csv" host="ricks-MacBook-Pro.local" index="last_rank_listen_count" sourcetype="csv" earliest=-2h latest=-h | eval listen_count=play_num | table works_id listen_count] 
| eval top_hot_=if(top_hot == 1, "热度榜", "") | eval top_origin_=if(top_origin == 1, "原创榜", "") | eval top_cover_=if(top_cover ==  1, "翻唱榜", "") | eval top_video_=if(top_video == 1, "视频榜", "") | eval rank_type= top_hot_ ."_". top_origin_ ."_". top_cover_ ."_". top_video_ | eval increase_ratio=listen_count/play_num | table  title, works_id, rank_type, play_num,, listen_count , increase_ratio | sort +increase_ratio | rename increase_ratio as 两小时增长
```

### 用户活跃度
```
index=behavior source=suiyue_behavior sourcetype=suiyue_behavior platform=android | fields behavior_event udid platform |stats count by udid  | sort -count | rename count as "活跃度" |table *

index=behavior source=suiyue_behavior sourcetype=suiyue_behavior platform=android AND (channel=yingyongbao1 OR channel= yingyongbaofufei1 OR channel= chuanbo1) | fields behavior_event udid platform |stats count by udid  | sort -count | rename count as "活跃度" |table *
```