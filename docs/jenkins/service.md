# 服务类部署

服务类的构建是指，需要用服务器才能运行的服务，例如用python、php、nodejs等后端语言编写的程序以及通过暴露端口对外提供服务的程序（例如jenkins也可以算是服务类的部署）

## 参数说明
此类服务通过docker构建之后，需要启动容器，并且绑定容器与宿主机的端口对外提供服务。

您可以在构建参数的`out_ip`和`in_ip`填写宿主机和容器的端口号对应关系，例如
* nginx的服务，容器内和宿主机都是侦听80，out_ip和in_ip都填写80
* api服务，容器内侦听7001端口，宿主机开放8081端口，out_ip则填写8081，in_ip填写7001
* jenkins服务，非容器版本默认侦听8080端口，宿主机开放8080端口，out_ip则填写8080，in_ip填写8080
* 以此类推……

配置文件通过WriteFile参数编写，格式为
```bash
cat>文件名<<EOF
文件内容
EOF
```
因此您的服务需要配置时尽量能提供配置文件
## 构建步骤
1. 拉取构建流水线[jenkins-template](https://github.com/flashtd1/jenkinsfile-template/blob/master/Jenkinsfile)
2. 读取流水线填写的脚本路径（Jenkinsfile）
3. 按照流水线步骤执行
   1.  git拉取项目
   2.  写入配置
   3.  根据项目内的dockerfile构建镜像
   4.  停止已经运行的容器
   5.  启动新构建的容器
   6.  清理镜像

