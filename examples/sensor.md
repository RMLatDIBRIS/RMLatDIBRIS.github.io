---
layout: default
title: Anomaly detection of sensors
show_downloads: false
---
# Anomaly detection of sensors

## Value range anomaly 

```js
// range: range anomaly detection for single or multiple sensors

sensor_in_range(min,max) matches {event:'func_post',name:'sensor',res:{value:val}} with min<=val && val<=max;

Main = CheckRange<22,24>;
CheckRange<min,max> = sensor_in_range(min,max)*;
```

## Timestamp anomaly

```js
// timestamp: timestamp anomaly detection for single or multiple sensors
// works with traces generated from timestamp.js

sensor(time) matches {event:'func_post',name:'sensor',res:{timestamp:time}};
check_time(time1,time2) matches sensor(time2) with time2 > time1;
relevant matches sensor(_);

Main = relevant >> {let time; sensor(time) CheckTime<time>}!;
CheckTime<time1> = {let time2; check_time(time1,time2) CheckTime<time2>};
```

## Value derivative anomaly

```js
// derivative: derivative anomaly detection for single or multiple sensors

sensor(val,time) matches {event:'func_post',name:'sensor',res:{value:val,timestamp:time}};
check_der(val1,time1,val2,time2) matches sensor(val2,time2)
			       with delta==(val2-val1)/(time2-time1) && abs(delta) <= 1; 

Main = {let val1,time1; sensor(val1,time1) CheckDer<val1,time1>}!;
CheckDer<val1,time1> = {let val2, time2; check_der(val1,time1,val2,time2) CheckDer<val2,time2>};
```
