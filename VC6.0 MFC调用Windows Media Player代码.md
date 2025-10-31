## VC6.0 MFC调用`Windows Media Player`代码

- 在相应的对话框中鼠标右键-`插入Active X 控件`-`Windows Media Player`；
- 更改控件名；
- 建立类向导-`Member Variables`-`Add variable`-提示中点击确定-绑定对象 `m_MediaPlayer`；
- 添加 `#include "wmpcontrols.h"`
- 创建CWMPControls对象 `m_WMPControls`;
- 初始化：`m_WMPControls = static_cast<CWMPControls>(m_MediaPlayer.GetControls());`
- 设置播放文件路径：`m_MediaPlayer.SetUrl(videoPath);`
- 开始播放：`m_WMPControls.play();`
- 暂停播放：`m_WMPControls.pause();`
- 停止播放：`m_WMPControls.stop();`
- 窗口最大化：`m_MediaPlayer.SetFullScreen(TRUE);`
- 窗口最小化：`m_MediaPlayer.SetFullScreen(FALSE);`
- 关闭后释放资源：`m_WMPControls.ReleaseDispatch();`

注意：

在调试时发现程序直接调用没有任何问题，但使用TCP控制视频最大化时总是报错

解决方法：在MediaPlayer控件中添加一个按钮，在此按钮的消息响应函数中添加视频播放器的控制命令，TCP控制时，直接给此按钮发送消息（PostMessage）即可。

原文链接：https://blog.csdn.net/GuitarCoder/article/details/83993482

youtube视频下载可以通过下面的网址找到真实的视频地址：
https://youtube.iiilab.com/