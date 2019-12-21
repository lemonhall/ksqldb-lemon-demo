1、继承上一个实验
https://docs.ksqldb.io/en/latest/concepts/stream-processing/

==============================================================================
2、创建流和表：
CREATE TABLE products (product_name VARCHAR, cost DOUBLE)
    WITH (kafka_topic='products', partitions=1, value_format='json', key='product_name');

CREATE STREAM orders (product_name VARCHAR)
    WITH (kafka_topic='orders', partitions=1, value_format='json', key='product_name');

产品为一个表，orders是一个流，这倒是很符合现实，因为订单建模的时候应该就是一个不可变数据项

-----------------------

ksql> CREATE TABLE products (product_name VARCHAR, cost DOUBLE)
>    WITH (kafka_topic='products', partitions=1, value_format='json', key='product_name');
>
>CREATE STREAM orders (product_name VARCHAR)
>    WITH (kafka_topic='orders', partitions=1, value_format='json', key='product_name');

CREATE TABLE products (product_name VARCHAR, cost DOUBLE)
    WITH (kafka_topic='products', partitions=1, value_format='json', key='product_name');
 Message
---------------
 Table created
---------------

CREATE STREAM orders (product_name VARCHAR)
    WITH (kafka_topic='orders', partitions=1, value_format='json', key='product_name');
 Message
----------------
 Stream created
----------------

==============================================================================
3、创建流表关联
使用（2）的终端，来创建一张新表：

CREATE TABLE order_metrics AS
    SELECT p.product_name, COUNT(*) AS count, SUM(p.cost) AS revenue
    FROM orders o JOIN products p ON p.product_name = o.product_name
    GROUP BY p.product_name EMIT CHANGES;

-----------------------

----------------
ksql> CREATE TABLE order_metrics AS
>    SELECT p.product_name, COUNT(*) AS count, SUM(p.cost) AS revenue
>    FROM orders o JOIN products p ON p.product_name = o.product_name
>    GROUP BY p.product_name EMIT CHANGES;

 Message
-----------------------------------------------------------------------------------------------
 Table ORDER_METRICS created and running. Created by query with query ID: CTAS_ORDER_METRICS_4
-----------------------------------------------------------------------------------------------
ksql>

==============================================================================

4、插入产品数据：
INSERT INTO products (product_name, cost) VALUES ('罗技k380键盘', 130);
INSERT INTO products (product_name, cost) VALUES ('iPhone 11 Pro', 13000);
INSERT INTO products (product_name, cost) VALUES ('飞利浦抽湿机H210', 2300);
INSERT INTO products (product_name, cost) VALUES ('HP显示器', 1650);


结果：
ksql> INSERT INTO products (product_name, cost) VALUES ('罗技k380键盘', 130);
>INSERT INTO products (product_name, cost) VALUES ('iPhone 11 Pro', 13000);
>INSERT INTO products (product_name, cost) VALUES ('飞利浦抽湿机H210', 2300);
>INSERT INTO products (product_name, cost) VALUES ('HP显示器', 1650);
ksql>

==============================================================================
5、插入流数据：

INSERT INTO orders (product_name) VALUES ('罗技k380键盘');
INSERT INTO orders (product_name) VALUES ('飞利浦抽湿机H210');
INSERT INTO orders (product_name) VALUES ('iPhone 11 Pro');
INSERT INTO orders (product_name) VALUES ('HP显示器');



=============================================================================
6、取得这个聚合物化视图表的结果：
ksql> SELECT * FROM order_metrics WHERE ROWKEY='罗技k380键盘';
+---------------------------------------------------------------+---------------------------------------------------------------+---------------------------------------------------------------+---------------------------------------------------------------+
|ROWKEY                                                         |P_PRODUCT_NAME                                                 |COUNT                                                          |REVENUE                                                        |
+---------------------------------------------------------------+---------------------------------------------------------------+---------------------------------------------------------------+---------------------------------------------------------------+
|罗技k380键盘                                                       |罗技k380键盘                                                       |5                                                              |650.0                                                          |
Query terminated
ksql>



===========================================================================
7、概念笔记：

订单表==orders表
产品表==普通的表

用groupby建立了一个聚合表
order_metrics

是用产品名做的，订单数的count，以及对应的利润数据，REVENUE

当然，还有一个，就是最终的query

这里唯一让我有点懵逼的地方是，这个ROWKEY,就是PRODUCT_NAME，但是却必须需要指定；
ROWKEY      | P_PRODUCT_NAME |COUNT  |REVENUE  
罗技k380键盘 | 罗技k380键盘    |5      |650.0|

一共产生了5条订单，并且最终的结果是650的利润

===========================================================================

8、这里有一个冗余表的问题

就是说，如果罗技k380键盘 ，作为一个产品，它的价格是130，然后产生的订单，关联后得到的一刻，就是1x130=130，然后得到了它的价格

如果罗技k380键盘 的价格变化后，理论上，之前的订单不应当理会这个价格改变，所以需要看最后这个流式处理怎么搞

如果不行就只能触发重算，这就是物化视图的麻烦之处；

如果要修改以前发生的订单价格的话就会很麻烦；

当然，这个例子本身就不好，因为价格这个东西，应该是订单的一部分做出来作为冗余；这是具体的建模问题；

===========================================================================

9、sink的问题
将group过后的数据，做沉降，沉降到es结果表里面去，这样就省去了es的聚合操作；

这也很重要；

另外，怎样做数据重放，这也是个问题，最直观的方法就是设置超大数据硬盘，讲所有的changes，原封不动得落硬盘

用kafka作为数据表的审计表