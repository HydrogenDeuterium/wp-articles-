#! https://zhuanlan.zhihu.com/p/400508782
# 为什么我从PostgreSQL回到MySQL

## 引言

最近在研究 go ，发现 mysql 没有数组功能，听别人说 pg 有，就考虑用那个。

## 安装

安装这个过程基本上还算是顺利的，不论是 直接 `apt install` 还是 `docker pull && docker run` 都还算顺利，但是配置数据库才是噩梦。

## 配置

安装完了，让我用 `psql` 去调用，然后我自己还用不了，非要 `su` 到一个他自己的用户 postgres 调用，很离谱。

忙活了半天给自己的用户添加了用户，忙活一通，连接！

结果一通报错。仔细看了一下，似乎是权限不够。好我这次加上权限，直接拉满上超管，连接！

pg: 认证失败。

有点离谱了哈，哥们。

pg：没有叫做 {{username}} 的数据库。连接失败！

黑人问号.jpg

为什么我连数据库会要你这个名字的数据库？！！

最后经过一番周折，终于算是能够在本机连上了。


## 连接

不会真的有人 ssh 到服务器上然后在上面随便干什么吧？

自带的 sqlshell 连历史回显都没有，用个der。

在本地 Navicat 上连了半天，死活连不上。一开始以为是服务器没开端口，后来发现开了也没有用。

后来查了一下，说是 pg 默认不能放开外网，要改配置文件才行——真有你的，pg。

改了半天，没成。幸好我没在本地装，晚上能搜到大把 linux 的，但是没看见有 win 的。

嗯，要不我去用 docker？ 说不定是 ubuntu 背锅呢。顺利的 pull 下来，顺利的 run 起来。

除了还是连不上之外好像一点问题都没有。

教程说可以用一个叫 pgadmin4 的东西，pull 之（因为看了下好像不能直接 `apt install`），run 之，顺利点亮，配置了半个小时之后发现这玩意对我一点帮助都没有(摔！)

这时候感觉有点骑虎难下，最后进去容器里面改配置，就这么鼓捣了一段时间，天亮了。（喷血）

想想只能破财消灾了。最后我上阿里云买了个打折的 pgsql 云数据库。然后突然意识到要是我在本机装的话，好像根本不用考虑外网访问的问题——起码暂时不用。所以合着我这是在解决一个不存在的问题，佛了。

## 建表

如果觉得就只是这样的话，那就太浅薄了。能连接上数据库了，那就可以开始建表了。

先把 CUD 三个字段安排上，CU 默认值当前时间，U 每次 `update` 自动更新。

在mysql里我一般是这么整的：

```sql
ALTER TABLE `test02`.`test01` 
ADD COLUMN `create` datetime NULL DEFAULT CURRENT_TIMESTAMP FIRST,
ADD COLUMN `update` datetime NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP AFTER `create`;
```

在 Navicat 里填好字段，勾选一下“根据当前时间戳更新”就行了。

![mysql](http://blog.deuterium.wiki/wp-content/uploads/2021/08/QQ截图20210817034932.png)

然后在 pg 的情况下，默认值的事情倒是好说。但是自动更新？哦我的上帝，这简直比隔壁苏珊奶奶的烤苹果馅饼还要恶心，如果我知道这是谁干的，我一定要我靴子狠狠的踢他的屁股，我发誓！

找了一大堆资料，都表示这得用一个叫做触发器的高级功能。我坚持认为这样简单的功能放给触发器这样的东西使用而不做恰当的封装的话，毫无疑问是一种极大的增加学习成本的事情，任何人都非常不应该这么做。

我找到的几乎所有资料都是互相抄袭，最后找到的来源大概是在 STO，摸索研究了很久才整出来一个面前能用的。首先是定义函数：

```sql
CREATE OR REPLACE FUNCTION "public"."auto_update"()
  RETURNS "pg_catalog"."trigger" AS $BODY$
begin
new.update = current_timestamp(0);
return new;
end
$BODY$
  LANGUAGE plpgsql VOLATILE
  COST 100
```

然后这里这些 COST100 之类的玩意，咱也不知道，咱也不敢问。终于定义好了触发器函数了，接下来才能建表：
```sql
CREATE TABLE "public"."Untitled" (
  "create" timestamp DEFAULT CURRENT_TIMESTAMP(0),
  "update" timestamp DEFAULT CURRENT_TIMESTAMP(0)
)
;

CREATE TRIGGER "foo" BEFORE UPDATE ON "public"."Untitled"
FOR EACH ROW
EXECUTE PROCEDURE "public"."auto_update"();
```

也确实是有一些比较好的地方，比如可以省略`add column`或者`after XXXX`这种，但是这个触发器真的是太蛋疼了。

接下来是 ID，一开始找不到怎么自动递增，后来才了解到要用 serial。然鹅这玩意实际上生成的sql是这样的：

```sql
ADD COLUMN "iid2" int4 NOT NULL DEFAULT nextval('images_iid_seq'::regclass);
```

除了 pg 怕是没有什么东西能认识的出来这是啥了，md自带的代码编辑器都认不出这 tm 是个 sql 了，染色都不肯给我染。

## 迁移

好容易 忙完了一番，终于把表建起来了，接下来是 gorm 那一头的数据模型……

唔，这个用啥注解啊？嗯，那个用啥注解啊？写了可能有三分之一个 struct 我就写不下去了。查了一下，gorm 好像有办法迁移。

嗯，AutoMigration，看起来似乎就是你……册那，怎么只能从结构体模型到数据库？！

找了一圈，gorm 官方没有给类似解决方案，那就去找找第三方吧。最后找了一圈找到了一万个把 mysql 迁移到 gorm 的，没有一个能给 pg 用。找到一个宣称支持的网页，然后直接报错。

一通好找，换了三个搜索引擎，最后找到俩：
- [wantedly/pq to gorm](https://github.com/wantedly/pq2gorm)
- [yishuihanj/db to gorm](https://github.com/yishuihanj/db2go)

第一个仓库已经 archive 了，最后一个commit 大概在五年之前。看看作者一大堆乱七八糟的项目，下下来试了一下，果然点不亮。

第二个好一点，最后一个更新在去年 12 月，今年 4 月的 issue 里看见作者在出没，甚至还有个 12 天前的 PR 下下来果然能过编译。

调了一通，运行！

> 错误! 查找数据库表 '' 包含的列失败：pq: column ad.adsrc does not exist

肯定不能这么容易的放弃对不对，肯定要进去看看源码对不对，说不定这个人的源码就是写的有问题没考虑到我这种特殊情况嘛。

这个库支持 my 和 pg ，问题好像处在pg里那个查询代码上。手动丢去查询一下发现 where 里确实有报错，手工删掉报错的填回去继续执行，还是报错。

继续开始 debug，F8 了一大圈最后大致定位到了一大堆读写缓冲区的东西，反正肯定是很底层的东西了，完全看不懂。看一眼表，又是五点了，rnm退钱！

## 放弃

放弃了，至少现在这个项目是放弃了，可能至少要有半年功夫我才可能继续考虑去用 pg，毕竟很多功能确实是比 mysql 强一些的，未来发展趋势我还是更加看好它一些。

但是，不是现在。整了这两个晚上，感觉很多新事物的成长还是需要足够而充分的时间。而目前 pg 还没有得到这些。

按照我的理解来说，至少还要再等半年，半年之内我认为都很难看到什么太大的起色。

最后的最后，一个项目真的不适合同时引入太多新东西，否则相互叠加之下学习成本会按照指数级上升，最后直接劝退。

