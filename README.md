# ptp-config

- Please refer to below guidance information for EC2 instance types that support Amazon Time Sync Service

[Official Amazon Time Sync Service Guidance](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/configure-ec2-ntp.html#connect-to-the-ptp-hardware-clock)

## EC2 PTP Enabling Congfig

>below operations are done using Amazon Linux OS

- Current Status Checking

  - as-is ena status checking, ENA driver version: 2.16.1g and above — supports PTP/PHC.
  ```sh
  modinfo ena | grep version
  ```
  - check phc_enable, as-is should be 0
  ```sh
  cat /sys/module/ena/parameters/phc_enable
  ```
  >⚠️ This will briefly drop network connectivity since it reloads the network driver. Alternatively, to make it persistent without reloading     >now:
  >```sh
  >echo "options ena phc_enable=1" | sudo tee /etc/modprobe.d/ena.conf
  >```
  >Then reboot, or reload the module.
  
  - check the time source of chronyc, as-is should be NTP
  ```sh
  chronyc sources -v
  ```
  - output looks like below, indicating NTP is the current time source.
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

- PTP Enablement
  - Enable PHC in the ENA driver
  ```sh
  sudo modprobe -r ena && sudo modprobe ena phc_enable=1
  ```
  - After PHC is enabled, /dev/ptp0 should appear. Verify with:
  ```sh
  ls /dev/ptp0
  ```
    reverts back with below output:
  ```sh
  /dev/ptp0
  ```
  - Configure chrony to use PTP by adding a refclock line. Create /etc/chrony.d/ptp.conf:
  ```txt
  refclock PHC /dev/ptp0 poll 0 dpoll -2 offset 0 prefer
  ```
  - Restart chrony
  ```sh
  sudo systemctl restart chronyd
  ```
- Status Re-check
  - check phc_enable, as-is should be 1
  ```sh
  cat /sys/module/ena/parameters/phc_enable
  ```
  - check the time source of chronyc, as-is should be NTP
  ```sh
  chronyc sources -v
  ```
  - output looks like below, indicating NTP is the current time source.
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
  #* PHC0                          0   0   377     1   -224ns[ -250ns] +/-   36ns 
  ^- 169.254.169.123               1   4    77     9  +4161ns[+3950ns] +/-   90us 
  ^- ec2-54-90-191-9.compute->     4   6    17    43    -19us[  -19us] +/-  502us 
  ^- ec2-44-201-148-133.compu>     4   6    17    42    -31us[  -31us] +/-  796us 
  ^- ec2-54-81-127-33.compute>     4   6    17    42    -28us[  -28us] +/-  512us 
  ^- ec2-3-87-127-143.compute>     4   6    17    42    -28us[  -28us] +/-  833us 
  ```
  
