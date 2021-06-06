docker run -d --net=host  --name redis-manager  \
-e DATASOURCE_DATABASE='redis_manager' \
-e DATASOURCE_URL='jdbc:mysql://192.168.31.101:3306/redis_manager?useUnicode=true&characterEncoding=utf-8&serverTimezone=GMT%2b8' \
-e DATASOURCE_USERNAME='root' \
-e DATASOURCE_PASSWORD='Trsadmin123!@#' \
reasonduan/redis-manager





https://github.com/ngbdf/redis-manager