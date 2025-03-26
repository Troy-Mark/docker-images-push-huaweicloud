# Docker Images Pusher huaweicloud

使用Github Action将国外的Docker镜像转存到华为云私有仓库，供国内服务器使用，免费易用<br>
- 支持DockerHub, gcr.io, k8s.io, ghcr.io等任意仓库<br>
- 支持最大40GB的大型镜像<br>
- 使用阿里云的官方线路，速度快<br>

本项目参考了大佬技术爬爬虾的(https://github.com/tech-shrimp/me)项目<br>

## 使用方式


### 配置华为云
#### 获取长期访问指令
华为云首页：https://www.huaweicloud.com/<br>
登录华为云，首页搜索容器，选择容器镜像服务 SWR<br>
![image](https://github.com/user-attachments/assets/29bf8d98-7d4e-437e-b172-82526202bc20)
<br>

点击控制台进入容器管理页面<br>
![image](https://github.com/user-attachments/assets/c46b2253-0a52-47ed-9d85-4ffadc4a8062)

<br>

在镜像服务控制台总览页面，选择右上角的登陆指令<br>
![image](https://github.com/user-attachments/assets/aff2a25f-1646-4fbe-9bf2-6ad289011cc1)

生成长期登陆指令并复制指令<br>
![image](https://github.com/user-attachments/assets/7f55a99a-a6ce-4d49-94d2-3853b20a0ab5)

例如长期登陆指令为：<br>
```
docker login -u cn-north-4@NKXXXXXXXXXXXXXS -p 2xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx6a2 swr.cn-north-4.myhuaweicloud.com
```


需要保存三个值，后面会用到：<br>
华为云用户名：cn-north-4@NKXXXXXXXXXXXXXS<br>
华为云用户密码：2xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx6a2<br>
华为云仓库地址：swr.cn-north-4.myhuaweicloud.com<br>

#### 创建组织

镜像服务控制台页面选择组织管理，然后创建组织，并复制组织名称<br>
![image](https://github.com/user-attachments/assets/4d5ba05f-00c2-4aac-aca2-0fc74da9269c)

现在已经获取到了到了四个值，以下四个变量分别代表：<br>

HW_REGISTRY：华为云仓库地址<br>
HW_ORG_NAME：华为云组织名称<br>
HW_REGISTRY_USER：华为云用户名<br>
HW_REGISTRY_PASSWORD：华为云用户密码<br>

### Fork本项目
Fork本项目<br>
#### 启动Action
进入自己的项目，点击Action，启用Github Action功能<br>
#### 配置环境变量
进入Settings->Secret and variables->Actions->New Repository secret<br>
将上一步的**四个值**<br>
HW_REGISTRY，HW_ORG_NAME，HW_REGISTRY_USER，HW_REGISTRY_PASSWORD<br>
配置成环境变量<br>
![image](https://github.com/user-attachments/assets/2aee2a32-9fb0-4d01-b026-c1909c414240)


### 添加镜像
#### json文件内容
打开images.json文件，根据docker官方的镜像信息填写需要同步的镜像信息<br>
- image字段：填写镜像仓库地址，支持以下格式：<br>
    - 官方镜像<br>
    - 第三方镜像：bitnami/nginx → 保持原命名空间<br>
    - 带标签镜像：alpine:3.18 → 自动剥离标签保留名称<br>
- version字段：镜像版本号，如latest，noble-20250127等。
- architectures字段：填写镜像的系统架构，支持同步多架构的镜像，需与Docker官方平台标识的架构严格对应，对应关系如下。
    - linux/386：386<br>
    - linux/amd64：amd64<br>
    - linux/arm/v5：armv5<br>
    - linux/arm/v6：armv6<br>
    - linux/arm/v7：armv7<br>
    - linux/arm64/v8：arm64v8<br>
    - linux/mips64le：mips64le<br>
    - linux/ppc64le：ppc64le<br>
    - linux/s390x：s390x<br>

#### 示例

**nginx镜像信息如下：**<br>
![image](https://github.com/user-attachments/assets/e3489aac-a15e-4e38-a087-5e1d924f81f0)
<br>
**ubuntu镜像信息如下：**<br>
![image](https://github.com/user-attachments/assets/b4f484ba-4fc1-4f1d-970c-cbea74da2b80)
<br>
根据图中所示的信息编写的json文件如下：
```
[
    {
    "image": "ubuntu",
    "version": "noble-20250127",
    "architectures": [
      "amd64", 
      "armv7",
      "arm64v8",
      "ppc64le",
      "riscv64",
      "s390x"
    ]
  },
  {
    "image": "nginx",
    "version": "stable-perl",
    "architectures": [
      "386",
      "amd64",
      "armv5"
    ]
  },
]
```

提交文件后，会自动执行Github Action，向华为云镜像仓库上传镜像<br>

### 使用镜像
回到华为云，镜像仓库，点击任意镜像，可查看镜像状态，可以改成公开，拉取镜像免登录。<br>
![image](https://github.com/user-attachments/assets/352d1d64-728e-49a9-809f-d44486b43b0d)
<br>
要在服务器上拉取华为云镜像仓库中的镜像, 具体使用方法请打开仓库中的镜像详情查看。<br>
![image](https://github.com/user-attachments/assets/3671a5f6-2c12-4b86-b469-495b3dae1164)


### 定时执行
修改/.github/workflows/docker.yaml文件
添加 schedule即可定时执行(此处cron使用UTC时区)
![](doc/定时执行.png)
