---
layout: post
title: 贝塞尔样条曲线
date:  2015-03-19 10:39:36  
category: 数学
tags: 贝塞尔
---

很长时间没写博客了，每次写篇博客可能都要花费我一个小时左右，感觉有点费时间，但是不管怎么样，还是尽量抽出时间保证每月三篇的量吧。

今天在学习Windows上编程的时候学到了一个贝塞尔样条曲线觉得比较有趣。虽然我是想以后找工作的方向是Linux后台服务器开发的，但是还是快速的把Windows上面的编程过一遍吧，Windows比较好理解一点，Linux比较酷一点: )

好的。 还是回到贝塞尔样条曲线上来。

##用处

据说这东西很有用，在计算机图形程序设计中的广泛程度仅次于矩形，椭圆等。PostScript字体的字符轮廓线就是用贝寒尔样条曲线来定义的，这些就不说了，百度都能搜到的。

##样子

它长的什么样呢？来先看看这幅图建立感性认识 - -

![bezier](http://shamospace.qiniudn.com/bezier.jpg)

##细说

贝塞尔样条曲线需要用四个点来确定它的形状，A(x0, y0), B(x1, y1), C(x2, y2), D(x3, y3). 

我们把A和D称为起点和终点，B和C称为控点。看看它的公式：（这个我还没用会怎么用Markdown写数学公式，先留一留 。）

贝塞尔样条曲线的如下几个特征使得它在计算机辅助设计中非常有用。

首先，只要经过少量的练习，通常就可以自由操纵曲线，画出与预期形状非常接近的曲线。

其次，贝塞尔样条曲线非常易于控制。对一些样条产曲线公式而言，它们代表的曲线不经过任何一个定义该曲线的点。而贝塞尔曲线总是固定在两个端点之间（因为这是用于推导贝塞尔公式时所做的假设之一）。而且，一些形式的样条曲线公式有奇点，在这些点曲线趋于无穷远，这在计算机辅助设计中是不希望出现的。贝塞尔样条曲线则从来没有这种奇点；事实上，贝塞尔样条曲线总是位于一个由端点和控点连接而成的四边形（称为“凸包”）之内。

然后，贝塞尔样条曲线的国外一个特性是端点和控点之间的关系。曲线总是和从起点到最后一个控点绘制的直线相切，并保持同一方向。同时，也和从第二个控点到终点绘制的直线相切，并保持同一方向。这是用于推导贝塞尔公式的另外两个假设。

最后，据说贝塞尔样条曲线通常是很有美感的，这种主观的标准…… ; )

##程序

~~~~~~

#include <windows.h>

LRESULT CALLBACK WndProc(HWND, UINT, WPARAM, LPARAM);

int WINAPI WinMain(HINSTANCE hInstance, HINSTANCE hPrevInstance, PSTR szCmdLine, int iCmdShow)
{
	static TCHAR szAppName[] = TEXT("Bezier");
	HWND hwnd;
	MSG msg;
	WNDCLASS wndclass;

	wndclass.style = CS_HREDRAW | CS_VREDRAW;
	wndclass.lpfnWndProc = WndProc;
	wndclass.cbClsExtra = 0;
	wndclass.cbWndExtra = 0;
	wndclass.hInstance = hInstance;
	wndclass.hIcon = LoadIcon(NULL, IDI_APPLICATION);
	wndclass.hCursor = LoadCursor(NULL, IDC_ARROW);
	wndclass.hbrBackground = (HBRUSH)GetStockObject(WHITE_BRUSH);
	wndclass.lpszMenuName = NULL;
	wndclass.lpszClassName = szAppName;

	if (!RegisterClass(&wndclass)){
		MessageBox(NULL, TEXT("Program requires Windows NT!"), szAppName, MB_ICONERROR);
		return 0;
	}

	hwnd = CreateWindow(szAppName, 
						TEXT("Bezier Splines"),
						WS_OVERLAPPEDWINDOW,
						CW_USEDEFAULT,
						CW_USEDEFAULT,
						CW_USEDEFAULT,
						CW_USEDEFAULT,
						NULL,
						NULL,
						hInstance, 
						NULL);

	ShowWindow(hwnd, iCmdShow);
	UpdateWindow(hwnd);

	while (GetMessage(&msg, NULL, 0, 0)){
		TranslateMessage(&msg);		
		DispatchMessage(&msg);
	}

	return msg.wParam;
}

void DrawBezier(HDC hdc, POINT apt[])
{
	PolyBezier(hdc, apt, 4);

	MoveToEx(hdc, apt[0].x, apt[0].y, NULL);
	LineTo(hdc, apt[1].x, apt[1].y);

	MoveToEx(hdc, apt[2].x, apt[2].y, NULL);
	LineTo(hdc, apt[3].x, apt[3].y);
}


LRESULT CALLBACK WndProc(HWND hwnd, UINT message, WPARAM wParam, LPARAM lParam)
{
	static POINT apt[4];
	static int cxClient, cyClient;
	HDC hdc;
	PAINTSTRUCT ps;

	switch (message){
		case WM_SIZE:
			cxClient = LOWORD(lParam);
			cyClient = HIWORD(lParam);

			apt[0].x = cxClient/4;
			apt[0].y = cyClient/2;

			apt[1].x = cxClient/2;
			apt[1].y = cyClient/4;

			apt[2].x = cxClient/2;
			apt[2].y = 3*cyClient/4;

			apt[3].x = 3*cxClient/4;
			apt[3].y = cyClient/2;
			return 0;

		case WM_LBUTTONDOWN:
		case WM_RBUTTONDOWN:
		case WM_MOUSEMOVE:
			if (wParam & MK_LBUTTON || wParam & MK_RBUTTON){
				hdc = GetDC(hwnd);

				SelectObject(hdc, GetStockObject(WHITE_PEN));
				DrawBezier(hdc, apt);

				if (wParam & MK_LBUTTON){
					apt[1].x = LOWORD(lParam);
					apt[1].y = HIWORD(lParam);
				}

				if (wParam & MK_RBUTTON){
					apt[2].x = LOWORD(lParam);
					apt[2].y = HIWORD(lParam);
				}

				SelectObject(hdc, GetStockObject(BLACK_PEN));
				DrawBezier(hdc, apt);
				ReleaseDC(hwnd, hdc);
			}
			return 0;

		case WM_PAINT:
			InvalidateRect(hwnd, NULL, TRUE);

			hdc = BeginPaint(hwnd, &ps);

			DrawBezier(hdc, apt);

			EndPaint(hwnd, &ps);
			return 0;

		case WM_DESTROY:
			PostQuitMessage(0);
			return 0;
	}

	return DefWindowProc(hwnd, message, wParam, lParam);
}


~~~~~~























