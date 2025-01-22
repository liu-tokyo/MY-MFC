# VC常用源码

## 1. 滚动条处理方法

- 设置范围

  ```c++
  m_spscroll.SetScrollRange(0,200); 
  SCROLLINFO si; 
  si.cbSize=sizeof(SCROLLINFO); 
  si.nPage=100; 
  si.fMask=SIF_PAGE;//设置页宽 
  m_spscroll.SetScrollInfo(&si); 
  ```

- 处理消息，垂直滚动条加WM_VSCROLL消息，水平加WM_HSCROLL消息

  ```c++
  void CPrintView::OnVScroll(UINT nSBCode, UINT nPos, CScrollBar* pScrollBar) 
  { 
  	// TODO: Add your message handler code here and/or call default 
  	if(pScrollBar->GetDlgCtrlID()==IDC_SCROLLBAR1) 
  	{ 
  		int nCurrentPos=pScrollBar->GetScrollPos(); 
  		SCROLLINFO si; 
  		si.fMask=SIF_PAGE;//取得页宽 
  		pScrollBar->GetScrollInfo(&si); 
  		switch(nSBCode) 
  		{ 
  			case SB_THUMBTRACK://移动滑块 
  			case SB_THUMBPOSITION: 
  				pScrollBar->SetScrollPos(nPos);//注意，设置页宽后滚动条的pos会以max/nPage倍数减少，所以在使用时注意把pos值*(max/nPage)才能得到原值 
  				break; 
  			case SB_LINEUP://点向上小三角 
  				pScrollBar->SetScrollPos(nCurrentPos-1); 
  				break; 
  			case SB_LINEDOWN://点向下小三角 
  				pScrollBar->SetScrollPos(nCurrentPos+1); 
  				break; 
  			case SB_PAGEUP://向上一页 
  				pScrollBar->SetScrollPos(nCurrentPos-si.nPage); 
  				break;
  			case SB_PAGEDOWN://向下一页 
  				pScrollBar->SetScrollPos(nCurrentPos+si.nPage); 
  				break; 
  		} 
  	} 
  	CDialog::OnVScroll(nSBCode, nPos, pScrollBar); 
  } 
  ```

  

## 2. 锁定鼠标

- 锁定鼠标

  ```c++
  bool pOld; 
  CRect rt; 
  SetForegroundWindow(); 
  SystemParametersInfo(SPI_SETSCREENSAVERRUNNING,true,&pOld,SPIF_UPDATEINIFILE); 
  GetWindowRect(rt); 
  ClipCursor(rt); 
  //加到 LRESULT CLockDlg::WindowProc(UINT message, WPARAM wParam, LPARAM lParam) 会有意外的效果 
  ```

  

## 3. 在列表字符前插入一个负数字符以修改乱码

- 在列表字符前插入一个负数字符以修改乱码

  ```c++
  int index=m_list.GetSelectionMark();
  //在列表字符前插入一个负数字符以修改乱码 
  CString cs; 
  cs=m_list.GetItemText(index,0); 
  char insert_char=-87; 
  cs.Insert(0,insert_char); 
  m_list.SetItemText(index,0,cs); 
  ```

  

## 4. 在列表中添加项目最大只能显示259个字符(不含'/0')

- 在列表中添加项目最大只能显示259个字符(不含'/0')

  ```c++
  int char_length=cs.GetLength();
  //cs,ct为CString类对象，是要发到列表框的文本但是可能大于259字节 
  while(char_length>259)//如果大于259字节 
  {
  	ct=cs.Left(259); 
  	m_list.InsertItem(0,ct);//在列表中添加项目最大只能显示259个字符(不含'/0') 
  	cs=cs.Right(char_length-259); 
  	char_length=cs.GetLength(); 
  } 
  m_list.InsertItem(0,cs);//在列表中添加项目最大只能显示259个字符(不含'/0') 
  //＜========================================================== 
  
  ```

  

## 5. 设置NT窗口的透明度

- 设置NT窗口的透明度

  ```c++
  OSVERSIONINFO osv; 
  osv.dwOSVersionInfoSize=sizeof OSVERSIONINFO; 
  GetVersionEx(&osv);//取得版本信息 
  if(osv.dwPlatformId==VER_PLATFORM_WIN32_NT)//VER_PLATFORM_WIN32_WINDOWS 98 Me用这个宏 
  { 
  	//加入WS_EX_LAYERED扩展属性 
  	SetWindowLong(this->GetSafeHwnd(),GWL_EXSTYLE, 
  	GetWindowLong(this->GetSafeHwnd(),GWL_EXSTYLE)^0x80000);//如果多次调用下面这个函数设置，这个函数只在一个位置调用一次就行了 
  	HINSTANCE hInst = LoadLibrary("User32.DLL"); 
  	if(hInst) 
  	{ 
  		typedef BOOL (WINAPI *MYFUNC)(HWND,COLORREF,BYTE,DWORD); 
  		MYFUNC fun = NULL; 
  		//取得SetLayeredWindowAttributes函数指针 
  		fun=(MYFUNC)GetProcAddress(hInst, "SetLayeredWindowAttributes"); 
  		if(fun)fun(this->GetSafeHwnd(),0, 
  			200,//0 ~ 255 
  			2); 
  	FreeLibrary(hInst); 
  	} 
  }
  ```

  

## 6. 字体对话框的初始化

- 字体对话框的初始化

  ```c++
  LOGFONT lf; 
  lf.lfHeight=-35; 
  lf.lfCharSet=134; 
  lf.lfWeight=400; 
  lf.lfOutPrecision=3; 
  lf.lfClipPrecision=2; 
  lf.lfQuality=1; 
  lf.lfPitchAndFamily=2; 
  strcpy(lf.lfFaceName,"宋体");//以上初始化为宋体26号字 
  CFontDialog cf(&lf);//字体 
  cf.m_cf.rgbColors=textcolor;//颜色 
  ```

  

## 7. 移动没有标题的窗口

- 移动没有标题的窗口

  ```c++
  //1定义： 
  CPoint just_point; 
  //2 
  void CClockfortecherDlg::OnLButtonDown(UINT nFlags, CPoint point) 
  { 
  	// TODO: Add your message handler code here and/or call default 
  	just_point=point; 
  	CDialog::OnLButtonDown(nFlags, point); 
  } 
  //3 
  void CClockfortecherDlg::OnMouseMove(UINT nFlags, CPoint point) 
  { 
  	// TODO: Add your message handler code here and/or call default 
  	WINDOWPLACEMENT wi; 
  	GetWindowPlacement(&wi); 
  	if(nFlags==MK_LBUTTON) 
  	SetWindowPos(&wndTop, 
  	wi.rcNormalPosition.left+(point.x-just_point.x), 
  	wi.rcNormalPosition.top+(point.y-just_point.y), 
  	0,0,SWP_NOSIZE); 
  	CDialog::OnMouseMove(nFlags, point); 
  } 
  ```

  

## 8. 线程与信号量

```c++
//1,定义信号量句柄 
HANDLE　event; 
//2,创建信号量 
event=CreateEvent(NULL, TRUE, FALSE, NULL); 
//3,创建线程, 
//1)定义线程函数，格式必须如下,其中lParam为AfxBeginThread的第二个参数值，可强制转化成所需类型 
UINT WorkThreadProc(LPVOID lParam)//必须是 UINT XXX..XXX(LPVOID lParam) 
{ 
	//代码示例WaitForSingleObject： 
	while(1) 
	{//-----------------------注意，如果线程间要求同步或互斥的时候，要在每一层循环体中加入WaitForSingleObject 
		WaitForSingleObject((HANDLE)lParam, INFINITE); 
		//WaitForSingleObject的使用方法：第一个为信号量HANDLE，是CreateEvent的返回值,第二个参数为等待的毫秒数（1/1000秒） 
		//第二个参数为INFINITE时则一直等待，直到调用SetEvent()设置信号量时函数返回；为数值（如1000）则函数在1秒后返回 
		//（即使你没调用SetEvent()设置信号量） 
		AfxMessageBox("fcuk");//不能用MessageBox()因为这不是在类中了... 
		ResetEvent((HANDLE)lParam); 
		/*重置信号量，以使WaitForSingleObject函数可以继续等待，否则（如果你已经调用过了SetEvent()设置了信号量） 
		WaitForSingleObject函数将会立刻返回 
		*/ 
	} 
 
} 
//2)用AfxBeginThread创建一个WorkThreadProc的线程 
AfxBeginThread(WorkThreadProc,event); 
//4,在主程序需要的地方调用SetEvent()设置信号量启动线程 
SetEvent(event); 
//-----------或者用WaitForMultipleObjects函数 

static UINT __stdcall WorkThreadProc(void* pThis);/*如果lParam参数为一个对话框的指针，想调用 
这个对话框的变量或函数那么就得这样定义线程函数， 
还要将WorkThreadProc改成CWait_forDlg::WorkThreadProc， 
这样WorkThreadProc就成为CWait_forDlg类的函数，在这个线程里就可以调用该类的变量了， 
注意得用_beginthreadex函数创建线程*/ 
UINT CWait_forDlg::WorkThreadProc(void * lParam) 
{ 
	CWait_forDlg *pThis=(CWait_forDlg *)lParam; 
	HANDLE hObjects[2]; 
	hObjects[0] = pThis->event1; 
	hObjects[1]= pThis->event2; 
	 
	while(1) 
	{ 
		DWORD dwWait = WaitForMultipleObjects(2,hObjects,TRUE,INFINITE); 
		/*第一个参数为信号量个数2，第二个为指针，第三个如果为TRUE函数要等待两个信号量都被SetEvent才返回，返回值为最后一个 
		SetEvent的WAIT_OBJECT_0+i；而为FALSE则只要有一个被SetEvent就返回，返回值为 WAIT_OBJECT_0+i 即信号量在数组中的位置 
		+WAIT_OBJECT_0 */ 
		if (dwWait == WAIT_OBJECT_0) 
		AfxMessageBox("fcuk 1 "); 
		 
		//开始ping 
		if (dwWait == WAIT_OBJECT_0 + 1) 
			AfxMessageBox("fcuk 2"); 
		ResetEvent(hObjects[1]); 
		ResetEvent(hObjects[0]); 
	} 
} 
 
#include <process.h> /* 调用_beginthread, _endthread 得包涵这个头文件*/ 
_beginthreadex(NULL, 
0, 
WorkThreadProc, 
(void*) this, 
0, 
0); 
//第三种创建线程的方法： 
HANDLE thread; 
DWORD threadrid; //线程ID 
DWORD WINAPI sniff(LPVOID no){} //线程函数这样定义 
thread=CreateThread(NULL, //安全属性 
0, //栈大小 
sniff, //要创建的线程名 
NULL, //参数（一般为调用线程的指针） 
0, //创建标志 
&threadrid); //线程ID 
CloseHandle(thread); 
```

## 9. 操作数据库

```C++
//在stdafx.h中加入
#include <afxdb.h> 
//1，用类向导，建立基于CRecordset或CDaoRecordset的新子类,并选择数据源m_setComplete 
//2，添加 
if ( ! m_setComplete . IsOpen () ) // if the recordset isn't already open.. 
m_setComplete . Open (); // open it 
m_setComplete . AddNew (); // begin the add 
 
m_setComplete . m_strCallsign = strCallsign; // change the recordset members 
m_setComplete . m_strFrequency = strFrequency; 
m_setComplete . m_strCity = strCity; 
m_setComplete . m_strState = strState; 
m_setComplete . m_strInput = strInput; 
 
m_setComplete . Update (); // complete the add by doing an update 
m_setComplete . Close (); // close the recordset 
//3,修改 
if ( ! m_setComplete . IsOpen () ) // if the recordset isn't already open.. 
m_setComplete . Open (); // open it 
m_setComplete . Edit (); // begin the edit 
 
m_setComplete . m_strCallsign = strCallsign; // change the recordset members 
m_setComplete . m_strFrequency = strFrequency; 
m_setComplete . m_strCity = strCity; 
m_setComplete . m_strState = strState; 
m_setComplete . m_strInput = strInput; 
 
m_setComplete . Update (); // complete the add by doing an update 
m_setComplete . Close (); // close the recordset 
//4，删除 
/* 1, DAO 数据库，不是ODBC 
if ( ! m_setComplete . IsOpen () ) 
m_setComplete . Open (); 
// cycle through the selected listbox elements. 
strRecordIdQuery =CString ( "[ID]=" ) +CString ( m_lcRepeaterList . GetItemText ( nItemIndex, 0 ) ); // put the ID into the query string 
MessageBox(strRecordIdQuery); 
if ( m_setComplete . FindFirst ( strRecordIdQuery ) ) { // looking for this ID in the database, ID is a unique 'autonumber' 
m_setComplete . Delete (); // delete the record 
　　　 m_setComplete . MoveFirst (); // move back to the first record 
m_bRecordsWereDeleted = TRUE; // make a note that we changed the database 
SetDlgItemText ( IDC_DELETE_STATUS, "Repeater Deleted From Database" ); // set the status field 
} 
else { 
// if we EVER end up here, either the database is in the crapper, or I will have screwed up horribly--been known to happen from time to time :) ... 
// so let's cover our ass-ets just in case. 
AfxMessageBox ( "Internal failure/n/nCannot find selected repeater in database/nor database is corrupted", MB_ICONSTOP ); 
} 
m_setComplete . Close (); // close the database*/ 
 
// ODBC here 
m_setComplete.m_strFilter="number=200301";//条件查询（where语句） 
if ( ! m_setComplete . IsOpen () ) 
m_setComplete . Open (); 
m_setComplete.Delete(); 
m_setComplete . MoveFirst (); 
m_setComplete . Close (); // close the database 
m_setComplete.m_strFilter="";//清空条件（where语句） 
//查询（where语句） 
m_setComplete.m_strFilter="number=200301"; 
if ( ! m_setComplete . IsOpen () ) 
m_setComplete . Open (); 
m_setComplete . Close (); // close the database 
 
//不用类向导写连接数据库的程序段，但是我只看明白了连接和查询，不会修改和添加，删除 
 
//1, 
CDatabase m_dbCust; //定义数据库类对象 
m_dbCust.OpenEx(_T("DSN=MQIS;UID=sa;PWD=1980623")//打开数据库　//数据源名，用户名，密码 
,CDatabase::forceOdbcDialog );//此参数只定是否打开连接确认对话框 
//MessageBox(m_dbCust.GetDatabaseName());　取得数据源名 
//m_dbCust.ExecuteSQL("select number from works");测试是否支持SQL语句 
CRecordset cs(&m_dbCust);//定义目录查询对象 
cs.Open( CRecordset::dynaset, 
_T( "select * from works" ) );//打开时执行的SQL语句 
short nFields = cs.GetODBCFieldCount( );//取得字段数，（列数） 
CDBVariant varValue;//定义通用数据类型 
CODBCFieldInfo co;//定义字段信息 
CString cc; 
 
while( !cs.IsEOF( ) ) 
{ 
for( short index = 0; index < nFields; index++ ) 
{ 
// do something with varValue 
cs.GetFieldValue(index,varValue);//取得某列的数据， 
//cs.GetFieldValue(index,cc); 也可直接取得由某列的数据直接转化成的文本 
if(varValue.m_dwType==DBVT_LONG)//m_dwType成员用来判断数据类型 
/* m_dwType Union data member 
DBVT_NULL No union member is valid for access. 
DBVT_BOOL m_boolVal 
DBVT_UCHAR m_chVal 
DBVT_SHORT m_iVal 
DBVT_LONG m_lVal 
DBVT_SINGLE m_fltVal 
DBVT_DOUBLE m_dblVal 
DBVT_DATE m_pdate 
DBVT_STRING m_pstring 
DBVT_BINARY m_pbinary 
*/ 
{ 
cc.Format("%d",varValue.m_lVal);//m_dwType==DBVT_LONG时为LONG型数据，成员为m_lVal，数值在其中 
MessageBox(cc); 
} 
cs.GetODBCFieldInfo(index,co);//取得字段信息 
/* struct CODBCFieldInfo 
{ 
CString m_strName;　　　　字段名 
SWORD m_nSQLType;　　　　SQL数据类型 
UDWORD m_nPrecision; 
SWORD m_nScale; 
SWORD m_nNullability; 
}; 
*/ 
} 
cs.MoveNext( );//下一个 
} 
cs.Close(); 
```

## A. 显示程序段

```C++
CString cstr; 
cstr.Format("%d", ); 
MessageBox(cstr); 
if(MessageBox("关闭Windows秘书您可会失去重要的提醒信息,确定要关闭吗?", 
"Windows秘书", 
MB_OKCANCEL|MB_DEFBUTTON2|MB_ICONWARNING)==IDOK) 
MessageBeep(MB_ICONQUESTION); 
```

## B. 注册与卸载OCX控件

```C++
//1，函数入口宏定义 
typedef HRESULT (STDAPICALLTYPE *CTLREGPROC)(); 
//2，注册函数 
BOOL RegisterOcx(CString ocxfilename) 
{ 
	BOOL bResult = FALSE ; 
	HMODULE hModule = ::LoadLibrary(ocxfilename) ; 
	//获得注册函数地址 
	CTLREGPROC DLLRegisterServer = 
	(CTLREGPROC)::GetProcAddress(hModule,"DllRegisterServer" ) ; 
	if (DLLRegisterServer != NULL) 
	{ 
	HRESULT regResult = DLLRegisterServer() ; 
	bResult = (regResult == NOERROR) ; 
	} 
	::FreeLibrary(hModule) ; 
	return bResult ; 
} 
//2，卸载函数 
BOOL UnRegisterOcx(CString ocxfilename) 
{ 
	BOOL bResult = FALSE ; 
	HMODULE hModule = ::LoadLibrary(ocxfilename) ; 
	//获得卸载函数地址 
	CTLREGPROC DLLUnregisterServer = 
	(CTLREGPROC)::GetProcAddress( hModule, 
	"DllUnregisterServer" ) ; 
	if (DLLUnregisterServer != NULL) 
	{ 
	HRESULT regResult = DLLUnregisterServer() ; 
	bResult = (regResult == NOERROR) ; 
	} 
	::FreeLibrary(hModule) ; 
	return bResult ; 
} 
```

## C. 处理文件拖动

```C++
//1 添加 WM_DROPFILES 静态消息 
//2 在OnInitDialog()函数中加入 
DragAcceptFiles(TRUE);//允许拖放 
//3 WM_DROPFILES 的处理函数为 
void CCXatDemoDlg::OnDropFiles(HDROP hDropInfo) 
{ 
	unsigned int nFiles=DragQueryFile(hDropInfo,0xFFFFFFFF,NULL,0);//取得拖放的文件总数 
	for (unsigned int i=0;i<nFiles;i++)//循环取得文件名 
	{ 
		unsigned int nLen=DragQueryFile(hDropInfo,i,NULL,0)+1;//取得文件名长度 
		char *psBuffer=new char[nLen]; 
		unsigned int sLen=DragQueryFile(hDropInfo,i,psBuffer,nLen);//取得文件名到psBuffer中,sLen为实际拷贝的字符数 
		//To add code here... 
		 
		delete [] psBuffer; 
	} 
	DragFinish(hDropInfo);//结束 
} 
```

## 设置窗口大小和位置

```C++
SetWindowPos(&wndTopMost,0,0,0,0,SWP_NOSIZE|SWP_NOMOVE);//设为最高层 
/* 
wndBottom 
wndTop 
wndTopMost 
wndNoTopMost 
*/ 
//在窗体中这样做 HWND_NOTOPMOST 为不是最高层 
::SetWindowPos(AfxGetMainWnd()->m_hWnd,HWND_TOPMOST,-1,-1,-1,-1,SWP_NOMOVE|SWP_NOSIZE); 
/* 
HWND_BOTTOM 
HWND_NOTOPMOST 
HWND_TOP 
HWND_TOPMOST 
SWP_DRAWFRAME Draws a frame, defined in the class description of the window, around the window. 
SWP_FRAMECHANGED Sends a WM_NCCALCSIZE message to the window, even if the size of the window is not being changed. If this flag is not specified, WM_NCCALCSIZE is sent only when the size of the window is being changed. 
SWP_HIDEWINDOW Hides the window. 
SWP_NOACTIVATE Does not activate the window. If this flag is not set, the window is activated and moved to the top of either the topmost or nontopmost group, depending on the setting of the hWndInsertAfter parameter. 
SWP_NOMOVE Retains the current position and ignores the X and Y parameters. 
SWP_NOOWNERZORDER Does not change the position in the Z order of the owner window. 
SWP_NOREPOSITION Same as the SWP_NOOWNERZORDER flag. 
SWP_NOSIZE Retains the current size and ignores the cx and cy parameters. 
SWP_NOZORDER Retains the current Z order and ignores the hWndInsertAfter parameter. 
SWP_SHOWWINDOW Displays the window. 
 
*/ 
```

## 在系统栏显示图标

```C++
//1定义结构
NOTIFYICONDATA systemicon; 
//2定义新的消息 
#define WM_SYSTEMMESSAGE WM_USER+998 
//3添加消息函数映射 
ON_MESSAGE(WM_SYSTEMMESSAGE,OnSystemMessage) 
//4初始化结构 
systemicon.cbSize=sizeof(NOTIFYICONDATA); 
systemicon.hWnd=this->m_hWnd; 
systemicon.uID=IDI_SYSICON; //图标 
systemicon.uFlags=NIF_MESSAGE|NIF_ICON|NIF_TIP; 
systemicon.uCallbackMessage=WM_SYSTEMMESSAGE; 
systemicon.hIcon=LoadIcon(AfxGetInstanceHandle(),MAKEINTRESOURCE(IDI_SYSICON)); 
strcpy(systemicon.szTip,"Windows秘书"); 
Shell_NotifyIcon(NIM_ADD,&systemicon);//显示 
strcpy(systemicon.szTip,cs); 
Shell_NotifyIcon(NIM_MODIFY,&systemicon);//修改 
Shell_NotifyIcon(NIM_DELETE,&systemicon);//删除 
//处理函数 
void OnSystemMessage(WPARAM wParam, LPARAM lParam) 
{ 
	if(wParam!=IDI_SYSICON) 
		return; 
	switch(lParam) 
	{ 
		case WM_RBUTTONUP://右键起来时弹出快捷菜单 
		{ 
			CMenu *menu=NULL; 
			LPPOINT mousepoint=new tagPOINT; 
			::GetCursorPos(mousepoint);//得到鼠标位置 
			BCMenu bmpmenu; 
			bmpmenu.LoadMenu(IDR_SYSPOPUPMENU); 
			menu = bmpmenu.GetSubMenu (0);//确定弹出式菜单的位置 
			if(IsWindowVisible()) 
				bmpmenu.ModifyODMenuA("隐藏主窗口",ID_SHOWHIDE,IDB_HIDE); 
			else 
				bmpmenu.ModifyODMenuA("显示主窗口",ID_SHOWHIDE,IDB_SHOW); 
			if(findtxdata->stoplowstep==0) 
				bmpmenu.ModifyODMenuA("停止普通级提醒",ID_STOPALLPT,IDB_STOP); 
			else 
				bmpmenu.ModifyODMenuA("启用普通级提醒",ID_STOPALLPT,IDB_NOTSTOP); 
			if(topornot) 
				bmpmenu.ModifyODMenuA("窗体不在最上层",ID_NOTONTOP,IDB_NOTTOP); 
			else 
				bmpmenu.ModifyODMenuA("窗体在最上层",ID_NOTONTOP,IDB_TOP); 
			if(!muteornot) 
				bmpmenu.ModifyODMenuA("静音",IDMUTE,IDB_NOSOUND); 
			else 
				bmpmenu.ModifyODMenuA("取消静音",IDMUTE,IDB_SOUND); 
			SetForegroundWindow();//必须先调用这个函数才能如果我不选择任何菜单而取消它 
			menu->TrackPopupMenu(TPM_LEFTALIGN,mousepoint->x,mousepoint->y,this); 
			//资源回收 
			//menu->Detach(); 
			//menu->DestroyMenu(); 
			delete mousepoint; 
			break; 
		} 
	} 
	return; 
} 
```

## 浮动鼠标提示

```C++
//1定义 
CToolTipCtrl tooltip; 
//2在OnCreate函数中创建 
tooltip.Create(this); 
//3在OnInitDialog函数中为控件添加提示字符 
tooltip.AddTool(GetDlgItem(IDC_DATEANDTIME),"显示日期/时间"); 
tooltip.Activate(TRUE);//启动 
//4在PreTranslateMessage(MSG* pMsg) 函数中加上 
tooltip.RelayEvent(pMsg); 
 
//tab控件的使用************************************************ 
 
m_tab.InsertItem(0,"提醒"); 
//定义结构 
WINDOWPLACEMENT wininfo; 
//取得窗口信息 
m_tab.GetWindowPlacement(&wininfo); 
//注意第一个参数要为&wndTop,如果是&wndTopMost就可能会函数功能无效 
qita.SetWindowPos(&wndTop,wininfo.rcNormalPosition.left-10,wininfo.rcNormalPosition.top+20, 
0,0,SWP_NOSIZE|SWP_HIDEWINDOW); 
//选择不同的标签 
void OnSelchangeTab(NMHDR* pNMHDR, LRESULT* pResult) 
{ 
	// TODO: Add your control notification handler code here 
	int i=m_tab.GetCurSel(); 
	switch(i) 
	{ 
	case 0: 
	break; 
	case 1: 
	break; 
	} 
	*pResult = 0; 
} 
```

## 注册热键

```C++
//m_filehotkey为热键控件变量
m_filehotkey.GetHotKey(WORD virtualkey,WORD flags); 
//注意要在主窗口中调用这个函数 
void RegHotKey(WORD virtualkey,WORD flags,int id) 
{ 
	if(flags==1||flags==9)//从热键控件中得到的系统键与下函数的标志不同,但有一一对应的关系: 
	{ 
		RegisterHotKey(GetSafeHwnd(),id,MOD_SHIFT,virtualkey); 
	}//每个ID只能注册一个热键,up,down,home,end等中部键盘的键的flags为9,10,11,12,13,14,15 
	if(flags==2||flags==10)//其余为1,2,3,4,5,6,7 
	{ 
		RegisterHotKey(GetSafeHwnd(),id,MOD_CONTROL,virtualkey); 
	} 
	if(flags==4||flags==12) 
	{ 
		RegisterHotKey(GetSafeHwnd(),id,MOD_ALT,virtualkey); 
	} 
	if(flags==3||flags==11) 
	{ 
		RegisterHotKey(GetSafeHwnd(),id,MOD_CONTROL|MOD_SHIFT,virtualkey); 
	} 
	if(flags==5||flags==13) 
	{ 
		RegisterHotKey(GetSafeHwnd(),id,MOD_ALT|MOD_SHIFT,virtualkey); 
	} 
	if(flags==6||flags==14) 
	{ 
		RegisterHotKey(GetSafeHwnd(),id,MOD_ALT|MOD_CONTROL,virtualkey); 
	} 
	if(flags==7||flags==15) 
	{ 
		RegisterHotKey(GetSafeHwnd(),id,MOD_ALT|MOD_CONTROL|MOD_SHIFT,virtualkey); 
	} 
} 
//处理消息 
LRESULT CWindowsDlg::WindowProc(UINT message, WPARAM wParam, LPARAM lParam) 
{ 
	// TODO: Add your specialized code here and/or call the base class 
	switch(message) 
	{ 
		case WM_HOTKEY : 
		switch(wParam) 
		{ 
		case IDC_HOTKEY: 
			if(IsWindowVisible()) 
				ShowWindow(SW_HIDE); 
			else 
				ShowWindow(SW_SHOWNORMAL); 
			SetForegroundWindow(); 
			break; 
		case IDC_HOTKEY2: 
			break; 
		} 
		return 0 ; 
	} 
	return CDialog::WindowProc(message, wParam, lParam); 
} 
```

## 控制系统音量

```C++
//在StdAfx.h中加入宏 
#include <mmsystem.h> 
//在setting->Link中加入下静态链接库! 
winmm.lib 
//播放声音 文件名 风格：第一个参数为文件名/找不到文件时不播放默认声音 
PlaySound( "", NULL, SND_FILENAME |SND_NODEFAULT); 
//定义结构 
UINT m_nNumMixers; //混合器的数量 
HMIXER m_hMixer; //当前混合器的句柄 
MIXERCAPS m_mxcaps; //当前混合器的性能参数 
CString m_strDstLineName, m_strVolumeControlName; //混合器控件的名称 
DWORD m_dwMinimum, m_dwMaximum; //最大，最小的音量值 
DWORD m_dwVolumeControlID; //混合器控件的音量控制ID 
DWORD m_dwMuteControlID; //混合器控件的静音控制ID 
BOOL muteornot; //是否静音 
DWORD nowvolume; //当前值 
double multiple; 
double difference; //最大最小差值 
int volumestep;//步长 
//加入消息映射函数 
ON_MESSAGE(MM_MIXM_CONTROL_CHANGE, OnMixerCtrlChange) 
//当系统音量改变时调用此函数 
LONG OnMixerCtrlChange(UINT wParam, LONG lParam) 
{ 
//响应控件音量改变的消息，然后获得当前音量并设置滚动条 
if ((HMIXER)wParam == m_hMixer && (DWORD)lParam == m_dwVolumeControlID) 
{ 
amdGetMasterVolumeValue(nowvolume); 
multiple=nowvolume; 
double percent; 
} 
if ((HMIXER)wParam == m_hMixer && (DWORD)lParam == m_dwMuteControlID) 
{ 
amdGetMasterMuteValue(muteornot); 
} 
return 0L; 
} 
//在Init函数中加入 
if (amdInitialize()) 
{ 
//获得音量控制控件的ID和名称 
amdGetMasterVolumeControl(); 
// 获得当前音量值，并设置滚动条的初始位置 
amdGetMasterVolumeValue(nowvolume); 
amdGetMasterMuteValue(muteornot); 
multiple=nowvolume; 
difference=(long)(m_dwMaximum-m_dwMinimum); 
} 
//函数 
BOOL amdInitialize()//初始化 
{ 
//获取当前混合设备数量 
m_nNumMixers = ::mixerGetNumDevs(); 
m_hMixer = NULL; 
::ZeroMemory(&m_mxcaps, sizeof(MIXERCAPS)); 
if (m_nNumMixers != 0) 
{ 
//打开混合设备 
if (::mixerOpen(&m_hMixer, 
0, 
(DWORD)this->GetSafeHwnd(), 
NULL, 
MIXER_OBJECTF_MIXER | CALLBACK_WINDOW) 
!= MMSYSERR_NOERROR) 
return FALSE; 
// 获取混合器性能 
if (::mixerGetDevCaps((UINT)m_hMixer, &m_mxcaps, sizeof(MIXERCAPS)) 
!= MMSYSERR_NOERROR) 
return FALSE; 
} 
return TRUE; 
} 
BOOL amdGetMasterVolumeControl()//获得音量控制控件的ID和名称 
{ 
m_strDstLineName.Empty(); 
m_strVolumeControlName.Empty(); 
if (m_hMixer == NULL) 
return FALSE; 
//获得混合器性能 
MIXERLINE mxl; 
mxl.cbStruct = sizeof(MIXERLINE); 
mxl.dwComponentType = MIXERLINE_COMPONENTTYPE_DST_SPEAKERS; 
if (::mixerGetLineInfo((HMIXEROBJ)m_hMixer, 
&mxl, 
MIXER_OBJECTF_HMIXER | 
MIXER_GETLINEINFOF_COMPONENTTYPE) 
!= MMSYSERR_NOERROR) 
return FALSE; 
MIXERCONTROL mxc; 
MIXERLINECONTROLS mxlc; 
mxlc.cbStruct = sizeof(MIXERLINECONTROLS); 
mxlc.dwLineID = mxl.dwLineID; 
mxlc.dwControlType = MIXERCONTROL_CONTROLTYPE_VOLUME;//此值表示取得音量 
mxlc.cControls = 1; 
mxlc.cbmxctrl = sizeof(MIXERCONTROL); 
mxlc.pamxctrl = &mxc; 
//获得混合器线控件 
if (::mixerGetLineControls((HMIXEROBJ)m_hMixer, 
&mxlc, 
MIXER_OBJECTF_HMIXER | 
MIXER_GETLINECONTROLSF_ONEBYTYPE) 
!= MMSYSERR_NOERROR) 
return FALSE; 
//记录控件的信息 
m_strDstLineName = mxl.szName; 
m_strVolumeControlName = mxc.szName; 
m_dwMinimum = mxc.Bounds.dwMinimum; 
m_dwMaximum = mxc.Bounds.dwMaximum; 
m_dwVolumeControlID = mxc.dwControlID; 
mxlc.dwControlType = MIXERCONTROL_CONTROLTYPE_MUTE;//此值表示取得静音与否 
if (::mixerGetLineControls((HMIXEROBJ)m_hMixer, 
&mxlc, 
MIXER_OBJECTF_HMIXER | 
MIXER_GETLINECONTROLSF_ONEBYTYPE) 
!= MMSYSERR_NOERROR) 
return FALSE; 
m_dwMuteControlID = mxc.dwControlID; 
return TRUE; 
} 
BOOL amdGetMasterVolumeValue(DWORD &dwVal)// 获得当前音量值，并设置滚动条的初始位置 
{ 
if (m_hMixer == NULL || 
m_strDstLineName.IsEmpty() || m_strVolumeControlName.IsEmpty()) 
return FALSE; 
MIXERCONTROLDETAILS_UNSIGNED mxcdVolume; 
MIXERCONTROLDETAILS mxcd; 
mxcd.cbStruct = sizeof(MIXERCONTROLDETAILS); 
mxcd.dwControlID = m_dwVolumeControlID; 
mxcd.cChannels = 1; 
mxcd.cMultipleItems = 0; 
mxcd.cbDetails = sizeof(MIXERCONTROLDETAILS_UNSIGNED); 
mxcd.paDetails = &mxcdVolume; 
//获取指定混合器控件 
if (::mixerGetControlDetails((HMIXEROBJ)m_hMixer, 
&mxcd, 
MIXER_OBJECTF_HMIXER | 
MIXER_GETCONTROLDETAILSF_VALUE) 
!= MMSYSERR_NOERROR) 
return FALSE; 
dwVal = mxcdVolume.dwValue; 
return TRUE; 
} 
BOOL amdGetMasterMuteValue(BOOL &yorn)//是否静音 
{ 
if (m_hMixer == NULL || 
m_strDstLineName.IsEmpty() || m_strVolumeControlName.IsEmpty()) 
return FALSE; 
MIXERCONTROLDETAILS_UNSIGNED mxcdVolume; 
MIXERCONTROLDETAILS mxcd; 
mxcd.cbStruct = sizeof(MIXERCONTROLDETAILS); 
mxcd.dwControlID = m_dwMuteControlID; 
mxcd.cChannels = 1; 
mxcd.cMultipleItems = 0; 
mxcd.cbDetails = sizeof(MIXERCONTROLDETAILS_UNSIGNED); 
mxcd.paDetails = &mxcdVolume; 
//获取指定混合器控件 
if (::mixerGetControlDetails((HMIXEROBJ)m_hMixer, 
&mxcd, 
MIXER_OBJECTF_HMIXER | 
MIXER_GETCONTROLDETAILSF_VALUE) 
!= MMSYSERR_NOERROR) 
return FALSE; 
yorn = mxcdVolume.dwValue; 
return TRUE; 
} 
BOOL amdSetMasterMuteValue(BOOL &yorn)//设置静音与否 
{ 
if (m_hMixer == NULL || 
m_strDstLineName.IsEmpty() || m_strVolumeControlName.IsEmpty()) 
return FALSE; 
MIXERCONTROLDETAILS_UNSIGNED mxcdVolume = { yorn }; 
MIXERCONTROLDETAILS mxcd; 
mxcd.cbStruct = sizeof(MIXERCONTROLDETAILS); 
mxcd.dwControlID = m_dwMuteControlID; 
mxcd.cChannels = 1; 
mxcd.cMultipleItems = 0; 
mxcd.cbDetails = sizeof(MIXERCONTROLDETAILS_UNSIGNED); 
mxcd.paDetails = &mxcdVolume; 
//放置混合器控件 
if (::mixerSetControlDetails((HMIXEROBJ)m_hMixer, 
&mxcd, 
MIXER_OBJECTF_HMIXER | 
MIXER_SETCONTROLDETAILSF_VALUE) 
!= MMSYSERR_NOERROR) 
return FALSE; 
return TRUE; 
} 
BOOL amdSetMasterVolumeValue(DWORD dwVal)//设置音量 
{ 
if (m_hMixer == NULL || 
m_strDstLineName.IsEmpty() || m_strVolumeControlName.IsEmpty()) 
return FALSE; 
MIXERCONTROLDETAILS_UNSIGNED mxcdVolume = { dwVal }; 
MIXERCONTROLDETAILS mxcd; 
mxcd.cbStruct = sizeof(MIXERCONTROLDETAILS); 
mxcd.dwControlID = m_dwVolumeControlID; 
mxcd.cChannels = 1; 
mxcd.cMultipleItems = 0; 
mxcd.cbDetails = sizeof(MIXERCONTROLDETAILS_UNSIGNED); 
mxcd.paDetails = &mxcdVolume; 
//放置混合器控件 
if (::mixerSetControlDetails((HMIXEROBJ)m_hMixer, 
&mxcd, 
MIXER_OBJECTF_HMIXER | 
MIXER_SETCONTROLDETAILSF_VALUE) 
!= MMSYSERR_NOERROR) 
return FALSE; 
return TRUE; 
} 
BOOL amdUninitialize()//关闭 
{ 
BOOL bSucc = TRUE; 
if (m_hMixer != NULL) 
{ 
//关闭混合器 
bSucc = ::mixerClose(m_hMixer) == MMSYSERR_NOERROR; 
m_hMixer = NULL; 
} 
return bSucc; 
} 
//在OnDestroy() 中加入 
amdUninitialize(); 
```

## 编辑注册表

```C++
DWORD disp,valuelength=260; 
HKEY parentkey; 
HKEY childkey; 
char filepath[MAX_PATH]; 
//打开主键 
RegOpenKeyEx(HKEY_LOCAL_MACHINE,"software//microsoft//windows//currentversion//run",//注册启动信息 
0L,KEY_WRITE,&parentkey); 
//查询键值 
RegQueryValueEx(parentkey,"Windows秘书",0,NULL,(unsigned char *)filepath,&valuelength); 
//写入键值 
RegSetValueEx(parentkey,"Windows秘书",0,REG_SZ,(const unsigned char *)filepath,strlen(filepath)); 
//删除键值 
RegDeleteValue(parentkey,"Windows秘书"); 
//关闭 
RegCloseKey(parentkey); 
//建立子键 
RegOpenKeyEx(HKEY_LOCAL_MACHINE,"software", 
0L,KEY_WRITE,&parentkey);//打开主键 
RegCreateKeyEx(parentkey,"宇光软件",NULL,"CWindowsDlg",REG_OPTION_NON_VOLATILE,KEY_ALL_ACCESS, 
NULL,&childkey,&disp);//把子键保存在childkey中 
RegCloseKey(parentkey); 
//在子键中写入子键值 
RegSetValueEx(childkey,"Windows秘书",0,REG_SZ,(const unsigned char *)filepath,strlen(filepath)); 
RegCloseKey(childkey); 

```

## 创建桌面快捷图标和创建开始菜单组

```C++
//初始化 
CoInitialize (NULL); 
//建立桌面快捷图标函数 
void CreateDesttop(char *filepath) 
{ 
strcat(filepath,"//Windows秘书.exe"); 
LPITEMIDLIST pidlBeginAt; 
char szLink[MAX_PATH]="";//快捷方式的数据文件名 
// 取得桌面的PIDL 
SHGetSpecialFolderLocation( HWND_DESKTOP, 
CSIDL_DESKTOPDIRECTORY, &pidlBeginAt) ; 
// 把PIDL转换为路径名 
SHGetPathFromIDList( pidlBeginAt, szLink) ; 
strcat(szLink,"//Windows秘书.lnk"); 
IShellLink * psl ; 
IPersistFile* ppf ; 
WORD wsz[ MAX_PATH] ; 
//创建一个IShellLink实例 
CoCreateInstance( CLSID_ShellLink, NULL, 
CLSCTX_INPROC_SERVER, IID_IShellLink, 
(void **)&psl) ; 
//设置目标应用程序 
psl -> SetPath( filepath) ; 
//设置快捷键(此处设为Shift+Ctrl+'R') 
psl -> SetHotkey( MAKEWORD( 'Q', 
HOTKEYF_SHIFT |HOTKEYF_CONTROL|HOTKEYF_ALT)) ; 
//从IShellLink获取其IPersistFile接口 
//用于保存快捷方式的数据文件 (*.lnk) 
psl -> QueryInterface( IID_IPersistFile, 
(void**)&ppf) ; 
// 确保数据文件名为ANSI格式 
MultiByteToWideChar( CP_ACP, 0, szLink, -1, 
wsz, MAX_PATH) ; 
//调用IPersistFile::Save 
//保存快捷方式的数据文件 (*.lnk) 
ppf -> Save( wsz, STGM_READWRITE) ; 
//释放IPersistFile和IShellLink接口 
ppf -> Release( ) ; 
psl -> Release( ) ; 
SHChangeNotify( SHCNE_CREATE|SHCNE_INTERRUPT, 
SHCNF_FLUSH | SHCNF_PATH, 
szLink,0); 
//取得szPath的父目录 
char* p; 
for( p=szLink+lstrlen(szLink)-1; 
*p != '//'; 
p--); 
*p='/0'; 
SHChangeNotify(SHCNE_UPDATEDIR 
|SHCNE_INTERRUPT, 
SHCNF_FLUSH | SHCNF_PATH,szLink,0); 
} 
//创建开始菜单组的函数 
void CWindowsDlg::CreateStarMenu(char *filepath) 
{ 
LPITEMIDLIST pidlBeginAt; 
char szPath[ MAX_PATH] ; 
// 取得开始菜单的PIDL 
SHGetSpecialFolderLocation( HWND_DESKTOP, // 
CSIDL_STARTMENU, &pidlBeginAt) ; 
SHGetPathFromIDList( pidlBeginAt, szPath) ;// 把PIDL转换为路径名 
strcat(szPath,"//宇光软件"); 
if( !CreateDirectory( szPath, NULL))return ; 
SHChangeNotify( SHCNE_MKDIR|SHCNE_INTERRUPT, 
SHCNF_FLUSH | SHCNF_PATH, 
szPath,0); 
//取得szPath的父目录 
char* p; 
for( p=szPath+lstrlen(szPath)-1; 
*p != '//'; 
p--); 
*p='/0'; 
SHChangeNotify(SHCNE_UPDATEDIR 
|SHCNE_INTERRUPT, 
SHCNF_FLUSH | SHCNF_PATH,szPath,0); 
strcat(filepath,"//Windows秘书.exe"); 
strcat(szPath,"//宇光软件//Windows秘书.lnk"); 
IShellLink * psl ; 
IPersistFile* ppf ; 
WORD wsz[ MAX_PATH] ; 
//创建一个IShellLink实例 
CoCreateInstance( CLSID_ShellLink, NULL, 
CLSCTX_INPROC_SERVER, IID_IShellLink, 
(void **)&psl) ; 
//设置目标应用程序 
psl -> SetPath( filepath) ; 
//设置快捷键(此处设为Shift+Ctrl+'Y') 
psl -> SetHotkey( MAKEWORD( 'Y', 
HOTKEYF_SHIFT |HOTKEYF_CONTROL|HOTKEYF_ALT)) ; 
//从IShellLink获取其IPersistFile接口 
//用于保存快捷方式的数据文件 (*.lnk) 
psl -> QueryInterface( IID_IPersistFile, 
(void**)&ppf) ; 
// 确保数据文件名为ANSI格式 
MultiByteToWideChar( CP_ACP, 0, szPath, -1, 
wsz, MAX_PATH) ; 
//调用IPersistFile::Save 
//保存快捷方式的数据文件 (*.lnk) 
ppf -> Save( wsz, STGM_READWRITE) ; 
//释放IPersistFile和IShellLink接口 
ppf -> Release( ) ; 
psl -> Release( ) ; 
SHChangeNotify( SHCNE_CREATE|SHCNE_INTERRUPT, 
SHCNF_FLUSH | SHCNF_PATH, 
szPath,0); 
//取得szPath的父目录 
for( p=szPath+lstrlen(szPath)-1; 
*p != '//'; 
p--); 
*p='/0'; 
SHChangeNotify(SHCNE_UPDATEDIR 
|SHCNE_INTERRUPT, 
SHCNF_FLUSH | SHCNF_PATH,szPath,0); 
} 
void OnDestroy() 
{ 
CoUninitialize(); 
} 
```

## 程序只允许一个实例运行

```C++
//在这个位置调用FirstInstance函数 
BOOL CWindowsApp::InitInstance() 
{ 
if (!FirstInstance()) 
return FALSE; //已经有实例存在了，退出 
AfxEnableControlContainer(); 
} 
//FirstInstance函数 
BOOL FirstInstance() 
{ 
CWnd *pWndPrev; 
//根据主窗口类名和主窗口名判断是否已经有实例存在了 
if (pWndPrev = CWnd::FindWindow("#32770","Windows秘书"))//第二个参数为程序名 
{//如果存在就将其激活，并显示出来 
pWndPrev->ShowWindow(SW_RESTORE); 
pWndPrev->SetForegroundWindow(); 
return FALSE; 
} 
else 
return TRUE; 
} 

```

## 链表操作

```C++
//建立一个有空链环的链表,这样就在删除时就不用判断是不是第一个或最后一个了 
head=new zbdata;//此为结构体 
head->next=NULL; 
//添加 
zbdata *newdata=new zbdata; 
*newdata=inputdata; 
newdata->next=head->next;//生成倒链表,即最先的放在最后 
head->next=newdata; 
//修改 
zbdata *rewrite=head->next; 
for(int j=0;j<i;j++) 
{ 
rewrite=rewrite->next; 
} 
*rewrite=inputdata; 
//删除 
zbdata *delbelow,*del=head; 
for(int j=0;j<i;j++) 
{ 
del=del->next; 
} 
delbelow=del->next; 
del->next=delbelow->next; 
//查找 
zbdata *find=head->next; 
for(int j=0;j<i;j++) 
{ 
find=find->next; 
} 
inputdata=*find; 

```

## 取得系统时间及计算时间差

```C++
DWORD system_runtime; 
system_runtime=GetTickCount()/60000;//取得系统运行时间 
SYSTEMTIME sysTime; // Win32 time information 
GetLocalTime(&sysTime); 
COleDateTime now(sysTime); 
//COleDateTime now; 
//now=now.GetCurrentTime(); 
//GetCurrentTime()取得时间与time( time_t *timer );和struct tm *localtime( const time_t *timer );所得时间相同只能到2037年 
//而GetLocalTime(&sysTime);可到9999年 
m_zbyear=now.GetYear();//年 
m_zbmonth=now.GetMonth();//月 
m_zbday=now.GetDay();//日 
now.GetDayOfWeek();//weekday 
COleDateTimeSpan sub; 
sub.SetDateTimeSpan(m_howdaysafteris,0,0,0);//设定时间差为天数 
now+=sub; 
then.SetDateTime(,,,,,,,);//其它方式添加时保存的当时时间 
if(!now.SetDate(m_jsyear,m_jsmonth,m_jsday)//检查时间是否正确 
&&!then.SetDate(m_jsyear2,m_jsmonth2,m_jsday2)) 
{ 
sub=then-now; 
CString cs; 
cs.Format("%d年%d月%d日 相距 %d年%d月%d日/n/n %.0f天",m_jsyear,m_jsmonth,m_jsday, 
m_jsyear2,m_jsmonth2,m_jsday2,sub.GetTotalDays()); 
MessageBox(cs); 
} 
else 
MessageBox("您输入的时间有误,请重新输入!","Windows秘书",MB_OK|MB_ICONSTOP); 

```

## 文件操作

```C++
//打开文件对话框 
CFileDialog dlg(TRUE, "mp3", NULL, OFN_FILEMUSTEXIST|OFN_NOCHANGEDIR, 
音乐/电影文件(*.mp3,*.wav,*.avi,*.asf)|*.mp3;*.wav;*.avi;*.asf|/ 
mp3 文件(*.mp3)|*.mp3| / 
音频文件 (*.wav)|*.wav| / 
视频文件 (*.avi)|*.avi| / 
Window Media 文件(*.asf)|*.asf| / 
所有文件 (*.*)|*.*||); 
if(dlg.DoModal()==IDOK) 
{ 
m_filename=dlg.GetPathName(); 
} 
//读取文件 
CFile file; 
if(file.Open(m_filename,CFile::modeRead)) 
{ 
file.Read(zbpassword,sizeof(zbpassword)); 
} 
file.Close(); 
//写入文件 
if(file.Open(m_zbfilename,CFile::modeCreate|CFile::modeWrite)) 
{ 
zbfile.Write(zbpassword,sizeof(zbpassword)); 
} 
file.Close(); 
```

## 列表框ListCtrl的使用

```C++
CSize move; 
move.cx=0;move.cy=13;//cy=13为滚动一行 
m_list.Scroll(move);//移动滚动条 
m_list.SetItemState(3, LVIS_SELECTED, LVIS_SELECTED);//设置要选择的项(高亮) 
m_list.SetSelectionMark(3);//设置下次调用GetSelectionMark()函数的返回值 
//初始化，四列 
m_zblist.InsertColumn(0,"日期",LVCFMT_CENTER,70); 
m_zblist.InsertColumn(1,"备注",LVCFMT_LEFT,100); 
m_zblist.InsertColumn(2,"收入(元)",LVCFMT_LEFT,100); 
m_zblist.InsertColumn(3,"支出(元)",LVCFMT_LEFT,100); 
//设定宽度 
m_txlist.SetColumnWidth(1,longth1); 
//加项目 
m_zblist.InsertItem(0,""); 
m_zblist.SetItemText(0,1,""); 
m_zblist.SetItemText(0,2,""); 
m_zblist.SetItemText(0,3,""); 
//删除所有项 
m_zblist.DeleteAllItems(); 
//取得单选选中的项 
m_zblist.GetSelectionMark(); 
//多选时 
int listnumber;//定义列表序号和内存序号 
int deldata[2002],delcount=0;//定义数组存放要删除的列表项的序号 
POSITION pos = m_zblist.GetFirstSelectedItemPosition(); 
if(pos==NULL) 
{ 
MessageBox("请点击要删除的记录先!","Windows秘书",MB_OK|MB_ICONWARNING); 
} 
while (pos)//如果用户是选择了项目... 
{ 
listnumber= m_mplist.GetNextSelectedItem(pos);//取得用户点击的序号,从0开始查找列表控件 
deldata[delcount]=listnumber;//把列表中要删除的序号赋给数组保存以便最后删除时用 
delcount++;//数组指针加1 
} 
for(i=delcount-1;i>=0;i--) 
{ 
m_mplist.DeleteItem(deldata[i]);//从最后最大项的序号开始删除以便正确的删除列表中的项 
data.del(deldata[i]); 
} //从前面删除则会出现错删 

```

## 进制转换

```C++
CString BinaryToDecimal(CString strBinary)//转换二进制为十进制 
{ 
int nLenth = strBinary.GetLength(); 
char* Binary = new char[nLenth]; 
Binary = strBinary.GetBuffer(0); 
int nDecimal = 0; 
for(int i=0;i<nLenth;i++) 
{ 
char h = Binary[nLenth-1-i]; 
char str[1]; 
str[0] = h; 
int j = atoi(str); 
for(int k=0;k<i;k++) 
{ 
j=j*2; 
} 
nDecimal += j; 
} 
CString strDecimal; 
strDecimal.Format("%d",nDecimal); 
return strDecimal; 
} 
CString BinaryToHex(CString strBinary)//转换二进制为十六进制 
{ 
int nLength = strBinary.GetLength(); 
CString str = strBinary; 
//位数不是四的倍数时补齐 
switch(nLength%4) 
{ 
case 0: 
break; 
case 1: 
strBinary.Format("%d%d%d%s",0,0,0,str); 
break; 
case 2: 
strBinary.Format("%d%d%s",0,0,str); 
break; 
case 3: 
strBinary.Format("%d%s",0,str); 
break; 
default: 
return ""; 
break; 
} 
CString strHex,str1; 
str1 = ""; 
nLength = strBinary.GetLength(); 
for(int i=1;i<=(nLength/4);i++) 
{ 
//每四位二进制数转换为一十六进制数 
str = strBinary.Left(4); 
CString strDecimal = BinaryToDecimal(str); 
int nDecimal = atoi(strDecimal.GetBuffer(0)); 
if(nDecimal<10) 
str1.Format("%d",nDecimal); 
else 
{ 
char c = 'A' + (nDecimal-10); 
str1.Format("%c",c); 
} 
strHex += str1; 
strBinary = strBinary.Right(strBinary.GetLength()-str.GetLength()); 
} 
return strHex; 
} 
unsigned char BtoH(char ch)//将16进制的一个字符转换为十进制的数 
{ 
//0-9 
if (ch >= '0' && ch <= '9') 
return (ch - '0'); 
//9-15 
if (ch >= 'A' && ch <= 'F') 
return (ch - 'A' + 0xA); 
//9-15 
if (ch >= 'a' && ch <= 'f') 
return (ch - 'a' + 0xA); 
return(255); 
} 
CString DecimalToBinary(CString strDecimal)//转换十进制为二进制 
{ 
int nDecimal = atoi(strDecimal.GetBuffer(0)); 
int nYushu; //余数 
int nShang; //商 
CString strBinary = ""; 
char buff[2]; 
CString str = ""; 
BOOL bContinue = TRUE; 
while(bContinue) 
{ 
nYushu = nDecimal%2; 
nShang = nDecimal/2; 
sprintf(buff,"%d",nYushu); 
str = strBinary; 
strBinary.Format("%s%s",buff,str); 
nDecimal = nShang; 
if(nShang==0) 
bContinue = FALSE; 
} 
return strBinary; 
} 
CString BinaryToDecimal(CString strBinary)//转换二进制为十进制 
{ 
int nLenth = strBinary.GetLength(); 
char* Binary = new char[nLenth]; 
Binary = strBinary.GetBuffer(0); 
int nDecimal = 0; 
for(int i=0;i<nLenth;i++) 
{ 
char h = Binary[nLenth-1-i]; 
char str[1]; 
str[0] = h; 
int j = atoi(str); 
for(int k=0;k<i;k++) 
{ 
j=j*2; 
} 
nDecimal += j; 
} 
CString strDecimal; 
strDecimal.Format("%d",nDecimal); 
return strDecimal; 
} 

```

## 数学函数

```C++
sin(); 
cos(); 
tan(); 
sqrt(); 
pow(x,y); 
log(); 
log10(); 

```

## 递归法遍历磁盘目录

```C++
void CQt::BrowseDir(CString strDir) 
{ 
CFileFind ff; 
CString str,cs,inifile; 
inifile=regfilepath; 
inifile+="//Windows秘书.ini"; 
CString szDir = strDir; 
int index; 
char jjj,ppp,ggg; 
if(szDir.Right(1) != "//") 
szDir += "//"; 
szDir += "*.*"; 
BOOL res = ff.FindFile(szDir); 
while(res) 
{ 
res = ff.FindNextFile(); 
if(ff.IsDirectory() && !ff.IsDots()) 
{ 
//如果是一个子目录，用递归继续往深一层找 
BrowseDir(ff.GetFilePath()); 
} 
else if(!ff.IsDirectory() && !ff.IsDots()) 
{ 
//显示当前访问的文件 
str=ff.GetFilePath(); 
/* index=str.GetLength(); 
//MessageBox(str); 
jjj=str.GetAt(index-3); 
ppp=str.GetAt(index-2); 
ggg=str.GetAt(index-1); 
if(((jjj=='J'||jjj=='j')&&(ppp=='P'||ppp=='p')&&(ggg=='G'||ggg=='g'))||((jjj=='B'||jjj=='b')&&(ppp=='M'||ppp=='m')&&(ggg=='P'||ggg=='p'))) 
{ 
intNumber++; 
cs.Format("%d",intNumber); 
WritePrivateProfileString("图片目录文件",cs,str,inifile);*/ 
//Sleep(3); 
} 
} 
} 
ff.Close();//关闭 
//cs.Format("%d",intNumber); 
//WritePrivateProfileString("图片目录文件","total",cs,inifile); 
//WritePrivateProfileString("图片目录文件","turntowhich","1",inifile); 
} 

```

## 公用目录对话框

```C++
//添加如下程序段 
LPITEMIDLIST pidlBeginAt, pidlDestination ; 
char szPath[ MAX_PATH] ; 
// 取得开始菜单或桌面的PIDL 
SHGetSpecialFolderLocation(AfxGetApp()->m_pMainWnd->m_hWnd,CSIDL_DESKTOP,&pidlBeginAt) ; 
// 取得新建文件夹的父文件夹 
if( !BrowseForFolder(pidlBeginAt , 
&pidlDestination, 
请选择图片目录的位置：)) 
{ 
return ; 
} 
// 把PIDL转换为路径名 
SHGetPathFromIDList( pidlDestination, szPath); 
//添加如下函数 
BOOL BrowseForFolder( 
LPITEMIDLIST pidlRoot,//浏览开始处的PIDL 
LPITEMIDLIST *ppidlDestination, 
//浏览结束时所选择的PIDL 
LPCSTR lpszTitle)//浏览对话框中的提示文字 
{ 
BROWSEINFO BrInfo ; 
ZeroMemory( &BrInfo, sizeof(BrInfo)) ; 
BrInfo.hwndOwner = HWND_DESKTOP ; 
BrInfo.pidlRoot = pidlRoot ; 
BrInfo.lpszTitle = lpszTitle ; 
//浏览文件夹 
*ppidlDestination= SHBrowseForFolder(&BrInfo); 
//用户选择了取消按钮 
if(NULL == *ppidlDestination) 
return FALSE ;/**/ 
return TRUE ; 
} 

```

## 读写INI/ini文件

```C++
//写入值 字段名 变量名 值 带目录文件名 
WritePrivateProfileString("目录","path",cs, regpath); 
//写入结构体 字段名 变量名 值 大小 带目录文件名 
WritePrivateProfileStruct("字体","font",&LF,sizeof(LOGFONT),regpath);//结构体 
//读入字符 字段名 变量名 默认值 字符缓冲区 长度 带目录文件名 
GetPrivateProfileString("目录","path","", buffer.GetBuffer(260),260,regpath); 
//读入整数值 字段名 变量名 默认值 带目录文件名 
GetPrivateProfileInt("colors","red",255, regpath); 
//读入结构体 字段名 变量名 值 大小 带目录文件名 
GetPrivateProfileStruct("字体","font",&LF,sizeof(LOGFONT),regpath); 

```

## 位图操作、画图

```C++
CClientDC client(this); 
BITMAP bmpInfo; 
CDC memdc; 
CBitmap picture; 
memdc.CreateCompatibleDC(pdc); 
memdc.SelectObject(&picture); 
CRect re; 
GetClientRect(&re); 
client.BitBlt(0,0,bmpInfo.bmWidth,bmpInfo.bmHeight,&memdc,0,0,SRCCOPY); 
client.SetBkMode(TRANSPARENT); 
client.SetTextColor(RGB(red,green,blue)); 
CFont font; 
CFont *oldfont; 
font.CreateFontIndirect(&LF); 
oldfont=client.SelectObject(&font); 
client.DrawText("",//注意这个字符串里如果只有一连串的字母或数字，没有空格或中文或标点符号，且总长度超过距形宽度，则不能自动换行!! 
&re,DT_CENTER |DT_WORDBREAK); 
client.SelectObject(oldfont); 

```

## 打开EXE/exe文件

```C++
ShellExecute(GetSafeHwnd(), 
open, 
http://home.jlu.edu.cn/~ygsoft,//改这个文件名就可以了 
NULL, 
NULL,SW_SHOWNORMAL); 
//or 
LPITEMIDLIST pidlBeginAt; 
// 取得桌面的PIDL 
SHGetSpecialFolderLocation(AfxGetApp()->m_pMainWnd->m_hWnd, 
CSIDL_DRIVES//这个标志为我的电脑 
,&pidlBeginAt) ; 
SHELLEXECUTEINFO exe; 
ZeroMemory(&exe,sizeof(SHELLEXECUTEINFO)); 
exe.cbSize=sizeof(SHELLEXECUTEINFO); 
exe.fMask=SEE_MASK_IDLIST; 
exe.lpVerb="open"; 
exe.nShow=SW_SHOWNORMAL; 
exe.lpIDList=pidlBeginAt; 
ShellExecuteEx(&exe); 

```

## 取得函数不能正常返回的原因字符

```C++
LPVOID lpMsgBuf; 
FormatMessage( 
FORMAT_MESSAGE_ALLOCATE_BUFFER | 
FORMAT_MESSAGE_FROM_SYSTEM | 
FORMAT_MESSAGE_IGNORE_INSERTS, 
NULL, 
GetLastError(), 
MAKELANGID(LANG_NEUTRAL, SUBLANG_DEFAULT), // Default language 
(LPTSTR) &lpMsgBuf, 
0, 
NULL 
); 
// Process any inserts in lpMsgBuf. 
// ... 
// Display the string. 
MessageBox((LPCTSTR)lpMsgBuf); 
// Free the buffer. 
LocalFree( lpMsgBuf ); 

```

## 关机函数

```C++
void ShutDown(void)//2000 OR NT 
{ 
OSVERSIONINFO osv; 
osv.dwOSVersionInfoSize=sizeof OSVERSIONINFO; 
GetVersionEx(&osv); 
if(osv.dwPlatformId==VER_PLATFORM_WIN32_NT)//VER_PLATFORM_WIN32_WINDOWS 98 Me用这个宏 
{ 
HANDLE hProcess,hToken; 
TOKEN_PRIVILEGES Privileges; 
LUID luid; 
hProcess=GetCurrentProcess(); 
OpenProcessToken(hProcess,TOKEN_ADJUST_PRIVILEGES,&hToken); 
Privileges.PrivilegeCount=1; 
LookupPrivilegeValue(NULL,SE_SHUTDOWN_NAME,&luid); 
Privileges.Privileges[0].Luid=luid; 
Privileges.Privileges[0].Attributes=SE_PRIVILEGE_ENABLED; 
AdjustTokenPrivileges(hToken,FALSE,&Privileges,NULL,NULL,NULL); 
} 
ExitWindowsEx(EWX_POWEROFF,0); 
} 

```

## 创建一个非模态对话框

```C++
//1声明父对话框 
#include "Tx.h" 
//2在头文件中改为CTx* pParent = NULL，并声明CTx *tx; 
public: 
CInputmr(CTx* pParent = NULL); // standard constructor 
CTx *tx; 
//3在CPP文件中改为CTx* pParent /*=NULL*/，并声明ID：CDialog(CInputmr::IDD, pParent)和tx(pParent) 
CInputmr::CInputmr(CTx* pParent /*=NULL*/) 
: CDialog(CInputmr::IDD, pParent),tx(pParent) 
{ 
if(Create(CInputmr::IDD,pParent))//创建 
ShowWindow(SW_SHOW);//显示窗口 
//就可以使用 tx-> 了 
} 

```

## 常用网络数据包报头结构->以太网、ARP、IP、TCP、UDP、ICMP、DNS、UDP伪报头

```C++
 
#pragma pack(1) 
typedef struct ethdr //以太网包头14字节 
{ 
unsigned char destination_mac[6];//目标MAC 6字节 
unsigned char source_mac[6];//源MAC 6字节 
unsigned short type;//后面的协议类型2字节 为0806表示后面跟的包为ARP包 
}ET_HEADER,*PETHDR; 
typedef struct arphdr //arp协议28字节 
{ 
unsigned short hard_tpye;//硬件类型2字节 通常为01 (以太网地址) 
unsigned short protocol;//协议类型2字节 通常为80 (IP地址) 
unsigned char hard_length;//硬件地址长度1字节 通常为6 
unsigned char protocol_length;//协议地址长度1字节 通常为4 (IP协议) 
unsigned short operation_type;//操作类型 1为ARP请求，2为ARP应答，3为RARP请求，4为RARP应答 
unsigned char source_mac[6];//源物理地址 
struct in_addr source_ip;//源IP地址 
unsigned char destination_mac[6];//目的物理地址 
struct in_addr destination_ip;//目的IP地址 
}ARP_HEADER,*PARPHDR; 
// IP Header -- RFC 791 IP数据报头 
typedef struct tagIPHDR 
{ 
u_char VIHL; // Version and IHL 版本4bit = 4 和 首部长度4bit = 5 
u_char TOS; // Type Of Service 服务类型 1字节 
short TotLen; // Total Length 总长度2字节，包括数据和报头 
short ID; // Identification 标识符2字节 
short FlagOff; // Flags and Fragment Offset 标志 3bit 和分段偏移量 13bit 
u_char TTL; // Time To Live 生存期1字节，为经过路由器的总次数 
u_char Protocol; // Protocol 协议类型1字节 
u_short Checksum; // Checksum 首部(只是IP首部!!不然就错了，过不了网关!!)校验和2字节 
struct in_addr source_ip; // Internet Address - Source 源IP地址 
struct in_addr destination_ip; // Internet Address - Destination 目的IP地址 
}IP_HEADER, *PIP_HEADER; 
//TCP Header TCP数据报头 
typedef struct tcp_hdr 
{ 
unsigned short source_port; //源端口 
unsigned short destination_port; //目的端口 
unsigned long index; //32位序号 
unsigned long makesuer_index; //32位确认序号 
unsigned short header_length_and_flags; //首部长度和标志位 
unsigned short window_size; //窗口大小 
unsigned short checksum; //检查和 
unsigned short exigency_pointer; //紧急指针 
}TCP_HEADER; 
//UDP Header --UDP数据报头 
typedef struct udp_hdr 
{ 
unsigned short source_port; //源端口 
unsigned short destination_port; //目的端口 
unsigned short length; //数据长度 
unsigned short checksum; //带数据!检查和 
} UDP_HEADER; 
// ICMP Header - RFC 792 ICMP数据报头 
typedef struct tagICMPHDR 
{ 
u_char Type; // Type 类型 
u_char Code; // Code 代码 
u_short Checksum; // Checksum 校验和 
u_short ID; // Identification 标识符 
u_short Seq; // Sequence 序列号 
char Data; // Data 数据(依情况而定) 
}ICMP_HEADER, *PICMP_HEADER; 
// ICMP Echo Request ICMP 请求数据报 
typedef struct tagECHOREQUEST 
{ 
ICMPHDR icmpHdr; 
DWORD dwTime; 
char cData[32]; 
}IMCP_REQUEST, *PIMCP_REQUEST; 
// ICMP Echo Reply ICMP 回响数据报 
typedef struct tagECHOREPLY 
{ 
IPHDR ipHdr; 
ECHOREQUEST echoRequest; 
char cFiller[256]; 
}ICMP_REPLY, *ICMP_REPLY; 
typedef struct dns //DNS数据报： 
{ 
unsigned short id;//标识，通过它客户端可以将DNS的请求与应答相匹配； 
unsigned short flags;//标志：[QR | opcode | AA| TC| RD| RA | zero | rcode ] 
unsigned short quests;//问题数目； 
unsigned short answers;//资源记录数目； 
unsigned short author;//授权资源记录数目； 
unsigned short addition;//额外资源记录数目； 
}DNS,*PDNS; 
//在16位的标志中：QR位判断是查询/响应报文，opcode区别查询类型，AA判断是否为授权回答，TC判断 
//是否可截断，RD判断是否期望递归查询，RA判断是否为可用递归，zero必须为0，rcode为返回码字段。 
typedef struct psd //伪报头，用于计算UDP校验和 
{ 
unsigned int source_ip; //源IP 
unsigned int destination_ip; //目的IP 
char mbz; // 0 
char protocol; //协议 UDP = 17 
unsigned short udp_length; //UDP 长度 
}PSD,*PPSD; 
//DNS查询数据报： 
typedef struct query 
{ 
unsigned short type; //查询类型，大约有20个不同的类型 
unsigned short classes; //查询类,通常是A类既查询IP地址。 
}QUERY,*PQUERY; 
//DNS响应数据报： 
typedef struct response 
{ 
unsigned short name; //查询的域名 
unsigned short type; //查询类型 
unsigned short classes; //类型码 
unsigned int ttl; //生存时间 
unsigned short length; //资源数据长度 
unsigned int addr; //资源数据 
}RESPONSE,*PRESPONSE; 
#pragma pack() 

```

## 网络编程部分源码->校验和、隐藏IP方法、取得本地MAC

```C++
//注意!!!!!! 
//IP的检查和只是IP头部的20个字节，不然过不了网关!!!! 
//而TCP和UDP和检查和是数据和头部都包括!! 
//在计算检查和之前IP或UDP或TCP报头中 所有的成员 必须被初始化，包括 checksum!!! 
//为了保证系统和机器的******兼容性******，在定义网络结构体或共用体时要加上 
#pragma pack(1) 
#pragma pack() 
//如： 
#pragma pack(1) 
typedef struct response 
{ 
unsigned short name; 
unsigned short type; 
unsigned short classes; 
unsigned int ttl; 
unsigned short length; 
unsigned int addr; 
}RESPONSE,*PRESPONSE; 
#pragma pack() 
//sizeof(RESPONSE)==16，如果不加#pragma pack那么sizeof(RESPONSE)==20 
//局域网内相同网段下的IP地址，数据包直接发给该主机的MAC地址，不同网段下的，数据包发给网关， 
//如：202.198.153.17->202.198.153.64 以太包目的地址为202.198.153.64 的MAC地址 
//如：202.198.153.17->202.198.159.139 以太包目的地址为202.198.153.254(网关) 的MAC地址 

```

### 隐藏IP的方法

```C++
WSADATA wsd; 
SOCKET s; 
BOOL bOpt; 
struct sockaddr_in remote; 
USHORT *buf=NULL; 
ECHOREQUEST icmpHdr; 
int ret; 
unsigned short iTotalSize, 
iIPVersion, 
iIPSize, 
cksum = 0; 
if (WSAStartup(MAKEWORD(2,2), &wsd) != 0)//打开WSA网络函数 
{ 
MessageBox("WSAStartup() failed: "); 
return ; 
} 
s = WSASocket(AF_INET, SOCK_RAW, IPPROTO_ICMP, NULL, 0,0);//建立SOCKET，注意第二个一定为SOCK_RAW 
if (s == INVALID_SOCKET) 
{ 
MessageBox("WSASocket() failed: "); 
return ; 
} 
bOpt = TRUE; 
//设置SOCKET属性 
ret = setsockopt(s, IPPROTO_IP, IP_HDRINCL, (char *)&bOpt, sizeof(bOpt)); 
if (ret == SOCKET_ERROR) 
{ 
MessageBox("setsockopt(IP_HDRINCL) failed: %d/n"); 
return ; 
} 
//修改IP包的值 
iTotalSize = sizeof(icmpHdr) ; 
iIPVersion = 4; 
iIPSize = sizeof(IP_HDR) / sizeof(unsigned long); 
icmpHdr.ipHdr.ip_verlen = (iIPVersion << 4) | iIPSize; 
icmpHdr.ipHdr.ip_tos = 0; 
icmpHdr.ipHdr.ip_totallength = htons(iTotalSize); 
icmpHdr.ipHdr.ip_id = 0; 
icmpHdr.ipHdr.ip_offset = 0; 
icmpHdr.ipHdr.ip_ttl = 128; 
icmpHdr.ipHdr.ip_protocol = 1; //ICMP 
buf=(USHORT *)&icmpHdr; 
icmpHdr.ipHdr.ip_checksum = checksum(buf,sizeof(IP_HDR)); 
icmpHdr.ipHdr.ip_srcaddr = inet_addr(m_source_ip); 
icmpHdr.ipHdr.ip_destaddr = inet_addr(m_destinition); 
icmpHdr.icmpHdr.Type=8; 
icmpHdr.icmpHdr.Code=0; 
icmpHdr.icmpHdr.ID=1; 
icmpHdr.icmpHdr.Seq=1; 
buf=(USHORT *)&icmpHdr; 
icmpHdr.icmpHdr.Checksum=checksum(buf+sizeof(IP_HDR),sizeof(icmpHdr)-sizeof(IP_HDR)); 
icmpHdr.dwTime=GetTickCount(); 
remote.sin_family = AF_INET; 
remote.sin_port = 0; 
remote.sin_addr.s_addr = inet_addr(m_destinition); 
struct timeval Timeout; 
fd_set readfds; 
readfds.fd_count = 1; 
readfds.fd_array[0] = s; 
Timeout.tv_sec = 1; 
Timeout.tv_usec = 0; 
int nRet; 
int nAddrLen = sizeof(struct sockaddr_in); 
ECHOREPLY echoReply; 
CString cs; 
for(int i=0;i<m_ping_times;i++) 
{ //发送IP包 
ret = sendto(s, (char *)&icmpHdr, iTotalSize, 0, (SOCKADDR *)&remote, sizeof(remote)); 
if (ret == SOCKET_ERROR) 
MessageBox("sendto() failed:"); 
/* 
select(1, &readfds, NULL, NULL, &Timeout); 
//接收请求回应 
nRet = recvfrom(s, 
(LPSTR)&echoReply, 
sizeof(ECHOREPLY), 
0, 
(LPSOCKADDR)&remote, 
&nAddrLen); 
//检查返回值 
if (nRet == SOCKET_ERROR) 
MessageBox("recvfrom()"); 
cs.Format("%d",echoReply.ipHdr.ip_ttl); 
MessageBox(cs); 
//返回发送的时间 
*/ 
Sleep(m_separete); 
} 
closesocket(s) ; 
WSACleanup() ;//关闭WSA 
} 

```

### 校验和程序

```C++
USHORT CFffDlg::checksum(USHORT *buffer, int size) 
{ 
unsigned long cksum=0; 
while (size > 1) 
{ 
cksum += *buffer++; 
size -= sizeof(USHORT); 
} 
if (size) 
{ 
cksum += *(UCHAR*)buffer; 
} 
cksum = (cksum >> 16) + (cksum & 0xffff); 
cksum += (cksum >>16); 
return (USHORT)(~cksum); 
} 

```

### 取得本地机器IP地址

```c++
WORD wv=MAKEWORD(1,1); 
WSADATA ws; 
WSAStartup(wv,&ws); 
char name[256]; 
gethostname(name,256); 
HOSTENT *hos=gethostbyname(name); 
CString cs; 
for(int i=0;hos!=NULL&&hos->h_addr_list[i]!=NULL;i++) 
{ 
cs=inet_ntoa(*(struct in_addr*)hos->h_addr_list[i]); 
MessageBox(cs); 
} 
WSACleanup(); 

```

### 取得本地MAC地址

```C++
#include <lm.h> 
NetApi32.lib 
unsigned char macdata[8]; 
WKSTA_TRANSPORT_INFO_0 * pwkti; 
DWORD dwe,dwt; 
BYTE *pb; 
NET_API_STATUS dws=NetWkstaTransportEnum( 
NULL, 
0, 
&pb, 
MAX_PREFERRED_LENGTH, 
&dwe, 
&dwt, 
NULL); 
pwkti=(WKSTA_TRANSPORT_INFO_0 *)pb; 
CString cs; 
for(DWORD i=1;i<dwe;i++) 
{ 
swscanf((wchar_t *)pwkti[i].wkti0_transport_address,L"%2hx%2hx%2hx%2hx%2hx%2hx",&macdata[0], 
&macdata[1],&macdata[2],&macdata[3],&macdata[4],&macdata[5]); 
cs.Format("%02x%02x%02x%02x%02x%02x",macdata[0], 
macdata[1],macdata[2],macdata[3],macdata[4],macdata[5]); 
MessageBox(cs); 
} 
NetApiBufferFree(pb); 

```

## ADO连接代码

```C++
//用#import引用msado15.dll。可以直接在Stdafx.h文件中加入 
#import "c:/program files/common files/system/ado/msado15.dll" / 
no_namespace / 
rename ("EOF", "adoEOF") 
// 定义ADO连接、命令、记录集变量指针 
_ConnectionPtr m_pConnection; 
_CommandPtr m_pCommand; 
_RecordsetPtr m_pRecordset; 
//初始化连接 
AfxOleInit(); 
m_pConnection.CreateInstance(__uuidof(Connection)); 
m_pRecordset.CreateInstance(__uuidof(Recordset)); 
// 在ADO操作语句中要常用try...catch()来捕获错误信息， 
// 因为它会经常出现一些意想不到的错误。 
try //打开库 
{ 
// 打开本地Access库Demo.mdb 
m_pConnection->Open("Provider=Microsoft.Jet.OLEDB.4.0;Data Source=Demo.mdb","","",adModeUnknown); 
} 
catch(_com_error e) 
{ 
AfxMessageBox("数据库连接失败，确认数据库Demo.mdb是否在当前路径下!"); 
return FALSE; 
} 
try //打开表 
{ 
m_pRecordset->Open("SELECT * FROM DemoTable", // 查询DemoTable表中所有字段 
m_pConnection.GetInterfacePtr(), // 获取库接库的IDispatch指针 
adOpenDynamic, 
adLockOptimistic, 
adCmdText); 
} 
catch(_com_error *e) 
{ 
AfxMessageBox(e->ErrorMessage()); 
} 
//m_CListBox.AddString((LPCSTR)_bstr_t(m_pRecordset->GetCollect("Name")));//获得Name字段的值 
//m_pRecordset->PutCollect("Name", _variant_t(m_Name)); // 写入各字段值 

```

第一部分：记录集 

 

 

记录集是从数据库中按一定查询条件读入到内存中的一批记录，以供快速的操作。 

记录集recordset对象的属性，方法： 

BOF:当记录集记录指针指到起始记录（第1条记录）再向前移（即超过第1条记录），这时返回true.常用来对付一些出错情况。注：在BOF或EOF时使用update方法会出错。 

EOF:当记录指针指到最后一条记录之后（即超过了最后1条记录）时，该属性返回true.注：当一个记录集为空时，其BOF和EOF属性都为True，可据此检测一个记录集是否为空。 

AbsolutePosition:返回当前记录指针，即当前记录是第几条记录，只读。 

BookMark: 设置/返回当前记录指针的书签，为字符串。如： 

在当前记录处设置书签：lxn=Adodc1.Recordset.BookMark 

当指针移动后回到指定书签位置：Adodc1.Recordset.BookMark=lxn. 

返回记录集中记录的总数：RecordCount属性。该属性对于表形式的记录集将返回精确数目，但对于仅向前型记录集(adOpenForwardOnly)，返回-1。动态记录集(adOpenDynamic)则不一定，可能返回-1，与记录集的CursorLocation有关；而对于静态记录集(adOpenStatic)，也总能返回精确数目。附：在DAO中，对动态集和快照集需要先用MoveFirst和Movelast方法，再用RecordCount取得记录精确数目。 

Move方法：移动记录集指针。该方法有两个参数，第一个参数指定要向前或向后移动多少条记录，第二个参数指定一个相对书签位置，表明从当前记录还是从第1条或最后1条记录开始算，缺省为0从当前记录开始移，将指针从当前位置向前（负数）或向后（正数）移动指定条记录（第二个‘按书签移动’参数设为0-adBookMarkCurrent从当前记录开始，缺省）或将指针从第1条记录算起移动指定条记录（第二个参数设为1-adBookMarkFirst从首记录）。或将指针从最后1条记录算起移动指定条记录（第二个参数设为adBookMarkLast），如：Adodc1.Recordset.Move -12，将指针从当前位置向前移动12条记录，再如Adodc1.Recordset.Move 6 , 1表示指针从首记录开始后移6条记录，即使指针移到第7条记录。Move方法有几个引申的方法，如下： 

movefirst,记录集指针移到第1条记录； 

movelast,记录集指针移到最后1条记录； 

moveprevious,记录集指针移到上一条记录； 

movenext,记录集指针移到下一条记录。 

Find方法：查找满足条件的记录。 

find方法简略格式为： 

adodc对象.recordset.find 查找表达式（为“字段  比较符号  值”） 

其他参数采用缺省，find工作方式是：从当前记录指针位置开始（含当前记录）向后逐条检索记录，遇到满足条件的1条记录就停下来，指针指到此记录。 

因此，若要实现“继续寻找下1个”的功能，只要将记录指针移到下1条记录（用movenext一下），再照原样使用find即可。  而重新查找必须先用movefirst将指针指到开头。 

Adodc1.Recordset.find "姓名 = '李长春'" 

其中查找表达式的样子为“字段  比较符号  值”，比较符号不仅可以是等号，还可以是>,<,<=,>=,<>,like等。其中Like和 = 很相近，只不过like专用于字符串比较，不区分字母的大小写可用通配符，而 = 号会区分字母大小写不能用通配符。 

一般情况下，查找表达式常采用变量表示，由用户来确定，如下： 

Dim lxn As String 

Static a As String 

a = InputBox("请输入查找姓名","查找") 

lxn = "NAME like '" & a & "'" 

.Find lxn 

 

LockType属性：设置记录集中的记录锁定方式，是否可修改及修改方式：有1-adLockReadOnly(只读)；2-adLockPessimistic(保守式修改)，当修改记录后立即将更改保存到数据源，3-adLockOptimistic(开放式修改)，当修改记录后只有调用Update方法才将更改保存到数据源；4-adLockBatchOptimistic(开放式批处理修改)。当修改记录后只有调用UpdateBatch方法才将更改保存到数据源。对于ADO对象而言，该属性的缺省值为1-adLockReadOnly只读，要编辑记录则必须加以改变。 

Update方法：对记录集当前记录的更改进行保存到数据库。 

UpdateBatch方法：成批保存更改的多条记录。只有当记录集使用锁定方式为adLockBatchOptimistic打开时该方法才有效。使用该方法，可以加快更新速度。因为一条一条更新的话，速度慢，而多条一起更新的话，其实等同于一个更新操作，因此更快。该方法有个可选参数AffectRecords提一下，它可设为：adAffectCurrent只更新当前记录；adAffectGroup只更新当前Filter属性满足的记录；adAffectAll（缺省）全部更新，包括被当前Filter属性隐藏的记录。 

CancelUpdate方法：放弃保存对当前记录自上次Update后的更改，即不保存当前所作的修改。通常在WillChangeRecord事件中进行数据验证时用。当然一般是直接将事件提供的参数adStatus设为adStatusCancel即可取消保存。这里要注意，在WillChangeRecord事件中取消一个操作，将发生“操作已取消”的错误。照我的感觉，还不如直接在要Update的代码前面去验证输入的数据。 

在记录集中添加新记录，用addnew方法先在缓存中添加一个新的空记录，这时它自动成为当前记录，通过修改，然后用update方法保存，data1.recordset.update，说明：如果用Movenext等方法将记录指针移开时记录集会自动保存缓存中的记录(等于调用Update方法)。 

修改当前记录，只要直接给字段赋值就可以了，注意赋值后也要用Update方法将缓存中的数据保存，否则不会自动更新，除非用MoveNext等方法将指针移开让它自动调用。 

删除当前记录用Delete方法就行了。有一点要注意，删除后，记录指针仍在被删除的记录上，因此要用MoveNext等方法将指针移开，或干脆Requery刷新一下。 

Sort：指定用来对全部记录排序的参照字段。 

Fields：包含记录集中各字段的集合。指定某个字段格式为：fields("字段名")。可省略。如有：m$=data1.recordset.fields("姓名").value为读当前记录(用value表示)的“姓名”字段值。还可用来在代码中修改（写）记录值（不是用绑定控件），如修改记录值为“李新能”：data1.recordset.fields("姓名").value="李新能"。也可写为data1.recordset("姓名").value，括号内写明字段名或“字段索引值”，第1个字段索引值从0开始。如data1.recordset(0).value. 

关闭记录集用close方法，格式为“记录集.close”。 

要读或写当前记录的某个字段值，只要用“记录集("字段名")”就可以了，Fields和Value都是缺省属性，但当值是一个记录集的除外，如数据环境中Command命令对象的子命令对象。 

刷新记录集（即重新打开记录集）：Requery方法。在记录集打开的情况下，迅速关闭又打开一次，以达到确保记录集始终处于激活状态。 

返回/设置记录集当前编辑状态：EditMode属性。有以下三个可能返回值：adEditNone没有编辑；adEditInProgress正在编辑（即当前记录已修改但未保存）；adEditAdd已经用AddNew方法添加新记录但还未存盘，在缓存中的是新记录。这个属性通常用来检测某些操作状态，比如可在窗体的Unload事件中防止修改了的记录未保存就卸载。 

★ AbsolutePage属性：指定当前记录所在的页。其值的范围在1—PageCount值之间，所谓页，是把Recordset对象按PageSize为标准分为若干页面，每一页（除最后一页）记录数相等（等于PageSize）。有三个特殊值：adPosUnkown未知位置；adPosBOF在文件头；adPosEOF在文件尾。 

★ ActiveCommand:属性：只读属性。返回关联的Command对象，如果记录集不是由Command对象创建的，则返回Null. 

★ ActiveConnection属性:设置/返回记录集基于哪个Connection对象，如果没有Connection对象，则直接指定一个连接字符串。 

★ Cancel: 取消执行异步 Execute（对Connection和Command对象而言）  或  异步Open 方法调用（即通过 adAsyncConnect、adAsyncExecute 或 adAsyncFetch 参数选项调用这些方法）。 

★ CacheSize属性：本地内存缓存的大小。 

★ CancelBatch方法: 取消批更新模式下记录集中所有还未执行的更新。 

★ Clone方法:复制一个记录集。格式：Set 记录集变量=记录集.Clone [adLockReadOnly]当指定可选参数adLockReadOnly表示创建只读的记录集副本。使用 Clone 方法可创建多个 Recordset 对象副本，这对于希望在给定的记录组中保留多个当前记录十分有用。使用 Clone 方法比使用与初始定义相同的定义创建和打开新 Recordset 对象要有效得多。新创建副本的当前记录将设置为首记录。无论游标类型如何，对某个 Recordset 对象所作的修改在其所有副本中都是可见的。不过一旦在原始 Recordset 上执行了 Requery，副本将不再与原始 Recordset 同步。关闭原始 Recordset 时并不关闭它的副本，而关闭某个副本也将不关闭原始 Recordset 或任何其他副本。用户只允许复制支持书签的 Recordset 对象。书签值是可交换的，也就是说，来自一个 Recordset 对象的书签引用可引用其任何副本中的相同记录。 

★ CompareBookmarks: 比较两个书签并返回它们相差值的说明。即谁先谁后。 

★ CursorLocation: 设置或返回游标服务的位置。游标：可以简单理解为指向若干行的指针。有三种设置值：adUseNone 没有使用游标服务。（该常量已过时并且只为了向后兼容才出现，通常不用）。adUseClient 使用由本地游标库提供的客户端游标，通常使用这种游标。adUseServer 使用数据提供者的或驱动程序提供的游标。 

该属性应在建立连接之前设置，更改 CursorLocation 属性不会影响现有的连接。 

远程数据服务用法  当用于客户端 (ADOR) Recordset 或 Connection 对象时，只能将 CursorLocation 属性设置为 adUseClient。 

★ CursorType: 指示在 Recordset 对象中使用的游标类型。AdOpenForwardOnly 仅向前游标，默认值。除了只能在记录中向前滚动外，与静态游标相同。当只需要在记录集中单向移动时，使用它可提高性能。 

AdOpenKeyset 键集游标。尽管从您的记录集不能访问其他用户删除的记录，但除无法查看其他用户添加的记录外，键集游标与动态游标相似。仍然可以看见其他用户更改的数据。 

AdOpenDynamic 动态游标。可以看见其他用户所作的添加、更改和删除。允许在记录集中进行所有类型的移动，但不包括提供者不支持的书签操作。 

AdOpenStatic 静态游标。可以用来查找数据或生成报告的记录集合的静态副本。另外，对其他用户所作的添加、更改或删除不可见。 

说明：使用 CursorType 属性可指定打开 Recordset 对象时应该使用的游标类型。Recordset 关闭时 CursorType 属性为读/写，而 Recordset 打开时该属性为只读。 

如果将 CursorLocation 属性设置为 adUseClient 则只支持 adOpenStatic 的设置。 

★ Filter属性：过滤器。对记录集进行筛选，返回记录集中满足条件的所有记录，该属性指定一个筛选字符串，为一个“字段-比较符号-值”的条件表达式，当该属性赋值后，记录集立即变成筛选后的方式，包括如 AbsolutePosition、AbsolutePage、RecordCount 和 PageCount等属性都将改变，“变”成了一个“新”记录子集，当前记录移动到记录子集的第一个记录。而当清除 Filter 属性后，记录集立即恢复，当前记录位置将移动到原Recordset 的第一个记录。 

关于“字段-比较符号-值”的说明：“字段”  必须为 Recordset 中的有效字段名。如果字段名包含空格，必须用方括号将字段名括起来。 

“比较符号”必须使用的操作符为：<、>、<=、>=、<>、= 或 LIKE。 

“值”  是用于与字段值（如 'Smith'、#8/24/95#、12.345 或 $50.00）进行比较的值。字符串使用单引号而日期使用井号 #(注:在SQL中不用#号)，对于数字，可以使用小数点、货币符号和科学记数法。如果 “比较符号”  为 LIKE，“值”  则可使用通配符。ADO和SQL只允许使用下划线(_) 和百分号 (%) 通配符，而且必须为字符串的尾字符。DAO只允许使用问号(?)和星号(*)作通配符.“值”  不可为 Null。在 LIKE 子句中，可在样式的开头和结尾使用通配符（如 LastName Like '*mit*'），或者只在结尾使用通配符（如，LastName Like 'Smit*'） 

Filter属性还有几个特殊值可供选择：AdFilterNone 删除当前筛选条件并恢复查看所有记录。同空字符串””。 AdFilterPendingRecords 允许只查看已更改且尚未发送到服务器的记录。只能应用于批更新模式。 AdFilterAffectedRecords 允许只查看上一次 Delete、Resync、UpdateBatch 或 CancelBatch 调用所影响的记录。 AdFilterFetchedRecords 允许查看当前缓冲区中的记录，即上一次从数据库中检索记录的调用结果。 AdFilterConflictingRecords 允许查看在上一次批更新中失败的记录。 

★ GetRows方法: 将Recordset的多个记录值复制到数组中，应当先定义一个变体变量，然后将GetRows的返回值赋给它，这个变量就成了一个二维数组，其第一维下标标识原所在的列（字段），第二维下标标识原所在的行（记录），每一个交叉点就是一个值了，如： 

Dim VariData As Variant 

VariData =  DataEnvironment1.rsCommand1.GetRows 

x = UBound(VariData, 1) 

y = UBound(VariData, 2) 

For m = 0 To x 

For n = 0 To y 

Print VariData(m, n); ‘用分号表示同一字段的数据打印在同一行。 

Next n 

Print 

Next m 

该方法格式：变体变量=记录集.GetRows([rows],[start],[fields])有三个可选参数，第一个参数Rows限制返回的记录数量，即要复制几行记录，它决定的是变体数组的第二维长度，缺省情况下，将读取记录集中的所有记录。第二个参数Start指定从哪个记录位置开始向数组复制记录，可以是一个记录书签字符串，或以下三个常数之一：adBookMarkCurrent(当前记录)adBookMarkFirst(首记录录)adBookMarkLast(尾记录)；第三个参数Fields限制返回的记录字段，即要复制哪几列，它决定的是变体数组的第一维长度，缺省情况下，将读取记录集中的所有字段，该参数可设置为单个字段名字符串，或多个字段名字符串组成的数组。 

注意：使用该方法复制记录值时，记录指针将随之移动，每复制一行后，指针自动移动到下一行。 

★ GetString: 将 Recordset 按字符串值的变体型 (BSTR) 返回。 

★ MarshalOptions: 汇集选项。指示要被调度返回服务器的记录。可选设置值：AdMarshalAll 默认值。表明所有行将返回到服务器。 AdMarshalModifiedOnly 表明只有已修改的行返回到服务器。当使用客户端 (ADOR) Recordset 时，已在客户端被修改的记录将通过称作“调度”的技术写回中间层或 Web 服务器。 

★ MaxRecords: 指示通过一次查询返回 Recordset 的记录的最大数目。使用 MaxRecords 属性可对从数据源返回的记录数加以限制。该属性的默认设置为零，表明提供者返回所有所需的记录。Recordset 关闭时，MaxRecords 属性为读/写，打开时为只读。 

★ NextRecordset:清除当前Recordset对象，执行下一个命令返回新的Recordset对象。当一个命令语句是复合语句（即用分号隔开的多条命令）？时，用该方法依次执行下一条命令。格式：Set recordset2=recordset1.NextRecordset。例如： 

Dim rst As ADODB.Recordset 

Dim strCnn As String 

Dim strCmd As String 

strCnn =  "Provider=Microsoft.Jet.OLEDB.3.51;Persist Security Info=False;Data  Source=C:/工商所收费系统/MyDatabase.mdb" 

strCmd = "SELECT * FROM unitrecord;SELECT *  FROM invoice" ‘AccessSQL不支持。 

Set rst = New ADODB.Recordset 

rst.Open strCmd, strCnn, adOpenDynamic,  adLockOptimistic, adCmdText 

Print rst("Name") 

Set rst = rst.NextRecordset 

Print rst("Name") 

rst.Close 

★ Open:打开一个游标,即记录集。 

★ PageCount:页面数。 

★ PageSize:每页大小，缺省为10。 

★ Properties: 

★ Resync: 

★ Save: 

★ Seek方法:使用索引进行查询，比用Find方法速度更快，但只能用于以表形式打开的记录集，不能用于动态记录集或快照型记录集，不能在远程服务器表上使用Seek，因为远程数据源不能以表形式打开。而且这个表预先定义了索引字段，使用Seek方法前，要在代码中用Index属性指定当前要使用的索引，格式为：记录集对象.Index=索引名。索引名是在表的设计阶段定义好的。注意索引名不等于字段名，只不过是以某个字段为标准的。设置好要使用的索引后，使用Seek进行查找，格式：Recordset对象.Seek 值，这里的值参数指定按当前索引所属字段进行查找的值，若找到，则指针指到此记录，若没找到，则EOF为True. 

一般情况下，在ADO中都不用Seek进行定位，而是用SQL查询生成动态记录集。只是在DAO中有一些使用，如： 

Private Sub Command2_Click() 

Data1.Recordset.Index = "indexName" 

Data1.Recordset.Seek "=", "李春生" 

Text1.Text = Data1.Recordset(2) 

End Sub 

其格式有一点不同，它的第一参数指定一个比较符号，第二个参数才是值，需要在属性窗口中将DATA1的RecordsetType属性设置为0-Table。 

★ Sort: 

★ Source:数据源。 

★ State:对象的当前状态，有adStateClosed（关闭）或adStateOpen（打开）。 

★ Status:批量操作或海量操作的状态。 

★ StayInSync: 

★ Supports方法:判断本记录集是否具有某个方面的功能。如：是否允许增添记录If rst.Supports(adAddnew)=True then rst.Addnew，如果具有某项功能则返回True，不具备则返回False，该方法的一个参数是指定哪个方面，如adDelete是否允许删除记录，adBookmark是否支持书签设置，adUpdate是否允许更新（即修改）数据源，adIndex是否可以使用index属性设置索引，adSeek是否可用Seek方法定位记录指针。再如判断是否支持索引：MsgBox DataEnvironment1.rsCommand1.Supports(adIndex)。 

记录集有五种不同的类型： 

Table:表示数据库中一张表，记录集与数据库中的数据同步，可通过记录集对数据库添加，删除等操作。 

Dynaset:一张查询结果集，可由多个表中不同数据组成，可通过记录集对数据库进行添加、删除等操作。 

Dynamic:与dynaset相似，但它有这样的功能：当其他用户修改记录集的基表（数据库表）时，会将修改反映到这个记录集中，主要用于多用户操作。 

SnapShot:一张只读的查询结果集，可包含不同表中数据记录，不能对记录添加，修改等，可用于浏览数据库。 

Forward-Only:一个没游标的SnapShot记录集，只能从头到尾顺序经过所有记录，不能任意移动。 

### ODBC动态创建数据源

```C++
#define ODBC_ADD_DSN 1 // Add data source 
#define ODBC_CONFIG_DSN 2 // Configure (edit) data source 
#define ODBC_REMOVE_DSN 3 // Remove data source 
#include <odbcinst.h> 
CString sPath; 
GetModuleFileName(NULL,sPath.GetBufferSetLength(MAX_PATH+1),MAX_PATH); 
sPath.ReleaseBuffer (); 
int nPos; 
nPos=sPath.ReverseFind ('//'); 
sPath=sPath.Left (nPos); 
nPos=sPath.ReverseFind('//'); 
sPath=sPath.Left (nPos); 
CString lpszFile = sPath + "//DataBase.mdb"; 
char* szDesc; 
int mlen; 
szDesc=new char[256]; 
sprintf(szDesc,"DSN=%s? DESCRIPTION=TOC support source? DBQ=%s? FIL=MicrosoftAccess? DEFAULTDIR=%s?? ","zth",lpszFile,sPath); 
mlen = strlen(szDesc); 
for (int i=0; i<mlen; i++) 
{ 
if (szDesc[i] == '?') 
szDesc[i] = '/0'; 
} 
if (FALSE == SQLConfigDataSource(NULL,ODBC_ADD_DSN,"Microsoft Access Driver (*.mdb)/0",(LPCSTR)szDesc)) 
AfxMessageBox("SQLConfigDataSource Failed");

```

## 打开一个网址

- 打开`http://www.sina.com.cn`这个站点如下：  
  
  ```
  ShellExecute(NULL, "open", "http://www.sina.com.cn",NULL, NULL, SW_MAXIMIZE );
  ```
  
  此命令将以默认浏览器打开`http://www.sina.com.cn`，并将加开后的窗口最大化。
  
- 又例：

  ```
  ShellExecute(NULL, "open", "IEXPLORE.exe http://www.sina.com.cn",NULL, NULL, SW_MAXIMIZE );
  ```

  此命令将直接用IE打开一个sina的站点。不过将开一个新的窗口。

## 控件的注册

- 注册
  
  ```
  regsvr32 x:\xxx\demo.ocx //不一定非得在 Windows 系统目录
  ```
- 卸载
  
  ```
  regsvr32 /u x:\xxx\demo.ocx
  ```
  
  

## 对话框透明特效

- 在OnInitDialog()中加入以下代码：

  ```C++
  //加入WS_EX_LAYERED扩展属性
  SetWindowLong(this->GetSafeHwnd(),GWL_EXSTYLE,
  GetWindowLong(this->GetSafeHwnd(),GWL_EXSTYLE)^0x80000);
  HINSTANCE hInst = LoadLibrary("User32.DLL");
  if(hInst)
  {
      typedef BOOL (WINAPI *MYFUNC)(HWND,COLORREF,BYTE,DWORD);
      MYFUNC fun = NULL;
      //取得SetLayeredWindowAttributes函数指针
      fun=(MYFUNC)GetProcAddress(hInst, "SetLayeredWindowAttributes");
      if(fun)fun(this->GetSafeHwnd(),0,128,2);
      FreeLibrary(hInst);
  }
  ```

  注意：fun的参数128不能太小，否则就完全透明了！
