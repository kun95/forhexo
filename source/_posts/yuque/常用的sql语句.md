
---

title: 常用的sql语句

date: 2019-02-25 20:16:55 +0800

tags: []

---
<a name="5042afb6"></a>
### 交换两列的值

```sql
update models_mapping as a, models_mapping as b 
set a.souche_model_code=b.third_model_code, a.third_model_code=b.souche_model_code
where a.id=b.id and a.src_domain='iautos.cn' and a.domain='souche.com';
```

<a name="c54edfc1"></a>
### 查询表的所有字段

```sql
select  GROUP_CONCAT(CONCAT('\`',COLUMN_NAME,'\`')) 
	from INFORMATION_SCHEMA.Columns 
  where table_name='brand' and table_schema='car_model'
  
select COLUMN_NAME,column_comment 
	from INFORMATION_SCHEMA.Columns 
  where table_name='表名' and table_schema='数据库名'
```

<a name="14d46fd0"></a>
### 一次新增多个字段

```sql
alter table models_mapping add  (`souche_series_name` varchar(255) DEFAULT NULL COMMENT '来源车系名称',
`third_brand_name` varchar(255) DEFAULT NULL COMMENT '目的域品牌名称',
`third_series_name` varchar(255) DEFAULT NULL COMMENT '目的域车系名称',
`third_model_name` varchar(255) DEFAULT NULL COMMENT '目的域车型名称');
```

<a name="65b5f47b"></a>
### 复制表数据

```sql
insert into dictionary(field,`key`,`value`,date_create,date_update) select field,dict_key ,`value`,data_create , data_create   from config_dictionary ;
```

<a name="tZuU2"></a>
### 新增唯一索引

```sql
alter table `dictionary` add UNIQUE uniq_field_value(`field`,`value`);
```

<a name="Ott9l"></a>
### 查询参数价格颜色

```sql


select 
a.model_code '车型编码',
a.model_name '车型名称',
a.category_code '车款编码',
a.category_name '车款名称',
a.series_code '车系编码',
a.series_name '车系名称',
a.brand_code '品牌编码',
a.brand_name '品牌名称',
(select Group_concat(color_name SEPARATOR ',')  from model_color where model_code = a.model_code and type=0) '外饰颜色',
(select Group_concat(color_name SEPARATOR ',')  from model_color where model_code = a.model_code and type=1) '内饰颜色',
(select guide_price  from model_price where model_code = a.model_code) '指导价',
(select assurance_period_month  from model_parameter where model_code = a.model_code) '保养周期-月',
(select assurance_period_km  from model_parameter where model_code = a.model_code) '保养周期-公里'

from model a

where a.model_code in 

(
	select model_code from model where brand_code in (
'brand-74','brand-86','brand-48'
)
)
```

<a name="uE6bG"></a>
### 查询车型参数

```sql

select 
a.model_code '车型编码',
a.model_name '车型名称',
a.category_code '车款编码',
a.category_name '车款名称',
a.series_code '车系编码',
a.series_name '车系名称',
a.brand_code '品牌编码',
a.brand_name '品牌名称',
(select value from dictionary where `field`='fuelForm' and `key`=b.fuel_form) '燃料形式',
(select value from dictionary where `field`='engineVolume' and `key`=b.engine_volume) '排量',
(select value from dictionary where `field`='gearBox' and `key`=b.gear_box) '变速',
(select value from dictionary where `field`='drivingMode' and `key`=b.driving_Mode) '驱动方式',
(select value from dictionary where `field`='intakeType' and `key`=b.intake_type) '进气形式',
(select value from dictionary where `field`='bodyFormId' and `key`=b.body_formid) '车身形式标识'

from 
	model a,
	model_parameter b
where a.model_code = b.model_code
and a.display_tag & 4>0;
```

<a name="UmvKp"></a>
### 修改列字段

```sql
alter table model_parameter modify column `max_power` decimal(6,2) unsigned DEFAULT NULL COMMENT '最大功率(kW)';
```


