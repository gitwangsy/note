C < logn < n < nlogn < n^x^

## 力扣/CSDN

1. 给你一个整数数组 nums 。如果任一值在数组中出现 至少两次 ，返回 true ；如果数组中每个元素互不相同，返回 false 。

	```c
	//未解决(超时)
	bool containsDuplicate(int* nums, int numsSize){
	    for(int i=0;i<numsSize;i++){
	        for(int j=1+i;j<numsSize;j++){
	            if(nums[i]==nums[j]){
	                return true;
	            }
	        }
	    }
	    return false;
	}//时间复杂度n^2（超时）
	```

2. 给定一个字符串长度为 nn 的字符串 s1 (10<n<100) , 求出将字符串循环向左移动 k 位的字符串 s2 (1<k<n) , 例如：字符串 abcdefghijk , 循环向左移动 3 位就变成 defghijkabc
	输入描述：输入仅两行，第一行为左移的位数 k , 第二行为字符串 s1 .
	输出描述：输出仅一行，为将字符串 s1 左移 k 位得到的字符串 s

	```c
	#include <stdio.h>
	#include <string.h>
	//已解决(中等)
	void reversion(char *str, int beg, int end)
	{
		char temp;
		for (int i = beg, j = end; i < j; i++, j--)
		{
			temp = str[i];
			str[i] = str[j];
			str[j] = temp;
		}
	}
	void turnleft(char *str, int k, int len)
	{
		int left = k % len;
		if (left == 0)
		{
			return;
		}
		// 先小反转再大反转
		reversion(str, 0, left - 1);
		reversion(str, left, len - 1);
		reversion(str, 0, len - 1);
		return;
	}
	
	int main(int argc, const char *argv[])
	{
		char str[100];
		int k = 0;
		scanf("%d", &k);
		scanf("%s", str);
		int len = strlen(str);
	
		turnleft(str, k, len);
	
		printf("%s\n", str);
		return 0;
	}
	```

	```c
	//计算字符串长度
	char str[10]; //char *str；
	scanf("%s",str);
	int i = 0;
	while (str[i++]);
	printf("%d\n",i-1);
	//用指针则段错误，str是野指针，指向不确定，不允许操作
	```

3. 