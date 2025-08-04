index=main | stats count by host

index=firewall | top limit=5 src_ip

index=auth action=failure


index=web status>=500 
| timechart span=1d count as error_count


index=access_logs | stats avg(response_time) by uri_path

index=auth action=success 
| stats count by user


index=auth action=failure 
| stats count by user 
| where count > 3


index=auth action=failure 
| stats latest(_time) as last_fail by user 
| join user [search index=auth action=success 
             | stats earliest(_time) as first_success by user] 
| where first_success > last_fail	



index=web sourcetype=access_combined 
| timechart span=10m count(eval(status>=500)) as errors, count as total 
| eval error_rate = round((errors/total)*100, 2)	


index=auth action=failure 
| bin _time span=5m 
| stats count by src_ip, _time 
| where count > 10
