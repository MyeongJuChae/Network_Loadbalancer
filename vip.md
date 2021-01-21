## 191(master), 95(backup)
***
191번(keepalived on)
eno1: 

inet 10.122.64.191/24 brd 10.122.64.255 scope global noprefixroute eno1

inet 10.122.64.119/24 scope global secondary eno1

95번(keepalived on)
enp5s0: 
    **inet 10.122.64.119/24 scope global secondary enp5s0**
       valid_lft forever preferred_lft forever
***
191번(keepliaved stop)
