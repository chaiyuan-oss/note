### 1.写函数创建长度为10的数组，数组中的元素为递增的奇数，首项为1.
{% reveal %}
```php
<?php
    function arrsort($first,$length){

        $arr = array();
        for($i=$first;$i<=$length;$i++){

            $arr[] = $i*2-1;
        }
        return $arr;
    }

    $arr1 = arrsort(1,10);
    print_r($arr1);
```
{% endreveal %}

### 2.创建长度为10的数组，数组中的数为递增的等比数，比值为3，首项为1.
{% reveal %}
```php
<?php
    //$num为比值
    function arrsort($first,$length,$num){

          $arr= array();
          for($i=$first;$i<=$length;$i++){

                //pow($num,$i-2);返回$num的($i-2)次方
                $arr[] = $num*pow($num,$i-2);
          }
          return $arr;
    }

    $arr1 = arrsort(1,10,3);
    print_r($arr1);
```
{% endreveal %}

### 3.求数组中最大数的下标.
{% reveal %}
```php
<?php
    function maxkey($arr){

    $maxval = max($arr);
    foreach($arr as $key=>$val){

        if($maxval == $val){

            $maxkey = $key;
        }
    }
    return $maxkey;
}

$arr = array(0,-1,-2,5,"b"=>15,3);
echo maxkey($arr);
```
{% endreveal %}

### 4.创建一个长度为10的数组，数组中的元素满足斐波拉契数列的规律.
( 斐波那契数列，又称黄金分割数列，指的是这样一个数列：1、1、2、3、5、8、13、21、……在数学上，斐波纳契数列以如下被以递归的方法定义：F0=0，F1=1，Fn=F(n-1)+F(n-2)（n>=2，n∈N*）. 特别指出：第0项是0，第1项是第一个1。)
{% reveal %}
```php
<?php

function arrFibo($len){

    $arr[0] = 0;
    $arr[1] = 1;
    for($i=2;$i<$len;$i++){

        $arr[$i] = $arr[$i-1]+$arr[$i-2];
    }
    return $arr;
}

echo "<pre>";
print_r(arrFibo(10));
echo "</pre>";
```
{% endreveal %}

### 5.计算数组中最大数和最小数的差.
{% reveal %}
```php
<?php
//max/min
function arrsub($arr){

    $maxval = max($arr);
    $minval = min($arr);
    $sub = $maxval - $minval;

    return $sub;
}

$arr = array(-1,-2,100);

echo arrsub($arr);

//sort把元素按从小到大排序/rsort吧元素按从大到小排序
function arrsub($arr){

    sort($arr);
    $min = $arr[0];

    rsort($arr);
    $max = $arr[0];

    $sub = $max - $min;

    return $sub;
}

$arr = array(-1,-2,100);

echo arrsub($arr);
```
{% endreveal %}

### 6.写一个方法，将一个长度超过10的数组最后5项直接截取，不改变顺序变为前5项，如{1,2,3,4,5,6,7,8,9,10}变为{6,7,8,9,10,1,2,3,4,5}.
{% reveal %}
```php
<?php

function arrsort($arr){

    $num = count($arr);

    if($num > 10){

        //array_slice($arr,起始位置,截取长度,保留索引(默认为false))
        $arr_firstpart = array_slice($arr,0,$num-5,true);
        $arr_lastpart = array_slice($arr,($num-5),5,true);
    }else{

        echo "数组不超过10个元素,请重新输入";
        exit();
    }

    //拼接
    $arr_new = array_merge($arr_lastpart,$arr_firstpart);

    return $arr_new;
}

$arr = array("a"=>1,2,3,8,9,6,"b"=>5,-1,"c"=>8,0,7);

echo "<pre>";

print_r($arr);

echo "<br>= = = = = 拼接后 = = = = <br><br>";

print_r(arrsort($arr));

echo "</pre>";
```
{% endreveal %}

### 7.将两个数组连接成一个新数组.
{% reveal %}
```php
//使用array_merge()函数
array_merge($arr1,$arr2);

//使用array_merge_recursive()函数递归追加数组
//array_merge_recursive() 函数与 array_merge() 函数 一样，将一个或多个数组的元素的合并起来，一个数组中的值附加在前一个数组的后面。并返回作为结果的数组。
//但是，与 array_merge() 不同的是，当有重复的键名时，值不会被覆盖，而是将多个相同键名的值递归组成一个数组
<?php

    $arr = array("a"=>1,"b"=>2,3);
    $arr2 = array("a"=>Dee,3,5);

    $arr3 = array_merge($arr,$arr2);
    $arr4 = array_merge_recursive($arr,$arr2);

    echo "<pre>";
    print_r($arr3);

    echo "<br> = = = = = <br><br>";

    print_r($arr4);
    echo "</pre>";

<?php

function arrsort($arr1,$arr2){

    $arr_new = $arr1;

    foreach($arr2 as $key=>$val){

            $arr_new[] = $val;
    }

    return $arr_new;
}

$arr1 = array("a"=>1,"b"=>2,3);
$arr2 = array("a"=>Dee,"c"=>3,5);

echo "<pre>";
print_r(arrsort($arr1,$arr2));
echo "</pre>";

```
{% endreveal %}

### 8.数组逆序( 不能使用rsort函数，不能生成新数组 )
{% reveal %}
```php
<?php

$arr = array("a","b","c",1,10);
$i = "";//要替换位置的数的下标
$j = "";//临时变量
$k = "";//被替换位置的数的下标

$len = count($arr);
$half_len = floor($len/2);//向下取整，取整的值是循环的次数

for($i=0;$i<$half_len;$i++){

    $j = $arr[$i];

    //判断数组个数奇偶
    if($len%2!=0){ //奇数

        $k = $half_len*2-$i;
    }else{

        //偶数
        $k = $half_len*2-$i-1;
    }

    $arr[$i] = $arr[$k];
    $arr[$k] = $j;
}

echo "<pre>";
print_r($arr);
echo "</pre>";
```
{% endreveal %}