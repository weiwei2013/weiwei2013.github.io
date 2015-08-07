---
layout: post
title: Run Loop小结
description: "How to Draw VN flag in OpenGL?"
modified: 2015-08-07
tags: [OpenGL, cpp]
---

<img src="http://i.imgur.com/cfxb4wY.gif?1"><br>
code here:<br>
<b>首先说一下什么是RunLoop？</b><br>
<b>Run Loop 本身听起来就和它本身的名字很像。它是一个循环，你的线程进入并使用它来运行响应输入事件的事件处理程序。你的代码要提供实现循环部分的控制语句，换言之就是要有while或for循环语句来驱动run loop。在你的循环中，使用Run loop object 来运行事件</b>
<b>那么在实际的开发中，简单的线程任务是不建议手动创建线程来实现的，因为手动创建并管理线程的生命周期比较麻烦，这么麻烦肿么办内，表怕，咱就用系统提供的一些异步方法(performSelectorInBackground:<#(SEL)#> withObject:<#(id)#>: 等)，或者是OperationQueue 后者是 Dispatch Queues等来实现。而只有当持续的异步任务需求时，我们才创建一个独立的生命周期可控的线程，而Run Loop就是控制线程生命周期并接收事件进行处理的机制。</b>
<br>
<b>接下来 看一下介个Run Loop的官方定义:</b>
<br>
<p><code>Run loops are part of the fundamental infrastructure associated with threads. A run loop is an event processing loop that you use to schedule work and coordinate the receipt of incoming events. The purpose of a run loop is to keep your thread busy when there is work to do and put your thread to sleep when there is none.</code></p>
<img src="https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/Multithreading/Art/runloop.jpg"><br>
<b>假设进程是一个工厂，线程是一个流水线，那么Run Loop就是流水线上的主管；当接工厂到商家订单，分配给这个流水线时，Run Loop就启动这个流水线，让流水线动起来，生产产品；而当订单的产品生产完毕时，Run Loop就会暂时停下流水线，节约资源。有Run Loop这个主管分配生产任务，流水线才不会因为无所事事被工厂干掉；而工厂转型或者产能升级等原因，不需要这个流水线时，就会辞掉Run Loop这个主管，不再接收任何的订单，即退出线程，把所有的资源释放。</b>
<br>
<img src="http://oncenote.com/assets/images/2015-03-22/assembly_line.jpg"><br>
<b>Run Loop并非iOS/OSX平台专属的概念，在任何平台的多线程编程中，为控制线程生命周期，接收处理异步消息，都需要类似Run Loop的循环机制来实现：从简单的一个无限顺序do{sleep(1);//执行消息}while(true)，到高级平台，如Android的Looper，都是类似的机制。
主线程的Run Loop在应用启动的时候就会自动创建，而其他自己创建的线程则需要在该线程下显式地调用[NSRunLoop currentRunLoop]，假如该线程还没有线程的话，系统会自动创建一个返回。你不能自己去创建一个Run Loop。需要注意的是Run Loop并非线程安全的，所以需要避免在其他线程上调用当前线程的Run Loop。
</b>
<b>RunLoop支持处理输入源(Input Source)事件和计时器(Timer)事件。其中输入源事件包括：系统的MachPort事件、以及其他自定义的输入事件。其中Mach Port是iOS/OSX系统支持的一种通讯事件；而自定义输入事件则故名思议，是需要你自己根据Run Loop的接口，实现相关的回调，来配置自定义的输入源，让Run Loop能够支持对这写输入源的监听和处理。</b>
<br>
<b>值得注意的是，在启动Run Loop，必须先添加监听的输入源事件或者Timer事件，否则调用[runLoop run]会直接返回，而不是进入循环让线程长驻。</b>

{% highlight cpp %}
- (void)main
{
    NSRunLoop *runLoop = [NSRunLoop currentRunLoop];
    while (!self.isCancelled && !self.isFinished) {
        [runLoop runUntilDate:[NSDate dateWithTimeIntervalSinceNow:3]];
    };
}
{% endhighlight %}
<br>
<b>上述代码，因为Run Loop没有添加任何输入源事件或Timer事件，会立刻返回，这样的话，线程其实是一直在无限循环空转中，虽然是让线程长驻不退出，但会一直占用着CPU的时间片，而没有实现资源的合理分配；在其他线程发送一个事件给该线程，系统会自动为Run Loop添加对应输入源或者Timer，让Run Loop正常运行。也可以手动添加输入源或者Timer来让Run Loop正常运行。添加了输入源或Timer事件的Run Loop在没有事件需要处理时，会让线程进行休眠，而不会占用着CPU的时间片。
</b>
<br>
<b>正确的姿势:如下</b><br>
{% highlight cpp %}
- (void)main
{
    NSRunLoop *runLoop = [NSRunLoop currentRunLoop];
    [runLoop addPort:[NSMachPort port] forMode:NSDefaultRunLoopMode];
    while (!self.isCancelled && !self.isFinished) {
        @autoreleasepool {
            [runLoop runUntilDate:[NSDate dateWithTimeIntervalSinceNow:3]];
        }
    }
}
{% endhighlight %}
<br
<b>注意，Run Loop的每个循环必须加上@autoreleasepool，用于释放每个循环结束后不再需要的内存。</b>








{% highlight cpp %}
#include<GL\glut.h>
#include<stdlib.h>
#include<math.h>
#include<windows.h>
#include<time.h>
float radian=3.1415/180;
int x=300;
int y=250;
float r=50;
int max=1000000;
void Init();
void Display();
void Reshape(int Width, int Height);
void Keyboard(unsigned char Key, int X, int Y);


void Init()
{
	glClearColor(0.0,0.0,0.0,0.0);
}
void Setpixel(double x,double y)
{
	glBegin(GL_POINTS);
	glVertex2f(x,y);
	glEnd();
}


void DINHNGOISAO(float tdx,float tdy,float r)
{
	glPointSize(5);
	glBegin(GL_POINTS);
	glColor3f(1.0,0.0,0.0);
	//5 dinh 
	glVertex2i(tdx,tdy+r);
	glVertex2f(tdx-r*cos(18*radian),tdy+r*cos(72*radian));
	glVertex2f(tdx+r*cos(18*radian),tdy+r*cos(72*radian));
	glVertex2f(tdx-r*cos(54*radian),tdy-r*cos(36*radian));
	glVertex2f(tdx+r*cos(54*radian),tdy-r*cos(36*radian));
	//2dinh giua tren
	glVertex2f(tdx-r*cos(72*radian)*tan(36*radian),tdy+r*cos(72*radian));
	glVertex2f(tdx+r*cos(72*radian)*tan(36*radian),tdy+r*cos(72*radian));
	//2dinh giua duoi
	glVertex2f(tdx-r*cos(72*radian)*cos(18*radian)/cos(36*radian),tdy-r*cos(72*radian)*sin(18*radian)/cos(36*radian));
	glVertex2f(tdx+r*cos(72*radian)*cos(18*radian)/cos(36*radian),tdy-r*cos(72*radian)*sin(18*radian)/cos(36*radian));
	//dinh cuoi
	glVertex2f(tdx,y-r*cos(72*radian)/cos(36*radian));
	glEnd();
	glPointSize(1);
	glColor3f(0.0,1.0,0.0);
//	Circle(tdx,tdy,r);
}
void NGOISAO(float tdx,float tdy,float r)
{
	glBegin(GL_POLYGON);
	glVertex2f(tdx-r*cos(72*radian)*tan(36*radian),tdy+r*cos(72*radian));
	glVertex2f(tdx-r*cos(18*radian),tdy+r*cos(72*radian));
	glVertex2f(tdx-r*cos(72*radian)*cos(18*radian)/cos(36*radian),tdy-r*cos(72*radian)*sin(18*radian)/cos(36*radian));
	glVertex2f(tdx-r*cos(54*radian),tdy-r*cos(36*radian));
	glVertex2f(tdx,y-r*cos(72*radian)/cos(36*radian));
	glVertex2f(tdx+r*cos(54*radian),tdy-r*cos(36*radian));
	glVertex2f(tdx+r*cos(72*radian)*cos(18*radian)/cos(36*radian),tdy-r*cos(72*radian)*sin(18*radian)/cos(36*radian));
	glVertex2f(tdx+r*cos(18*radian),tdy+r*cos(72*radian));
	glVertex2f(tdx+r*cos(72*radian)*tan(36*radian),tdy+r*cos(72*radian));
	glVertex2i(tdx,tdy+r);
	glVertex2f(tdx-r*cos(72*radian)*tan(36*radian),tdy+r*cos(72*radian));
	glEnd();
}
void HinhVuong(float tdx,float tdy,float r)
{
	glPointSize(5);
	glBegin(GL_POLYGON);
	glVertex2i(tdx-r-100,tdy+r+50);
	glVertex2i(tdx-r-100,tdy-r-50);
	glVertex2i(tdx+r+100,tdy-r-50);
	glVertex2i(tdx+r+100,tdy+r+50);
	glEnd();
}
void Display()
{
	glClear(GL_COLOR_BUFFER_BIT);
	DINHNGOISAO(x,y,r);
	glColor3f(1.0,0.0,0.0);
	HinhVuong(x,y,r);
	glColor3f(1.0,1.0,0.0);
	NGOISAO(x,y,r);
	glFlush();
}

void Reshape(int Width,int Height)
{
	glViewport(0,0,(GLsizei)Width,(GLsizei)Height);
	glMatrixMode(GL_PROJECTION);
	glLoadIdentity();
	gluOrtho2D(0.0,(GLdouble)Width-1,0.0,(GLdouble)Height-1);
}
void Keyboard(unsigned char Key,int X,int Y)
{
	switch(Key)
	{
	case 27:
		exit(EXIT_SUCCESS);
		break;
	case 114:
		r+=5;
		break;
	case 102:
		r-=5;
		break;
	case 97:
		x-=5;break;
	case 100:
		x+=5;break;
	case 119:
		y+=5;break;
	case 115:
		y-=5;break;
	}
	glutPostRedisplay();
}
void MyIdle()
{
	Sleep(200);
		//x = rand() %300+200;
		//y = rand() %300+200;
		//r = rand() %(100);
	glutPostRedisplay();
}
void main(int Argc,char **Argv)
{
	glutInit(&Argc,Argv);
	glutInitDisplayMode(GLUT_SINGLE|GLUT_RGB);
	glutInitWindowSize(640,480);
	glutInitWindowPosition(0,0);
	glutCreateWindow("Test");
	Init();
	glutDisplayFunc(Display);
	glutReshapeFunc(Reshape);
	glutKeyboardFunc(Keyboard);
	glutIdleFunc(MyIdle);
	glutMainLoop();
}
{% endhighlight %}
