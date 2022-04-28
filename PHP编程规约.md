# 编码规范

- PSR-12：扩展编码风格
  https://www.php-fig.org/psr/psr-12/

# 推荐IDE

- Visual Studio Code

# 命名风格
1. 【强制】命名中严禁使用拼音或包含拼音，更不允许直接使用中文。
正例：jd/taobao/aliyun/bilibili/pinyin 国际通用的名称视同英文；array2map 谐音to；map4login 谐音for；

2. 【强制】类名使用 大驼峰 风格。方法名/参数名/成员变量/局部变量都使用 小驼峰 风格。

3. 【强制】常量名全部大写，单词间用下划线隔开，力求语义表达完整清楚。

4. 【强制】异常类名使用 Exception 结尾；测试类名使用要测试的类名加 Test。

5. 【强制】包名使用小写，分隔符之间有且仅有一个自然语义的英语单词。
正例：\app\controller\admin；\app\middleware；

6. 【强制】杜绝不规范的缩写，避免望文不知义。
反例：condition 缩写成 condi；Function 缩写成 Fu；

7. 【推荐】在常量与变量的命名时，表示类型的名词放在词尾，以提升辨识度。
正例：startTime/workQueue/userList/APP_ACCESS_COUNT

# 常量定义

1. 【强制】在 方法及函数 中不要使用字面量，应事先定义 常量。
```php
// 正例：
public const STATUS_SUCCESS = 1;
public function test() {
    if ($status == self::STATUS_SUCCESS) {
        ...
    }
}

// 反例：
public function test() {
    if ($status == 1) {
        ...
    }
}
```

2. 【推荐】不要使用一个常量类维护所有常量，要按对应功能类进行常量定义，分开维护。

# 控制语句

1. 【强制】在 if/else/while/for/foreach 语句中必须使用大括号。
```php
// 正例：
if ($condition) {
    statements;
}

// 反例：
if ($condition) statements;
```

2. 【强制】在 switch 块内，每个 case 必须通过 continue/break/return 来终止，或者注释说明将继续执行到哪一个 case 为止。在 switch 块内，必须有 default 语句且放在最后，即使什么代码也没有。
```php
// 正例：
switch ($condition) {
    case self::STATUS_SUCCESS:
        statements;
        break;
    default:
        break;
}
```

3. 【强制】数量限制场景中，避免使用 等于 判断作为中断或退出的条件，应使用大于或小于的区间判断。避免控制没有处理好，产生等值判断被 击穿 的情况。
```php
// 正例：数量为小于等于0时，正常返回
if ($stockQty <= 0) {
    return;
}

// 反例：数量为负时，会出错，不能正常返回
if ($stockQty == 0) {
    return;
}
```

4. 【推荐】 if/else/switch 分支语句应只用在 流程跳转及异常抛出 中。判断赋值和处理，应使用短路逻辑运算，或数组映射。
```php
// 正例：满足条件，返回
if ($condition) {
    return;
}
// 正例：满足条件，跳转到方法
if ($condition) {
    $this->doNext();
}
// 正例：满足条件，抛出异常
if ($condition) {
    throw new \Exception();
}
// 正例：赋值（不满足为默认值）
$data['succ_time'] = $condition ? time() : $default;
// 正例：满足条件才进行赋值（不满足则短路）
$condition && $data['succ_time'] = time();
// 正例：不满足条件才进行赋值（满足则短路）
$condition || $data['succ_time'] = time();

// 反例：
if ($condition) {
    $data['succ_time'] = time();
}
```

5. 【推荐】尽可能避免使用 while/for/foreach 循环语句对数组或集合进行操作。使用 箭头/闭包 处理数组和集合。
```php
// 正例：遍历数组，返回关联数组
$datas = array_map(fn($item) => [
    'key_' => $item['defKey'],
    'label' => $item['defName'],
    'intro' => $item['intro'],
], $items);

// 正例：遍历数组，组合到映射数组
array_walk($array, function ($val, $key) use (&$map) {
    is_array($val) && $map[] = [$key, 'in', $val];
});
```
