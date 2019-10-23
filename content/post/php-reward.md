---
layout: post
title: PHP游戏抽奖应用
date: 2015-10-31
author: opso
tags:
- php
cover: "/images/php_logo.jpg"
---

PHP游戏概率中的运用

<!--more-->

### 概率累加

```php
<?php
function get_rand($proArr) { 
    $result = ''; 
    //概率数组的总概率精度 
    $proSum = array_sum($proArr); 
    //概率数组循环 
    foreach ($proArr as $key => $proCur) { 
        $randNum = mt_rand(1, $proSum); 
        if ($randNum <= $proCur) { 
            $result = $key; 
            break; 
        } else { 
            $proSum -= $proCur; 
        } 
    } 
    unset ($proArr); 
    return $result; 
} 
//test
$p = array('a'=>50,'b'=>50,'c'=>50);
for($i = 0; $i<100; $i++){
	$arr[] =  get_rand($p);
}
$arr1 = array_count_values($arr);
ksort($arr1);
print_r($arr1);

```
