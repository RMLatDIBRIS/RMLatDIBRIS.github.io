---
layout: default
title: Sensors and anomaly detection
show_downloads: false
---
# Sensors and anomaly detection

## Value range anomaly 

```js
// range: range anomaly detection for single or multiple sensors

sensor_in_range(min,max) matches sensor(val) with min<=val && val<=max;

Main = CheckRange<22,24>;
CheckRange<min,max> = sensor_in_range(min,max)*;
```

## Timestamp anomaly

```js
// timestamp: timestamp anomaly detection for single or multiple sensors

check_time(time1,time2) matches sensor(time2) with time2 > time1;

Main = {let time; sensor(time) CheckTime<time>}!;
CheckTime<time1> = {let time2; check_time(time1,time2) CheckTime<time2>};
```

## Value derivative anomaly

```js
// derivative: derivative anomaly detection for single or multiple sensors

check_der(val1,time1,val2,time2) matches sensor(val2,time2)
			       with delta==(val2-val1)/(time2-time1) && abs(delta) <= 1; 

Main = {let val1,time1; sensor(val1,time1) CheckDer<val1,time1>}!;
CheckDer<val1,time1> = {let val2, time2; check_der(val1,time1,val2,time2) CheckDer<val2,time2>};
```

## Damped harmonic oscillator

```js
// anomaly detection of a damped harmonic oscillator with a distance sensor 
// works with traces generated from oscillator.js

// based on the standard equation
// pos(time) = amplitude e^(-zeta omega_0 time) sin(omega_1 time + phase)
// where zeta=c/(2 sqrt(mk)) is the damping ratio,
//       omega_0=sqrt(k/m) is the undamped angular frequency,
//      k is the spring constant, c is the viscous damping coefficient,
//      omega_1=sqrt(1 - zeta^2) omega_0 is the angular frequency

sensor(pos,time) matches {event:'func_post',name:'sensor',res:{position:pos, time:time}};

check_dots(pos1, time1, pos2, time2) matches sensor(pos2,time2)
			       with
			       e==2.718 && k==5000 && c==18.6*10^-6 && m==1 && phase==0 && error==10^-5
			       &&
			       zeta==c/(2*(m*k)^0.5) && omega0==(k/m)^0.5 && omega1==omega0*(1-zeta^2)^0.5
			       &&
			       delta1==pos1-amplitude*(e^(-zeta*omega0*time1)*sin(omega1*time1+phase))
			       &&
			       delta2==pos2-amplitude*(e^(-zeta*omega0*time2)*sin(omega1*time2+phase))
			       &&
			       delta1 <= error
			       &&
			       delta1 >= -error
			       &&
			       delta2 <= error
			       &&
			       delta2 >= -error;
			       
Main = {let pos1, time1; sensor(pos1, time1) CheckDot<pos1, time1>}!;
CheckDot<pos1, time1> = {let pos2, time2; check_dots(pos1, time1, pos2, time2) CheckDot<pos2,time2>};
```
