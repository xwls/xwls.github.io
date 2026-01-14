+++
title = '工具方法-判断当前时间是否在指定时间范围'
date = 2020-08-13T13:43:09+08:00
categories = ['Util']
tags = ['java', 'golang', 'c#', 'util']
summary = '项目中遇到一个需求，简单来说就是预约和抢购，预约有开始时间和结束时间，抢购也有开始时间和结束时间，这些时间只需要在数据库中存储的是 time 类型，因为只需要小时分钟秒，不需要年月日。分享几个工具方法，用来比较当前时间是否在指定时间范围。'
+++

## Golang

```golang
//当前时间是否在指定范围内
//参数为时间字符串，格式为"时：分：秒"
func IsNowInTimeRange(startTimeStr, endTimeStr string) bool {
 //当前时间
 now := time.Now()
 //当前时间转换为"年-月-日"的格式
 format := now.Format("2006-01-02")
 //转换为 time 类型需要的格式
 layout := "2006-01-02 15:04:05"
 //将开始时间拼接"年-月-日 "转换为 time 类型
 timeStart, _ := time.ParseInLocation(layout, format+" "+startTimeStr, time.Local)
 //将结束时间拼接"年-月-日 "转换为 time 类型
 timeEnd, _ := time.ParseInLocation(layout, format+" "+endTimeStr, time.Local)
 //使用 time 的 Before 和 After 方法，判断当前时间是否在参数的时间范围
 return now.Before(timeEnd) && now.After(timeStart)
}
```

## Java

```java
/**

* 判断当前时间是否在指定时间内
* @param startTimeStr 开始时间
* @param endTimeStr 结束时间
* @return true：在；false：不在
 */
public static boolean isNowInTimeRange(String startTimeStr, String endTimeStr) {
    LocalTime startTime = LocalTime.parse(startTimeStr);
    LocalTime endTime = LocalTime.parse(endTimeStr);
    int startCompare = startTime.compareTo(LocalTime.now());
    int endCompare = endTime.compareTo(LocalTime.now());
return startCompare* endCompare <= 0;
}
```

## C#

```csharp
/// <summary>
/// 判断当前时间是否在时间段之间
/// </summary>
/// <param name="startTime">开始时间</param>
/// <param name="endTime">结束时间</param>
/// <returns></returns>
public static bool NowTimeBetweenTwoTimes(string startTimeStr, string endTimeStr, int serverTimeStamp)
{
    // 解析时间
    DateTime startTime = DateTime.Parse(startTimeStr);
    DateTime endTime = DateTime.Parse(endTimeStr);

    TimeZone.CurrentTimeZone.ToLocalTime(new System.DateTime(1970, 1, 1));
    DateTime serverTime = TimeStampToDateTime(serverTimeStamp);
    // t1 小于 t2 小于零 t1 等于 t2 等于零 t1 大于 t2 大于零
    int startCompare = DateTime.Compare(startTime, serverTime);
    int endCompare = DateTime.Compare(endTime, serverTime);
    return startCompare * endCompare <= 0;
}