## 题目规划
- 动态规划15道
- 回溯法15道
- 链表操作20道
- 二叉树20道
- 排序10道
- 常规算法
- 字符串10道

### 1.回文子串
回文子串可以使用动态规划解法，但是需要附加判定条件，因为回文子串并不是简单的寻找子序列，需要两个子串的位置相对相同。也可以使用Manacher（马拉车算法）。

##### 动态规划方法
$P(i,j) = \left\{\begin{matrix}
true \; \; \; s[i,j] \; is \\
\;\;\;\;\;\;false \; \; \; s[i,j] \;is\; not
\end{matrix}\right.$

动态规划方程： $P(i,j) = (P(i+1,j-1)  \&\& S[i] == S[j])$

用s[i,j]表示字符串，dp[i,j]表示是否是回文串，dp[i,j]可以由s[i]和s[j]是否是回文推断，具体步骤如下：
1. dp[i,i] = 1
2. 从子串长度为2开始，判断dp[i,j]时，判断s[i],s[j]是否相等，且dp[i+1,j-1]是否为回文串
3. 记录最长子串位置

##### Manacher算法
对字符串进行分隔，在其中加入#，使得所有子串都为奇数个字符，再以每个字符为回文中心进行遍历，**其最主要的工作是利用了回文性质缩小了比较步骤**，具体做法：

用p表示以c为中心的回文串长度，以r表示回文串中半径达到最远的字符下标，以i表示中心扩展时的判断，i_mirror则表示以c为中心i的镜像。
1. r = 0, c = 0
2. 从第一个字符开始，使用中心扩展判断是否是回文串
3. 更新r
4. 以下一个字符为中心判断时，如果i没有超过r并且半径范围也在R内，则直接更新，并在下一个字符重复第四步。
5. 如果i大于了r，或者等于R或者因为镜像节点边境问题，需要重新使用中心扩展。
```
public String preProcess(String s) {
    int n = s.length();
    if (n == 0) {
        return "^$";
    }
    String ret = "^";
    for (int i = 0; i < n; i++)
        ret += "#" + s.charAt(i);
    ret += "#$";
    return ret;
}

// 马拉车算法
public String longestPalindrome2(String s) {
    String T = preProcess(s);
    int n = T.length();
    int[] P = new int[n];
    int C = 0, R = 0;
    for (int i = 1; i < n - 1; i++) {
        int i_mirror = 2 * C - i;
        if (R > i) {
            P[i] = Math.min(R - i, P[i_mirror]);// 防止超出 R
        } else {
            P[i] = 0;// 等于 R 的情况
        }

        // 碰到之前讲的三种情况时候，需要利用中心扩展法
        while (T.charAt(i + 1 + P[i]) == T.charAt(i - 1 - P[i])) {
            P[i]++;
        }

        // 判断是否需要更新 R
        if (i + P[i] > R) {
            C = i;
            R = i + P[i];
        }

    }

    // 找出 P 的最大值
    int maxLen = 0;
    int centerIndex = 0;
    for (int i = 1; i < n - 1; i++) {
        if (P[i] > maxLen) {
            maxLen = P[i];
            centerIndex = i;
        }
    }
    int start = (centerIndex - maxLen) / 2; //最开始讲的求原字符串下标
    return s.substring(start, start + maxLen);
}


```

### 2.公共子序列
最长公共子序列可以使用动态规划算法，其动态方程是

$P(i,j) = \left\{\begin{matrix}
P(i-1,j-1) +1 \; \; \;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\; text1[i]\;==\;text2[j] \\
Max(P(i-1,j),P(i,j-1)) \; \; \; text1[i]\;!=\;text2[j]
\end{matrix}\right.$

最后得到就是子序列长度
```
int longestCommonSubsequence(string text1, string text2) {
	int len1 = text1.length();
	int len2 = text2.length();
	int** dp = new int*[len1];
	for (int i = 0; i < len1; ++i) {
		dp[i] = new int[len2];
		memset(dp[i], 0, len2 * sizeof(int));
	}
	for (int i = 0; i < len1; ++i) {
		if (text1[i] == text2[0]) {
			dp[i][0] = 1;
		}
	}
	for (int i = 0; i < len2; ++i) {
		if (text2[i] == text1[0]) {
			dp[0][i] = 1;
		}
	}
	for (int i = 0; i < len1; ++i) {
		for (int j = 0; j < len2; ++j) {
			if (text1[i] == text2[j]) {
				dp[i][j] = dp[i - 1][j - 1] + 1;
			}
			else {
				dp[i][j] = max(dp[i - 1][j], dp[i][j - 1]);
			}
		}
	}
	return dp[len1 - 1][len2 - 1];

}
```

### 3. 最长公共子串
子串跟子序列相比，得到的结果是需要连续的，而且需要比较后保存最大值。可以在最长公共子序列上改进得到。
```
int findLength(vector<int>& A, vector<int>& B) {
	int len1 = A.size();
	int len2 = B.size();
	int** dp = new int*[len1];
	for (int i = 0; i < len1; ++i) {
		dp[i] = new int[len2];
		memset(dp[i], 0, len2 * sizeof(int));
	}
	for (int i = 0; i < len1; ++i) {
		if (A[i] == B[0]) {
			dp[i][0] = 1;
		}
	}
	for (int i = 0; i < len2; ++i) {
		if (A[0] == B[i]) {
			dp[0][i] = 1;
		}
	}
	int Max = 0;
	for (int i = 1; i < len1; ++i) {
		for (int j = 1; j < len2; ++j) {
			if (A[i] == B[j]) {
				dp[i][j] = dp[i - 1][j - 1] + 1;
				Max = max(Max, dp[i][j]);
			}
			else {
				dp[i][j] = 0;
			}
		}
	}
	return Max;
}

```

---

### 4.快速排序
快排的传统写法需要将right的循环判断写在前面，这样做是为了先找到一个比他小的数，再找比它大的数，如果在左边没有比它大的数了，说明右边比它小的数就是它的位置，就算最后没有找到ij最后一次交换也是互相交换，这里提供另外一种高效写法
```
void quickSort(vector<int>& nums, int l, int r) {
        if (l >= r) return;
        int m = l + (r - l) / 2;
        swap(nums[r], nums[m]);
        int t = l;
        for (int i = l; i < r; ++i) {
            if (nums[i] < nums[r]) {
                swap(nums[t++], nums[i]);
            }
        }
        swap(nums[t], nums[r]);
        quickSort(nums, l, t - 1);
        quickSort(nums, t + 1, r);
    }
```
这种写法先将mid放入最后，再从头开始比较，依次将比mid小的放入前面位置，到最后所有比mid小的都在前面，再讲mid放入正确位置。其他排序写法[参见](https://leetcode-cn.com/problems/sort-an-array/solution/c-ge-chong-jie-fa-tong-pai-xu-zui-kuai-by-da-li-wa/)

[双路快排，三路快排](https://blog.csdn.net/k_koris/article/details/80585979)

---

### 5. 二分查找元素区间
查找排序好的元素的区间问题，通过二分法将其化为$O(log_{2}^{N})$复杂度。
- 二分的时候lo = mid + 1;不然的话会陷入死循环无法退出。
- 利用一个变量去判断当前是左边界还是右边界。
- hi在定义为size()时边界处理更加方便。

```
int find(vector<int>& nums, int target, int left){
	int lo = 0;
	int hi = nums.size();
	while(lo < hi){
		int mid = (lo + hi) / 2;
		if(nums[mid] > target || (left && mums[mid] == target)){
			hi = mid;
		}else{
			lo = mid + 1;
		}
	}
	return lo;
}

int searchRange(vector<int>& nums, int target){
	vector<int> res{-1,-1};
	int leftIndex = find(nums, target, true);
	if(nums[leftIndex] != target || leftIndex == nums.size()){
		return res;
	}
	res[0] = leftIndex;
	res[1] = find(nums, target, false);
	return res;
}

```

### 6. 删除倒数第N个节点
通过前后指针进行删除，在进行链表操作时，满足以下几点可以更快地操作：
- 如果首节点有可能变更，则引入一个新的头节点res，并且`res->next = head`，在返回时`return res`。
- 节点在进行删除、交换位置时，最好在其先驱节点上进行操作，因为删除和交换都需要其先驱节点，这在单向链表中很难得到。

```
ListNode* removeNthFromEnd(ListNode* head, int n) {
		ListNode* res  = new ListNode(0);
		res->next = head;

		ListNode* lo = res;
		ListNode* hi = res;
		for(int i = 0;i < n;++i){
				hi = hi->next;
		}
		while(hi->next != NULL){
				hi = hi->next;
				lo = lo->next;
		}
		lo->next = lo->next->next;
		return res->next;
}
```

### 7. 对两个有序链表进行整合
注意操作顺序，没有什么难点
```
ListNode* mergeTwoLists(ListNode* l1, ListNode* l2) {
		ListNode* res = new ListNode(0);
		ListNode* ite = res;

		while(l1 != NULL && l2 != NULL){
				if(l1->val > l2->val){
						ite->next = l2;
						ite = ite->next;
						l2 = l2->next;
				}else{
						ite->next = l1;
						ite = ite->next;
						l1 = l1->next;
				}
		}

		if(l1 != NULL){
				ite->next = l1;
		}else{
				ite->next = l2;
		}
		return res->next;
}
```

### 8. 链表中的节点交换
同上
```
ListNode* swapPairs(ListNode* head) {
		ListNode* res = new ListNode(0);
		res->next = head;

		ListNode* ite = res;
		while(ite->next != NULL && ite->next->next != NULL){
				ListNode* t = ite->next;

				ite->next = t->next;
				t->next = ite->next->next;
				ite->next->next = t;

				ite = t;
		}

		return res->next;
}
```
