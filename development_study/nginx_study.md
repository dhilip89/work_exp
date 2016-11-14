## return 200 json
 /user_behavior {
          default_type application/json;
          return 200 '{"stauts": 0, "msg":"", "data":[]}';
      }
      
## nginx json_log
```
log_format  log_json '{ "@timestamp": "$time_local", '
                         '"@fields": { '
                         '"remote_addr": "$remote_addr", '
                         '"remote_user": "$remote_user", '
                         '"body_bytes_sent": "$body_bytes_sent", '
                         '"request_time": "$request_time", '
                         '"status": "$status", '
                         '"request": "$request", '
                         '"request_method": "$request_method", '
                         '"http_referrer": "$http_referer", '
                         '"body_bytes_sent":"$body_bytes_sent", '
                         '"http_x_forwarded_for": "$http_x_forwarded_for", '
                         '"http_user_agent": "$http_user_agent",'
                         '"query_string": "$query_string",'
                         '"http_cookie" : "$http_cookie",'
                         '"http_header" : "$http_header",'
                         '"request_body": "$request_body"} }';
```
