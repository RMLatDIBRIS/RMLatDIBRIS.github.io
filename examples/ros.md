---
layout: default
title: Verification robotic systems
show_downloads: false
---
# Speed control of Curiosity rover

```js
left_speed matches {event:'func_pre', name:'command', args:[{topic:'wheels_control',direction:'left',speed:val}]} with val <= 10;
right_speed matches {event:'func_pre', name:'command', args:[{topic:'wheels_control',direction:'right',speed:val}]} with val <= 10;
forward_speed matches {event:'func_pre', name:'command', args:[{topic:'wheels_control',direction:'forward',speed:val}]} with val <= 15;
backward_speed matches {event:'func_pre', name:'command', args:[{topic:'wheels_control',direction:'backward',speed:val}]} with val <= 15;

relevant matches {event:'func_pre', name:'command'};

Main = relevant>>(left_speed \/ right_speed \/ forward_speed \/ backward_speed)*;
```

