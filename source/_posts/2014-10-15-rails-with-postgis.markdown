---
layout: post
title: "使用Rails和Postgis开发基于地理位置应用的安装与配置"
date: 2014-10-15 15:24:58 +0800
comments: true
categories: [Rails, Postgis, LBS]

---

[`PostGIS`](http://postgis.refractions.net/) 是关系型数据库 [`PostgreSQL`](http://www.postgresql.org/) 一个扩展，它为PostgreSQL提供了存储空间地理数据的支持，使PostgreSQL成为了一个空间数据库，能够进行空间数据管理、数量测量与几何拓扑分析。现在已经有好多为 Rails 提供地理空间数据操作的 gem。所以处理地理空间数据还是比较简单的，但是开发环境配置还有不少工作要做。

###环境依赖
本文以 Mac OS X系统为例，需要安装以下内容：

   * XCode
   * Homebrew   
   * Postgresql
   * Postgis   
   * Ruby
   * Rails

下面单说 Postgresql 及 Postgis 安装

####安装 Postgresql 和 Postgis
```
brew install postgresql
brew install postgis
```

####检查系统是否正确安装 postgis 
```
~$ psql -U username
=# select pg_available_extensions where name like ‘postgis%’;
```

Mac 用户还可以直接下载 [`Postgres.app`](http://postgresapp.com/)，拖入applications文件夹。Postgres.app 包含 PostgreSQL 常用的扩展和工具：

<!-- more -->

* PostgreSQL 9.3.5
* PostGIS 2.1.3
* Procedural languages: PL/pgSQL, PL/Perl, PL/Python, and PLV8 (Javascript)
* Popular extensions, including hstore and uuid-ossp, and more
* A number of command-line utilities for managing PostgreSQL and working with GIS data

###Rails 相关配置
####新建 Rails 应用
先使用 postgresql 适配器新建应用（database参数可选，默认sqlite）

```
rails new my_app --database=postgresql
```

####配置 Gemfile
打开gemfile文件，添加如下 gem ：


   * pg
   * rgeo
   * activerecord-postgis-adapter

```
gem 'pg'
gem 'rgeo'
gem 'activerecord-postgis-adapter'
```

####配置 database.yml
打开 database.yml 文件，adapter 使用 postgis ：

```
development:
  username:           your_username
  adapter:            postgis
  host:               localhost
  postgis_extension: postgis
  schema_search_path: public
```
postgis_extension 表明应用在创建数据库的时候安装 postgis extension。当执行 `rake db:create` 命令的时候，postgresql会有类似如下的操作：

```
CREATE SCHEMA postgis;
CREATE EXTENSION postgis WITH SCHEMA postgis;
```

如果 postgis extension 不是安装在 public schema (默认安装在 public schema)，还得把 postgis 的 schema 添加到 `schema_search_path`：

```
development:
  schema_search_path: public, postgis
```

其他支持的选项：

```
development:
  adapter: postgis
  encoding: unicode
  postgis_extension: postgis      # default is postgis
  schema_search_path: public,postgis
  pool: 5
  database: my_app_development    # your database name
  username: my_app_user           # the username your app will use to connect
  password: my_app_password       # the user's password
  su_username: my_global_user     # a superuser for the database
  su_password: my_global_pasword  # the superuser's password
```

####创建 model 和 table

```
rails g model earthquake location:point
```
要存储空间数据，必须得创建空间类型的字段。Postgis 提供了很多空间类型，包括：point, linestring, polygon以及不同类型的集合。activerecord-postgis-adapter 继承了 ActiveRecord 的 migration 句法，可以支持空间类型。

```
class CreateEarthquakes < ActiveRecord::Migration  
	def change    
		create_table :earthquakes do |t|   
			t.column :shape1, :geometry
			t.geometry :shape2
			t.line_string :path, :srid => 3785   
			t.point :location, :geographic => true
			t.point :lonlatheight, :geographic => true, :has_z => true    
			t.timestamps    
		end  
		
		add_index :earthquakes, :location, :spatial => true
	end
end
```
第一个字段“shape1”，使用的是“geometry”类型，这是空间数据的基础类型（base class），表明可以容纳任何空间类型的数据。

第二个字段“shape2”，使用的是第一个字段定义的简写句法。

第三个字段“path”，使用了一个明确的几何类型 `line_string`，并且定义了srid（spatial reference ID）。表明这个字段只接受 `line_string` 类型的数据，而且SRID等于3785。

第四个字段“location”，是 `point`类型，同时定义了“geographic”，表明这个字段接受 经度/纬度 数据，并且以球型做相关计算（比如两点之间的距离）。

第五个字段“lonlatheight”，包含了“z”坐标，一般用来记录高度。

以下是activerecord-postgis-adapter所支持的postgis类型：

* :geometry -- Any geometric type
* :point -- Point data
* :line_string -- LineString data
* :polygon -- Polygon data
* :geometry_collection -- Any collection type
* :multi_point -- A collection of Points
* :multi_line_string -- A collection of LineStrings
* :multi_polygon -- A collection of Polygons

最后，还支持创建空间索引：

```
def change
	add_index :earthquakes, :location, :spatial => true
end
```

####配置 Model

```
class Earthquake < ActiveRecord::Base

  # 默认使用 Geos 空间字段实现
  self.rgeo_factory_generator = RGeo::Geos.factory_generator

  # 也可以设置某个字段为 geographic 空间字段实现
  set_rgeo_factory_for_column(:location, RGeo::Geographic.spherical_factory(:srid => 4326))
end
```

###空间数据操作
一旦安装了以上gem，并且配置了数据库以及执行了migration，就可以以RGeo对象与model中的空间数据交互了。

[`RGeo`](https://github.com/rgeo/rgeo) 是Opengis Simple Features 标准的 Ruby 实现。它是一个空间数据类型结合，包括points, lines, polygons, 和 collections。还提供标准的空间数据分析操作，比如计算交集或包含关系，计算长度或区域等等。	
####空间数据读写

```
record = Earthquake.find(1)
p = record.location                # Returns an RGeo::Feature::Point
puts p.x                           # displays the x coordinate
puts p.geometry_type.type_name     # displays "Point"

record.location = 'POINT(-122 47)'	# sets the value with WKT string
record.save

factory = p.factory                # returns a spherical factory
# factory = Earthquake.rgeo_factory_for_column(:location)
p2 = factory.point(-122, 47)       # p2 is a point in a spherical factory
record.lonlat = p2                 # sets the value to the given point
record.shape1 = p2                 # shape1 uses a flat geos factory, so it
                                   # will cast p2 into that coordinate system
                                   # before setting the value
record.save
```

####空间数据查询
location距离坐标(116.458104 39.966293) 10公里以内的Earthquake：

```
Earthquake.where("st_dwithin(location, 'point(116.458104 39.966293)', 10000)").count
```
location距离坐标(116.458104 39.966293) 10公里以内的Earthquake，并按距离排序：

```
Order.select("orders.*, st_distance(location, 'point(116.458104 39.966293)') as distance").where("st_dwithin(location, 'point(116.458104 39.966293)', 10000)").order("distance”)
```
location在矩形区域POLYGON((116.424031 39.955045, 116.468731 39.955045, 116.468731 39.983357, 116.424031 39.983357, 116.424031 39.955045))内的Earthquake：

```
Order.where("ST_Intersects('SRID=4326;POLYGON((116.424031 39.955045, 116.468731 39.955045, 116.468731 39.983357, 116.424031 39.983357, 116.424031 39.955045))',location)")
```
<small>说明：矩形是由4个点组成，但是使用POLYGON需要5个点，其中第一个和最后一个是同一个点，个人理解是为了形成一个闭合。另外使用此方法可以构建任意的多边形。</small>

###参考文章
* [ActiveRecord connection adapter for PostGIS, based on postgresql and rgeo](https://github.com/rgeo/activerecord-postgis-adapter)
* [POSTGIS AND RAILS](http://www.waratuman.com/2012/10/26/postgis_and_rails/)
