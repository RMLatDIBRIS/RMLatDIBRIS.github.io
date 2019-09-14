---
layout: default
title: Anomaly detection of sensors
show_downloads: false
---
# Anomaly detection of sensors

## Temperature range

```js
// temp1: temperature range anomaly detection for single or multiple sensors

temp_sensor_in_range(min,max) matches {event:'func_post',name:'temperature',res:temp} with min<=temp && temp<=max;

Main = CheckRange<22,24>;
CheckRange<min,max> = temp_sensor_in_range(min,max)*;
```
