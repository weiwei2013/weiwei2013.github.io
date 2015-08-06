---
layout: post
title: Draw VN flag in OpenGL
description: "How to Draw VN flag in OpenGL?"
modified: 2015-05-12
tags: [OpenGL, cpp]
---

<img src="http://i.imgur.com/cfxb4wY.gif?1"><br>
code here:

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
