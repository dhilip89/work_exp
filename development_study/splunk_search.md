## Splunk Search
```
index=behavior source=suiyue_behavior sourcetype=suiyue_behavior behavior_event=app_on_open | fields channel udid |dedup 1 channel udid | timechart span=1d count by channel
```