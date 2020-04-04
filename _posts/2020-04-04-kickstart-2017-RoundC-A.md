---
layout: post
title: Kickstart 2017 Round C Problem A
categories: [blog ]
tags: [kickstart, ]
description: 
---

问题地址：[Ambiguous Cipher](https://code.google.com/codejam/contest/dashboard?c=4344486#s=p0&a=0)

问题输入是一个大写字符串，表示加密后的信息。要求输出加密前的原始字符串，如果原始字符串不唯一，那么输出“ANBIGUOUS”。

编码规则是先将字符转为对应的数字，然后将相邻数字相加对26取模得到新的数字，再转化为对应的字母。第一个和最后一个字符只有一个相邻数字。示例如下：

```
* S = 18, O = 14, U = 20, and P = 15.
* Calvin encrypts each letter based on the values of its neighbor(s):
	* First letter: 14 mod 26 = 14.
	* Second letter: (18 + 20) mod 26 = 12.
	* Third letter: (14 + 15) mod 26 = 3.
	* Fourth letter: 20 mod 26 = 20.
* The values 14 12 3 20 correspond to the letters OMDU, and this is the encrypted word that Calvin will write on the note for Susie.
```
分析：

当字符串长度为偶数时，原始的第二个字符就是加密后的第一个字符，然后原始字符串中的第四个、第六个直到最后一个字符，都可以根据第二个推算出来。同理，倒数第二个、倒数第四个、直到第一个字符，也可以推算出来。

当字符串长度为奇数时，可以得到第二个和倒数第二个字符，但是这两个都是偶数位，奇数位的数字不能唯一确定。所以字符串长度为奇数时，都输出“ANBIGUOUS”。

{% highlight c++ %}
#include <iostream>
#include <vector>

using namespace std;

int main(){
    int n;
    cin >> n;
    string w;
    for(int i=1; i<=n; i++){
        cin >> w;
        int l = w.length();
        string ret = w;
        vector<int> a(l);
        vector<int> b(l);
        if(l % 2 == 1){
            ret = "AMBIGUOUS";
        }else{
            for(int j=0; j<l; j++){
                a[j] = w[j] - 'A';
            }
            b[1] = a[0];
            for(int j=3; j<l; j+=2){
                b[j] = a[j-1] - b[j-2];
            }
            b[l-2] = a[l-1];
            for(int j=l-4; j>=0; j-=2){
                b[j] = a[j+1] - b[j+2];
            }
            for(int j=0; j<l; j++){
                b[j] = b[j] % 26;
                if(b[j]<0){
                    b[j] += 26;
                }
                ret[j] = b[j] + 'A';
            }
        }
        cout << "Case #" << i << ": " << ret << endl;
    }
}
{% endhighlight %}
