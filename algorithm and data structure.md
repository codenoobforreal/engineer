# Algorithm and Data Structure

## Algorithm

### Sorting

#### 冒泡排序

将数组中的元素进行两两比对，根据比较结果，调换两个元素的位置，重复这一操作，直到完成整个数组的排序

```js
// 简单版本，以数组由小到大进行排序为例
function bubbleSort(arr) {
    for (let i = 0; i < array.length; i++) {
        // 由于每一次外层循环都能确定一个最大或最小数，所以后续内层循环则不再需要对比所有元素
        for (let j = 0; j < array.length - i - 1; j++) {
            if (arr[j] > arr[j + 1]) {
                // 使用数组解构赋值语法交换两个元素位置
                [arr[j], arr[j + 1]] = [arr[j + 1], arr[j]]
            }
        }
    }
    return arr;
}
```

简单版本实现存在一些问题，即在对比过程中，数组有可能已经完成了排序，此时外层循环还没有走完的情况，造成了不必要的元素对比操作，可以进行优化

```js
function bubbleSort(arr) {
    let swapped;
    for (let i = 0; i < array.length; i++) {
        swapped = false;
        for (let j = 0; j < array.length - i - 1; j++) {
            if (arr[j] > arr[j + 1]) {
                [arr[j], arr[j + 1]] = [arr[j + 1], arr[j]];
                swapped = true;
            }
        }
        // 如果并没有发现元素交换则意味着数组已经完成了排序，则跳出外层循环
        if(!swapped) break;
    }
    return arr;
}
```

冒泡排序的时间复杂度由数组情况决定，如果数组刚好从小到大排列则不需要进行交换操作，如果数组刚好从大到小排列，则需要进行(n-1)+(n-2)+...+1次数的交换，记作O(n2)

#### 归并排序

![归并排序](/assets/images/merge%20sort.jpg "归并排序")

这种排序采取了分治的思想，将问题拆分成小问题，得到小问题的解决方案，再形成完成问题的解决方案。

```js
function mergeSort(arr) {
    if (arr.length < 2) {
        return arr;
    }
    const middle = Math.floor(arr.length / 2);
    const left = arr.slice(0, middle);
    const right = arr.slice(middle);
    return merge(mergeSort(left), mergeSort(right));
}

// 合并两个数组并进行排序
function merge(left, right) {
    // 这里是从小到大的排序，如果是从大到小则应该将反向操作
    const result = [];
    while (left.length && right.length) {
        if (left[0] <= right[0]) {
            result.push(left.shift())
        } else {
            result.push(right.shift())
        }
    }
    while (left.length) result.push(left.shift());
    while (right.length) result.push(right.shift());
    return result;
}
```

在执行`merge`方法时，如果两个数组都是n/2个元素，那么最坏情况就需要比较n次，记为n，我们把T(n/2)表示排序n/2个元素要花的时间，那么排序n个元素的总时间为2*T(n/2)+n。

层级 | 每个子序列的元素个数 | 时间表达式
-----|----------------------|--------------
1    | n                    | T(n)
2    | n/2                  | 2*T(n/2)+n
3    | n/4                  | 4*T(n/4)+2n
4    | n/8                  | 8*T(n/8) + 3n
最后一层| 1 | n*T(1) + (logn)*n|

因为T(1)为0即不需要花费时间排序，所以T(n)=(logn)*n

## Data Structure

### tree

#### 二叉树遍历

![二叉树遍历](/assets/images/tree%20travel%20order.jpg "二叉树遍历")

```js
// 前序遍历
// 可以将处理语句放在不同的位置可以达成不同的遍历顺序要求
function travelTreeNode(node) {
    if (!node) {
        return;
    }
    // 视为处理当前节点的操作
    console.log(node.value);
    travelTreeNode(node.left);
    travelTreeNode(node.right);
}
```
