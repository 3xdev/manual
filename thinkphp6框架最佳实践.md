# 模型

## 关联模型
关联名使用一个小写单词
正例：ologs  tmembers  members
反例：optLogs team_members

一对一关联和相对的关联使用单数，其他关联使用复数
正例：owner(文章模型的发布用户)  comments(文章模型的评论)  article(评论模型的关联文章)
反例：optLogs team_members

用户 的 关联非常多，几乎所有的模型都对与之关联，模型文件中应该只加入以用户为主体的关联，不应该有其他模型为主体的关联。
正例：profile(个人资料)  cards(会员卡)  coupons(优惠券)  addrs(地址)
反例：orders(订单)  comments(评论)


