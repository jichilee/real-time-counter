real-time-counter
=================

A real time counter system (We call it rtc) will be used for counter user login and logout 
in some scenario? what is hard to know when user login or logout.e.g. 
A video web want to counter uv/pv/vv of one vedio real time, and its use may be a cookie, 
which is alive only when its heartbeat log is reveived each interval seconds by the counter.
How to deal this scenario? and how to reduce latency?

Implementation:
We use storm to real time calculate, redis to cache (but not use its ttl APIs), and implement a
rotatingmap based on redis to manage key expire and update counter realtime.

Important:
1.RTC is base on storm but not must!
2.RTC uses reids as cache, we all know that redis has some API to deal with key expire.Here is why we dont use:
  Solution 1.We first use its API to manage key expire, and use "keys pattern" to counter uv/pv everytime.
    But "keys pattern" is low efficiency when keys is increasing big and big. And the redis will manage more and
    more keys, efficiency is lower and memory is bigger!
  Solution 2.We give up "keys pattern", use key-expire-notification event. We just want when the key is expire, redis
    can nofify us then we can update counter.It looks well,but redis send notification is so lazy, even uncertainty!
  After tried the two ways, We implement our expire mechanism base on redis, and to promote its efficiency, we use bucket
  to expire, which means there are more keys expired not only one(this is more like the real world),each time we deal a
  expired bucket, and update counter only once(if not, each key expire should update counter once)!
  To reduce the memory store, we use bitmap to store cookie status and calculate total uv/pv.
  To reduce the bitmap collision, we use double hash to check, which keeps collision only about 0.0066392 when test dataset is 10,000,000.
  
