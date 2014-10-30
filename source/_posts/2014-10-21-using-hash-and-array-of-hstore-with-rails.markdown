---
layout: post
title: "在Rails中使用hstore和Array类型"  
date: 2014-10-21 12:05:15 +0800  
comments: true
categories: [Rails, PostgreSQL, Hstore]

---

最近项目技术选型从 MySQL+MongoDB 转到 PostgreSQL，但是 MongoDB 支持存储 Array 和 JSON 数据类型在某些场景下还是很方便的。比如手机通讯录里的联系人，一个联系人有多个号码、多个地址，如果使用传统的数据类型，可能要三个表来存储这些信息；另一个问题在于，一般编辑联系人的时候，会同时增删改N个号码 + N条地址，这是再去操作三个表逻辑上也会复杂不少。但是如果把多个号码和多个地址以 JSON 的 array 及 key-value 形式存储在同一个字段中，这样曾删改的时候会方便很多。索性PostgreSQL，也有相应的字段类型来支持此场景，比如 hstore。


Rails 4.1 中的一个新特性就是对 PostgreSQL 的hstore类型的支持。而 Rails 之前的版本需要安装 [`activerecord-postgres-hstore`](https://github.com/diogob/activerecord-postgres-hstore) gem。下面说说 Rails 中使用 hstore 的方法。

###增加 hstore extenstion
首先创建一个migration，启用 hstore extension。  

``` ruby
class AddHstoreExtension < ActiveRecord::Migration
  def self.up
    execute "CREATE EXTENSION IF NOT EXISTS hstore"
  end

  def self.down
    execute "DROP EXTENSION IF EXISTS hstore"
  end
end
```

还有一种更简便的写法，不需要用 execute 执行原生的 SQL 语句。

```
class AddHstoreExtension < ActiveRecord::Migration
  def self.up
    enable_extension :hstore
  end

  def self.down
    disable_extension :hstore
  end
end
```

<!-- more -->

###使用 hstore 创建 model

```
class CreateContacts < ActiveRecord::Migration
  def change
    create_table :contacts do |t|
      t.references :user, index: true
      t.string :name
      t.string :gender
      t.hstore :telphones, :array => true
      t.hstore :settings

      t.timestamps
    end        
  end
end
```

`:array => true` 表示当前字段是数组类型。字段 `telphones` 是 `hstore` 类型的数组，字段 `settings` 是 `hstore` 类型。

###使用示例

####创建 model

``` ruby
contact = Contact.create(user_id: 1, name: 'windstill', gender: '先生', telphones: 
[{type: '手机', num: '15155556666'}, {type: '工作', num: '010-86118233'}], 
settings: {theme: 'white', language: 'cn'})
```
####获取字段

```
contact.settings 					# => {theme: 'white', language: 'cn'}
contact.settings["theme"] 			# => "white"

contact.telphones 					# => [{type: '手机', num: '15155556666'}, {type: '工作', num: '010-86118233'}]
contact.telphones[0] 				# => {type: '手机', num: '15155556666'}
contact.telphones[0]["num"] 		# => "15155556666"
```

####更新字段
如果是直接对 contact 的字段（比如name、settings、telphones）进行赋值，那么可以直接 save。
但是如果对 settings 和 telphones 里面的 hash 值进行修改，需要用 `attr_will_change` 通知 rails，因为 hash 的key-value不是 Active Record model 对象。如下所示：

```
contact.settings = {theme: 'blue', language: 'en'}
contact.save

contact.telphones[1]["type"] = "手机"
contact.telphones[1]["num"] = "137700001111"
contact.telphones_will_change!
contact.save
```

####查询

```
Contact.where("settings -> 'language' = 'en'")
Contact.where("'15155556666' = ANY (SELECT unnest(telphones) -> 'num')")
```

也可以写个 scope：

```
scope :where_any, ->(column, key, value) { where("? = ANY (SELECT unnest(#{column}) -> ?)", value, key) }
scope :where_all, ->(column, key, value) { where("? = ALL (SELECT unnest(#{column}) -> ?)", value, key) }
```

###建立索引
在 PostgreSQL 中可以对普通的 array 建立索引，可以对 hstore 建立索引。但是要对 hstore 类型的 array 建立索引就会有问题，会提示你要对 hstore[] 定义 operator class。

下面是对 hstore 建立索引的示例：

```
add_index :contacts, :telphones, :using => :gin
```

这里有两种索引可以选择：gist 和 gin。他们的性能区别如下：

* GIN index lookups are about three times faster than GiST
* GIN indexes take about three times longer to build than GiST
* GIN indexes are about ten times slower to update than GiST
* GIN indexes are two-to-three times larger than GiST

###参考文章
* [Using arrays of hstore with Rails 4](http://inopinatus.org/2013/07/12/using-arrays-of-hstore-with-rails-4/)
* [Using Postgres Hstore with Rails 4](http://jes.al/2013/11/using-postgres-hstore-rails4/)