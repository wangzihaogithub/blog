---
layout: post
title:  "我的Netty项目-优化响应头的Date"
tags: 我的Netty项目
---

### 发现设置响应头的CPU时间长
    
![dateformat](../../../images/postimg/dateformat.png)

    发现别的都是赋值语句，应该没问题。 那么锁定到下面这句话
    headers.set(HttpHeaderConstants.DATE, DATE_FORMAT_GMT_LOCAL.get().format(new Date()));

这个日期应该会生成相对多的小对象和一些计算。相比这个方法中的其他赋值语句来说。
其实它1秒生成一次就行，于是改成如下.

      private static final FastThreadLocal<DateFormat> DATE_FORMAT_GMT_LOCAL = new FastThreadLocal<DateFormat>() {
            private TimeZone timeZone = TimeZone.getTimeZone("GMT");
            @Override
            protected DateFormat initialValue() {
                DateFormat df = new SimpleDateFormat("EEE, dd-MMM-yyyy HH:mm:ss z", Locale.ENGLISH);
                df.setTimeZone(timeZone);
                return df;
            }
        };
        private static long lastTimestamp;
        private static String nowRFCTime;
    
        public static String getDateByRfcHttp(){
            long timestamp = System.currentTimeMillis();
            if(timestamp - lastTimestamp > 1000){
                lastTimestamp = timestamp;
                nowRFCTime = DATE_FORMAT_GMT_LOCAL.get().format(new Date(timestamp));
                System.out.println("cacheCount="+cache);
                System.out.println(nowRFCTime);
                cache= 0;
            }else {
                cache++;
            }
            return nowRFCTime;
        }
        public static void main(String[] args) {
            while (true){
                getDateByRfcHttp();
            }
        }

效果如下

    Sat, 07-Dec-2019 16:36:14 GMT
    cacheCount=209380959次
    Sat, 07-Dec-2019 16:36:15 GMT
    cacheCount=208618483次
    Sat, 07-Dec-2019 16:36:16 GMT
    cacheCount=236709287次
    Sat, 07-Dec-2019 16:36:17 GMT
    cacheCount=234956261次
    Sat, 07-Dec-2019 16:36:18 GMT
    cacheCount=86650219次
    Sat, 07-Dec-2019 16:36:19 GMT
    cacheCount=73971423次
    Sat, 07-Dec-2019 16:36:20 GMT
    cacheCount=201444191次
    Sat, 07-Dec-2019 16:36:21 GMT
    cacheCount=235206764次
    Sat, 07-Dec-2019 16:36:22 GMT
    cacheCount=236035989次
    Sat, 07-Dec-2019 16:36:23 GMT

改完后，JMeter测试吞吐量从12900+/s 变成了13650+/s

![datetimeafter](../../../images/postimg/dateformatafter.jpg)

