---
layout: post
title: Codeforces Round 124(Div.2) 总结
abstract: 一场codeforces - div2 的总结

tags:
- Personal
- acm
- algorithm
- codeforces

type: post
---

## 缘起 ##

水人前几天发现阔别已久的codeforces竟然有9点场，看来比午夜场淡定很多，于是拉上比[泽裕](https://github.com/ZeyuChen)一起去水。水人目测9点前离开实验室有点废所以便在实验室开打了。

## 流水 ##

codeforces只有两个小时但是因为我们做div.2所以略水啊。当然只是前几题略水。本来想着半小时水三道冲四的，结果一进场就被吓到了.我汗啊5道题全是3000分，然后看完第一道类似博弈的东西，彻底被镇住。

A. [Plate Game](http://www.codeforces.com/contest/197/problem/A)

说给个长方形，一种固定大小的圆，两个人往里面轮流放，不能重叠不能越出长方形。谁先没得放就输了。

其实这题一看就觉得有点水，想想第一个圆放下去以后first对称着放不就行了？然后和废人说了一下想法，他觉得也就是这样了。他就去敲并且过了，我竟然敲错一点。。尼嘛几行代码能敲错我真是水到暴。然后WA了两次怀疑正确性=。=，最后拖沓地过了小数据。

B. [Limit](http://www.codeforces.com/contest/197/problem/B) 

全场最大水题，算两个多项式相除的极限，这种东西，不是无限就是0不然就是一个除法搞定拉。不过被A射了一箭搞得分数奇低。

C. [Lexicographically Maximum Subsequence](http://www.codeforces.com/contest/197/problem/C)

此时泽裕貌似过了三道了=。=我就继续看第三道，竟然看错题..堪称最长递减子序列，不过就算看错也是水题，速敲速交WA去..问了下泽裕，原来是字典序最大！！！尼嘛水到暴了。

D. [Infinite Maze](http://www.codeforces.com/contest/197/problem/D)

因为之前被三道水题吓到，开D的时候已经没有信心拉。问了下题意，他说是水题，我一看也觉得很水，说给个矩阵地图，有些点可以走有些不可以，问这个小地图组成的大地图（横竖挨着放）能否走到无限远。

一上眼直接开暴力，模拟走到第100个就说可以拉。结果敲完已经1小时56分了，交第一个Runtime Error心碎，以为没可能了，发现数组开很小，改超级大交上去。过了小数据！！！还是泽裕看到我过的。。当时真是开心呀。得瑟回宿舍。
结果出来结果Time Limited 10，哭了，想想应该是开太大了暴力，就改小一点，WA46蛋碎..又水掉了一场。什么时候能变色呢。

后来想想D其实是水题，只要走到别的地图的点是在本地图走过的点就行了。

充分性:如果能够走到别的地图上走过的点，那么可以照这种走法走无限远，因为地图之间肯定有一个坐标是偏离的。

必要性:因为可以走无限远，而小地图大小有限，所以必然可以走到相同的点。想到这一层就敲之结果也过了，代码如下:


{% highlight c++ linenos %}
/* codeforces 124 div.2 D.cpp */
#include <iostream>
#include <queue>
#include <cstdio>
#include <cstring>
#include <cstdlib>
using namespace std;

#define INF 50
#define MX  1555
int n, m;
int st;
int board[1501][1501];
char input[1510];
bool had[MX+100][MX+100];
int po[MX][MX][2];
//#define HOME 

int _x[] = {1, -1, 0 ,0 };
int _y[] = {0, 0, 1, -1 };
int main()
{
#ifdef HOME
    freopen("1.txt", "r", stdin);
#endif
    scanf("%d %d", &n, &m);
    memset(had, false, sizeof(had));
    for(int i = 0;i < n;i++)
    {
        scanf("%s", input);
        int len = strlen(input);
        for(int j = 0;j < len;j++)
        {
            if(input[j] == '#') board[i][j] = 1;
            else if(input[j] == '.') board[i][j] = 0;
            else
            {
                board[i][j] = 0;
                st = i * m + j;
                po[i][j][0] = 0;
                po[i][j][1] = 0;
            }
        }
    }
    
    queue<int> q;
    q.push(st);
    had[st/m][st%m] = true;
    bool ok = false;
    while(!q.empty())
    {
        if(ok) break;
        int _top = q.front(); q.pop();
        int x = _top / m, y  = _top % m;

        for(int i = 0;i < 4;i++)
        {
            int x_nxt = x + _x[i], y_nxt = y + _y[i];

            int opx = po[x][y][0], opy = po[x][y][1];

            if(x_nxt == -1)
            {
                opx--;
                x_nxt = n-1;
            }
            else if(x_nxt == n)
            {
                opx++;
                x_nxt = 0;
            }
            else if(y_nxt == -1)
            {
                opy--;
                y_nxt = m-1;
            }
            else if(y_nxt == m)
            {
                opy++;
                y_nxt = 0;
            }
            if(board[x_nxt][y_nxt] == 1) continue;
            else if(!had[x_nxt][y_nxt])
            {
                had[x_nxt][y_nxt] = true;
                q.push(x_nxt*m+y_nxt);
                po[x_nxt][y_nxt][0] = opx;
                po[x_nxt][y_nxt][1] = opy;
            }
            else
            {
                if(opx != po[x_nxt][y_nxt][0] || opy != po[x_nxt][y_nxt][1])
                {
                    ok = true;
                    break;
                }
            }

        }
    }
    if(ok)
        printf("Yes\n");
    else
        printf("No\n");
}
{% endhighlight %}

[E. Paint Tree](http://www.codeforces.com/contest/197/problem/E)

水人表示这个不是水人的菜。

## 后记 ##
现在生活已经没那么清闲，毕竟比大一大二老了很多，一想到大一大二为之奋斗的东西现在要告别就胸口受伤。其实我想坚持，只是又觉得没多大必要了，就让我这个水人越来越水吧。

这可能就是为什么我要花两个小时做那么几道水题的原因吧，明明肯定不是为了锻炼了，或许只是因为我依然喜欢这个游戏。虽然可是毫无压力什么都不理只刷题的日子已经不在。但依然会为每一个Accepted而开心，水题又怎么样，别人打机看小说浪费时间，我刷水题能高兴还怕浪费时间么=。=

其实我们两一起水题也有一段日子了，codeforces很多次都感觉能暴4题但最后老差那么一点。很奇怪。好吧以后我决定以后要多玩这个了毕竟很开心。什么时候能稳稳过4题，就变色吧～～

