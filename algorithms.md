# 算法

## 排序算法

![](https://images2017.cnblogs.com/blog/849589/201710/849589-20171015233043168-1867817869.png)

- 内部排序：将所有数据读入内存并排序
- 外部排序：排序数据很大，一次不能容纳全部的排序记录，在排序过程需要访问外存
- 稳定/不稳定：A == B，排序前 A 在 B 之前，排序后 A 仍在 B 之前即为稳定
- 比较类排序：通过比较来决定元素间的相对次序，时间复杂度不能突破 O(nlogn)，适用于各种规模的数据，也不需要关注数据的分布，即适用于一切情况
- 非比较类排序：不通过比较元素间的相对次序，而是通过确定每个元素前应有多少元素来排序，因此能够达到 O(n) 的时间复杂度，但需要占用额外空间来确定唯一位置

### 冒泡排序

每次从一端开始，递归与前一个元素比较，若不符合顺序，则将两者的位置交换，否则冒泡元素变更为前一个元素，重复直到不再发生交换

```java
public void bubbleSort(int[] nums) {
    
    int n = nums.length;
    int tmp;
    for(int i = 0; i < n - 1; i++) {
        boolean flag = false;
        for(int j = 0; j < n - 1 - i; j++){
            if(nums[i] > nums[j]){
                tmp = nums[i];
                nums[i] = nums[j];
                nums[j] = tmp;
                flag = true;
            }
        }
        if(!flag) break;
    }
    
}
```

- 稳定性：稳定
- 时间复杂度：最佳 O(n)；最差 O(n^2)； 平均 O(n^2)
- 空间复杂度：O(1)
- 排序方式：内部排序

### 选择排序

无论处理什么数据都是 O(n^2) 的时间复杂度，唯一的好处是不占用额外的内存空间

每次遍历寻找应该放在当前位置的元素，直到遍历所有元素

```java
public void select(int[] nums) {
    
    int len = nums.length;
    // 选择需要填充的位置
    for(int i = 0; i < len; i++) {
        
        int tmp = nums[i];
        // 选择用于填充的元素
        for(int j = i; j < len; j++) {
            
            if(tmp < nums[j]) {
                
                tmp = nums[j];
                
            }
            
        }
        
        nums[j] = nums[i];
        nums[i] = tmp;
        
    }
    
}
```

- 稳定性：不稳定
- 时间复杂度：最佳/最差/平均：O(n^2)
- 空间复杂度：O(1)
- 排序方式：内部排序

### 插入排序

通过构建有序序列，对于未排序数据，在已排序序列中从后往前扫描，找到相应位置并插入

```java
public void insert(int[] nums) {

    int len = nums.length;
    // 遍历未排序数据
    for (int i = 1; i < len; i++) {

        int idx = i;
        // 在已排序数据中寻找插入位置
        for (int j = i - 1; j >= 0; j--) {

            if (nums[j] > nums[idx]) {

                int tmp = nums[idx];
                nums[idx] = nums[j];
                nums[j] = tmp;
                idx = j;

            } else {
                break;
            }

        }

    }

    return;

}
```

- 稳定性：稳定
- 时间复杂度：最佳 O(n)，最差 O(n^2)，平均 O(n^2)
- 空间复杂度：O(1)
- 排序方式：内部排序

### 希尔排序

改进后的插入排序，先将整个待排序的记录序列分割为若干子序列分别进行直接插入排序，待整个序列中的记录基本有序时，再对全体记录进行依次直接插入排序

![](https://images2018.cnblogs.com/blog/1192699/201803/1192699-20180319094116040-1638766271.png)

```java
public void shell(int[] nums) {
    
    int len = nums.length;
    int gap = n / 2;
    while(gap > 0) {
        for(int i = gap; i < n; i++) {
            int current = nums[i];
            int preIndex = i - gap;
            // 对分组进行插入排序
            while(preIndex >= 0 && nums[preIndex] > current) {
                nums[preIndex + gap] = nums[preIndex];
                preIndex -= gap;
            }
            nums[preIndex + gap] = current;
        }
        gap /= 2;
    }
    
}
```

- 稳定性：不稳定
- 时间复杂度：最佳 O(nlogn)，最差 O(n^2)，平均 O(nlogn)
- 空间复杂度：O(1)

### 归并排序

采用分治法，将已有序的子序列合并，得到完全有序的序列

1. 将长度 n 的输入序列分成两个长度为 n / 2 的子序列
2. 分别对这两个子序列进行归并排序，使子序列变为有序状态
3. 设定两个指针，分别指向两个已经排序子序列的起始位置
4. 比较两个指针所指向的元素，选择合适的元素放入合并空间并移动指针
5. 重复步骤 3、4 直到一个指针到达序列尾
6. 将另一序列剩下的所有元素直接复制到合并序列尾

![](https://images2017.cnblogs.com/blog/849589/201710/849589-20171015230557043-37375010.gif)

```java
public void mergeSort(int[] nums) {
    
    // 排序数组长度为一时无法分割
    if(nums.length <= 1) {
        return nums;
    }
    
    // 将排序数组划分为等长的两个子数组
    int mid = nums.length / 2;
    int[] left = Arrays.copyOfRange(nums, 0, mid);
    int[] right = Arrays.copyOfRange(nums, mid, nums.length);
    merge(mergeSort(left), mergeSort(right));
    
}

public int[] merge(int[] left, int[] right) {
    
    // 两者合并后的数组
    int[] result = new int[left.length + right.length];
    // 各个数组的指针
    int res_idx = 0, left_idx = 0, right_idx;
    // 遍历两个数组，选择合适的元素插入合并数组
    while(left_idx < left.length && right_idx < right.length) {
        
        if(left[left_idx] < right[right_idx]) {
            result[res_idx++] = left[left_idx++];
        }
        else {
            result[res_idx++] = right[right_idx++];
        }
        
    }
    // 将剩余的元素全部加入合并数组
    while(left_idx < left.length) {
        result[res_idx++] = left[left_idx++];
    }
    while(right_idx < right.length) {
        result[res_idx++] = right[right_idx++];
    }
    return result;
    
}
```

- 稳定性：稳定
- 时间复杂度：最佳/最差/平均：O(nlogn)
- 空间复杂度：O(n)

### 快速排序

使用分治法，每次选择一个元素，将大于该元素的全部移至一边，将小于该元素的移至另一边，再对两边的元素重复上述步骤，直到不能再分出子序列

```java
public void quickSort(int[] nums, int start, int end) {
    
    if(start < end) {
        // 选择基点
        int p = nums[start];
        // 左指针
        int left = start;
        // 右指针
        int right = end;
        while(left < right) {
            // 从右侧开始遍历寻找到小于基点的元素 
            while(left < right && nums[right] >= p) {
                right--;
            }
            nums[left] = nums[right];
            // 从左侧开始遍历寻找到大于基点的元素
            while(left < right && nums[left] >= p) {
                left++;
            }
            nums[right] = nums[left];
            
        }
        // 此时 left == right，用 p 替换这个位置的数字
        nums[left] = p;
        // 遍历排序左右两个子序列
        quickSort(nums, start, left - 1);
        quickSort(nums, left + 1, end);
    }
    
}
```

- 稳定性：不稳定
- 时间复杂度：最佳/最差/平均：O(nlogn)
- 空间复杂度：O(logn)

### 堆排序

利用堆的数据结构特性进行排序

1. 将待排序的数组构成一个大根堆，此时整个数组的最大值就是堆结构的顶端
2. 将顶端的数与末尾的数交换，此时末尾的数为最大值，剩余待排序数组个数为 n - 1
3. 将剩余的 n - 1 个数再构成大根堆，再将顶端数与 n - 1 位置的数交换，如此反复执行得到有序数组

```java
    //堆排序
    public static void heapSort(int[] arr) {
        //构造大根堆
        heapInsert(arr);
        int size = arr.length;
        while (size > 1) {
            //固定最大值
            swap(arr, 0, size - 1);
            size--;
            //构造大根堆
            heapify(arr, 0, size);
 
        }
 
    }
 
    //构造大根堆（通过新插入的数上升）
    public static void heapInsert(int[] arr) {
        for (int i = 0; i < arr.length; i++) {
            //当前插入的索引
            int currentIndex = i;
            //父结点索引
            int fatherIndex = (currentIndex - 1) / 2;
            //如果当前插入的值大于其父结点的值,则交换值，并且将索引指向父结点
            //然后继续和上面的父结点值比较，直到不大于父结点，则退出循环
            while (arr[currentIndex] > arr[fatherIndex]) {
                //交换当前结点与父结点的值
                swap(arr, currentIndex, fatherIndex);
                //将当前索引指向父索引
                currentIndex = fatherIndex;
                //重新计算当前索引的父索引
                fatherIndex = (currentIndex - 1) / 2;
            }
        }
    }
    //将剩余的数构造成大根堆（通过顶端的数下降）
    public static void heapify(int[] arr, int index, int size) {
        int left = 2 * index + 1;
        int right = 2 * index + 2;
        while (left < size) {
            int largestIndex;
            //判断孩子中较大的值的索引（要确保右孩子在size范围之内）
            if (arr[left] < arr[right] && right < size) {
                largestIndex = right;
            } else {
                largestIndex = left;
            }
            //比较父结点的值与孩子中较大的值，并确定最大值的索引
            if (arr[index] > arr[largestIndex]) {
                largestIndex = index;
            }
            //如果父结点索引是最大值的索引，那已经是大根堆了，则退出循环
            if (index == largestIndex) {
                break;
            }
            //父结点不是最大值，与孩子中较大的值交换
            swap(arr, largestIndex, index);
            //将索引指向孩子中较大的值的索引
            index = largestIndex;
            //重新计算交换之后的孩子的索引
            left = 2 * index + 1;
            right = 2 * index + 2;
        }
 
    }
    //交换数组中两个元素的值
    public static void swap(int[] arr, int i, int j) {
        int temp = arr[i];
        arr[i] = arr[j];
        arr[j] = temp;
    }
```

- 稳定性：不稳定
- 时间稳定性：最佳/最差/平均：O(nlogn)
- 空间复杂度：O(1)

### 计数排序

将输入的数据值转化为 key 存储在额外开辟的数组空间中，作为一种线性时间复杂度的排序，计数排序要求输入的数据必须是有确定范围的整数(用于建立对应大小的数组)

![](https://images2017.cnblogs.com/blog/849589/201710/849589-20171015231740840-6968181.gif)

```java
/**
 * Gets the maximum and minimum values in the array
 *
 * @param arr
 * @return
 */
private static int[] getMinAndMax(int[] arr) {
    int maxValue = arr[0];
    int minValue = arr[0];
    for (int i = 0; i < arr.length; i++) {
        if (arr[i] > maxValue) {
            maxValue = arr[i];
        } else if (arr[i] < minValue) {
            minValue = arr[i];
        }
    }
    return new int[] { minValue, maxValue };
}

/**
 * Counting Sort
 *
 * @param arr
 * @return
 */
public static int[] countingSort(int[] arr) {
    if (arr.length < 2) {
        return arr;
    }
    int[] extremum = getMinAndMax(arr);
    int minValue = extremum[0];
    int maxValue = extremum[1];
    int[] countArr = new int[maxValue - minValue + 1];
    int[] result = new int[arr.length];

    for (int i = 0; i < arr.length; i++) {
        countArr[arr[i] - minValue] += 1;
    }
    for (int i = 1; i < countArr.length; i++) {
        countArr[i] += countArr[i - 1];
    }
    for (int i = arr.length - 1; i >= 0; i--) {
        int idx = countArr[arr[i] - minValue] - 1;
        result[idx] = arr[i];
        countArr[arr[i] - minValue] -= 1;
    }
    return result;
}
```

当输入的元素是 n 个 0-k 之间的整数时，运行时间是 O(n + k)，由于不是比较排序，故速度快于任何比较排序算法

由于用来计数的数组长度取决于待排序数组中数据的范围(最大值 - 最小值 + 1)，使得计数排序对于数据范围很大的数组，需要大量额外内存空间

- 稳定性：稳定
- 时间复杂度：最佳/最差/平均：O(n + k)
- 空间复杂度：O(k)

### 桶排序

计数排序的升级版，利用函数的映射关系，映射函数直接影响排序效率

假设输入数据服从均匀分布，将数据分到有限数量的桶中，每个桶再分别排序

![](https://images2017.cnblogs.com/blog/849589/201710/849589-20171015232107090-1920702011.png)

```java
/**
 * Gets the maximum and minimum values in the array
 * @param arr
 * @return
 */
private static int[] getMinAndMax(List<Integer> arr) {
    int maxValue = arr.get(0);
    int minValue = arr.get(0);
    for (int i : arr) {
        if (i > maxValue) {
            maxValue = i;
        } else if (i < minValue) {
            minValue = i;
        }
    }
    return new int[] { minValue, maxValue };
}

/**
 * Bucket Sort
 * @param arr
 * @return
 */
public static List<Integer> bucketSort(List<Integer> arr, int bucket_size) {
    if (arr.size() < 2 || bucket_size == 0) {
        return arr;
    }
    int[] extremum = getMinAndMax(arr);
    int minValue = extremum[0];
    int maxValue = extremum[1];
    int bucket_cnt = (maxValue - minValue) / bucket_size + 1;
    List<List<Integer>> buckets = new ArrayList<>();
    for (int i = 0; i < bucket_cnt; i++) {
        buckets.add(new ArrayList<Integer>());
    }
    for (int element : arr) {
        int idx = (element - minValue) / bucket_size;
        buckets.get(idx).add(element);
    }
    for (int i = 0; i < buckets.size(); i++) {
        if (buckets.get(i).size() > 1) {
            buckets.set(i, sort(buckets.get(i), bucket_size / 2));
        }
    }
    ArrayList<Integer> result = new ArrayList<>();
    for (List<Integer> bucket : buckets) {
        for (int element : bucket) {
            result.add(element);
        }
    }
    return result;
}
```

- 稳定性：稳定
- 时间复杂度：最佳 O(n + k)，最差 O(n^2)，平均 O(n + k)
- 空间复杂度：O(n + k)

### 基数排序

对元素中的每一位数字进行排序，从最低位开始排序，然后收集，再按照高位排序再收集，直到最高位

![](https://images2017.cnblogs.com/blog/849589/201710/849589-20171015232453668-1397662527.gif)

```java
/**
 * Radix Sort
 *
 * @param arr
 * @return
 */
public static int[] radixSort(int[] arr) {
    if (arr.length < 2) {
        return arr;
    }
    // 取最大值的位数为迭代次数
    int N = 1;
    int maxValue = arr[0];
    for (int element : arr) {
        if (element > maxValue) {
            maxValue = element;
        }
    }
    while (maxValue / 10 != 0) {
        maxValue = maxValue / 10;
        N += 1;
    }
    for (int i = 0; i < N; i++) {
        // 初始化收集数组
        List<List<Integer>> radix = new ArrayList<>();
        for (int k = 0; k < 10; k++) {
            radix.add(new ArrayList<Integer>());
        }
        // 对各个元素基数进行排序
        for (int element : arr) {
            int idx = (element / (int) Math.pow(10, i)) % 10;
            radix.get(idx).add(element);
        }
        int idx = 0;
        for (List<Integer> l : radix) {
            for (int n : l) {
                arr[idx++] = n;
            }
        }
    }
    return arr;
}
```

- 稳定性：稳定
- 时间复杂度：最佳/最差/平均：O(n*k)
- 空间复杂度：O(n + k)

## 经典算法

- 贪心算法：局部最优 → 全局最优
- 动态规划：每个状态由上一个状态推导而来
- 回溯算法：在搜索尝试过程中寻找问题的解，当发现不满足求解条件时就回溯返回(穷举)
- 分治算法：将问题分解为若干个规模较小的子问题，子问题相互独立且与原问题性质相同