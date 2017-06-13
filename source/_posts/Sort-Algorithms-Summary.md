---
title: 排序算法总结
date: 2017-03-14 10:47:04
categories: Algorithms
tags: 
- Sort
-  Algorithms
---
### 比较排序
#### 选择排序(Selection Sort)
**思路**：
　每次选择最小的元素与第i个元素进行交换
**特点**：
- 运行时间与输入无关，即使输入数据主键相同或有序，时间仍和最差相同，时间复杂度为 $N^2$
- 数据移动最少，仅需要$N$次交换
- 在第i次，会进行1次exchange和$N-i-1$次比较，总的比较次数为$N(N-1)/2$次
- 该算法为不稳定算法，因为有可能将

**代码**：
```java
    public static void sort(int[] arr) {
        int n = arr.length;
        for (int i = 0; i < n; i++) {
            int min = i;
            //从第i+1个开始寻找最小值
            for (int j = i + 1; j < n; j++) {
                if (arr[j] < arr[min]) min = j;
            }
            SortUtils.exchane(arr, i, min);
        }
    }
```
#### 插入排序(Insertion Sort)
**思路**
　将第i个元素插入到前面有序元素中合适的位置
**特点**
- 最好情况下，即已经有序，则需要$N-1$次比较，$N$次移动
- 最差情况下，即完全逆序，需要$N^2$次比较和$N^2$次交换
- 平均情况下，需要$1/4*N^2$次比较和$1/4*N^2$次交换
- 交换次数和倒置数相同，需要就比较次数大于倒置数量，小于等于倒置的数量加上数组的大小再减一

***倒置：***数组中两个顺序颠倒的元素(可重复计数)

**代码**
```java
    public static void sort(int[] arr) {
        int n = arr.length;
        for (int i = 1; i < n; i++) {
            //从i-1个开始交换，直至到正确的位置
            for (int j = i - 1; j >= 0 && arr[j + 1] < arr[j]; j--)
                SortUtils.exchane(arr, j + 1, j);
        }
    }
```

#### 折半查找插入排序（BinaryInsertionSort)
**思路**
在查找第i个元素在前面有序元素的位置时，采用二分查找的策略。
**特点：**
- 最差情况下，也是需要$N^2$的时间复杂度。
- 最好情况下，需要$O(nlgn)$的时间复杂度。

**代码**
```java
    public static void sort(int[] arr) {
        int n = arr.length;
        for (int i = 1; i < n; i++) {
            int key = arr[i];
            //查询需要插入的位置
            int index = binarySearch(arr, 0, i - 1, key);
            //移动数据
            for (int j = i - 1; j >= index; j--) arr[j + 1] = arr[j];
            arr[index] = key;
        }
    }

    private static int binarySearch(int[] arr, int i, int j, int key) {
        int low = i, high = j, mid;
        while (low <= high) {
            mid = low + (high - low) / 2;
            if (arr[mid] == key) {
                return mid;
            } else if (arr[mid] < key) {
                low = mid + 1;
            } else {
                high = mid - 1;
            }
        }
        return low;
    }
```

#### 希尔排序(Shell Sort)
**思路**
　选取一个序列作为步长，然后通过大步长排序将元素移动到较远的地方（原始插入排序为一步步移动），从而使数组局部有序，最终再利用插入排序进行全局排序。
**特点**
 - 算法性能也取决于步长数组的选取
 - 在最坏的情况下，算法的复杂度和$N^{3/2}$

**代码**
```java
 public static void sort(int[] arr) {
        int n = arr.length;
        int step = n / 2;
        //不断缩小step直至为1
        while (step >= 1) {
            //对每一组进行排序，一共step组
            for (int i = 0; i < step; i++) {
                //对每一组从第二个开始插入排序
                for (int j = i + step; j < n; j += step) {
                    //一直交换到合适的位置
                    for (int k = j - step; k >= 0 && arr[k + step] < arr[k]; k -= step)
                        SortUtils.exchange(arr, k + step, k);
                }
            }
            step = step / 2;
        }
    }
```
#### 冒泡排序
**思路**
每一次冒泡过程两两比较，如果发生逆序，则进行交换。总共需要n-1次冒泡，地k次冒泡需要比较n-k次。
**特点**
- 时间复杂度为$O(N^2)$，空间复杂度为$O(1)$
- 可以使用一个标志位，如果没有产生交换，则直接返回。

**代码**
```java
public static void sort(int[] arr) {
        int n = arr.length;
        //冒泡n-1次
        for (int i = 0; i < n - 1; i++) {
            //改进，如果当前次已经没有冒泡，则直接返回
            boolean flag = true;
            //每次冒出一个最大值到最后边
            for (int j = 0; j < n - i - 1; j++)
                if (arr[j] > arr[j + 1]) {
                    //交换相邻位置的
                    SortUtils.exchange(arr, j, j + 1);
                    flag = false;
                }
            if (flag)
                return;
        }
    }
```

#### 快速排序
**思路**
　选取一个切分元素，将原始元素按照此标准进行切分为两个子数组，分别对子数组进行排序。快速排序和归并排序区别：归并排序是在递归后对数组进行操作，而快速排序则在递归钱对数组进行操作。
**简单的切分**
	选取切分元key，索引i，j分别为序列开始，索引j不断后移，当遇到a[j]<=key时，则交换a[j]和a[i+1]，最后交换a[i]和key。这样索引i的左侧都为<=key，右侧都为>key。
![Alt text](/images/Sort Algorithms Summary/1489046415296.png)
```java
    private static int simplePartition(int[] arr, int start, int end) {
        //默认选取第一个元素为切分元
        int key = arr[start];
        int i = start;
        //j后移
        for (int j = start + 1; j <= end; j++) {
            //遇到小于切分元的key，则交换
            if (arr[j] <= key)
                SortUtils.exchange(arr, ++i, j);
        }
        SortUtils.exchange(arr, i, start);
        return i;
    }
```
**双工切分**
　选定a[lo]作为切分元素，使用指针i从左向右扫描比切分元素大的元素，指针j从右向左扫描比切分元素小的元素，交换两个元素的位置，重复上述过程，直至i和j相等，最后将a[lo]和a[j]交换，并返回j的索引。这种方法比上述方法交换次数较少。
![Alt text](/images/Sort Algorithms Summary/1489050265957.png)

```java
    private static int twoWayPartition(int[] arr, int start, int end) {
        //默认选择第一个元素为切分元
        int key = arr[start];
        int i = start + 1, j = end;
        while (i <= j) {
            //向右寻找大的元素
            while (i <= j && arr[i] <= key) ++i;
            //向左寻找小的元素
            while (i <= j && arr[j] >= key) --j;
            //交换元素
            if (i < j) {
                SortUtils.exchange(arr, i, j);
                ++i;
                --j;
            }
        }
        //将切分元移到中间
        SortUtils.exchange(arr, start, j);
        return j;
    }
```
**半双工切分**
	与上述类似，但每次扫描完一边时就赋值数据，不需要交换数据。
![Alt text](/images/Sort Algorithms Summary/1489051255182.png)
```java
    private static int oneWayPartition(int[] arr, int start, int end) {
        int key = arr[start];
        int i = start, j = end;
        while (i < j) {
            //向左寻找最小元素
            while (i < j && arr[j] >= key) j--;
            //覆盖元素
            if (i < j) arr[i++] = arr[j];
            //向右寻找最大元素
            while (i < j && arr[i] <= key) i++;
            //覆盖元素
            if (i < j) arr[j--] = arr[i];
        }
        arr[j] = key;
        return j;
    }
```
　
**特点**
- 将长度为Ｎ的无重复数组进行排序，快速排序平均需要～$N*lnN$此比较和$1/6N*lnN$次数据交换。
- 快速排序最多需要约为$N^2/2$次比较，此时就退化为冒泡排序。

**优化**
- 将小数组切换到插入排序
- 三取样排序，选取三个样本，基本为a[0],a[mid],a[high]三个，并选取中间数作为切分元素。
- 随机划分元，将最右边的元素与数组的随机一个元素进行交换，防止运行时间不依赖于输入序列的顺序。

```java
    private static int randomPartition(int[] arr, int start, int end) {
        //选取随机元素索引
        int randomIndex = new Random().nextInt(end - start + 1) + start;
        //交换
        SortUtils.exchange(arr, start, randomIndex);
        return simplePartition(arr, start, end);
    }
```
- 使用算法导论上的算法，选取中位数。

**代码**
```java
    public static void sort(int[] arr) {
        sort(arr, 0, arr.length - 1);
    }

    private static void sort(int[] arr, int start, int end) {
        if (start >= end) return;
        //划分数组
//        int index = simplePartition(arr, start, end);
//        int index = twoWayPartition(arr, start, end);
        int index = randomPartition(arr, start, end);
        //递归排序
        sort(arr, start, index - 1);
        sort(arr, index + 1, end);
    }
```

#### 归并排序
##### 自顶向下的归并排序
**思路**
　先将数组的两个子数组进行排序，然后进行归并。采用递归的思想，即不断递归左子数组，使其有序，再递归右子数组。
**特点**
- 对于长度为$N$的任意数组，自顶向下的归并排序需要$1/2*N*lgN$（最好情况，左数组最大值比右数组最小值小）至$NlgN$（最坏情况，左数组和右数组交叉排序）次比较。证明思路：$C(2^n)=2C(2^{n-1})+ 2^n$==>$C(2^n)/2^n=C(2^{n-1})/2^{n-1}+ 1$==>$C(2^n)/2^n=C(2^0)/2^0+ n$==>$C(N)=C(2^n)=n2^n=NlgN$
- 对于长度为$N$的任意数组，自顶向下的归并排序需要$6*N*lgN$次比较，其中2N次复制，2N次移动回去，2N次比较时访问

**代码**
```java
    public static void sort(int[] arr) {
        //创建一个临时数组
        int[] temp = new int[arr.length];
        sort(arr, 0, arr.length - 1, temp);
    }

    public static void sort(int[] arr, int start, int end, int[] temp) {
        if (start >= end) return;
        int mid = start + (end - start) / 2;
        //左侧数组排序
        sort(arr, start, mid, temp);
        //右侧数组排序
        sort(arr, mid + 1, end, temp);
        //合并
        merge(arr, start, mid, end, temp);
    }

    public static void merge(int[] arr, int start, int mid, int end, int[] temp) {
        int i = start, j = mid + 1, k = start;
        while (i <= mid && j <= end) {
            if (arr[i] <= arr[j]) {
                temp[k++] = arr[i++];
            } else {
                temp[k++] = arr[j++];
            }
        }
        while (i <= mid) temp[k++] = arr[i++];
        while (j <= end) temp[k++] = arr[j++];
        //复制回原数组
        for (k = start; k <= end; k++) {
            arr[k] = temp[k];
        }
    }
```

##### 自底向上的归并排序
**思路**
　先对微型数组归并，例如相邻的元素，在得到子数组。此时不需要采用递归。例如2==>4==>8==>...的思路
**特点**
- 自底向上的归并排序，其比较次数和访问数组次数都相同。但可以节省调用方法的时间

**代码**
```java
    public static void downToUpSort(int[] arr) {
        int n = arr.length;
        int[] temp = new int[n];
        //从最低的1开始，依次向上归并，每次步长为上次的2倍
        for (int step = 1; step < n; step += step) {
            for (int low = 0; low < n - step; low += (step * 2))
                merge(arr, low, low + step - 1, Math.min(low + step * 2 - 1, n - 1), temp);
        }
    }
```
**优化**
- 对小规模子数组采用插入排序
- 不将元素复制到辅助数组，采用角色互换的方法，减少一次复制的过程

#### 堆排序
**相关概念**
	堆有序：当一颗二叉树的每个节点都大于等于它的子节点时，它被称为堆有序。
	上浮：因为一个节点比其父节点大而产生无序状态，需交换该节点与父节点的位置。将新元素插入到数组末尾，然后增加堆的大小，再让该元素上浮到合适的位置。
　下沉：因为一个节点比其子节点小而产生无序状态，需交换该节点与子节点中较大的位置。删除头部元素，再让数组末端元素放到顶端，减小堆的大小，让该元素下沉到合适位置。
**思路**
   将待排序元素构造成一个大顶堆，每次将堆顶元素和最后一个元素互换，然后再维护前n-1个元素形成大顶堆。
**特点**
- 建堆的复杂度政委$O(n)$，每一次调整堆的复杂度为$O(lgN)$，因此总的复杂小于$O(n) + O(nlgn)$

**代码**
```java
    public static void sort(int[] arr) {
        int n = arr.length;
        //调整所有非叶子结点，形成最大堆
        for (int i = n / 2 - 1; i >= 0; i--)
            adjustHeap(arr, i, n - 1);
        for (int i = n - 1; i >= 0; i--) {
            //取出堆顶元素到最后
            SortUtils.exchange(arr, 0, i);
            //调整前面的元素仍称为最大堆
            adjustHeap(arr, 0, i - 1);
        }
    }

    private static void adjustHeap(int[] arr, int start, int end) {
        int dad = start;
        int son = dad * 2 + 1;
        while (son <= end) {
            //选中son中较大的结点
            if (son < end && arr[son] < arr[son + 1]) ++son;
            if (arr[dad] < arr[son]) {
                //dad小于son则调整
                SortUtils.exchange(arr, dad, son);
                dad = son;
                son = dad * 2 + 1;
            } else {
                //如果不需要调整，则直接退出
                break;
            }
        }
    }
```
**优化**
- 多叉堆：对于完全三叉树，数组对应索引需要进行修改。父节点为k/3，子节点为3k-1,3k,3k+1
- 调整数组大小：不采用固定数组大小的模式，而是采用前面实现栈的方式，动态调整数组大小。

### 其它排序
#### 计数排序
**思路**
建立一个排序序列取值范围大小的数组C，然后根据C的下标，统计序列中每个元素出现的次数，然后累加每个元素之前元素的个数到该元素所在C中。最后根据C得到最后的排序序列。
**特点**
计数排序的限制（待排序序列的取值范围为k）：T(n) = (n + k)，
- 如果k < n，T(n)  = O(n);
- 否则，如果k < nlgn，T(n) = O(nlgn);
- 否则，如果k > n^2，T(n) = O(n^2);

**代码**
```java
    public static void sort(int[] arr, int max) {
        int n = arr.length;
        int[] count = new int[max + 1];
        for (int i = 0; i <= max; i++) count[i] = 0;
        //计数
        for (int i = 0; i < n; i++) count[arr[i]]++;
        //累加
        for (int i = 0; i < max; i++) count[i + 1] += count[i];
        //排序
        int[] temp = new int[n];
        for (int i = n - 1; i >= 0; i--) {
            count[arr[i]]--;
            temp[count[arr[i]]] = arr[i];
        }
        //复制回原数组
        System.arraycopy(temp, 0, arr, 0, n);
    }
```

#### 桶排序
**思路**
1. 将所有数的取值范围均匀划分成M个桶。
2. 将N个元素分不到各个桶当中去。
3. 对每个桶进行排序。
4. 再依次将各个桶内元素取出，即为有序的序列。
![Alt text](/images/Sort Algorithms Summary/1489110832115.png)
**特点**
时间复杂度为$O(N+C)$，$O(C)=O(M(N/M)log(N/M))=O(NlogN-NlogM)$，空间复杂度为$O(N+M)$，算法是稳定的，且与初始序列无关。

### 各类算法复杂度汇总
![Alt text](/images/Sort Algorithms Summary/1489115690357.png)
***参考链接***
[排序算法总结](https://segmentfault.com/a/1190000004994003#articleHeader40)
[ 面试算法之排序算法集锦](http://blog.csdn.net/anonymalias/article/details/11547039)
