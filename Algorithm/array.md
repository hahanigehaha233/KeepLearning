## 最长回文字符串
[Leetcode第五题](https://leetcode-cn.com/problems/longest-palindromic-substring/submissions/)

采用马拉车算法的思想，但是第一版的代码没有剪枝，导致时间上只超过了50的人，马拉车算法分三部：
- 构造特殊字符串
- **根据先前计算出的回文半径进行剪枝**
- 遍历回文半径数组得到最优解

第一版代码
```
public class number5 {
    public String longestPalindrome(String s) {
        StringBuilder ss = new StringBuilder(2001);
        int len = s.length();
        if(len < 2) return s;
        ss.append('#');
        for(int i =0;i<len;i++){
            ss.append(s.charAt(i));
            ss.append('#');
        }
        int cc = 0;
        int mark = 0;
        for(int i = 1;i < ss.length();i++){
            int c = 0;
            int j = i + 1;
            int k = i - 1;
            while (j < ss.length() && k >= 0){
                if(ss.charAt(j) == ss.charAt(k)){
                    ++j;
                    --k;
                    c++;
                }else{
                    break;
                }
            }
            if(cc < c){
                mark = i;
                cc = c;
            }
        }
        StringBuilder res = new StringBuilder(100);
        for(int i = mark - cc;i < mark + cc;i++){
            if(ss.charAt(i) == '#'){
                continue;
            }
            else {
                res.append(ss.charAt(i));
            }
        }
        return res.toString();
    }
}

```

## 最长公共子序列和最长公共子串有区别
子序列可以分开，子串必须连续

[子串问题-leetcode](https://leetcode-cn.com/problems/maximum-length-of-repeated-subarray/)
