# VC++获取系统精确到毫秒时间

1. 使用CTime类（获取系统当前时间，精确到秒）

   ```C++
   CString str; 				   // 获取系统时间
   CTime tm; 
   tm=CTime::GetCurrentTime();	 	// 获取系统日期
   str=tm.Format("现在时间是%Y年%m月%d日 %X");
   MessageBox(str,NULL,MB_OK);
   ```
   
   ```C++
   // 从CTimet中提取年月日时分秒
   CTime t = CTime::GetCurrentTime();
   int d=t.GetDay(); 				//获得几号
   int y=t.GetYear(); 				// 获取年份
   int m=t.GetMonth(); 			// 获取当前月份
   int h=t.GetHour(); 				// 获取当前为几时
   int mm=t.GetMinute(); 			// 获取分钟
   int s=t.GetSecond(); 			// 获取秒
   int w=t.GetDayOfWeek(); 		// 获取星期几，注意1为星期天，7为星期六
   ```

   计算两段时间的差值，可以使用CTimeSpan类，具体使用方法如下：
   
   ```C++
   CTime t1( 1999, 3, 19, 22, 15, 0 );
   CTime t = CTime::GetCurrentTime();
   CTimeSpan span=t-t1; 			// 计算当前系统时间与时间t1的间隔
   int iDay=span.GetDays(); 		// 获取这段时间间隔共有多少天
   int iHour=span.GetTotalHours();  // 获取总共有多少小时
   int iMin=span.GetTotalMinutes(); // 获取总共有多少分钟
   int iSec=span.GetTotalSeconds(); // 获取总共有多少秒
   ```
   
   获得当前日期和时间，并可以转化为CString
   
   ```C++
   CTime tm=CTime::GetCurrentTime();
   CString str=tm.Format("%Y-%m-%d");// 显示年月日
   ```
   
   
   
2. 使用GetLocalTime：Windows API 函数，获取当地的当前系统日期和时间 （精确到毫秒）

   此函数会把获取的系统时间信息存储到SYSTEMTIME结构体里边

   ```C++
   typedef struct _SYSTEMTIME 　　{
       WORD wYear;					// 年
       WORD wMonth;				// 月
       WORD wDayOfWeek;			// 星期：0为星期日，1为星期一，2为星期二……
       WORD wDay;					// 日
       WORD wHour;					// 时
       WORD wMinute;				// 分
       WORD wSecond;				// 秒
       WORD wMilliseconds;			// 毫秒
   }SYSTEMTIME,*PSYSTEMTIME;
   ```

    　　