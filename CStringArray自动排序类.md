# CStringArray自动排序类

## 创建CSortStringArray类

从CStringArray继承，对CStringArray中的字符串数组，增加排序功能。

- **SortStringArray.h**

  ```C++
  // SortStringArray.h: interface for the CSortStringArray class.
  //
  ///////////////////////////////////////////////////////////////////////////////
  
  #if !defined(AFX_SORTSTRINGARRAY_H__ABE7EF30_22B1_4643_BBF7_47CED2B75E78__INCLUDED_)
  #define AFX_SORTSTRINGARRAY_H__ABE7EF30_22B1_4643_BBF7_47CED2B75E78__INCLUDED_
  
  #if _MSC_VER > 1000
  #pragma once
  #endif // _MSC_VER > 1000
  ///////////////////////////////////////////////////////////////////////////////
  
  class CSortStringArray : public CStringArray
  {
  public:
  	CSortStringArray();
  	virtual ~CSortStringArray();
  
  public:
  	void Sort();
  private:
  	BOOL CompareAndSwap(int pos);
  };
  
  ///////////////////////////////////////////////////////////////////////////////
  #endif // !defined(AFX_SORTSTRINGARRAY_H__ABE7EF30_22B1_4643_BBF7_47CED2B75E78__INCLUDED_)
  ```

- **SortStringArray.cpp**

  ```C++
  // SortStringArray.cpp: implementation of the CSortStringArray class.
  //
  ///////////////////////////////////////////////////////////////////////////////
  
  #include "stdafx.h"
  #include "SortStringArray.h"									// 字符串排序类
  ///////////////////////////////////////////////////////////////////////////////
  
  #ifdef _DEBUG
  #undef THIS_FILE
  static char THIS_FILE[]=__FILE__;
  #define new DEBUG_NEW
  #endif
  
  ///////////////////////////////////////////////////////////////////////////////
  // Construction/Destruction
  ///////////////////////////////////////////////////////////////////////////////
  
  CSortStringArray::CSortStringArray()
  {
  
  }
  
  CSortStringArray::~CSortStringArray()
  {
  
  }
  
  void CSortStringArray::Sort()
  {
  	BOOL bNotDone = TRUE;
  
  	while (bNotDone)
  	{
  		bNotDone = FALSE;
  		for(int pos = 0;pos < GetUpperBound();pos++)
  			bNotDone |= CompareAndSwap(pos);
  	}
  }
  
  BOOL CSortStringArray::CompareAndSwap(int pos)
  {
  	CString temp;
  	int posFirst = pos;
  	int posNext = pos + 1;
  
  	if (GetAt(posFirst).CompareNoCase(GetAt(posNext)) > 0)
  	{
  		temp = GetAt(posFirst);
  		SetAt(posFirst, GetAt(posNext));
  		SetAt(posNext, temp);
  
  		return TRUE;
  	}
  
  	return FALSE;
  }
  ```

  

## 字符串排序类测试

- Console

  ```C++
  ...
  #include <iostream>
  using namespace std;
  ...
  	CSortStringArray sortArray;
  
  	sortArray.Add(CString("Zebra"));
  	sortArray.Add(CString("Bat"));
  	sortArray.Add(CString("Apple"));
  	sortArray.Add(CString("Mango"));
  
  	for (int i = 0; i <= sortArray.GetUpperBound(); i++) {
  		cout << sortArray[i] << endl;
  	}
  
  	sortArray.Sort();
  	cout << endl;
  
  	for (int j = 0; j <= sortArray.GetUpperBound(); j++) {
  		cout << sortArray[j] << endl;
  	}
  ```

- Dialog

  ```C++
  ...
  	CSortStringArray sortArray;
  
  	sortArray.Add(CString("Zebra"));
  	sortArray.Add(CString("Bat"));
  	sortArray.Add(CString("Apple"));
  	sortArray.Add(CString("Mango"));
  
  	for (int i = 0; i <= sortArray.GetUpperBound(); i++) {
  		TRACE(sortArray[i]);
  		TRACE("\n");
  	}
  
  	sortArray.Sort();
  	TRACE("\n");
  
  	for (int j = 0; j <= sortArray.GetUpperBound(); j++) {
  		TRACE(sortArray[j]);
  		TRACE("\n");
  	}
  ...
  ```

  

## 参照资料

- 微软官方代码  
  https://jeffpar.github.io/kbarchive/kb/120/Q120961/
  
  ```C++
  MORE INFORMATION
  ================
  Sample Code
  -----------
  
    /*
     * Compile options needed: /MT
     */ 
  
    #include <afx.h>
    #include <iostream.h>
    #include <afxcoll.h>
  
    class CSortStringArray : public CStringArray {
    public:
       void Sort();
    private:
       BOOL CompareAndSwap(int pos);
    };
    void CSortStringArray::Sort()
    {
       BOOL bNotDone = TRUE;
  
       while (bNotDone)
       {
          bNotDone = FALSE;
          for(int pos = 0;pos < GetUpperBound();pos++)
             bNotDone |= CompareAndSwap(pos);
       }
    }
    BOOL CSortStringArray::CompareAndSwap(int pos)
    {
       CString temp;
       int posFirst = pos;
       int posNext = pos + 1;
  
       if (GetAt(posFirst).CompareNoCase(GetAt(posNext)) > 0)
       {
          temp = GetAt(posFirst);
          SetAt(posFirst, GetAt(posNext));
          SetAt(posNext, temp);
          return TRUE;
  
       }
       return FALSE;
    }
    void main()
    {
       CSortStringArray sortArray;
  
       sortArray.Add(CString("Zebra"));
       sortArray.Add(CString("Bat"));
       sortArray.Add(CString("Apple"));
       sortArray.Add(CString("Mango"));
  
       for (int i = 0; i <= sortArray.GetUpperBound(); i++)
          cout << sortArray[i] << endl;
  
       sortArray.Sort();
       cout << endl;
  
       for (int j = 0; j <= sortArray.GetUpperBound(); j++)
          cout << sortArray[j] << endl;
    }
  
  Additional query words: kbinf 7.00 1.00 1.50 2.00 2.10 2.50 3.00 4.00 kbNoUpdate
  
  ======================================================================
  Keywords          : kbcode kbnokeyword kbMFC kbVC100 kbVC150 kbVC200 kbVC400 kbGrpDSMFCATL 
  Technology        : kbAudDeveloper kbMFC
  Version           : winnt:1.0,2.0,2.1,4.0
  Issue type        : kbhowto
  =============================================================================
  ```
  
  
