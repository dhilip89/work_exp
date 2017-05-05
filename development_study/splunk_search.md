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


### test
```
| savedsearch "tbl_suiyuedb_user_relation" relation_type=1 | top 100 to_user_id | join type=left to_user_id [| savedsearch "tbl_suiyuedb_user" | rename id as to_user_id | table to_user_id, nickname] | table to_user_id, nickname, count, percent | rename count as 粉丝数, nickname as 用户名称, to_user_id as 用户id, percent as 占比


| savedsearch "tbl_suiyuedb_user" | rename id as to_user_id | table to_user_id, nickname | join type=left to_user_id [| savedsearch "tbl_suiyuedb_user_relation" relation_type=1 | top 100 to_user_id] | table to_user_id, nickname, count, percent | rename count as 粉丝数, nickname as 用户名称, to_user_id as 用户id, percent as 占比


-------
碎乐单
| savedsearch "tbl_suiyuedb_user"  | rename id as user_id | table user_id, nickname| join user_id  [| savedsearch "tbl_suiyuedb_play_list" | top 100 user_id | table user_id count percent] | sort -count|rename count as 碎乐单, nickname as 用户名称, user_id as 用户id, percent as 占比

粉丝数
| savedsearch "tbl_suiyuedb_user"  | rename id as user_id | table user_id, nickname| join user_id  [| savedsearch "tbl_suiyuedb_user_relation"  | top 100 to_user_id |rename to_user_id as user_id |table user_id count percent] | sort -count|rename count as 粉丝数, nickname as 用户名称, user_id as 用户id, percent as 占比
```


###审核作品数

```
|dbxquery query="select count(*) as dc, date_trunc('day', create_time::TIMESTAMP without time zone) as ts from works where status=2 group by ts order by ts" connection="suiyue_db" shortnames="yes" | eval _time=strptime(ts,"%Y-%m-%d %H:%M:%S") | eval weeknumber= tonumber(strftime(_time,"%U"))+1,monthnumber=strftime(_time,"%m"),yearnumber=strftime(_time,"%Y") | timechart span=1d values(dc) as 审核通过作品数
```

```

1.index=behavior source=suiyue_behavior sourcetype=suiyue_behavior behavior_event= subscribe_channel_on_entry | rex "behavior_info=\"(?<behavior_info>.+)\", geo_info=" | fields behavior_info | eval _raw=behavior_info | spath input=behavior_info | rename item_id as channel_id | stats count by channel_id | join type=left channel_id [| savedsearch "tbl_suiyuedb_channel" | rename id as channel_id | table channel_id,name] | table name channel_id count| sort -count |rename count as 浏览数, channel_id as 频道id, name as 频道名称

2.index=behavior source=suiyue_behavior sourcetype=suiyue_behavior behavior_event= subscribe_btn_on_click | rex "behavior_info=\"(?<behavior_info>.+)\", geo_info=" | fields behavior_info | eval _raw=behavior_info | spath input=behavior_info | rename item_id as channel_id | stats count by channel_id | join type=left channel_id [| savedsearch "tbl_suiyuedb_channel" | rename id as channel_id | table channel_id,name] | table name channel_id count| sort -count |rename count as 点击数, channel_id as 频道id, name as 频道名称

3.index=behavior source=suiyue_behavior sourcetype=suiyue_behavior behavior_event= guess_like_on_click | rex "behavior_info=\"(?<behavior_info>.+)\", geo_info=" | fields behavior_info | eval _raw=behavior_info | spath input=behavior_info | rename item_id as works_id | stats count by works_id | join works_id [| savedsearch "snap_suiyuedb_works_dbxquery" | rename id as works_id | table works_id,title] | table title works_id count| sort -count |rename count as 浏览数, works_id as 作品id, title as 作品名称

4.index=behavior source=suiyue_behavior sourcetype=suiyue_behavior behavior_event= subscribe_channel_on_entry | rex "behavior_info=\"(?<behavior_info>.+)\", geo_info=" | fields behavior_info | eval _raw=behavior_info | spath input=behavior_info | rename item_id as channel_id | stats count by channel_id | join type=left channel_id [| savedsearch "tbl_suiyuedb_channel_dbxquery" | rename id as channel_id | table channel_id,name] | table name channel_id count| sort -count | rename count as 点击数, channel_id as 频道id, name as 频道名称
```

### 根据版本号，时间，查数据
```
index=behavior source=suiyue_behavior sourcetype=suiyue_behavior behavior_event=feed_on_entry referer != "" |rex "device_info=\"(?<device_info>.+)\", behavior_info=" | fields device_info, referer | eval json_data=device_info | spath input=json_data  | where appv=172  |eval feed_type=case(
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

```
SELECT create_time::TIMESTAMP without time zone as ts, * FROM "node_music_weapon"."public"."works_playtimes"


|dbxquery query="select count(*) as dc, date_trunc('day', create_time::TIMESTAMP without time zone) as ts from works_playtimes group by ts order by ts" connection="suiyue_db" shortnames="yes" | eval _time=strptime(ts,"%Y-%m-%d %H:%M:%S") | eval weeknumber= tonumber(strftime(_time,"%U"))+1,monthnumber=strftime(_time,"%m"),yearnumber=strftime(_time,"%Y") | timechart span=1d values(dc) as 每日播放量
```

### 每日作品收听报表
```
| savedsearch "tbl_suiyuedb_play_times" | top 200 works_id | rename count as listen_num | join type=outer works_id [search index="suiyuedb"  sourcetype="suiyuedb_works" earliest=-5y latest=now | rename id as works_id | table works_id,title] | table works_id title listen_num | rename works_id as 作品id, title as 作品名称, listen_num as 试听次数
```

### 每日搜索热词报表
```
index=behavior source=suiyue_behavior sourcetype=suiyue_behavior behavior_event= search_on_display | rex "behavior_info=\"(?<behavior_info>.+)\", geo_info=" | fields behavior_info | eval _raw=behavior_info | spath input=behavior_info | where event_result=1 | stats count by item | sort -count | head 300 | rename item as 搜索词语, count as 搜索次数
```