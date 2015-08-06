---
layout: post
title: How many Zeroes will David write down?
description: "abcd"
modified: 2015-05-11
tags: [count, integer, interval, cpp]
---

**Description:**<br>
David writes down the decimal representations of all natural numbers between and including m and n, (m ≤ n). How many zeroes will he write down?

**Input:**<br>
Input starts with an integer T (≤ 11000), denoting the number of test cases.
Each case contains two unsigned 32-bit integers m and n, (m ≤ n).

**Output:**<br>
For each case, print the case number and the number of zeroes written down by David.

>Sample input:<br>
5<br>
10 11<br>
100 200<br>
0 500<br>
1234567890 2345678901<br>
0 4294967295<br>

>Sample output:<br>
Case 1: 1<br>
Case 2: 22<br>
Case 3: 92<br>
Case 4: 987654304<br>
Case 5: 3825876150<br>

{% highlight cpp %}
#include <cstdio>
#include <iostream>
using namespace std;
typedef long long int64;
int64 U[12];
int64 cal(int64 n)
{
    if(n<0)
        return 0;
    int64 sum=1,l,r,mid;
    for(int i=1;i<12;i++)
    {
        l=n/U[i];
        r=n%U[i-1];
        mid=mid=(n-l*U[i])/U[i-1];
        if(mid==0)
            sum+=(l-1)*U[i-1]+r+1;
        else
            sum+=l*U[i-1];
        if(U[i]>n)
            break;
    }
    return sum;
}

int main()
{
    int64 n,m;
    int cas;
    cin>>cas;
    U[0]=1;
    for(int i=1;i<12;i++)
        U[i]=U[i-1]*10;
    for(int i=1;i<=cas;i++)
    {
        cin>>n>>m;
        cout<<"Case "<<i<<": ";
        cout<<cal(m)-cal(n-1)<<endl;
    }
    return 0;
}
{% endhighlight %}
