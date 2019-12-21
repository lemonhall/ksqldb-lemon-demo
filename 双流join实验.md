
1、建表
CREATE STREAM s_test4_orders (order_id VARCHAR,product_name VARCHAR,user_id VARCHAR)
    WITH (kafka_topic='s_test4_orders', partitions=1, value_format='json', key='order_id');

CREATE STREAM s_test4_users (user_id VARCHAR,user_name VARCHAR)
    WITH (kafka_topic='s_test4_users', partitions=1, value_format='json', key='user_id');

===============================================================

2、监听
SELECT o.order_id,o.product_name,u.user_name
FROM s_test4_orders o JOIN s_test4_users u WITHIN 10 minute ON o.user_id = u.user_id EMIT CHANGES;

===============================================================

3、插入订单
INSERT INTO s_test4_orders (order_id, product_name,user_id) VALUES ('dd0001','DRETEC温湿度计','001');
INSERT INTO s_test4_orders (order_id, product_name,user_id) VALUES ('dd0002','华硕路由器TEK','001');
INSERT INTO s_test4_orders (order_id, product_name,user_id) VALUES ('dd0003','BOSE音箱 MINI','002');


===============================================================
4、插入用户
INSERT INTO s_test4_users (user_id, user_name) VALUES ('001', '余柠');
INSERT INTO s_test4_users (user_id, user_name) VALUES ('002', '张三');
INSERT INTO s_test4_users (user_id, user_name) VALUES ('003', '李四');
INSERT INTO s_test4_users (user_id, user_name) VALUES ('004', '王五');


===============================================================
5、这才是我真正想要的东西......

可又有什么用呢？

|dd0001                                                                                  |DRETEC温湿度计                                                                              |余柠                                                                                      |
|dd0002                                                                                  |华硕路由器TEK                                                                                |余柠                                                                                      |
|dd0003                                                                                  |BOSE音箱 MINI                                                                             |张三                                                                                      |
