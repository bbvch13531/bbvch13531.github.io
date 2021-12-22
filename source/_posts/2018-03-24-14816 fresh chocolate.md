---
layout: post
title:  "14816 fresh chocolate"
date: 2018-03-24 17:00:00 +0900
description: You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. # Add post description (optional)
img:  # Add image post (optional)
tags: [알고리즘, 백준, 14816, 14816 fresh chocolate]
---
# 14816 fresh chocolate (large)
google code jam 2017 round2 A2
단순하게 풀면 되는 문제

```
//
//  14816 fresh chocolate.cpp
//  Algorithm
//
//  Created by bbvch13531 on 2018. 3. 24..
//  Copyright © 2018년 bbvch13531. All rights reserved.
//

#include <iostream>
#include <cstdio>
#include <algorithm>
#include <cstring>
using namespace std;

int main(void){
    int T,N,P,a[110],b[10]={0,},ans,ret;
    cin>>T;
    
    for(int i=0;i<T;i++){
        ans=ret=0;
        cin>>N>>P;
        memset(b, 0, sizeof(b));
        for(int j=0;j<N;j++){
            cin>>a[j];
            b[ a[j] % P ]++;
        }
        ans+=b[0];
        if(P==2){
            ans+=(b[1]+1)/2;
        }
        
        else if(P==3){
            ret=min(b[1],b[2]);
            b[1]-=ret;
            b[2]-=ret;
            ans+=ret;
            
            ans+=(b[2]+2)/3;
            ans+=(b[1]+2)/3;
        }
        else{   //P==4
            ret=min(b[1],b[3]);
            b[1]-=ret;
            b[3]-=ret;
            ans+=ret;
            
            ret=b[2]/2;
            b[2]%=2;
            ans+=ret;
            //위 과정을 거치면 2는 0개 or 1개, 1만 남거나 3만 남는다.
            if(b[2]>0 && b[1]>=2){  //2+1+1
                b[2]=0;
                b[1]-=2;
                ans++;
            }
            if(b[2]>0 && b[3]>=2){  //2+3+3
                b[2]=0;
                b[3]-=2;
                ans++;
            }
            if(b[2]>0){ //2 1개 남는 경우
                b[2]=0;
                ans++;
            }
            else{   //1과 3이 남지만 위 조건에 해당하지 않는 경우
                ans+=(b[1]+3)/4;
                ans+=(b[3]+3)/4;
            }
        }
        printf("Case #%d: %d\n",i+1,ans);
    }
    return 0;
}
```