# web应用框架自动生成工具

## 初衷

为了快速完成单体应用的开发，选择了如下固定架构。

- docker：开发或发布的运行环境
- React：实现前端页面
- Golang：实现后端程序
- Sqlite：负责数据存储

本程序的目的是基于以上技术快速初始化一个应用的基本框架。


## 框架介绍

### 框架目录结构示例

```
.  
├── Dockerfile  
├── README.md  
├── docker-compose-dev.yml  
├── docker-compose.yml  
├── server  
│   ├── entrypoint.sh  
│   ├── go.mod  
│   ├── go.sum  
│   ├── main.go  
│   ├── ocr  
│   └── tmp.png  
└── ui  
    ├── README.md  
    ├── entrypoint.sh
    ├── index.html  
    ├── package-lock.json  
    ├── package.json  
    ├── postcss.config.js  
    ├── public  
    ├── src  
    ├── tailwind.config.js  
    └── vite.config.js  
```


### 配置项 !!!


文件`.config.json`中包括了框架的重要参数，参考下表。

|   参数名  |   含义    |
| :-        |:-         |
| project   | 项目名称。重要，go module名称，容器名称均基于本参数。  |
| output_dir | 生成项目框架的文件系统路径，默认当前路径 |
| pm | 包管理工具，目前仅测试过npm |
| template | 前端框架，目前仅测试过react |
| frontend_img | 前端容器镜像，默认node:18-alpine |
| backend_img | 后端容器镜像，默认golang:alpine |


### 主流程


#### 1. 系统环境检测

检查依赖程序是否安装，如docker

#### 2. 创建项目根目录

即创建文件夹：`$output_dir/$project/`

#### 3. 生成根目录文件

1. Dockerfile。用于构建生产版本镜像。
2. docker-compose.yml。基于Dockerfile文件，通过docker-compose实现生成版本构建、部署。
3. Docker-compose-dev.yml。通过docker-compose构建、部署开发版本。
4. README.md。项目介绍文档模板。


#### 4. 后端目录server

```
server/
├── entrypoint.sh
├── go.mod
├── go.sum
└── main.go
```

1. entrypoint.sh。开发容器启动脚本。
2. go.mod。`go mod init`生成。
3. go.sum。`go mod tidy`生成。
4. main.go。主函数文件，程序的入口。


#### 5. 前端目录ui

```
ui/
├── README.md
├── entrypoint.sh
├── index.html
├── package-lock.json
├── package.json
├── postcss.config.js
├── public
│   └── vite.svg
├── src
│   ├── App.jsx
│   ├── NotFound.jsx
│   ├── Root.jsx
│   ├── assets
│   │   └── react.svg
│   ├── components
│   │   ├── InputCard.jsx
│   │   └── UploadFile.jsx
│   ├── index.css
│   ├── main.jsx
│   └── output.css
├── tailwind.config.js
└── vite.config.js
```

1. README.md。说明文档。
2. entrypoint.sh。开发容器启动脚本。
3. 其他。通过`vite`初始化项目框架。

## 模式

框架包括**开发模式**和**生产模式**。


### 开发模式

- 前端运行于node容器内，负责UI界面。
- 后端运行于golang容器内，负责处理前端请求、数据库操作等。

分离的原因如下：

1. 前端开发过程中服务自动刷新，不用重启，效率高
2. 后端可独立编译，不必每次修改都嵌入前端界面影响效率


### 生产模式


1. 前端界面build生成单页应用
2. Golang程序集成前端界面文件

单一bin文件部署于容器中。








