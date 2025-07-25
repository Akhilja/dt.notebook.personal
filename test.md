1.index=trade sourcetype=tradexml " Process is up and running" | rex max_match=1 "date and time:(?<HealthCheckTS>.*)"
| eval healthCheckEpoch = strptime(HealthCheckTS,"%Y-%m-%d %H:%M:%S.%3N")
| eval secondsSinceLastHC = round(now()-healthCheckEpoch,0)
| eval status=case(secondsSinceLastHC>120,"ERROR",secondsSinceLastHC>65 AND secondsSinceLastHC<=120,"Warning",secondsSinceLastHC<=65,"OK")
| table host,HealthCheckTS,secondsSinceLastHC,status
| sort secondsSinceLastHC,host
| dedup host
| rename host as Server
 
 
2. index=auth (action=success OR action=failure)
|stats count(eval(action="success")) as success_count,
count(eval(action="failure")) as fail_count by host
|eval login_ratio=round(success_count/(fail_count + 1), 2)
|where login_ratio < 1
 
 
3. index=auth action=success
|fields user _time
|join user
[search index=web_logs uri_path="/admin" |fields user uri_path]
|table user _time uri_path
 
4. index=web sourcetype=access_combined
|timechart span=10m 
count(eval(status>=500)) AS errors, count AS total
|eval error_rate = round((errors/total)*100, 2)
 
 
5. index=auth action=success
|stats earliest(_time) as first_seen by user src_ip
|where first_seen >= relative_time(now(),"-1d@d")
