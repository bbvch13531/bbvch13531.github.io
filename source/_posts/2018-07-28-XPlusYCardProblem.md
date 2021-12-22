---
layout: post
title:  "X + Y Card Problem"
date: 2018-07-28 13:00:00 +0900
description: 카드 뒤집기 문제와 증명
img:  # Add image post (optional)
tags: [네이든, 카드 뒤집기, 증명, X + Y]
---
# X + Y Card Problem
우리나라에서 '네이든' 이라는 이름으로 개봉한 'X + Y' 에 나오는 카드 뒤집기 문제  
N개의 카드가 일렬로 놓여있다.  
각 카드의 상태는 앞면, 혹은 뒷면이고 앞면은 1 뒷면은 0 으로 나타낸다.

카드를 뒤집게되면 인접한 오른쪽의 카드 1장도 같이 뒤집어진다.  
(만약 맨 오른쪽 카드 한 장만 앞면이고, 나머지 카드가 모두 뒷면이면 맨 오른쪽 카드 한 장만 뒷면으로 뒤집을 수 있다.)
  
카드를 뒤집어서 모든 카드가 뒷면을 향하도록 만드는 동작이 항상 가능함을 증명하라.

예)  
000010 -> 000001 -> 000000  
011100 -> 000100 -> 000010 -> 000001 -> 000000

쉬운 DP문제.. 근데 증명은?

카드의 상태를 이진수로 봤을 때(unsigned), 카드를 뒤집는 동작에서는 항상 이진수가 감소한다.  
이진수가 음수가 되는 경우는 없으므로 모든 카드가 뒷면을 향하도록 (이진수로 0이 되도록)하는 이 동작은 항상 가능하다.

```

#include <iostream>

using namespace std;

string process(string str);
bool isFinished(string str);
void flipCard(string& str, int index);

int main(void){
    string input,res;
    
    cin>>input;
    
    res = process(input);
    cout<<res<<endl;
    
    return 0;
}

string process(string str){
    int len = str.length();
    for(int i = 0; i<len; i++){
        for(int j=i; j<len-1; j++){
            if(str[j]=='0') continue;
            
            else{
                flipCard(str,j);
                flipCard(str,j+1);
            }
            cout<<str<<endl;
        }
    }
    if(str[len-1] == '1'){
        str[len-1] = '0';
        cout<<str<<endl;
    }
    return str;
}

bool isFinished(string str){
    for(int i=0; i<str.length() - 1; i++){
        if(str[i]=='1') return false;
    }
    return true;
}

void flipCard(string& strp, int index){
    if(strp[index] == '1') strp[index]='0';
    else strp[index]='1';
    
    return;
}

```
