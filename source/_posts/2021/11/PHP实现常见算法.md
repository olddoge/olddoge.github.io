---
title: PHP实现常见算法
date: 2021-11-14 18:54:23
tags: [PHP, 算法]
categories: PHP
description: PHP 常见算法整理
---

# 冒泡排序

> 比较相邻的元素。如果第一个比第二个大，就交换他们两个。对每一对相邻元素作同样的工作，从开始第一对到结尾的最后一对。这步做完后，最后的元素会是最大的数。针对所有的元素重复以上的步骤，除了最后一个

```php
function bubbleSort($arr)
{
  	for($i=0;$i<count($arr);$i++) {
      	for($j=$i+1;$j<count($arr); $j++) {
          	if ($arr[$j] > $arr[$i]) {
              	$arr[$j] = $arr[$j] + $arr[$i];
              	$arr[$i] = $arr[$j] - $arr[$i];
              	$arr[$j] = $arr[$j] - $arr[$i];
            }
        }
    }
  	return $arr;
}
```

# 快速排序

> 从数列中挑出一个元素，作为基准;重新排序数列，所有元素比基准值小的摆放在基准前面，所有元素比基准值大的摆在基准的后面（相同的数可以到任一边）。在这个分区退出之后，该基准就处于数列的中间位置。递归地把小于基准值元素的子数列和大于基准值元素的子数列排序；

```php
function quickSort($arr)
{
    $arrLength = count($arr);
    if ($arrLength <= 1) {
        return $arr;
    }
    $divider = $arr[0]; // 获取中间值
    $left = [];
    $right = [];
    for ($i=1; $i<$arrLength; $i++) {
        if ($divider < $arr[$i]) {
            $right[] = $arr[$i];    // 小于中间值的放右边
        } else {
            $left[] = $arr[$i];     // 大于中间值的放左边
        }
    }
    // 递归排好另外两侧
    $left = quickSort($left);
    $right = quickSort($right);
    
    return array_merge($left, [$divider], $right);
}
```

# 选择排序

> 首先在未排序序列中找到最小（大）元素，存放到排序序列的起始位置。再从剩余未排序元素中继续寻找最小（大）元素，然后放到已排序序列的末尾。重复第二步，直到所有元素均排序完毕。

```php
function selectSort($arr)
{
    $len = count($arr);
    for ($i = 0; $i < $len - 1; $i++) {
        $minIndex = $i;
        for ($j = $i + 1; $j < $len; $j++) {
            if ($arr[$j] < $arr[$minIndex]) {
                $minIndex = $j;
            }
        }
        $temp = $arr[$i];
        $arr[$i] = $arr[$minIndex];
        $arr[$minIndex] = $temp;
    }
    return $arr;
}
```

# 二维数组根据某个值来排序

```php
$arr=[
    [
        'name'=>'StuA',
        'age'=>28
    ],
    [
        'name'=>'StuB',
        'age'=>14
    ],
    [
        'name'=>'StuC',
        'age'=>59
    ],
    [
        'name'=>'StuD',
        'age'=>23
    ],
    [
        'name'=>'StuE',
        'age'=>23
    ],
    [
        'name'=>'StuF',
        'age'=>21
    ],
];
// 根据 age 键来排序
array_multisort(array_column($arr,'age'),SORT_DESC,$arr);
print_r($arr);
```