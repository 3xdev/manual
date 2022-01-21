# 目标
- 充分利用IDE的提示及补全，提升开发效率。
- 避免开发人员的记忆错误或拼写错误。




# 控制器







# 验证器








# 模型

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
            //$query->where('user_id', 'in', function ($query) use ($value) {
            //    $query->table((new User())->getTable())->where('nickname', 'like', "%{$value}%")->field('id');
            //});
            $uids = User::where('nickname', 'like', "%{$value}%")->column('id') ?: [0];
            $tids = TeamMember::where('user_id', 'in', $uids)->column('team_id') ?: [0];
            $query->where(function ($query) use ($uids, $tids) {
                $query->whereOr([
                    ['user_id', 'in',  $uids],
                    ['id', 'in', $tids]
                ]);
            });

                $plist = \think\facade\Db::view('UserPoint', 'id,user_id,point_time,point')
                        ->view('TeamPoint', 'user_point_id', 'UserPoint.id=TeamPoint.user_point_id and TeamPoint.team_id='.$team->id, 'LEFT')
                        ->where('UserPoint.user_id', 'in', $uids)
                        ->where('UserPoint.point_time',  '>', $team->getData('start_time'))
                        ->where('user_point_id', 'null')
                        ->select();

    public function searchConsigneeAttr($query, $value)
    {
        if ($value) {
            $query->where('id', 'in', function ($query) use ($value) {
                $query->table((new GiftOrder())->getTable())->where('consignee', 'like', "%{$value}%")->field('gift_log_id');
            });
        }
    }
    public function searchMobileAttr($query, $value)
    {
        if ($value) {
            $query->where('id', 'in', function ($query) use ($value) {
                $query->table((new GiftOrder())->getTable())->where('mobile', 'like', "%{$value}%")->field('gift_log_id');
            });
        }
    }

```


# 参考
PSR-12：扩展编码风格
https://www.php-fig.org/psr/psr-12/



