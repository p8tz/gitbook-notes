安装redis

```bash
docker run -p 6379:6379 -d \ # -d 后台启动
-v /LOCAL_CONF_PATH/redis-6379.conf:/usr/local/etc/redis/conf/redis-6379.conf \
--name redis \
redis:6.0 \ # 镜像
redis-server /usr/local/etc/redis/conf/redis-6379.conf # 指定配置文件启动

docker run -p 6379:6379 -d -v /LOCAL_CONF_PATH/redis-6379.conf:/usr/local/etc/redis/conf/redis-6379.conf --name redis redis:6.0 redis-server /usr/local/etc/redis/conf/redis-6379.conf

# 客户端连接
docker exec -it redis redis-cli
```

