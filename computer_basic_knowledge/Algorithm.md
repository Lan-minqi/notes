# 算法

## KMP算法

KMP算法是一种字符串匹配算法，可以在 O(n+m) 的时间复杂度内实现两个字符串的匹配

字符串匹配问题

```txt
字符串 P 是否为字符串 S 的子串？如果是，它出现在 S 的哪些位置？” 其中 S 称为主串；P 称为模式串
```

介绍

<https://wiki.jikexueyuan.com/project/kmp-algorithm/define.html>

```cpp
int KmpSearch(char* s, char* p)  
{  
    int i = 0;  
    int j = 0;  
    int sLen = strlen(s);  
    int pLen = strlen(p);  
    while (i < sLen && j < pLen)  
    {  
        //①如果j = -1，或者当前字符匹配成功（即S[i] == P[j]），都令i++，j++      
        if (j == -1 || s[i] == p[j])  
        {  
            i++;  
            j++;  
        }  
        else  
        {  
            //②如果j != -1，且当前字符匹配失败（即S[i] != P[j]），则令 i 不变，j = next[j]      
            // j = next[j]等于是模式串向右移动了j - next[j]位
            j = next[j];  
        }  
    }  
    if (j == pLen)  
        return i - j;  
    else  
        return -1;  
}  

//优化过后的next 数组求法, 动态规划
void GetNextval(char* p, int next[])  
{  
    // 代码结构个KMP差不多, 自匹配
    int i = 0;  
    int j = -1;  
    int pLen = strlen(p);  
    next[0] = j;  
    while (i < pLen - 1)  
    {  
        if (j == -1 || p[i] == p[j])  
        {  
            ++i;  
            ++j;  
            // next[i+1]的值 决定于 p0 ~ pi的分布
            if (p[i] != p[j])  
                next[i] = j;
            else  
                //因为不能出现p[i] = p[ next[i ]]，所以当出现时需要继续递归
                next[i] = next[j];  
        }  
        else  
        {  
            j = next[j];  
        }  
    }  
}  
```
