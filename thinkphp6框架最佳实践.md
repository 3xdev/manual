# 目标
- 充分利用IDE的提示及补全，提升开发效率。
- 避免开发人员的记忆错误或拼写错误。




# 控制器







# 验证器








# 模型

## 约定
- 字段列表必须使用数组表示
正例：['id']  ['id', 'title'] ['id' => 'uid', 'name' => 'uname']
反例：'id'  'id,title'

## 常量
适用场景
- 定义类型或状态的约定值，避免记忆错误和拼写错误。
```php
// 类型1:模块
const TYPE_MODULE = 1;
// 类型2:分组
const TYPE_GROUP = 2;
// 类型3:菜单
const TYPE_MENU = 3;
// 类型4:操作
const TYPE_ACTION = 4;

// 通知类型
const TYPE_REPORT_DEDUCT = 'report_deduct';
const TYPE_CERT_TARGET = 'cert_target';
const TYPE_CERT_ADJUST = 'cert_adjust';

// 状态
const STATUS_SUCC = 1;
const STATUS_WAIT = 0;
const STATUS_FAIL = -1;
```


## 获取器
适用场景
- 格式化输出。
- 计算属性，根据已有字段计算出的扩展属性。
```php
// 格式化输出登录时间
public function getLoginTimeAttr($value)
{
    return $value ? date('Y-m-d H:i:s', $value) : '';
}

// 根据短信模板内容计算参数数组
public function getParamsAttr($value, $data)
{
    preg_match_all('/\{(\w+?)\}/', $data['content'], $matches);
    return $matches[1];
}

```

## 修改器
适用场景
- 数据转换写入。
- 涉及其他字段的条件或组合写入。
```php
// 转换写入发送时间
public function setSendTimeAttr($value)
{
    return strtotime($value) ?: $value;
}

// 转换去重批量手机号统一格式并更新手机号数量字段
public function setMobilesAttr($value, $data)
{
    $mobiles = array_unique(explode($data['separator'], $value));
    $this->set('mobile_count', count($mobiles));
    return implode(',', $mobiles);
}

```

## 搜索器


## 模型事件


## 条件查询

### whereOr 不同字段条件组合
whereOr 只能用在闭包查询中，直接使用会影响其他条件限制
正例：
```php
$map = [
    ['nickname', 'like', '%哥'],
    ['realname', 'like', '%姐'],
];
\app\model\User::where('status', 1)->where(function ($query) use($map) {
    $query->whereOr($map);
})->select();
```
```sql
SELECT * FROM `gen_user` WHERE (  `status` = 1  AND (  `nickname` LIKE '%哥'  OR `realname` LIKE '%姐' ) ) AND `gen_user`.`delete_time` = '0'";
```
反例：
```php
\app\model\User::where('status', 1)->whereOr([['nickname', 'like', '%哥'], ['realname', 'like', '%姐']])->select();
```
```sql
SELECT * FROM `gen_user` WHERE (  `status` = 1 OR `nickname` LIKE '%哥'  OR `realname` LIKE '%姐' ) AND `gen_user`.`delete_time` = '0';
```

### whereIn 数组查询
先使用 column 获取符合条件的 id 数组，再使用 whereIn 条件。适用于 小量 查询。
```php
$uids = \app\model\User::where('nickname', 'like', "%妈%")->column('id') ?: [0];
$orders = \app\model\Order::whereIn('user_id', $uids)->select();
```
```sql
SELECT * FROM `gen_order` WHERE (  `id` IN (5,28,31,39) ) AND `gen_order`.`delete_time` = '0';
```

### whereIn 子查询
in 将外表和内表做哈希连接(hash join)，先查询内表得出结果集，再查询外表。适用于 大表in小表。
```php
$mapUser = [
    ['gender', '=', 'f'],
    ['realname', 'like', '%姐'],
];
$orders = \app\model\Order::whereIn('user_id', function (&$query) use($mapUser) {
    $query = (new \app\model\User())->db();
    $query->field('id')->where($mapUser);
})->select();
```
```sql
SELECT * FROM `gen_order` WHERE (  `user_id` IN (SELECT `id` FROM `gen_user` WHERE ( `sex` = 'f' AND `realname` LIKE '%姐' ) AND `gen_user`.`delete_time` = '0') ) AND `gen_order`.`delete_time` = '0';
```

### whereExists 子查询
exists 先对外表做loop循环，然后每次再对内表进行查询。适用于 小表exists大表。
```php
$mapOrder = [
    ['status', '=', 1],
    ['amounts', 'between', [100, 200]],
];
$users = \app\model\User::where('status', 1)->whereExists(function (&$query) use($mapOrder) {
    $query = (new \app\model\Order())->db();
    $query->whereExp('buyer_user_id', '=' . (new \app\model\User())->db()->getTable() . '.id')->where($mapOrder);
})->select();
```
```sql
SELECT * FROM `gen_user` WHERE (  `status` = 1  AND EXISTS ( SELECT * FROM `gen_order` WHERE (  ( `buyer_user_id` = gen_user.id )  AND `status` = 1  AND `amounts` BETWEEN '0' AND '200'  ) AND `gen_order`.`delete_time` = '0' ) ) AND `gen_user`.`delete_time` = '0';
```

### 视图 子查询
子查询中用到两表或多表时，需要使用视图来作为子查询。
```php
$mapGoods = [
    ['Goods.type', '=', 1],
    ['Goods.title', 'like', '%面包%'],
];
$orders = \app\model\Order::where('status', 1)->whereExists(function ($query) use($mapGoods) {
    $query->view('OrderGoods')->view('Goods', ['id' => 'gid'], 'Goods.id=OrderGoods.goods_id and Goods.delete_time=0');
    $query->whereExp('OrderGoods.order_id', '=' . (new \app\model\Order())->db()->getTable() . '.id')->where($mapGoods);
})->select();
```
```sql
SELECT * FROM `gen_order` WHERE ( `status` = 1 AND EXISTS ( SELECT `OrderGoods`.*,Goods.id AS gid FROM `gen_order_goods` `OrderGoods` INNER JOIN `gen_goods` `Goods` ON `Goods`.`id`=OrderGoods.goods_id and Goods.delete_time=0 WHERE ( `OrderGoods`.`order_id` =gen_order.id ) AND `Goods`.`type` = '1' AND `Goods`.`title` LIKE '%面包%' ) ) AND `gen_order`.`delete_time` = '0';
```

## 模型关联
关联名使用一个小写单词
正例：ologs  tmembers  members
反例：optLogs  team_members

一对一关联或相对关联使用单数，一对多关联或多对多关联使用复数
正例：owner(文章模型的发布用户)  comments(文章模型的评论)  article(评论模型的关联文章)
反例：owners  comment  articles

用户模型的关联非常多，大多数模型都有与之对应的关联，模型中应该只加入以用户为主体的关联，不应该有其他模型为主体的关联。
正例：profile(个人资料)  cards(会员卡)  coupons(优惠券)  addrs(地址)
反例：orders(订单)  comments(评论)


## 复杂查询

```php
$plist = \think\facade\Db::view('UserPoint', ['id', 'user_id', 'point_time', 'point'])
    ->view('TeamPoint', ['user_point_id'], 'UserPoint.id=TeamPoint.user_point_id and TeamPoint.team_id='.$team->id, 'LEFT')
    ->where('UserPoint.user_id', 'in', $uids)
    ->where('UserPoint.point_time', '>', $team->getData('start_time'))
    ->select();


```


# 参考
PSR-12：扩展编码风格
https://www.php-fig.org/psr/psr-12/



