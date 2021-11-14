---
title: PHP常用代码片段
date: 2021-11-13 21:20:21
tags: [PHP]
categories: PHP
description: PHP常用代码片段记录
---

### 1.获取上一周/上月/前几天时间
```php
date("Y-m-d",strtotime("-1 week")); 
date("Y-m-d",strtotime("-1 month"));
date("Y-m-d",strtotime("-1 day"));
```

### 2.二维数组去重
```php
function remove_duplicate($arr,$key)
{
	// 建立一个目标数组 
	$result = []; 
	foreach ($arr as $value){ 
		// 查看有没有重复项 
		$if_exit = isset($result[$value[$key]]); 
		if ($if_exit) { 
			unset($value[$key]); 
		} else { 
			$result[$value[$key]] = $value; 
		}
	}
	foreach($result as $key => $row) {
		$result[] = $row; 
		unset($result[$key]); 
	}
	return $result; 
}
```

### 3.多维数组中按 7 个分成一组
```php
$memberList = []; 
$length = ceil(count($allMember));
for ($i = 0; $i < $length; $i++) {
	$memberList[] = array_slice($allMember, $i * 7, 7);
}
```

### 4.二维数组按特定值排序
```php
$data = [
	[ 'id' => 5698, 'first_name' => 'Bill', 'last_name' => 'Gates'], 
	[ 'id' => 4767, 'first_name' => 'Steve', 'last_name' => 'Aobs'],
	[ 'id' => 3809, 'first_name' => 'Mark', 'last_name' => 'Zuckerberg']
]; 
//根据字段last_name对数组$data进行降序排列 
$last_names = array_column($data,'last_name'); 
array_multisort($last_names,SORT_DESC,$data);
var_dump($data);
```

### 5.将中文拆分成数组
```php
function mb_str_split($str) 
{
	return preg_split('/(?<!^)(?!$)/u', $str); 
}
```

### 6.模拟发送 POST 请求
```php
function curl_post($url , $data=[])
{ 
	$ch = curl_init(); 
	curl_setopt($ch, CURLOPT_URL, $url); 	// 要访问的地址
	curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1); 	// 获取的信息以文件流的形式返回
	curl_setopt($ch, CURLOPT_SSL_VERIFYPEER, FALSE); 	// 对认证证书来源的检查
	curl_setopt($ch, CURLOPT_SSL_VERIFYHOST, FALSE); 	// 从证书中检查SSL加密算法是否存在
	// POST数据 
	curl_setopt($ch, CURLOPT_POST, 1); 
	// 把post的变量加上 
	curl_setopt($ch, CURLOPT_POSTFIELDS, $data); 
	$output = curl_exec($ch); 
	curl_close($ch); 
	return $output; 
}
```

### 7.美化输出变量
```php
function p($data){
    // 定义样式
    $str = '<pre style="display: block;padding: 9.5px;margin: 44px 0 0 0;font-size: 13px;line-height: 1.42857;color: #333;word-break: break-all;word-wrap: break-word;background-color: #F5F5F5;border: 1px solid #CCC;border-radius: 4px;">';
    // 如果是boolean或者null直接显示文字；否则print
    if (is_bool($data)) {
        $show_data = $data ? 'true' : 'false';
    } elseif(is_null($data)) {
        $show_data = 'null';
    } else {
        $show_data = print_r($data,true);
    }
    $str .= $show_data;
    $str .= '</pre>';
    echo $str;
}
```

### 8.支持中文的字符串截取
```php
/**
 * 字符串截取，支持中文和其他编码
 * @param string $str 需要转换的字符串
 * @param string $start 开始位置
 * @param string $length 截取长度
 * @param string $suffix 截断显示字符
 * @param string $charset 编码格式
 * @return string
 */
function re_substr($str, $start=0, $length, $suffix=true, $charset="utf-8")
{
    if(function_exists("mb_substr")) {
		$slice = mb_substr($str, $start, $length, $charset);
	} elseif(function_exists('iconv_substr')) {
        $slice = iconv_substr($str,$start,$length,$charset);
    } else {
        $re['utf-8'] = "/[\x01-\x7f]|[\xc2-\xdf][\x80-\xbf]|[\xe0-\xef][\x80-\xbf]{2}|[\xf0-\xff][\x80-\xbf]{3}/";
        $re['gb2312'] = "/[\x01-\x7f]|[\xb0-\xf7][\xa0-\xfe]/";
        $re['gbk']  = "/[\x01-\x7f]|[\x81-\xfe][\x40-\xfe]/";
        $re['big5'] = "/[\x01-\x7f]|[\x81-\xfe]([\x40-\x7e]|\xa1-\xfe])/";
        preg_match_all($re[$charset], $str, $match);
        $slice = join("",array_slice($match[0], $start, $length));
    }
    $omit = mb_strlen($str) >= $length ? '...' : '';
    return $suffix ? $slice.$omit : $slice;

}
```

### 9.获取一定范围内的随机数字
```php
/**
 * 获取一定范围内的随机数字
 * 跟rand()函数的区别是 位数不足补零 例如
 * rand(1,9999)可能会得到 465
 * rand_number(1,9999)可能会得到 0465  保证是4位的
 * @param integer $min 最小值
 * @param integer $max 最大值
 * @return string
 */
function rand_number ($min=1, $max=9999) {
	$strlen = strlen($max);
    return sprintf("%0" . $strlen . "d", mt_rand($min, $max));

}
```

### 10.生成一定数量的随机数，并且不重复
```php
/**
 * 生成一定数量的随机数，并且不重复
 * @param integer $number 数量
 * @param string $len 长度
 * @param string $type 字串类型
 * 0 字母 1 数字 其它 混合
 * @return string
 */
function build_count_rand ($number, $length=4, $mode=1)
{
    if($mode == 1 && $length < strlen($number)) {
        //不足以生成一定数量的不重复数字
        return false;
    }
    $rand = [];
    for($i = 0; $i < $number; $i++) {
        $rand[] = rand_string($length, $mode);
    }
    $unqiue = array_unique($rand);
    if(count($unqiue) == count($rand)) {
        return $rand;
    }
    $count = count($rand) - count($unqiue);
    for($i = 0; $i < $count*3; $i++) {
        $rand[] = rand_string($length, $mode);
    }
    $rand = array_slice(array_unique($rand), 0, $number);
    return $rand;
}
```

### 11.生成不重复的随机数
```php
/**
 * 生成不重复的随机数
 * @param  int $start  需要生成的数字开始范围
 * @param  int $end 结束范围
 * @param  int $length 需要生成的随机数个数
 * @return array       生成的随机数
 */
function get_rand_number($start = 1, $end = 10, $length = 4)
{
    $connt = 0;
    $temp = [];
    while($connt < $length){
        $temp[] = rand($start, $end);
        $data = array_unique($temp);
        $connt = count($data);
    }
    sort($data);
    return $data;
}

```

### 12.curl 获取远程数据
```php
/**
 * 使用curl获取远程数据
 * @param  string $url url连接
 * @return string      获取到的数据
 */
function curl_get_contents($url)
{
    $ch = curl_init();
    curl_setopt($ch, CURLOPT_URL, $url);                //设置访问的url地址
    // curl_setopt($ch,CURLOPT_HEADER,1);               //是否显示头部信息
    curl_setopt($ch, CURLOPT_TIMEOUT, 5);               //设置超时
    curl_setopt($ch, CURLOPT_USERAGENT, $_SERVER['HTTP_USER_AGENT']);   //用户访问代理 User-Agent
    curl_setopt($ch, CURLOPT_REFERER,$_SERVER['HTTP_HOST']);        //设置 referer
    curl_setopt($ch,CURLOPT_FOLLOWLOCATION,1);          //跟踪301
    curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);        //返回结果
    $r = curl_exec($ch);
    curl_close($ch);
    return $r;
}
```

### 13.不区分大小写的 in_array
```php
/**
 * 不区分大小写的in_array()
 * @param  string $str   检测的字符
 * @param  array  $array 数组
 * @return boolear       是否in_array
 */
function in_iarray($str,$array)
{
    $str = strtolower($str);
    $array = array_map('strtolower', $array);
    if (in_array($str, $array)) {
        return true;
    }
    return false;
}
```

### 14.根据时间戳计算距离现在的时间
```php
/**
 * 传入时间戳,计算距离现在的时间
 * @param  number $time 时间戳
 * @return string     返回多少以前
 */
function word_time($time) 
{
    $time = (int) substr($time, 0, 10);
    $int = time() - $time;
    $str = '';
    if ($int <= 2){
        $str = sprintf('刚刚', $int);
    } elseif ($int < 60){
        $str = sprintf('%d秒前', $int);
    }elseif ($int < 3600){
        $str = sprintf('%d分钟前', floor($int / 60));
    }elseif ($int < 86400){
        $str = sprintf('%d小时前', floor($int / 3600));
    }elseif ($int < 1728000){
        $str = sprintf('%d天前', floor($int / 86400));
    }else{
        $str = date('Y-m-d H:i:s', $time);
    }
    return $str;
}
```

### 15.获取访问设备的类型
```php
/**
 * 获取当前访问的设备类型
 * @return integer 1：其他  2：iOS  3：Android
 */
function get_device_type()
{
    //全部变成小写字母
    $agent = strtolower($_SERVER['HTTP_USER_AGENT']);
    $type = 1;
    //分别进行判断
    if(strpos($agent, 'iphone') !== false || strpos($agent, 'ipad') !== false){
        $type = 2;
    } 
    if(strpos($agent, 'android') !== false){
        $type = 3;
    }
    return $type;
}
```

### 16.下载文件
```php
function download($file_url, $new_name = '')
{    
	if(!isset($file_url) || trim($file_url) == ''){  
		return '500';  
	}
	if(!file_exists($file_url)){ 
		//检查文件是否存在  
		return '404';  
	}
	$file_name = basename($file_url);  
	$file_type = explode('.', $file_url);  
	$file_type = $file_type[count($file_type) - 1];  
	$file_name = trim($new_name == '') ? $file_name : urlencode($new_name) .'.'. $file_type;  
	$file_type = fopen($file_url, 'r'); //打开文件  
	//输入文件标签  
	header("Content-type: application/octet-stream");  
	header("Accept-Ranges: bytes");  
	header("Accept-Length: ".filesize($file_url));  
	header("Content-Disposition: attachment; filename=".$file_name);  
	//输出文件内容  
	echo fread($file_type, filesize($file_url));  
	fclose($file_type);  
}
```

