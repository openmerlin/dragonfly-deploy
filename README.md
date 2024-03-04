# dragonfly-config
本文件主要用于配置记录Dragonfly集群在hub生产环境中的脚本，包含docker compose文件，部署指导等。
# 部署说明
具体部署方案地址: https://tommylikeworkspace.notion.site/Dragonfly-b704d0215c1d4a21acb32db558936a55?pvs=4
## 部署的节点
当前是以docker compose+system service的模式部署了2套环境到生产环境中
基本信息:
```
1. node1
spec: 8u16g
ip address: 172.21.0.66
/data1: 100G 普通SSD
/data2: 10T 高速SSD，用于缓存Pieces

2. node2
spec: 8u16g
ip address: 172.21.0.65
/data1: 100G 普通SSD
/data2: 10T 高速SSD，用于缓存Pieces

2. node3
spec: 8u16g
ip address: 172.21.0.165
/data1: 100G 普通SSD
/data2: 10T 高速SSD，用于缓存Pieces

3. 2个节点的日志均通过挂载本地磁盘记录到了同样的位置，确保每次启动日志一致
scheduler和manager
/data1
|-- manager
|   `-- dragonfly
|       |-- cache
|       |-- home
|       |-- logs
|       |   `-- manager
|       |       |-- core.log
|       |       |-- gin.log
|       |       |-- grpc.log
|       |       |-- stderr.log
|       |       `-- stdout.log
|       `-- plugin
`-- scheduler
    `-- dragonfly
        |-- cache
        |   `-- dynconfig
        |       `-- scheduler
        |-- data
        |   |-- download.csv
        |   `-- networktopology.csv
        |-- home
        |-- logs
        |   `-- scheduler
        |       |-- core.log
        |       |-- gc.log
        |       |-- grpc.log
        |       |-- job.log
        |       |-- stderr.log
        |       `-- stdout.log
        `-- plugins
seed-peer和peer
/data2
|-- 123.bin
|-- hug.bin
|-- huge.bin
|-- peer
|   `-- dragonfly
|       |-- cache
|       |   `-- dynconfig
|       |       `-- daemon
|       |-- daemon.lock
|       |-- data
|       |   |-- 3e8bc749493bcbf9f4feb7d236854c6bc677118ee5a9a3e58e589c7495f1314d
|       |   |   `-- 172.21.0.66-1-502d68cf-4968-4a7f-97ec-08445398c567
|       |   `-- bea74c17b5172f7a59cd56a5bd484f7337d8cc63445e1792d2caf7e04809854d
|       |       `-- 172.21.0.66-1-61898e9b-ef26-47ad-986d-b3f5ae69d94f
|       |-- logs
|       |   `-- daemon
|       |       |-- core.log
|       |       |-- grpc.log
|       |       |-- stderr.log
|       |       `-- stdout.log
|       `-- plugins
`-- seed_peer
    `-- dragonfly
        |-- cache
        |   `-- dynconfig
        |       `-- daemon
        |-- data
        |   |-- 3e8bc749493bcbf9f4feb7d236854c6bc677118ee5a9a3e58e589c7495f1314d
        |   |   `-- 172.21.0.66-1-418bef27-2f45-41ab-95f5-f0c7417cd498_Seed
        |   `-- bea74c17b5172f7a59cd56a5bd484f7337d8cc63445e1792d2caf7e04809854d
        |       `-- 172.21.0.66-1-d22be8a2-b1c5-403c-b8a1-8e763334e8e7_Seed
        |-- home
        |   `-- daemon.lock
        |-- logs
        |   `-- daemon
        |       |-- core.log
        |       |-- gin.log
        |       |-- grpc.log
        |       |-- stderr.log
        |       `-- stdout.log
        `-- plugin

3. redis
172.21.0.115:6379 白名单加用户认证, 配置文件中的REDIS_PASSWORD需要替换成真正的密码
4. mysql
172.21.0.180:3306 白名单加用户认证，配置文件中的MYSQL_PASSWORD需要替换成真正的密码
```
4. 为了避免IP地址冲突,每个node部署的docker compose设置了他自己的子网段
```yaml
networks:
  default:
    driver: bridge
    ipam:
      driver: default
      config:
      - subnet:  172.30.0.0/16
```
## 其他注意事项
1. dragonfly集群的redis和mysql配置密码需要在同步前修改
2. cce/cci集群中的使用的反向代理证书，需要通过修改secret.yaml中的tls.crt和tls.key配置
# TODO
1. lfs regex优化，避免不必要的文件缓存(done)
2. kubernetes部署
3. docker compose重启自动加载(done)
4. node节点补充监控告警(todo)
5. peer节点部署deployment用于在CCI中部署多节点，用于集群加速(done)
6. peer节点部署daemonset用于在CCE或者自建集群中，用于集群加速(done)
7. 数据库连接加密(doing)
8. dragonfly的配置优化(todo)
9. 支持repository任务预热(doing)
10.对接预热任务(todo)
11. CCI&CCE集群环境不同，因为Peer与Peer之间还需要互相访问，如果需要打通，需要(done)
    1. CCI里面使用ELB+Service+Port IP的方式建立VPC连接, 内部访问用loadbalancer服务，外部，因为peer绑定了elb，同样可以访问(done)
    2. CCE里面使用NodePort的方式建立VPC连接
12. CCI集群scheduler hostname不支持设置，每次重启生成新的hostname(todo)
13. modelfoundry_hub支持proxy，整体逻辑是参考git proxy的逻辑，通过环境变量，支持将lfs的请求指向proxy代理(done)
14. 基于hijackHTTPS配置https的支持(done)
```
git config  http.proxy http://127.0.0.1:65001
git config --global url.http://foundation-model-china2.test.osinfra.cn/.insteadOf https://foundation-model-china2.test.osinfra.cn/
git config --global lfs.transfer.enablehrefrewrite true
```
    1. 需要支持设置全局代理 http.proxy与https.proxy配置
