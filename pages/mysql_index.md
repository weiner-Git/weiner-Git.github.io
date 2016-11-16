# MySql存储引擎与索引

MySQL中的经常使用的两个存储引擎 InnoDB和Myisam，总结一些区别和记录吧，使用时候在这里再复习。

###### InnoDB：

- 支持事务
- 适合大量Insert、update操作
- 为行级锁，当条件不是确定行行为时（like %tt%）会整表锁
- 不保存具体行数，count(*)时会遍历整个表
- 默认存储在一个ibd文件中，通过my.cnf修改innodb_file_per_table可独立表空间，表结构文件无区别都是frm文件。
- 聚集索引，必须有主键，没指定会生成隐含字段为主键

###### Myisam：

- 不支持事务
- 适合大量select操作
- 为表级索
- 保存具体行数，不包含where的count(*) 会直接返回值
- 表独立存储，数据文件MYD格式，索引文件MYI格式。
- 非聚集索引，非必须有主键，主索引要求key唯一。

##### InnoDB中索引：

主键(primary key)，这个列会被做为聚集索引，如果没有声明主键，则会用一个唯一且不为空的索引列做为主键，成为此表的聚集索引，上面二个条件都不满足，InnoDB会自己产生一个虚拟的聚集索引。 因为InnoDB的数据文件本身要按主键聚集，所以InnoDB要求表必须有主键，如果没有显式指定，则MySQL系统会自动选择一个可以唯一标识数据记录的列作为主键，如果不存在这种列，则MySQL自动为InnoDB表生成一个隐含字段作为主键，这个字段长度为6个字节，类型为长整形。

知道了InnoDB的索引实现后，就很容易明白为什么不建议使用过长的字段作为主键，因为所有辅助索引都引用主索引，过长的主索引会令辅助索引变得过大。聚集索引这种实现方式使得按主键的搜索十分高效，但是辅助索引搜索需要检索两遍索引：首先检索辅助索引获得主键，然后用主键到主索引中检索获得记录。

##### Myisam中索引：

在MyISAM中，主索引和辅助索引在结构上没有任何区别，只是主索引要求key是唯一的，而辅助索引的key可以重复。MyISAM中索引检索的算法为首先按照B+Tree搜索算法搜索索引，如果指定的Key存在，则取出其data域的值，然后以data域的值为地址，读取相应数据记录。

##### 索引使用：

索引以左侧原则使用，当条件左侧为%时不使用索引，当有多个范围条件只使用左侧范围索引，当条件中有函数或表达式时不会使用索引。sql中可以使用USE INDEX告知查询使用哪个索引，来优化索引使用。

##### explain：

在优化索引过程中可以使用explain 加 sql语句来分析

- id：标识符
- select_type：select的类型，
  - simple ：简单的select,没有union和子查询
  - primary：有子查询语句中外侧的select
  - union： 需要联合查询的那个语句
- table ： 所用的表
- type 连接类型
  - system 表仅有一行
  - const 仅有一个匹配行（主键或者唯一索引） 
  - eq_ref 关联表中前表的结果与后表比较。 (a.id = b.id)
  - ref 非一个匹配行结果或者联表左侧表使用索引
  - index_merge 表示使用索引合并
  - range 范围内使用索引 in(1,10)
  - index 使用索引读取
- possible_keys 提示查询使用哪个索引找到结果行
- keys 使用的索引
- key_len 使用的索引长度
- ref 显示使用哪个列或常数与key一起从表中选择行
- rows 查询的行数，判断是否使用好索引
- Extra 解决查询的详细信息