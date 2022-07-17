# Flink

# flink

参照指导，下载和配置docker  
1、下载代码
git clone https://github.com/apache/flink-playgrounds.git  
2、创建flink运行需要的检查点和保存点目录
```
mkdir -p /tmp/flink-checkpoints-directory
mkdir -p /tmp/flink-savepoints-directory
```
3、用开发工具打开源码，修改代码，实现基于flink table api的数据计算和输出
```
    public static Table report(Table table) {
        //每分钟计算在五分钟内每个账号的平均交易金额（滑动窗口）
        return table
                //拆分滑动窗口
                .window(Slide
                        .over(lit(5).minutes())
                        .every(lit(1).minutes())
                        .on($("transaction_time"))
                        .as("sw")
                )
                //分组
                .groupBy($("sw"), $("account_id"))
                //聚合结果
                .select($("account_id"),
                        $("transaction_time").max(),
                        $("amount").avg()
                );
    }

```

4、编译源码
执行下面命令，进行编译，将消耗几分钟
```
cd flink-playgrounds/table-walkthrough
docker-compose build
```  
5、启动docker  
```
docker-compose up -d
```

6、访问查看效果  
查看flink控制台 
http://localhost:8082  
查看grafana监控 
http://localhost:3000/d/FOe0PbmGk/walkthrough?viewPanel=2&orgId=1&refresh=5s  
参看mysql统计结果
```
docker-compose exec mysql mysql -Dsql-demo -usql-demo -pdemo-sql
mysql> use sql-demo;
mysql> select * from spend_report order by log_ts desc limit 20;
+------------+-------------------------+--------+
| account_id | log_ts                  | amount |
+------------+-------------------------+--------+
|          2 | 2021-11-29 09:11:12.000 |    930 |
|          5 | 2021-11-29 09:00:59.000 |    701 |
|          4 | 2021-11-29 08:55:08.000 |    133 |
|          2 | 2021-11-29 08:43:34.000 |    977 |
|          1 | 2021-11-29 08:38:01.000 |    778 |
|          4 | 2021-11-29 08:27:08.000 |    349 |
|          3 | 2021-11-29 08:21:18.000 |    827 |
|          1 | 2021-11-29 08:10:01.000 |    955 |
|          5 | 2021-11-29 08:04:41.000 |    574 |
|          4 | 2021-11-29 07:59:16.000 |    391 |
|          2 | 2021-11-29 07:48:22.000 |    386 |
|          1 | 2021-11-29 07:42:57.000 |    180 |
|          5 | 2021-11-29 07:37:13.000 |    885 |
|          3 | 2021-11-29 07:25:30.000 |    218 |
|          2 | 2021-11-29 07:19:36.000 |    118 |
|          1 | 2021-11-29 07:13:44.000 |    343 |
|          5 | 2021-11-29 07:08:41.000 |     68 |
|          4 | 2021-11-29 07:03:30.000 |     71 |
|          3 | 2021-11-29 06:58:26.000 |    799 |
|          2 | 2021-11-29 06:52:45.000 |    602 |
+------------+-------------------------+--------+
20 rows in set (0.00 sec)
```
7、遇到的问题  
7.1 启动docker报 share mount错误  
```
由于我是win10下wsl内嵌ubuntu实现的docker环境，执行此步骤时遇到1个问题，启动过程中报错：
/mnt/d 不是1个share mount。针对此问题，网上搜索了了各种办法，最后找到解决方案：
sudo mkdir /d
sudo mount /mnt/d /d
sudo mount --make-shared /d
```

7.2 打开http 8082失败  
执行下面命令查看日志，检查错误原因,然后根据原因调整代码，并重新build和启动docker
```
docker-compose logs -f jobmanager
```
8、退出docker
```
docker-compose down -v
```
