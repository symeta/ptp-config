# ptp-config

- Please refer to below guidance information for EC2 instance types that support Amazon Time Sync Service

[Official Amazon Time Sync Service Guidance](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/configure-ec2-ntp.html#connect-to-the-ptp-hardware-clock)

## EC2 PTP Enabling Congfig

- Amazon Linux OS

- - as-is ena status checking, ENA driver version: 2.16.1g and above — supports PTP/PHC.
```sh
modinfo ena | grep version
```
#check phc_enable, as-is should be 0
cat /sys/module/ena/parameters/phc_enable

#check the time source of chronyc, as-is should be NTP
chronyc sources -v
```
output looks like below:
```sh
  .-- Source mode  '^' = server, '=' = peer, '#' = local clock.
 / .- Source state '*' = current best, '+' = combined, '-' = not combined,
| /             'x' = may be in error, '~' = too variable, '?' = unusable.
||                                                 .- xxxx [ yyyy ] +/- zzzz
||      Reachability register (octal) -.           |  xxxx = adjusted offset,
||      Log2(Polling interval) --.      |          |  yyyy = measured offset,
||                                \     |          |  zzzz = estimated error.
||                                 |    |           \
MS Name/IP address         Stratum Poll Reach LastRx Last sample
===============================================================================
^* 169.254.169.123               1   4   377    15    +13us[  +15us] +/-   87us
^- ec2-3-87-127-143.compute>     4   6   177    49    -18us[  -21us] +/-  843us
^- ec2-44-201-148-133.compu>     4   6   177    48    -18us[  -21us] +/-  792us
^- ec2-54-81-127-33.compute>     4   6   177    49    -89us[  -92us] +/-  585us
^- ec2-54-90-191-9.compute->     4   6   177    48    -20us[  -23us] +/-  493us
```
