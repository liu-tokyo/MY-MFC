#  CListCtrl的Report风格自绘

> https://www.cnblogs.com/huhu0013/p/4618551.html

原文链接: http://jingyan.baidu.com/article/5bbb5a1b38af1113eaa17910.html

CListCtrl是MFC中运用最广泛的控件之一，很多软件都有CListCtrl的身影，但是对于CListCtrl的自绘，很多朋友都犯了难，网上虽然有很多人讲解怎么自绘，但是实现出的效果都不是太友好，本篇讲解CListCtrl的Report自绘，对于LVS_ICON风格，我们采用窗口模拟进行绘制。先说一下Report重绘的注意事项。


方法/步骤
    
CListCtrl控件分为上下两部分，上面的标头位置为标头控件，下方是我们需要绘制的CListCtrl，标头控件我们需要继承CHeaderCtrl进行标头控件的重绘，然后在CListCtrl中我们子类化CHeaderCtrl控件

CHeaderCtrl控件高度设置，有两种方法
    
1：通过SetFont强制撑大控件的高度

//创建字体

CFont Font;

Font.CreatePointFont(m_uItemHeight,TEXT("宋体"));

//设置字体

SetFont(&Font);

2：通过HDM_LAYOUT消息修改高度，添加ON_MESSAGE(HDM_LAYOUT, OnLayout)

LRESULT CSkinHeaderCtrl::OnLayout( WPARAM wParam, LPARAM lParam )

{

     LRESULT lResult = CHeaderCtrl::DefWindowProc(HDM_LAYOUT, 0, lParam); 
    
     HD_LAYOUT &hdl = *( HD_LAYOUT * ) lParam; 
    
     RECT *prc = hdl.prc; 
    
     WINDOWPOS *pwpos = hdl.pwpos; 
    
     int nHeight = 28;
    
     pwpos->cy = nHeight; 
    
     prc->top = nHeight; 
    
     return lResult; 

}
    
    上述代码中设置了控件的高度为28

重绘CListCtrl，先将Owner Draw Fixed设为true或者Create的时候添加LVS_OWNERDRAWFIXED属性，目的就是让控件能响应DrawItem从而实现自绘，此处需要注意，对于LVS_ICON风格，DrawItem不会被系统调用，不管是否添加LVS_OWNERDRAWFIXED属性，当然可以通过在WM_PAINT内进行绘制，不过这样会给Report带来诸多的麻烦，所以，我们通过模拟来实现LVS_ICON的绘制
    
CListCtrl高度的设置，因为CListCtrl并没有诸如SetItemHeight之类的函数进行节点高度的设置，所以我们必须自己实现这样的功能

void CSkinListCtrl::SetItemHeight( int nHeight )

{

      m_nHeightItem = nHeight;
    
      CRect rcWin;
    
      GetWindowRect(&rcWin);
    
      WINDOWPOS wp;
    
      wp.hwnd = m_hWnd;
    
      wp.cx = rcWin.Width();
    
      wp.cy = rcWin.Height();
    
      wp.flags = SWP_NOACTIVATE | SWP_NOMOVE | SWP_NOOWNERZORDER | SWP_NOZORDER;
    
      SendMessage(WM_WINDOWPOSCHANGED, 0, (LPARAM)&wp);

}

同时添加ON_WM_MEASUREITEM_REFLECT消息反射

void CSkinListCtrl::MeasureItem( LPMEASUREITEMSTRUCT lpMeasureItemStruct )

{

      if (m_nHeightItem>0)
    
      {
    
            lpMeasureItemStruct->itemHeight = m_nHeightItem;
    
      }

}

这里提到了消息发射，注意区分和消息映射的区别，关于消息反射的概念不做赘述，可以通过MSDN进行查阅

5 CListCtrl为我们提供了SetExtendedStyle函数可以添加CListCtrl的很多扩展属性，这里强调一点LVS_EX_CHECKBOXES会导致ON_WM_MEASUREITEM_REFLECT消息反射不被接收，LVS_EX_GRIDLINES会造成节点矩形不准确，所以，我们需要重载屏蔽这些属性

DWORD CSkinListCtrl::SetExtendedStyle( DWORD dwNewStyle )

{

      if ( dwNewStyle & LVS_EX_CHECKBOXES )
    
      {
    
            dwNewStyle &=~LVS_EX_CHECKBOXES;
    
            dwNewStyle &=~LVS_EX_GRIDLINES;
    
      }


​     

      return __super::SetExtendedStyle(dwNewStyle);

}

6 Check设置，因为我们上面屏蔽了LVS_EX_CHECKBOXES，所以默认情况下，当我们插入节点的时候，默认是被Check的，所以， 还需要重载四个InsertItem函数在里面默认将Check取消掉，例如

int CSkinListCtrl::InsertItem( const LVITEM* pItem )

{

      int nResult = __super::InsertItem(pItem);
    
      SetCheck(pItem->iItem,FALSE);
    
      return nResult;

}

7 关于标头控件节点的拖动，当我们将鼠标放在两个节点的交叉处，鼠标会变成拖动的样式，此时按住左键可以拉伸节点的宽度，但是很多时候我们不想让前面的几个进行拖动，这个时候，我们需要在CHeaderCtrl派生类中重载OnChildNotify函数

BOOL CSkinHeaderCtrl::OnChildNotify(UINT uMessage, WPARAM wParam, LPARAM lParam, LRESULT * pLResult)

{

      //变量定义
    
      NMHEADER * pNMHearder=(NMHEADER*)lParam;


​     

      //拖动消息
    
      if ((pNMHearder->hdr.code==HDN_BEGINTRACKA)||(pNMHearder->hdr.code==HDN_BEGINTRACKW))
    
      {
    
            //禁止拖动
    
            if (pNMHearder->iItem<(INT)m_uLockCount)
    
            {
    
                  *pLResult=TRUE;
    
                  return TRUE;
    
            }
    
      }


​     

      return __super::OnChildNotify(uMessage,wParam,lParam,pLResult);

}
    
    其中m_uLockCount就是禁止拖动的个数，比如设置为2，则前面2个节点将不能拉伸宽度，我们可以在写一个方法
    
    //设置锁定
    
    VOID CSkinHeaderCtrl::SetLockCount(UINT uLockCount)
    
    {
    
          //设置变量
    
          m_uLockCount=uLockCount;
    
          return;
    
    }


​     
​    END

绘制的步骤

    1
    
    创建CHeaderCtrl的派生类CSkinHeaderCtrl，现在主要看一下OnPaint的绘制
    
    //重画函数
    
    VOID CSkinHeaderCtrl::OnPaint() 
    
    {
    
          CPaintDC dc(this);


​     

          //获取位置
    
          CRect rcRect;
    
          GetClientRect(&rcRect);
    
         //双缓冲，内存DC  
    
          CMemoryDC BufferDC(&dc,rcRect);


​     

          //设置 DC
    
          BufferDC.SetBkMode(TRANSPARENT);
    
          BufferDC.SetTextColor(m_colNormalText);
    
          BufferDC.SelectObject(GetCtrlFont());


​     

          //绘画背景
    
          if (m_pBackImg != NULL && !m_pBackImg->IsNull())
    
                m_pBackImg->Draw(&BufferDC,rcRect);


​     

          if (m_pPressImg != NULL && !m_pPressImg->IsNull() && m_bPress)
    
          {
    
                CRect rcItem;
    
                GetItemRect(m_uActiveItem,&rcItem);
    
                m_pPressImg->Draw(&BufferDC,rcItem);
    
          }


​     

          //绘画子项
    
          CRect rcItem;
    
          HDITEM HDItem;
    
          TCHAR szBuffer[64];
    
          for (INT i=0;i<GetItemCount();i++)
    
          {
    
                //构造变量
    
                HDItem.mask=HDI_TEXT;
    
                HDItem.pszText=szBuffer;
    
                HDItem.cchTextMax=CountArray(szBuffer);


​     

                //获取信息
    
                GetItem(i,&HDItem);
    
                GetItemRect(i,&rcItem);


​     

                if (m_pGridImg != NULL && !m_pGridImg->IsNull())
    
                      m_pGridImg->DrawImage(&BufferDC,(rcItem.right-m_pGridImg->GetWidth()),(rcItem.Height()-m_pGridImg->GetHeight())/2);


​     

                //绘画标题
    
                rcItem.DeflateRect(3,1,3,1);
    
                   BufferDC.DrawText(szBuffer,lstrlen(szBuffer),&rcItem,DT_END_ELLIPSIS|DT_SINGLELINE|DT_VCENTER|DT_CENTER);
    
          }


​     

          return;
    
    }
    2
    
    鼠标按下
    
    void CSkinHeaderCtrl::OnLButtonDown( UINT nFlags, CPoint point )
    
    {
    
          CRect rcItem;


​     

          for (INT i=0;i<GetItemCount();i++)
    
          {
    
                GetItemRect(i,&rcItem);


​     

                if ( PtInRect(&rcItem,point) )
    
                {
    
                      m_bPress = true;
    
                      m_uActiveItem = i;
    
                      break;
    
                }
    
          }


​     

          RedrawWindow(NULL,NULL,RDW_FRAME|RDW_INVALIDATE|RDW_ERASE|RDW_ERASENOW);


​     

          __super::OnLButtonDown(nFlags, point);
    
    }


​     
​    3
​    
​    鼠标抬起
​    
    void CSkinHeaderCtrl::OnLButtonUp( UINT nFlags, CPoint point )
    
    {
    
          m_bPress = false;


​     

          RedrawWindow(NULL,NULL,RDW_FRAME|RDW_INVALIDATE|RDW_ERASE|RDW_ERASENOW);


​     

          __super::OnLButtonUp(nFlags, point);
    
    }


​     
​    4
​    
​    新建CListCtrl的派生类CSkinListCtrl
​    
    struct  tagItemImage 
    
    {
    
          CImageEx *pImage;
    
          int nItem;
    
    };


​     

    typedef vector<tagItemImage> CItemImgArray;


​     

    重点看一下绘制
    5
    
    //绘画函数
    
    VOID CSkinListCtrl::DrawItem(LPDRAWITEMSTRUCT lpDrawItemStruct)
    
    {
    
          //变量定义
    
          CRect rcItem=lpDrawItemStruct->rcItem;
    
          CDC * pDC=CDC::FromHandle(lpDrawItemStruct->hDC);


​     

          CMemoryDC BufferDC(pDC,rcItem);


​     

          //获取属性
    
          INT nItemID=lpDrawItemStruct->itemID;
    
          INT nColumnCount=m_SkinHeaderCtrl.GetItemCount();


​     

          //绘画区域
    
          CRect rcClipBox;
    
          BufferDC.GetClipBox(&rcClipBox);


​     

          //设置环境
    
          BufferDC.SetBkMode(TRANSPARENT);
    
          BufferDC.SetTextColor(m_colNormalText);
    
          BufferDC.SelectObject(GetCtrlFont());
    
          BufferDC->FillSolidRect(&rcItem,m_colBack);


​     

          //绘画焦点
    
          if (lpDrawItemStruct->itemState&ODS_SELECTED)
    
          {
    
                if (m_pSelectImg != NULL && !m_pSelectImg->IsNull())
    
                      m_pSelectImg->Draw(&BufferDC,rcItem);
    
          }
    
          else if ( m_uActiveItem == nItemID )
    
          {
    
                if (m_pHovenImg != NULL && !m_pHovenImg->IsNull())
    
                      m_pHovenImg->Draw(&BufferDC,rcItem);
    
          }


​     

          //绘画子项
    
          for (INT i=0;i<nColumnCount;i++)
    
          {
    
                //获取位置
    
                CRect rcSubItem;
    
                GetSubItemRect(nItemID,i,LVIR_BOUNDS,rcSubItem);


​     

                //绘画判断
    
                if (rcSubItem.left>rcClipBox.right) break;
    
                if (rcSubItem.right<rcClipBox.left) continue;


​     

                //绘画数据
    
                DrawReportItem(&BufferDC,nItemID,rcSubItem,i);
    
          }


​     

          return;
    
    }


​     
​    6
​    
​    //绘画数据
​    
    VOID CSkinListCtrl::DrawReportItem(CDC * pDC, INT nItem, CRect & rcSubItem, INT nColumnIndex)
    
    {
    
          //获取文字
    
          TCHAR szString[256]=TEXT("");
    
          GetItemText(nItem,nColumnIndex,szString,CountArray(szString));


​     

          //绘画文字
    
          rcSubItem.left+=5;


​     

          //绘制CheckButton
    
          if( nColumnIndex == 0 )
    
          {
    
                if ((m_pCheckImg != NULL && !m_pCheckImg->IsNull()) && (m_pUnCheckImg != NULL && !m_pUnCheckImg->IsNull()))
    
                {
    
                      if( GetCheck(nItem) )
    
                            m_pCheckImg->DrawImage(pDC,rcSubItem.left+2,rcSubItem.top+(rcSubItem.Height()-m_pCheckImg->GetHeight())/2);
    
                      else
    
                            m_pUnCheckImg->DrawImage(pDC,rcSubItem.left+2,rcSubItem.top+(rcSubItem.Height()-m_pUnCheckImg->GetHeight())/2);


​     

                      rcSubItem.left+=(8+m_pCheckImg->GetWidth());
    
                }


​     

                CItemImgArray::iterator iter = m_ItemImgArray.begin();


​     

                for (;iter != m_ItemImgArray.end(); ++iter )
    
                {
    
                      if ( iter->nItem == nItem )
    
                      {
    
                            CImageEx *pImage = iter->pImage;


​     

                            if (pImage != NULL && !pImage->IsNull())
    
                            {
    
                                  pImage->DrawImage(pDC,rcSubItem.left+2,rcSubItem.top+(rcSubItem.Height()-pImage->GetHeight())/2);
    
                                  rcSubItem.left+=(8+pImage->GetWidth());
    
                            }
    
                            break;
    
                      }
    
                }
    
          }


​     

          pDC->DrawText(szString,lstrlen(szString),&rcSubItem,DT_VCENTER|DT_SINGLELINE|DT_END_ELLIPSIS);


​     

          return;
    
    }


​     
​    7
​    
​    这里说一下节点的图标，因为每个节点的图标都可能不一样，而且，节点的图标随时可能因为其他的需要发生更换，所以这里我们采用vector进行图标的管理，
​    
    BOOL CSkinListCtrl::InsertImage( int nItem,LPCTSTR lpszFileName )
    
    {
    
          CItemImgArray::iterator iter = m_ItemImgArray.begin();


​     

          for (;iter != m_ItemImgArray.end(); ++iter )
    
          {
    
                if ( iter->nItem == nItem )
    
                {
    
                      //当该节点存在图片时，我们更新新的图片资源
    
                      if( iter->pImage != NULL )
    
                      {
    
                            RenderEngine->RemoveImage(iter->pImage);


​     

                            iter->pImage = RenderEngine->GetImage(lpszFileName);


​     

                            return TRUE;
    
                      }
    
                }
    
          }


​     

          tagItemImage ItemImage;


​     

          //设置数据
    
          ItemImage.nItem = nItem;
    
          ItemImage.pImage = RenderEngine->GetImage(lpszFileName);


​     

          if (NULL == ItemImage.pImage)
    
                return FALSE;
    
          else
    
          {
    
                m_ItemImgArray.push_back(ItemImage);


​     

                return TRUE;
    
          }
    
    }


​     
​    8
​    
​    void CSkinListCtrl::OnMouseMove( UINT nFlags, CPoint point )
​    
    {
    
          CRect rcItem;


​     

          static UINT uOldActiveItem = -1;


​     

          for (int i=0;i<GetItemCount();i++)
    
          {
    
                GetItemRect(i,rcItem,LVIR_BOUNDS);


​     

                if ( PtInRect(&rcItem,point) )
    
                {
    
                      m_uActiveItem = i;


​     

                      if( uOldActiveItem != m_uActiveItem )
    
                      {
    
                            uOldActiveItem = m_uActiveItem;


​     

                            Invalidate(FALSE);
    
                      }


​     

                      break;
    
                }
    
          }


​     

          __super::OnMouseMove(nFlags, point);
    
    }
    
    这里注意，我们设置了一个静态变量来保存之前鼠标移动过的节点，此变量的意义很简单，因为无法保证用户是不是在同一个节点晃动鼠标，如果只要监听到鼠标移动消息就进行刷新，这样的话会消耗过多的cpu资源
    9
    
    void CSkinListCtrl::OnLButtonDown( UINT nFlags, CPoint point )
    
    {
    
          if (m_pCheckImg != NULL && !m_pCheckImg->IsNull())
    
          {
    
                CRect rcSubItem,rcIcon;
    
                for (int i=0;i<GetItemCount();i++)
    
                {
    
                      GetItemRect(i,rcSubItem,LVIR_BOUNDS);


​     

                      rcIcon.left = rcSubItem.left+7;
    
                      rcIcon.top = rcSubItem.top+(rcSubItem.Height()-m_pCheckImg->GetHeight())/2;
    
                      rcIcon.right = rcIcon.left + m_pCheckImg->GetWidth();
    
                      rcIcon.bottom = rcIcon.top + m_pCheckImg->GetHeight();


​     

                  if ( PtInRect(&rcIcon,point) )
    
                  {
    
                        SetCheck(i,!GetCheck(i));


​     

                        SetItemState(i, LVIS_FOCUSED | LVIS_SELECTED,LVIS_FOCUSED | LVIS_SELECTED);
    
                        SetSelectionMark(i);


​     

                        Invalidate(FALSE);
    
                        break;
    
                  }
    
            }
    
      }


​     

​     

      __super::OnLButtonDown(nFlags, point);

}


​     
10

关于Report的自绘就完成了，看一下调用过程
END

调用过程

1

m_ListCtrl5.m_SkinHeaderCtrl.SetBackImage(TEXT("Res\\ListCtrl\\folder_nav_item_bg_hover.png"),&CRect(2,2,2,2));

m_ListCtrl5.m_SkinHeaderCtrl.SetPressImage(TEXT("Res\\ListCtrl\\folder_nav_item_bg_pressed.png"),&CRect(2,2,2,2));

m_ListCtrl5.m_SkinHeaderCtrl.SetGridImage(TEXT("Res\\ListCtrl\\category_sep.png"));

m_ListCtrl5.SetHovenImage(TEXT("Res\\ListCtrl\\item_bg_hover.png"),&CRect(2,2,2,2));

m_ListCtrl5.SetSelectImage(TEXT("Res\\ListCtrl\\item_bg_selected.png"),&CRect(2,2,2,2));

m_ListCtrl5.SetCheckImage(TEXT("Res\\ListCtrl\\check.png"),TEXT("Res\\ListCtrl\\uncheck.png"));

m_ListCtrl5.SetScrollImage(&m_ListCtrl5,TEXT("Res\\Scroll\\SKIN_SCROLL.bmp"));


​     

m_ListCtrl5.InsertColumn( 0, TEXT("文件名"), LVCFMT_LEFT, 150 );

m_ListCtrl5.InsertColumn( 1, TEXT("大小"), LVCFMT_LEFT, 100 );

m_ListCtrl5.InsertColumn( 2, TEXT("修改时间"), LVCFMT_LEFT, 150 );


​     

for (int i=0;i<8;i++)

{

      m_ListCtrl5.InsertItem(i,TEXT("跟我学MFC.zip"));
    
      m_ListCtrl5.SetItemText(i,1,TEXT("126MB"));
    
      m_ListCtrl5.SetItemText(i,2,TEXT("2013-10-17 18:13"));

}


​     

m_ListCtrl5.InsertImage(0,TEXT("Res\\ListCtrl\\DocType.png"));

m_ListCtrl5.InsertImage(1,TEXT("Res\\ListCtrl\\FolderType.png"));

m_ListCtrl5.InsertImage(2,TEXT("Res\\ListCtrl\\ImgType.png"));

m_ListCtrl5.InsertImage(3,TEXT("Res\\ListCtrl\\PdfType.png"));

m_ListCtrl5.InsertImage(4,TEXT("Res\\ListCtrl\\PptType.png"));

m_ListCtrl5.InsertImage(5,TEXT("Res\\ListCtrl\\RarType.png"));

//m_ListCtrl5.InsertImage(6,TEXT("Res\\ListCtrl\\VsdType.png"));

m_ListCtrl5.InsertImage(7,TEXT("Res\\ListCtrl\\VideoType.png"));


​     

//更新资源图片

m_ListCtrl5.InsertImage(0,TEXT("Res\\ListCtrl\\VideoType.png"));


​     

m_ListCtrl5.SetItemHeight(30);


​     

//标头控件禁止拖动

//m_ListCtrl6.m_SkinHeaderCtrl.EnableWindow(FALSE);

//让第一个节点禁止拖动

m_ListCtrl6.m_SkinHeaderCtrl.SetLockCount(1);


​     