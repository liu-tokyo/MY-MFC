# VC 获取系统特殊文件夹的路径

```c++
TCHAR MyDir[_MAX_PATH];  
SHGetSpecialFolderPath(this->GetSafeHwnd(),MyDir,CSIDL_APPDATA,0);
AfxMessageBox(MyDir);
```

其中第三个参数CSIDL_APPDATA可以为以下的值：

|CSIDL_APPDATA|说明|
| - |- |
|CSIDL_BITBUCKET |回收站 |
|CSIDL_CONTROLS |控制面板 |
|CSIDL_DESKTOP Windows |桌面Desktop |
|CSIDL_DESKTOPDIRECTORY |Desktop的目录 |
|CSIDL_DRIVES |我的电脑 |
|CSIDL_FONTS |字体目录 |
|CSIDL_NETHOOD |网上邻居 |
|CSIDL_NETWORK |网上邻居虚拟目录 |
|CSIDL_PERSONAL |我的文档 |
|CSIDL_PRINTERS |打印机 |
|CSIDL_PROGRAMS |程序组 |
|CSIDL_RECENT |最近打开的文档 |
|CSIDL_SENDTO |“发送到”菜单项 |
|CSIDL_STARTMENU |任务条启动菜单项 |
|CSIDL_STARTUP |启动目录 |
|CSIDL_TEMPLATES |文档模板|