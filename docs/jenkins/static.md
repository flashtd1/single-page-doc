# 静态网站类部署

静态网站的构建是指，只需要静态服务器即可运行的服务。此类服务直接将构建出的静态文件上传oss即可访问。

## 参数说明
此类服务通过docker构建之后，不需要启动容器。

配置文件通过WriteFile参数编写，格式为
```bash
cat>文件名<<EOF
文件内容
EOF
```
因此您的服务需要配置时尽量能提供配置文件

## 构建步骤
1. 拉取构建流水线[jenkins-template](https://github.com/flashtd1/jenkinsfile-template/blob/master/Jenkinsfile-oss)
2. 读取流水线填写的脚本路径（Jenkinsfile-oss）
3. 按照流水线步骤执行
   1.  git拉取项目
   2.  写入配置
   3.  根据项目内的dockerfile构建镜像(这一步里已经包含将内容推送到oss中，因此构建完成时就已经更新好了)
   4.  清理镜像

