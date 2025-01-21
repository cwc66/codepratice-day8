# codepratice-day8

代码随想录第八次笔记

今天内容由于生病、回家、见同学，晚了四五天的内容。之后补上

重点在于字符串操作,kmp算法，双指针法。需要在之后多刷几次，在这里先承诺补刷到6次的提交数。之后来更新。

[反转字符串中的单词](https://leetcode.cn/problems/reverse-words-in-a-string/description/)
移除多余空格操作
先进行整体反转，再单个单词内部反转。

```CPP
class Solution {
public:
    void reverse(string& s,int start,int end)
    {//翻转，区间写法，左闭右闭
        for(int i=start,j=end;i<j;i++,j--)
        {
            swap(s[i],s[j]);
        }
    }

    void removeExtraspaces(string& s)
    {//去除所有空格并在相邻单词之间添加空格，快慢指针
        int slow=0;
        for(int i=0;i<s.size();++i)
        {
            if(s[i]!=' ')//遇到非空格就处理，即删除所有空格
            {
                if(slow!=0)
                {
                    s[slow++]=' ';//手动控制空格，给单词之间添加空格。slow!=0说明不是第一个单词，需要在单词前添加空格
                }
                while(i<s.size()&&s[i]!=' ')//补上该单词，遇到空格说明单词结束.
                {
                    s[slow++]=s[i++];
                }
            }
        }
        s.resize(slow);//slow的大小即为去除多余空格后的大小
    }

    string reverseWords(string s) {
        removeExtraspaces(s);//去除多余空格，保证单词之间只有一个空格，且字符串首尾没有空格
        reverse(s,0,s.size()-1);
        int start=0;//removeextraspaces后保证第一个单词的开始下标一定是0
        for(int i=0;i<=s.size();++i)
        {
            if(i==s.size()||s[i]==' ')//到达空格或者串尾，说明一个单词结束。进行翻转。
            {
                reverse(s,start,i-1);//反转，注意是左闭右闭的反转。
                start=i+1;//更新下一个单词的开始下班start
            }
        }
        return s;
    }
};
```


[右旋字符串](https://kamacoder.com/problempage.php?pid=1065)
右旋字符串其实就是将某几位字符移到首位。
三次旋转，整体旋转，然后再分别旋转两个子部分。
```CPP
#include<iostream>
#include<algorithm>
using namespace std;
int main() {
    int n;
    string s;
    cin >> n;
    cin >> s;
    int len = s.size(); //获取长度
    reverse(s.begin(), s.begin() + n); //  反转第一段长度为n 
    reverse(s.begin() + n, s.end()); // 反转第二段长度为len-n 
    reverse(s.begin(), s.end());  // 整体反转
    cout << s << endl;

}
```

[找出字符串中第一个匹配项的下标](https://leetcode.cn/problems/find-the-index-of-the-first-occurrence-in-a-string/description/)
KMP算法。找到模式串。
由于暴力法，如果一次匹配不成功，需要重新重头匹配。
其实有可以利用的信息，不需要完全跳到头再匹配。

KMP的重点是前缀表，前缀表是用于记录最大相等前后缀的，知道最大相等前后缀之后，当匹配失败时，退回到最大相等前后缀的位置，重新匹配。
时间复杂度可以降到O(n)。

前缀表构造：1.初始化  2.处理前后缀不相同的情况  3.处理前后缀相同的情况。
j指针指向前缀末尾，i指针指向后缀末尾位置。
前后缀不同时，j回退。

```CPP
void getNext(int* next, const string& s){
    int j = -1;
    next[0] = j;
    for(int i = 1; i < s.size(); i++) { // 注意i从1开始
        while (j >= 0 && s[i] != s[j + 1]) { // 前后缀不相同了
            j = next[j]; // 向前回退
        }
        if (s[i] == s[j + 1]) { // 找到相同的前后缀
            j++;
        }
        next[i] = j; // 将j（前缀的长度）赋给next[i]
    }
}
```

第二个难点在于用数组next来做匹配。
与构造next数组有点像。但是模式串和字符串的比较。一是匹配失败时，j回退，二是如果字符串和模式串对应字符匹配，j移动，i在每次循环中移动。三是如果j的长度到模式串的长度，则匹配成功了，
答案是i的位置减去模式串长度+1。

```CPP
int j = -1; // 因为next数组里记录的起始位置为-1
for (int i = 0; i < s.size(); i++) { // 注意i就从0开始
    while(j >= 0 && s[i] != t[j + 1]) { // 不匹配
        j = next[j]; // j 寻找之前匹配的位置
    }
    if (s[i] == t[j + 1]) { // 匹配，j和i同时向后移动
        j++; // i的增加在for循环里
    }
    if (j == (t.size() - 1) ) { // 文本串s里出现了模式串t
        return (i - t.size() + 1);
    }
}
```

两种方法，next统一减1的：
```CPP
class Solution {
public:
    void getNext(int* next, const string& s) {
        int j = -1;
        next[0] = j;
        for(int i = 1; i < s.size(); i++) { // 注意i从1开始
            while (j >= 0 && s[i] != s[j + 1]) { // 前后缀不相同了
                j = next[j]; // 向前回退
            }
            if (s[i] == s[j + 1]) { // 找到相同的前后缀
                j++;
            }
            next[i] = j; // 将j（前缀的长度）赋给next[i]
        }
    }
    int strStr(string haystack, string needle) {
        if (needle.size() == 0) {
            return 0;
        }
		vector<int> next(needle.size());
		getNext(&next[0], needle);
        int j = -1; // // 因为next数组里记录的起始位置为-1
        for (int i = 0; i < haystack.size(); i++) { // 注意i就从0开始
            while(j >= 0 && haystack[i] != needle[j + 1]) { // 不匹配
                j = next[j]; // j 寻找之前匹配的位置
            }
            if (haystack[i] == needle[j + 1]) { // 匹配，j和i同时向后移动
                j++; // i的增加在for循环里
            }
            if (j == (needle.size() - 1) ) { // 文本串s里出现了模式串t
                return (i - needle.size() + 1);
            }
        }
        return -1;
    }
};
```

前缀表不减1的。
```CPP
class Solution {
public:
    void getNext(int* next, const string& s) {
        int j = 0;
        next[0] = 0;
        for(int i = 1; i < s.size(); i++) {
            while (j > 0 && s[i] != s[j]) {
                j = next[j - 1];
            }
            if (s[i] == s[j]) {
                j++;
            }
            next[i] = j;
        }
    }
    int strStr(string haystack, string needle) {
        if (needle.size() == 0) {
            return 0;
        }
        vector<int> next(needle.size());
        getNext(&next[0], needle);
        int j = 0;
        for (int i = 0; i < haystack.size(); i++) {
            while(j > 0 && haystack[i] != needle[j]) {
                j = next[j - 1];
            }
            if (haystack[i] == needle[j]) {
                j++;
            }
            if (j == needle.size() ) {
                return (i - needle.size() + 1);
            }
        }
        return -1;
    }
};
```

字符串、双指针总结[双指针总结](https://programmercarl.com/%E5%8F%8C%E6%8C%87%E9%92%88%E6%80%BB%E7%BB%93.html#%E5%AD%97%E7%AC%A6%E4%B8%B2%E7%AF%87)：
这部分的题需要复习重做。
