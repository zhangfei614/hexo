---
title: 剑指Offer面试题总结
date: 2017-06-13 14:19:15
categories: Algorithms
tags: 
- Algorithms
---

#### 面试3 二维数组中的查找
二维数组中，从左到右，从上到下都是有序的，查找一个数是否存在。
**思路：**
1. 从右上角向左开始搜索，删除右侧的列，直到遇到比目标值小的数。
2. 从右上角向下开始搜索，删除上方的行，直到遇到比目标值大的数。
**代码**
```java
boolean find (int[][] board,int rows,int columns,int key){
	boolean found = false;
	int row = 0, column = columns - 1;
	while(row < rows && column >= 0){
		if(board[row][column] == key){
			found = true;
			break;
		//右上角->左搜索，删除右侧列
		}else if(board[row][column] > key){
			column --;
		}else{
		//右上角->下搜索，删除上侧行
			row ++;
		}
	}
	return found;
}
```

#### 面试8 旋转数组中的最小数
把一个数组的最开始的若干个元素放到最后，成为一旋转数组。给定一个递增数组的旋转数组，求该数组中的最小值。
**思路：**
分别用index1表示前面子数组的第一个元素，index2表示后面子数组的最后一个元素。
1. 如果是旋转数组，则arr[index1] >= arr[index2]，否则直接返回。
2. 如果arr[mid] >= arr[index1]，则说明mid在前面的递增数组中，则可以将index1=mid。
3. 如果arr[mid] <= arr[index2]，则说明mid在后面的递增数组中，则可以将index2=mid。
4. 考虑特殊情况：arr[mid]=arr[index1]=arr[index2]，则无法确定在哪个数组中；例如{1,0,1,1,1}的情况或者{1,1,1,0,1}的情况。


```java
int min(int[] rArr){
	int n = rArr.length;
	int index1 = 0,index2 = n-1, mid = index1;
	while(rArr[index1] >= rArr[index2]){
		//如果差1，则找到结果
		if(index2 - index1 == 1){
			mid = index2;
			break;
		}
		mid = index1 + (index2 - index1)/2;
		//如果相等，则无法判断，只能顺序查找
		if(rArr[mid] == rArr[index1] && rArr[mid] == rArr[index2]){
			int result = rArr[index1];
			for(int i = index1 + 1; i <= index2; i++){
				if(rArr[i] < result){
					result = rArr[i]
				}
			}
			return result;
		//如果大于rArr[index1]，则说明在后面
		}else if(rArr[mid] >= rArr[index1]){
			index1 = mid;
		//如果小于rArr[index2]，则说明在前面
		}else if(rArr[mid] <= rArr[index2]){
			index2 = mid;
		}
	}
	return rArr[mid];
}
```
#### 面试10 二进制中1的个数
求一个整数中1的个数
**思路1**
求32次，但是注意不能使用n>>1来判断循环是否结束（如果是负数，则符号位一直未1），而是利用flag<<1来判断。

```java
int bitCount(int n){
	int count = 0;
	unsigned int flag = 1;
	while(flag){
		if(n & flag) count++;
		flag <<= 1;
	}
	return count;
}
```
**思路2**
当一个数n与n-1相与时，就将包括最后一位1的位全部之0，例如：1101 & 1100 -> 1100 ， 1010 & 1001 -> 1000，所以利用此种方案有多少1，则执行多少次循环。
```java
int bitCount(int n){
	int count = 0;
	while(n){
		count ++;
		n = n & (n-1);
	}
	return count;
}
```
**拓展：**
两个整数m、n，m经过多少次位变换可以得到n：现将m、n异或，然后求异或值1的个数。
利用&1来判断是否是级数，右移来表示除以2

#### 面试14 调整数组使所有奇数位于偶数前面
输入一个整数数组，将所有的奇数位于前半部分，偶数位于后半部分。
**思路：**
利用快速排序中的partition的思路，进行一次交换O(n)的扫描，并交换。
**代码：**
```java
    public void partitionArray(int[] array) {
        int start = -1;
        for (int i = 1; i < array.length; i++) {
            if ((array[i] & 1) != 0) {
                start++;
                exchange(array, start, i);
            }
        }
        exchange(array, start, 0);
    }
```

#### 面试15 输入一个链表的倒数第k个节点
给定一个单向链表，输出该链表的倒数第k个节点。
**思路：**
1. 如果空间复杂度允许，则可以使用栈，但肯定不是最优解。
2. 维护两个指针，当第一个指针走到k时，第二个指针开始走。
**代码：**
```java
public ListNode lastK(ListNode head,int k){
	ListNode p = head, q = null;
	int count = 0;
	while(p != null){
		count++;
		if(count >= k){
			q = q == null ? head : q.next;
		}
		p = p.next;
	}
	return q;
}
```
**拓展：**
- 求链表的中间节点，快指针每一次走两步，慢指针每一次走一步，则最后为中间节点。
- 求链表中是否存在一个环，则同样使用两个指针，一个快指针，一个慢指针。

#### 面试22 判断一个序列是否为另一个序列的出栈序列
输入两个整数序列，第一个序列为入栈的序列，判断第二个序列是否为第一个序列合法的出栈序列。
**思路**
1. 如果栈顶元素不为当前出栈元素，则顺序遍历输入序列，将当前元素之前的元素都入栈。如果无法找到该元素，则输出错我
2. 如果栈顶元素为当前元素，或者栈为空，则直接出栈或执行下一个元素。
**代码**
```java
 public boolean isStackOut(int[] in, int[] out) {
        if (in.length != out.length) return false;
        Stack<Integer> stack = new Stack<Integer>();
        int i = 0, j = 0;
        for (; j < out.length; j++) {
            while (stack.isEmpty() || stack.peek() != out[j]) {
                if (i >= in.length) return false;
                stack.push(in[i++]);
            }
            stack.pop();
            continue;
        }
        return true;
    }
```

#### 面试26 复杂链表的复制
如果一个链表内不仅含有next指针，还有一个random指针，请完成这个链表的深拷贝。
**思路1**
利用一个Hash表存储<N,N'>对，最后再通过映射的方法来完成所有复制。时间复杂度为O(n)，空间复杂度为O(N)
**思路2**
1. 将新建结点放到原有结点的后面。
![Alt text](/images/Offer Algorithms Summary/1496650732120.png)
2. 根据前一个结点的random指针，对后一个结点的random指针进行修改。
![Alt text](/images/Offer Algorithms Summary/1496650770141.png)
3. 拆分原来的大链表，使其变为两个链表。
![Alt text](/images/Offer Algorithms Summary/1496651008444.png)
```java
private void buildNodes(ListNode node){
	if(node == null) return ;
	ListNode p = node;
	while(p != null){
		ListNode q = new ListNode(p.val);
		q.next = p.next;
		p.next = q;
		p = q.next;
	}
}
private void copyRandomPointer(ListNode node){
	if(node == null) return ;
	ListNode p = node;
	while(p != null){
		if(p.random != null){
			p.next.random = p.random.next;
		}
		p = p.next.next;
	}
}
private ListNode spiltList(ListNode node){
	if(node == null) return null;
	ListNode p = node, q = node.next, newList = node.next;
	while( p != null){
		p.next = q.next;
		p = p.next;
		q.next = p == null ? null : p.next;
		q = q.next;
	}
	return newList;
}

public ListNode copyList(ListNode node){
	buildNodes(node);
	copyRandomPointer(node);
	return spiltList(node);
}
```

#### 面试28 字符的全排序
给出一个字符串，打印出这个字符串所有的排序。
**思路**
1. 将第一个字符与后面的字符依次进行交换。
2. 然后对后面的字符递归调用全排序。
3. 将字符交换回来。
```java
    public static void Permutation(String s) {
        char[] chars = s.toCharArray();
        Permutation(chars, 0);
    }

    private static void Permutation(char[] chars, int begin) {
        if (begin == chars.length - 1 ) {
            System.out.println(Arrays.toString(chars));
        } else {
            for (int i = begin ; i < chars.length; i++) {
                exchagne(chars, begin, i);
                Permutation(chars, begin + 1);
                exchagne(chars, begin, i);
            }
        }
    }

    private static void exchagne(char[] chars, int i, int j) {
        char t = chars[i];
        chars[i] = chars[j];
        chars[j] = t;
    }
```


#### 面试29 数组中此时超过一半的数字
给定一个输入数组，其中一个数字出现次数超过了数组长度的一半，求出该数。
**思路1**
利用Partition的思路，对数组进行划分，不断缩小搜索长度。
1. 如果返回index=n/2，则直接返回对应的数。
2. 如果返回index<n/2，则继续搜索后面的子数组。
3. 如果返回index>n/2，则继续搜索前面的子数组。

**思路2**
因为其中一个数字出现次数比其他数字出现次数总和都多，因此遍历时可以保存上一个数字和出现次数。
1. 如果和上一个次数相同，则加1。
2. 如果和上一个次数不同，则减1。
3. 如果次数等于0，则更换上一个数字，并将次数重新置为1。

```java
public findHalfNumber(int[] arr){
	int result = arr[0];
	int count = 1;
	for(int i = 1; i < arr.length; i++){
		if(count == 0){
			result = arr[i];
			count = 1;
		}else if(arr[i] == result){
			count++;
		}else{
			count--;
		}
	}
	//check whether the result is right number
	return result;
}
```

#### 面试32 从1到n整数中1出现的次数
输入一个整数n，求从1到n所有整数中1出现的次数。
思路：
递归求解，例如21345，可以分为两部分：1~1345,1346~21345来求解。
在求解过程中，可以一次求解：
1. 第一位为1的情况，即10000~19999中一的个数（如果第一位不为1）。
2. 后面几位中为1的情况，可以固定一位，然后其他位任意去0~9
3. 然后递归求解1~1245的情况。

```java
    public int numberOf1(char[] chars, int begin) {
        if (begin >= chars.length) return 0;
        int first = chars[begin] - '0';
        if (begin == chars.length - 1) {
            if (first == 0) return 0;
            else return 1;
        }
        int result = 0;
        //第一位为1的所有数字
        if (first > 1) {
            result += (int) Math.pow(10, chars.length - begin - 1);
        } else if (first == 1) {
            result += Integer.valueOf(String.valueOf(chars, begin + 1, chars.length - begin - 1)) + 1;
        }
        //1346~21345中，除了第一位为1的其他为为1的所有数字,固定某一位为1，其他为取值0~9
        result += first * (chars.length - begin - 1) * (int) Math.pow(10, chars.length - begin - 2);
        //递归求解1~1345的个数
        result += numberOf1(chars, begin + 1);
        return result;
    }
```

#### 面试33 第n个丑数
定义一个数的因子只要2,3,5的数称为丑数，求第n个丑数。
**思路**
第n个丑数肯定是由前n-1个丑数生成的，用三个指针分别指向生成下一个丑数的位置，然后计算最小值来生成丑数，然后更新指针。
```java
public int nthUglyNumber(int n){
	int[] pointer = new int[3]{1,1,1};
	int[] numbers = new int[n+1];
	numbers[1] = 1;
	for(int i = 2; i <= n; i++){
		numbers[i] = Math.min(numbers[pointer[0]]*2,Math.min(numbers[pointer[1]*3],numbers[pointer[2]*5]));
		//更新指针
		pointer[0] = numbers[i] == pointer[0]*2 ? pointer[0]+1:pointer[0];
		pointer[1] = numbers[i] == pointer[1]*3 ? pointer[1]+1:pointer[1];
		pointer[2] = numbers[i] == pointer[2]*5 ? pointer[2]+1 : pointer[2];
	}

}
```

#### 面试37 两个链表的第一个公共结点
两个链表，找到第一个公共结点
**思路**
遍历一遍先求出两个链表的长度，然后让长的链表先走，最后在一起走。


#### 面试40 数组中只出现一次的数
一个整形数组中除了两个数字之外，其它数字都出现了两次，找出这两个值出现一次的数字。
**思路**
1. 将数组中所有的数进行一次异或，获得的是两个只出现一次的数的异或。
2. 找出这个异或结果的某一位的1取出来，作为划分数组的掩码，则两个数会分到不同的子数组。
3. 然后对每个子数组进行一次异或，分别获得每个子数组的只出现一次的数。

```java
public int[] findNumbers(int[] arr){
		
        int data = 0;
        for (int i = 0; i < arr.length; i++) data ^= arr[i];

        int mask = 1;
        while ((data & mask) == 0) mask <<= 1;

        int num1 = 0, num2 = 0;
        for (int i = 0; i < arr.length; i++) {
            if ((arr[i] & mask) != 0) {
                num1 ^= arr[i];
            } else {
                num2 ^= arr[i];
            }
        }
        return new int[]{num1, num2};
}
```

#### 面试41 打印和为n的序列和
给定n，打印序列1~n中和为n的所有连续子序列。
**思路**
1. 定义small和big用来存储序列最小值和序列最大值。
2. 计算small到big的和，如果和为n，则打印序列。
3. 如果和小于n则增大big。
4. 如果和大于n则增大smll。

```java
    public void printSequenceOfSum(int sum) {
        int small = 1, big = 2;
        while (small <= (sum + 1) / 2) {
            int curSum = (small + big) * (big - small + 1) / 2;
            if (curSum == sum) {
                print(small, big);
            }
            if (curSum >= sum) {
                small++;
            } else {
                big++;
            }
        }
    }
```

#### 面试43 n个骰子和的概率分布
给定n个筛子，求所有和的情况的概率分布

**思路：**
1. 两个数组，第一个数组用于保存n-1个骰子时，各个和出现的次数；第二个数组用于保存n个骰子是，各个和出现的次数。
2. 利用n-1,n-2,n-3,n-4,n-5,n-6来计算当前n的的次数。
3. 交换两个数组，进行下一轮迭代。

```java
    private static final int MAX_VALUE = 6;

    public void printProbability(int n) {
        if (n <= 0) return;
        int[][] count = new int[2][n * MAX_VALUE + 1];
        int flag = 0;
        for (int i = 1; i <= MAX_VALUE; i++) count[flag][i] = 1;
        for (int i = 2; i <= n; i++) {
            //小于i的都为0
            for (int j = 0; j < i; j++) count[1 - flag][j] = 0;
            for (int j = i; j <= i * MAX_VALUE; j++) {
                for (int k = 1; k <= j && k <= MAX_VALUE; k++)
                    count[1 - flag][j] += count[flag][j - k];
            }
            flag = 1 - flag;
        }
        double total = Math.pow(MAX_VALUE, n);
        for (int i = n; i <= n * MAX_VALUE; i++) {
            System.out.printf("%d : % .6f\n", i, count[flag][i] / total);
        }
    }
```

