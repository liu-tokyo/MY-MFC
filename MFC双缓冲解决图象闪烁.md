# MFC双缓冲解决图象闪烁

> 转载网上找到的一篇双缓冲的文章，很好用。http://www.cnblogs.com/piggger/archive/2009/05/02/1447917.html

显示图形如何避免闪烁，如何提高显示效率是问得比较多的问题。而且多数人认为MFC的绘图函数效率很低，总是想寻求其它的解决方案。
MFC的绘图效率的确不高但也不差，而且它的绘图函数使用非常简单，只要使用方法得当，再加上一些技巧，用MFC可以得到效率很高的绘图程序。
我想就我长期（呵呵当然也只有2年多）使用MFC绘图的经验谈谈我的一些观点。

## 1、显示的图形为什么会闪烁？

我们的绘图过程大多放在OnDraw或者OnPaint函数中，OnDraw在进行屏幕显示时是由OnPaint进行调用的。当窗口由于任何原因需要重绘时，总是先用背景色将显示区清除，然后才调用OnPaint，而背景色往往与绘图内容反差很大，这样在短时间内背景色与显示图形的交替出现，使得显示窗口看起来在闪。如果将背景刷设置成NULL，这样无论怎样重绘图形都不会闪了。当然，这样做会使得窗口的显示乱成一团，因为重绘时没有背景色对原来绘制的图形进行清除，而又叠加上了新的图形。有的人会说，闪烁是因为绘图的速度太慢或者显示的图形太复杂造成的，其实这样说并不对，绘图的显示速度对闪烁的影响不是根本性的。例如在OnDraw(CDC *pDC)中这样写：

```C++
pDC->MoveTo(0,0);
pDC->LineTo(100,100);
```

这个绘图过程应该是非常简单、非常快了吧，但是拉动窗口变化时还是会看见闪烁。其实从道理上讲，画图的过程越复杂越慢闪烁应该越少，因为绘图用的时间与用背景清除屏幕所花的时间的比例越大人对闪烁的感觉会越不明显。比如：清楚屏幕时间为1s绘图时间也是为1s，这样在10s内的连续重画中就要闪烁5次；如果清楚屏幕时间为1s不变，而绘图时间为9s，这样10s内的连续重画只会闪烁一次。这个也可以试验，在OnDraw(CDC *pDC)中这样写：

```C++
for(int i=0;i<100000;i++)
{
    pDC->MoveTo(0,i);
    pDC->LineTo(1000,i);
}
```

呵呵，程序有点变态，但是能说明问题。

说到这里可能又有人要说了，为什么一个简单图形看起来没有复杂图形那么闪呢？这是因为复杂图形占的面积大，重画时造成的反差比较大，所以感觉上要闪得厉害一些，但是闪烁频率要低。那为什么动画的重画频率高，而看起来却不闪？这里，我就要再次强调了，闪烁是什么？闪烁就是反差，反差越大，闪烁越厉害。因为动画的连续两个帧之间的差异很小所以看起来不闪。如果不信，可以在动画的每一帧中间加一张纯白的帧，不闪才怪呢。

## 2、如何避免闪烁

在知道图形显示闪烁的原因之后，对症下药就好办了。首先当然是去掉MFC提供的背景绘制过程了。实现的方法很多，
* 可以在窗口形成时给窗口的注册类的背景刷付NULL
* 也可以在形成以后修改背景 

  ```C++
  static CBrush brush(RGB(255,0,0));
  SetClassLong(this->m_hWnd,GCL_HBRBACKGROUND,(LONG)(HBRUSH)brush);
  ```

* 要简单也可以重载OnEraseBkgnd(CDC* pDC)直接返回TRUE

这样背景没有了，结果图形显示的确不闪了，但是显示也象前面所说的一样，变得一团乱。怎么办？这就要用到双缓存的方法了。双缓冲就是除了在屏幕上有图形进行显示以外，在内存中也有图形在绘制。我们可以把要显示的图形先在内存中绘制好，然后再一次性的将内存中的图形按照一个点一个点地覆盖到屏幕上去（这个过程非常快，因为是非常规整的内存拷贝）。这样在内存中绘图时，随便用什么反差大的背景色进行清除都不会闪，因为看不见。当贴到屏幕上时，因为内存中最终的图形与屏幕显示图形差别很小（如果没有运动，当然就没有差别），这样看起来就不会闪。

## 3、如何实现双缓冲

首先给出实现的程序，然后再解释，同样是在OnDraw(CDC *pDC)中：

```C++
CDC MemDC; //首先定义一个显示设备对象
CBitmap MemBitmap;//定义一个位图对象
//随后建立与屏幕显示兼容的内存显示设备
MemDC.CreateCompatibleDC(NULL);
//这时还不能绘图，因为没有地方画 ^_^
//下面建立一个与屏幕显示兼容的位图，至于位图的大小嘛，可以用窗口的大小
MemBitmap.CreateCompatibleBitmap(pDC,nWidth,nHeight);
//将位图选入到内存显示设备中
//只有选入了位图的内存显示设备才有地方绘图，画到指定的位图上
CBitmap *pOldBit=MemDC.SelectObject(&MemBitmap);
//先用背景色将位图清除干净，这里我用的是白色作为背景
//你也可以用自己应该用的颜色
MemDC.FillSolidRect(0,0,nWidth,nHeight,RGB(255,255,255));
//绘图
MemDC.MoveTo(……);
MemDC.LineTo(……);
//将内存中的图拷贝到屏幕上进行显示
pDC->BitBlt(0,0,nWidth,nHeight,&MemDC,0,0,SRCCOPY);
//绘图完成后的清理
MemBitmap.DeleteObject();
MemDC.DeleteDC();
```

上面的注释应该很详尽了，废话就不多说了。

## 4、如何提高绘图的效率

我主要做的是电力系统的网络图形的CAD软件，在一个窗口中往往要显示成千上万个电力元件，而每个元件又是由点、线、圆等基本图形构成。如果真要在一次重绘过程重画这么多元件，可想而知这个过程是非常漫长的。如果加上了图形的浏览功能，鼠标拖动图形滚动时需要进行大量的重绘，速度会慢得让用户将无法忍受。怎么办？只有再研究研究MFC的绘图过程了。

实际上，在OnDraw(CDC *pDC)中绘制的图并不是所有都显示了的，例如：你在OnDraw中画了两个矩形，在一次重绘中虽然两个矩形的绘制函数都有执行，但是很有可能只有一个显示了，这是因为MFC本身为了提高重绘的效率设置了裁剪区。裁剪区的作用就是：只有在这个区内的绘图过程才会真正有效，在区外的是无效的，即使在区外执行了绘图函数也是不会显示的。因为多数情况下窗口重绘的产生大多是因为窗口部分被遮挡或者窗口有滚动发生，改变的区域并不是整个图形而只有一小部分，这一部分需要改变的就是pDC中的裁剪区了。因为显示（往内存或者显存都叫显示）比绘图过程的计算要费时得多，有了裁剪区后显示的就只是应该显示的部分，大大提高了显示效率。但是这个裁剪区是MFC设置的，它已经为我们提高了显示效率，在进行复杂图形的绘制时如何进一步提高效率呢？那就只有去掉在裁剪区外的绘图过程了。可以先用pDC->GetClipBox()得到裁剪区，然后在绘图时判断你的图形是否在这个区内，如果在就画，不在就不画。

如果你的绘图过程不复杂，这样做可能对你的绘图效率不会有提高。 

http://www.cnblogs.com/watsonlong/archive/2011/04/19/2021486.html

MFC双缓冲绘图

——————————————————————————
```C++
BOOL CDataStructureView::OnEraseBkgnd(CDC* pDC)
{
     CRect rc;
     CDC dcMem;
     GetClientRect(&rc);
     CBitmap bmp; //内存中承载临时图象的位图


     dcMem.CreateCompatibleDC(pDC); //依附窗口DC创建兼容内存DC
     //创建兼容位图(必须用pDC创建，否则画出的图形变成黑色)
     bmp.CreateCompatibleBitmap(pDC,rc.Width(),rc.Height());
     CBitmap *pOldBit=dcMem.SelectObject(&bmp);
     //按原来背景填充客户区，不然会是黑色
     dcMen.FillSolidRect(rc,RGB(255,255,255))
     
     //画图，添加你要画图的代码，不过用dcMem画，而不是pDC；
     
     ......
     
     pDC->BitBlt(0,0,rc.Width(),rc.Height(),&dcMem,0,0,SRCCOPY);
     
     //将内存DC上的图象拷贝到前台
     //绘图完成后的清理
     dcMem.DeleteDC();     //删除DC
     bmp.DeleteObject(); //删除位图
     return true;
     //这里一定要用return true，如果用自动生成的，会调用基类，把画出来的覆盖，就什     么结果也没有了
}
```
——————————————————————————

Good Luck !
MFC GDI双缓冲避免图形闪烁[转]

如何实现双缓冲
首先给出实现的程序，然后再解释，同样是在OnDraw(CDC *pDC)中：

```C++
CDC MemDC; //首先定义一个显示设备对象
CBitmap MemBitmap;//定义一个位图对象

//随后建立与屏幕显示兼容的内存显示设备
MemDC.CreateCompatibleDC(NULL);
//这时还不能绘图，因为没有地方画 ^_^
//下面建立一个与屏幕显示兼容的位图，至于位图的大小嘛，可以用窗口的大小
MemBitmap.CreateCompatibleBitmap(pDC,nWidth,nHeight);

//将位图选入到内存显示设备中
//只有选入了位图的内存显示设备才有地方绘图，画到指定的位图上
CBitmap *pOldBit=MemDC.SelectObject(&MemBitmap);

//先用背景色将位图清除干净，这里我用的是白色作为背景
//你也可以用自己应该用的颜色
MemDC.FillSolidRect(0,0,nWidth,nHeight,RGB(255,255,255));

//绘图
MemDC.MoveTo(......);
MemDC.LineTo(......);

//将内存中的图拷贝到屏幕上进行显示
pDC->BitBlt(0,0,nWidth,nHeight,&MemDC,0,0,SRCCOPY);

//绘图完成后的清理
MemBitmap.DeleteObject();
MemDC.DeleteDC();

禁止系统擦掉原来的图象
可以重载OnEraseBkgnd()函数，让其直接返回TRUE就可以了。如
BOOL CMyWin::OnEraseBkgnd(CDC* pDC)
{
　　return TRUE;
　　//return CWnd::OnEraseBkgnd(pDC);//把系统原来的这条语句注释掉。
}
```

多数人认为MFC的绘图函数效率很低，总是想寻求其它的解决方案。
MFC的绘图效率的确不高但也不差，而且它的绘图函数使用非常简单，
只要使用方法得当，再加上一些技巧，用MFC可以得到效率很高的绘图程序。
我想就我长期（呵呵当然也只有2年多）使用MFC绘图的经验谈谈
我的一些观点。
1、显示的图形为什么会闪烁？
    我们的绘图过程大多放在OnDraw或者OnPaint函数中，OnDraw在进行屏
幕显示时是由OnPaint进行调用的。当窗口由于任何原因需要重绘时，
总是先用背景色将显示区清除，然后才调用OnPaint，而背景色往往与绘图内容
反差很大，这样在短时间内背景色与显示图形的交替出现，使得显示窗口看起来
在闪。如果将背景刷设置成NULL，这样无论怎样重绘图形都不会闪了。
当然，这样做会使得窗口的显示乱成一团，因为重绘时没有背景色对原来
绘制的图形进行清除，而又叠加上了新的图形。
    有的人会说，闪烁是因为绘图的速度太慢或者显示的图形太复杂造成的，
其实这样说并不对，绘图的显示速度对闪烁的影响不是根本性的。
例如在OnDraw(CDC *pDC)中这样写：
pDC->MoveTo(0,0);
pDC->LineTo(100,100);
这个绘图过程应该是非常简单、非常快了吧，但是拉动窗口变化时还是会看见
闪烁。其实从道理上讲，画图的过程越复杂越慢闪烁应该越少，因为绘图用的
时间与用背景清除屏幕所花的时间的比例越大人对闪烁的感觉会越不明显。
比如：清楚屏幕时间为1s绘图时间也是为1s，这样在10s内的连续重画中就要闪
烁5次；如果清楚屏幕时间为1s不变，而绘图时间为9s，这样10s内的连续重画
只会闪烁一次。这个也可以试验，在OnDraw(CDC *pDC)中这样写：
for(int i=0;i<100000;i++)
{
  pDC->MoveTo(0,i);
  pDC->LineTo(1000,i);
}
呵呵，程序有点变态，但是能说明问题。
    说到这里可能又有人要说了，为什么一个简单图形看起来没有复杂图形那么
闪呢？这是因为复杂图形占的面积大，重画时造成的反差比较大，所以感觉上要
闪得厉害一些，但是闪烁频率要低。
    那为什么动画的重画频率高，而看起来却不闪？这里，我就要再次强调了，
闪烁是什么？闪烁就是反差，反差越大，闪烁越厉害。因为动画的连续两个帧之间
的差异很小所以看起来不闪。如果不信，可以在动画的每一帧中间加一张纯白的帧，
不闪才怪呢。

2、如何避免闪烁
    在知道图形显示闪烁的原因之后，对症下药就好办了。首先当然是去掉MFC
提供的背景绘制过程了。实现的方法很多，
  * 可以在窗口形成时给窗口的注册类的背景刷付NULL
  * 也可以在形成以后修改背景
static CBrush brush(RGB(255,0,0));
SetClassLong(this->m_hWnd,GCL_HBRBACKGROUND,(LONG)(HBRUSH)brush);
  * 要简单也可以重载OnEraseBkgnd(CDC* pDC)直接返回TRUE
    这样背景没有了，结果图形显示的确不闪了，但是显示也象前面所说的一样，
    变得一团乱。怎么办？这就要用到双缓存的方法了。双缓冲就是除了在屏幕上有
    图形进行显示以外，在内存中也有图形在绘制。我们可以把要显示的图形先在内存中绘制好，然后再一次性的将内存中的图形按照一个点一个点地覆盖到屏幕上去（这个过程非常快，因为是非常规整的内存拷贝）。这样在内存中绘图时，随便用什么反差大的背景色进行清除都不会闪，因为看不见。当贴到屏幕上时，因为内存中最终的图形与屏幕显示图形差别很小（如果没有运动，当然就没有差别），这样看起来就不会闪。

3、如何实现双缓冲
    首先给出实现的程序，然后再解释，同样是在OnDraw(CDC *pDC)中：

```C++
CDC MemDC; //首先定义一个显示设备对象
CBitmap MemBitmap;//定义一个位图对象
//随后建立与屏幕显示兼容的内存显示设备
MemDC.CreateCompatibleDC(NULL);
//这时还不能绘图，因为没有地方画 ^_^
//下面建立一个与屏幕显示兼容的位图，至于位图的大小嘛，可以用窗口的大小
MemBitmap.CreateCompatibleBitmap(pDC,nWidth,nHeight);
//将位图选入到内存显示设备中
//只有选入了位图的内存显示设备才有地方绘图，画到指定的位图上
CBitmap *pOldBit=MemDC.SelectObject(&MemBitmap);
//先用背景色将位图清除干净，这里我用的是白色作为背景
//你也可以用自己应该用的颜色
MemDC.FillSolidRect(0,0,nWidth,nHeight,RGB(255,255,255));
//绘图
MemDC.MoveTo(……);
MemDC.LineTo(……);
//将内存中的图拷贝到屏幕上进行显示
pDC->BitBlt(0,0,nWidth,nHeight,&MemDC,0,0,SRCCOPY);
//绘图完成后的清理
MemBitmap.DeleteObject();
MemDC.DeleteDC();
```

上面的注释应该很详尽了，废话就不多说了。

4、如何提高绘图的效率
    我主要做的是电力系统的网络图形的CAD软件，在一个窗口中往往要显示成千上万个电力元件，而每个元件又是由点、线、圆等基本图形构成。如果真要在一次重绘过程重画这么多元件，可想而知这个过程是非常漫长的。如果加上了图形的浏览功能，鼠标拖动图形滚动时需要进行大量的重绘，速度会慢得让用户将无法忍受。怎么办？只有再研究研究MFC的绘图过程了。
实际上，在OnDraw(CDC *pDC)中绘制的图并不是所有都显示了的，例如：你
在 OnDraw中画了两个矩形，在一次重绘中虽然两个矩形的绘制函数都有执行，但是很有可能只有一个显示了，这是因为MFC本身为了提高重绘的效率设置了裁剪区。裁剪区的作用就是：只有在这个区内的绘图过程才会真正有效，在区外的是无效的，即使在区外执行了绘图函数也是不会显示的。因为多数情况下窗口重绘的产生大多是因为窗口部分被遮挡或者窗口有滚动发生，改变的区域并不是整个图形而只有一小部分，这一部分需要改变的就是pDC中的裁剪区了。因为显示（往内存或者显存都叫显示）比绘图过程的计算要费时得多，有了裁剪区后显示的就只是应该显示的部分，大大提高了显示效率。但是这个裁剪区是MFC设置的，它已经为我们提高了显示效率，在进行复杂图形的绘制时如何进一步提高效率呢？那就只有去掉在裁剪区外的绘图过程了。可以先用 pDC->GetClipBox()得到裁剪区，然后在绘图时判断你的图形是否在这个区内，如果在就画，不在就不画。
如果你的绘图过程不复杂，这样做可能对你的绘图效率不会有提高。

关键字 双缓冲
原作者姓名 戚高

介绍
在论坛中经常见到关于刷新时界面闪烁的帖子,如何控制在进行高效绘图时不出现界面闪烁的感觉呢,下文就双缓冲方法进行讲解.

正文
图形为什么会闪烁的原因是:我们的绘图过程大多放在OnDraw或者OnPaint函数中，OnDraw在进行屏幕显示时是由OnPaint进行调用的。当窗口由于任何原因需要重绘时，总是先用背景色将显示区清除，然后才调用OnPaint，而背景色往往与绘图内容反差很大，这样在短时间内背景色与显示图形的交替出现，使得显示窗口看起来在闪。如果将背景刷设置成NULL，这样无论怎样重绘图形都不会闪了。当然，这样做会使得窗口的显示乱成一团，因为重绘时没有背景色对原来绘制的图形进行清除，而又叠加上了新的图形。有的人会说，闪烁是因为绘图的速度太慢或者显示的图形太复杂造成的，其实这样说并不对，绘图的显示速度对闪烁的影响不是根本性的。
如何实现双缓冲:在OnDraw(CDC *pDC)中：

```C++
      CDC MemDC; //首先定义一个显示设备对象
      CBitmap MemBitmap;//定义一个位图对象
      //随后建立与屏幕显示兼容的内存显示设备
      MemDC.CreateCompatibleDC(NULL);
      //这时还不能绘图，因为没有地方画 ^_^
      //下面建立一个与屏幕显示兼容的位图，至于位图的大小嘛，可以用窗口的大小
      MemBitmap.CreateCompatibleBitmap(pDC,nWidth,nHeight);
      //将位图选入到内存显示设备中
      //只有选入了位图的内存显示设备才有地方绘图，画到指定的位图上
      CBitmap *pOldBit=MemDC.SelectObject(&MemBitmap);
      //先用背景色将位图清除干净，这里我用的是白色作为背景
      //你也可以用自己应该用的颜色
      MemDC.FillSolidRect(0,0,nWidth,nHeight,RGB(255,255,255));
      //绘图
      MemDC.MoveTo(……);
      MemDC.LineTo(……);
      //将内存中的图拷贝到屏幕上进行显示
      pDC->BitBlt(0,0,nWidth,nHeight,&MemDC,0,0,SRCCOPY);
      //绘图完成后的清理
      MemBitmap.DeleteObject();
      MemDC.DeleteDC();
```

以论坛的一个帖子例子为例来说明一些具体如何解决问题.
帖子那容是:

我想让一个区域动起来，
如何解决窗口刷新时区域的闪烁。

```C++
void CJhkljklView::OnDraw(CDC* pDC)
{
    CJhkljklDoc* pDoc = GetDocument();
    ASSERT_VALID(pDoc);
    // TODO: add draw code for native data here
            int i;
    int x[20],y[20];
    CPen hPen;
     POINT w[5];
             x[0]=a/100+10;
             x[1]=a/100+30;
                 x[2]=a/100+80;
                 x[3]=a/100+30;
                 x[4]=a/100+10;

                     y[0]=10;
                  y[1]=10;
                   y[2]=25;
                    y[3]=40;
                     y[4]=40;   
    
      for (i=0;i<5;i++)
      {         w[i].x=x[i];
      w[i].y=y[i];
      }
      //CClientDC dc(this);
             //hPen=CreatePen(PS_SOLID,1,RGB(255,0,0));
      CRgn argn,Brgn;
        CBrush abrush(RGB(40,30,20));
        argn.CreatePolygonRgn(w, 5, 1);// point为CPoint数组，
        pDC->FillRgn(&argn, &abrush);
         abrush.DeleteObject();

}

void CJhkljklView::OnTimer(UINT nIDEvent)
{
    // TODO: Add your message handler code here and/or call default
        InvalidateRect(NULL,true);
        UpdateWindow();
        a+=100;
    CView::OnTimer(nIDEvent);
}

int CJhkljklView::OnCreate(LPCREATESTRUCT lpCreateStruct)
{
    if (CView::OnCreate(lpCreateStruct) == -1)
        return -1;
    // TODO: Add your specialized creation code here
    SetTimer(1,10,NULL);
    return 0;
}
```

利用定时器直接进行10毫秒的屏幕刷新,这样效果会出现不停的闪烁的情况.

解决方法利用双缓冲,首先触发WM_ERASEBKGND,然后修改返回TRUE;
定义变量:

```C++
CBitmap *m_pBitmapOldBackground ;
CBitmap m_bitmapBackground ;
CDC m_dcBackground;

//绘制背景
if(m_dcBackground.GetSafeHdc()== NULL|| (m_bitmapBackground.m_hObject == NULL))
    {
        m_dcBackground.CreateCompatibleDC(&dc);
        m_bitmapBackground.CreateCompatibleBitmap(&dc,rect.Width(),rect.Height()) ;
        m_pBitmapOldBackground = m_dcBackground.SelectObject(&m_bitmapBackground) ;
        //DrawMeterBackground(&m_dcBackground, rect);
        CBrush brushFill, *pBrushOld;        
        // 背景色黑色
        brushFill.DeleteObject();
        brushFill.CreateSolidBrush(RGB(255, 255, 255));
        pBrushOld = m_dcBackground.SelectObject(&brushFill);
        m_dcBackground.Rectangle(rect);
        m_dcBackground.SelectObject(pBrushOld);
    }
    memDC.BitBlt(0, 0, rect.Width(), rect.Height(),
                       &m_dcBackground, 0, 0, SRCCOPY) ;

    //绘制图形
    int i;
    int x[20],y[20];
    CPen hPen;
    POINT w[5];
    x[0]=a/100+10;
    x[1]=a/100+30;
    x[2]=a/100+80;
    x[3]=a/100+30;
    x[4]=a/100+10;
    y[0]=10;
    y[1]=10;
    y[2]=25;
    y[3]=40;
    y[4]=40;
    for (i=0;i<5;i++)
    {         w[i].x=x[i];
    w[i].y=y[i];
    }
    //CClientDC dc(this);
    //hPen=CreatePen(PS_SOLID,1,RGB(255,0,0));
    CRgn argn,Brgn;
    CBrush abrush(RGB(40,30,20));
    argn.CreatePolygonRgn(w, 5, 1);// point为CPoint数组，
    memDC.FillRgn(&argn, &abrush);
    abrush.DeleteObject();

}
```

这样编译运行程序就会出现屏幕不闪烁的情况了.
《MFC游戏开发》笔记六 图像双缓冲技术：实现一个流畅的动画

本系列文章由七十一雾央编写，转载请注明出处。

 http://blog.csdn.net/u011371356/article/details/9334121

作者：七十一雾央 新浪微博：http://weibo.com/1689160943/profile?rightmod=1&wvr=5&mod=personinfo

在前几节的笔记里，大家肯定会为一个问题感到心烦：画面怎么老是一闪一闪的啊，太难受了。确实是的，如果玩这样的游戏简直就是一种折磨。但是大家玩游戏的时候，从来没有遇到过这种情况吧？那么游戏开发者是怎么解决这个问题的呢？雾央在这一节笔记里给大家讲解一种简单通用的方法——图像双缓冲。

一、闪烁原因

为了解决问题，我们得首先搞清楚闪烁的原因是什么，然后才能对症下药。能够导致游戏画面闪烁的原因非常多，但是对于我们做游戏开发的同学来说，最主要的就是一种：贴图贴的太频繁。
    
如果大家是一个细心人的话，那么应该可以发现，在笔记三讲解贴图的时候，当我们贴出背景图的时候，是根本不会闪烁的，但是当我们贴出人物后，闪烁就出来了，而当我们移动人物的时候，闪烁的画面简直惨不忍睹啊。
    
想弄清楚真正的原因就得要理解GDI绘图的原理:GDI绘图的时候是先绘制到显存里面，然后显存每隔一段时间就需要把里面的内容输出到屏幕上，这个时间就是刷新周期。在绘图的时候，系统会先用一种背景色擦除掉原来的图像，然后再绘制新的图像。如果这几次绘制不在同一个刷新周期中，那么我们看到的就是先看到背景色，再看到内容出来，就会有闪烁的感觉，而绘制的次数越多，看到这种现象的可能性就越大，就闪烁的越厉害。

二、图像双缓冲技术

大家清楚了闪烁的原因后，再结合我们只贴出背景的时候并没有闪烁的事实，那么或许大家就可以想到一种解决方法了：我们事先将要画的所有东西画在一张图片上，然后将这张图直接贴出来，不就解决了吗？
    
如果你想到这里，那么恭喜你，你已经想到了图像双缓冲技术。其实看起来很高端的这个名词其实非常简单。我们之前画图的时候都是直接画在窗口DC上，在之前我们可以自己先创建一个内存DC，然后把画图都画在内存DC中，最后再一次性的将内存DC输出到窗口DC中，就可以解决画面闪烁的问题了。
    
下面我们讲述写代码的方法
    
1.定义变量
    
首先在CChildView.h中定义两个变量

```C++
CDC m_cacheDC;   //缓冲DC
CBitmap m_cacheCBitmap;//缓冲位图
```

2.创建缓冲DC
    
然后呢，在CChildView.cpp中OnPaint中创建缓冲DC

```C++
//创建缓冲DC
m_cacheDC.CreateCompatibleDC(NULL);
m_cacheCBitmap.CreateCompatibleBitmap(cDC,m_client.Width(),m_client.Height());
m_cacheDC.SelectObject(&m_cacheCBitmap);
```

3.在缓冲DC上绘图
    
后面贴图都贴在缓冲DC上就可以了，如

```C++
m_bg.Draw(m_cacheDC,m_client);
```

4.缓冲DC输出到窗口DC
    
最后一次性的将缓冲DC中的内容输出到窗口DC中去，函数都是之前笔记二介绍过的，不熟悉的同学请阅读笔记二。 

```C++
cDC->BitBlt(0,0,m_client.Width(),m_client.Height(),&m_cacheDC,0,0,SRCCOPY);
```

此时OnPaint函数中的内容就如同下面这样

```C++
void CChildView::OnPaint() 
{
	//获取窗口DC指针
	CDC *cDC=this->GetDC();
	//获取窗口大小
	GetClientRect(&m_client);
	//创建缓冲DC
	m_cacheDC.CreateCompatibleDC(NULL);
	m_cacheCBitmap.CreateCompatibleBitmap(cDC,m_client.Width(),m_client.Height());
	m_cacheDC.SelectObject(&m_cacheCBitmap);
	

	//————————————————————开始绘制——————————————————————
	//贴背景,现在贴图就是贴在缓冲DC：m_cache中了
	m_bg.Draw(m_cacheDC,m_client);
	//贴英雄
	MyHero.hero.Draw(m_cacheDC,MyHero.x,MyHero.y,80,80,MyHero.frame*80,MyHero.direct*80,80,80);
	//最后将缓冲DC内容输出到窗口DC中
	cDC->BitBlt(0,0,m_client.Width(),m_client.Height(),&m_cacheDC,0,0,SRCCOPY);
	
	//————————————————————绘制结束—————————————————————
	
	//在绘制完图后,使窗口区有效
	ValidateRect(&m_client);
	//释放缓冲DC
	m_cacheDC.DeleteDC();
	//释放对象
	m_cacheCBitmap.DeleteObject();
	//释放窗口DC
	ReleaseDC(cDC);

}
```

三、实现一个真正意义上的动画demo

雾央在这里实现的是一个骑着白马的少年在场景中闲逛的demo，按下WASD人物会向四个方向移动，移动的过程中动态更换图片。图片使用的是一张大图，然后每次去取其中一小块显示出来，当然大家也可以使用一张张分开好的图。具体的请看代码。
    
先来几张截图看看效果，呵呵。

头文件

```C++
// ChildView.h : CChildView 类的接口
//


#pragma once


// CChildView 窗口

class CChildView : public CWnd
{
// 构造
public:
	CChildView();

// 特性
public:
	struct shero
	{
		CImage hero;     //保存英雄的图像
		int x;             //保存英雄的位置
		int y;
		int direct;        //英雄的方向
		int frame;         //运动到第几张图片
	}MyHero;

	CRect m_client;    //保存客户区大小
	CImage m_bg;      //背景图片
	
	CDC m_cacheDC;   //缓冲DC
	CBitmap m_cacheCBitmap;//缓冲位图

// 操作
public:

// 重写
	protected:
	virtual BOOL PreCreateWindow(CREATESTRUCT& cs);

// 实现
public:
	virtual ~CChildView();

	// 生成的消息映射函数

protected:
	afx_msg void OnPaint();
	DECLARE_MESSAGE_MAP()
public:
	afx_msg void OnKeyDown(UINT nChar, UINT nRepCnt, UINT nFlags);
	afx_msg void OnLButtonDown(UINT nFlags, CPoint point);
	afx_msg void OnTimer(UINT_PTR nIDEvent);
	afx_msg int OnCreate(LPCREATESTRUCT lpCreateStruct);
};
```


CPP文件

```C++
// ChildView.cpp : CChildView 类的实现
//

#include "stdafx.h"
#include "GameMFC.h"
#include "ChildView.h"

#ifdef _DEBUG
#define new DEBUG_NEW
#endif

//定时器的名称用宏比较清楚
#define TIMER_PAINT 1
#define TIMER_HEROMOVE 2
//四个方向
#define DOWN 0
#define LEFT 1
#define RIGHT 2
#define UP 3

// CChildView

CChildView::CChildView()
{
}

CChildView::~CChildView()
{
}


BEGIN_MESSAGE_MAP(CChildView, CWnd)
	ON_WM_PAINT()
	ON_WM_KEYDOWN()
	ON_WM_LBUTTONDOWN()
	ON_WM_TIMER()
	ON_WM_CREATE()
END_MESSAGE_MAP()


//将png贴图透明
void TransparentPNG(CImage *png)
{
	for(int i = 0; i <png->GetWidth(); i++)
	{
		for(int j = 0; j <png->GetHeight(); j++)
		{
			unsigned char* pucColor = reinterpret_cast<unsigned char *>(png->GetPixelAddress(i , j));
			pucColor[0] = pucColor[0] * pucColor[3] / 255;
			pucColor[1] = pucColor[1] * pucColor[3] / 255;
			pucColor[2] = pucColor[2] * pucColor[3] / 255;
		}
	}
}

// CChildView 消息处理程序

BOOL CChildView::PreCreateWindow(CREATESTRUCT& cs) 
{
	if (!CWnd::PreCreateWindow(cs))
		return FALSE;

	cs.dwExStyle |= WS_EX_CLIENTEDGE;
	cs.style &= ~WS_BORDER;
	cs.lpszClass = AfxRegisterWndClass(CS_HREDRAW|CS_VREDRAW|CS_DBLCLKS, 
		::LoadCursor(NULL, IDC_ARROW), reinterpret_cast<HBRUSH>(COLOR_WINDOW+1), NULL);
	
	//-----------------------------------游戏数据初始化部分-------------------------
	
	//加载背景
	m_bg.Load("bg.png");
	//加载英雄图片
	MyHero.hero.Load("heroMove.png");
	TransparentPNG(&MyHero.hero);
	//初始化英雄状态
	MyHero.direct=UP;
	MyHero.frame=0;
	//设置英雄初始位置
	MyHero.x=100;    
	MyHero.y=400;
	
	return TRUE;

}

void CChildView::OnPaint() 
{
	//获取窗口DC指针
	CDC *cDC=this->GetDC();
	//获取窗口大小
	GetClientRect(&m_client);
	//创建缓冲DC
	m_cacheDC.CreateCompatibleDC(NULL);
	m_cacheCBitmap.CreateCompatibleBitmap(cDC,m_client.Width(),m_client.Height());
	m_cacheDC.SelectObject(&m_cacheCBitmap);
	

	//————————————————————开始绘制——————————————————————
	//贴背景,现在贴图就是贴在缓冲DC：m_cache中了
	m_bg.Draw(m_cacheDC,m_client);
	//贴英雄
	MyHero.hero.Draw(m_cacheDC,MyHero.x,MyHero.y,80,80,MyHero.frame*80,MyHero.direct*80,80,80);
	//最后将缓冲DC内容输出到窗口DC中
	cDC->BitBlt(0,0,m_client.Width(),m_client.Height(),&m_cacheDC,0,0,SRCCOPY);
	
	//————————————————————绘制结束—————————————————————
	
	//在绘制完图后,使窗口区有效
	ValidateRect(&m_client);
	//释放缓冲DC
	m_cacheDC.DeleteDC();
	//释放对象
	m_cacheCBitmap.DeleteObject();
	//释放窗口DC
	ReleaseDC(cDC);

}

//按键响应函数
void CChildView::OnKeyDown(UINT nChar, UINT nRepCnt, UINT nFlags)
{
	//nChar表示按下的键值
	switch(nChar)
	{
	case 'd':         //游戏中按下的键当然应该不区分大小写了
	case 'D':
		MyHero.direct=RIGHT;
		MyHero.x+=5;
		break;
	case 'a':
	case 'A':
		MyHero.direct=LEFT;
		MyHero.x-=5;
		break;
	case 'w':
	case 'W':
		MyHero.direct=UP;
		MyHero.y-=5;
		break;
	case 's':
	case 'S':
		MyHero.direct=DOWN;
		MyHero.y+=5;
		break;
	}
}

//鼠标左键单击响应函数
void CChildView::OnLButtonDown(UINT nFlags, CPoint point)
{
	char bufPos[50];
	sprintf(bufPos,"你单击了点X:%d,Y:%d",point.x,point.y);
	AfxMessageBox(bufPos);
}

//定时器响应函数
void CChildView::OnTimer(UINT_PTR nIDEvent)
{
	

	switch(nIDEvent)
	{
	case TIMER_PAINT:OnPaint();break;  //若是重绘定时器，就执行OnPaint函数
	case TIMER_HEROMOVE:               //控制人物移动的定时器
		{
			MyHero.frame++;              //每次到了间隔时间就将图片换为下一帧
			if(MyHero.frame==4)          //到最后了再重头开始
				MyHero.frame=0;
		}
		break;
	}

}


int CChildView::OnCreate(LPCREATESTRUCT lpCreateStruct)
{
	if (CWnd::OnCreate(lpCreateStruct) == -1)
		return -1;

	// TODO:  在此添加您专用的创建代码
	
	//创建一个10毫秒产生一次消息的定时器
	SetTimer(TIMER_PAINT,10,NULL);
	//创建人物行走动画定时器
	SetTimer(TIMER_HEROMOVE,100,NULL);
	return 0;

}
```

