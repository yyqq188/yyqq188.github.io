## 安装register
#### 1 添加修改配置 /etc/docker/daemon.json 添加
`{"insecure-registries":["10.1.100.111:5000"]}`
#### 2 重启docker
```
sudo systemctl daemon-reload
sudo systemctl restart docker
```
#### 3 下载并启动register
```
docker run -d \
  -e REGISTRY_HTTP_ADDR=0.0.0.0:5000 \
  -p 5000:5000 \
  --restart=always \
  --name registry \
  -v /home/puaiuc/registry:/var/lib/registry \
  registry:2

```
#### 4 使用
```
docker pull ubuntu
docker tag ubuntu xxxx:5000:ubuntu
docker push xxxx:5000:ubuntu
```
#### 5 停止并删除register
```
docker container stop registry && docker container rm -v registry
```
## 安装gitlab
#### 1 编写start.sh代码
```
#!/bin/bash
HOST_NAME=dict.gitlab.com
GITLAB_DIR=/home/puaiuc/yhl/gitlab  #start.sh也放在这个目录下
docker stop gitlab
docker rm gitlab
docker run -d \
    --hostname ${HOST_NAME} \
    -p 8443:443 -p 9080:80 -p 2222:22 \
    --name gitlab \
    -v ${GITLAB_DIR}/config:/etc/gitlab \
    -v ${GITLAB_DIR}/logs:/var/log/gitlab \
    -v ${GITLAB_DIR}/data:/var/opt/gitlab \
    gitlab/gitlab-ce:latest

```
#### 2 可以提前下载gitlab/gitlab-ce 或者直接执行 start.sh
`bash start.sh`
#### 3 进入到start.sh中GITLAB_DIR指定的目录下找到config目录，并找到config目录里的gitlab.rb文件,在gitlab.rb文件的第399行把注释打开，并更改为start.sh中映射的2222端口
```
gitlab_rails['gitlab_shell_ssh_port'] = 2222
```
#### 4 重新执行bash start.sh命令
