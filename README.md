# 《数据可视化》第五次个人作业

## 项目说明

本项目是《数据可视化》课程的个人作业五。本项目使用Vite构建Vue3项目呈现网页，在网页中使用D3.js进行数据可视化。选择任务1：世界最高建筑可视化和任务3：FIFA世界杯数据可视化，可视化结果在网页中。

## 项目结构

```plaintext
D3ProVisual
├─ 📁public                         # 公共文件夹，用来存放静态资源
│  ├─ 📁buildings                   # 存放与建筑物相关的数据和图像
│  │  ├─ 📁img                      # 存放建筑物相关的图片
│  │  └─ 📄buildings.csv            # 存放建筑物数据的CSV文件
│  ├─ 📄fifa-world-cup.csv          # 存放FIFA世界杯相关数据的CSV文件
│  └─ 📄vite.svg                    # Vite框架的SVG图标
├─ 📁src                            # 源文件夹，用于存放项目的主要源代码
│  ├─ 📁assets                      # 资源文件夹，存放各种静态资源，如图片、样式表等
│  ├─ 📁components                  # 组件文件夹，存放Vue组件
│  │  ├─ 📄TallestBuildings.vue     # 展示最高建筑物的Vue组件
│  │  └─ 📄WorldCup.vue             # 展示世界杯数据的Vue组件
│  ├─ 📄App.vue                     # Vue的根组件
│  ├─ 📄main.js                     # 项目的入口文件，用于初始化Vue实例等
│  └─ 📄style.css                   # 项目的主要样式表
├─ 📄index.html                     # 项目的主HTML文件，通常作为单页面应用的入口
├─ 📄package-lock.json              # 自动生成的文件，描述项目依赖的具体版本，确保依赖一致性
├─ 📄package.json                   # 描述项目的元数据和依赖关系的文件
├─ 📄README.md                      # 项目的README文件，用于提供项目说明、使用方法等信息
└─ 📄vite.config.js                 # Vite的配置文件，用于自定义Vite的构建和开发行为
```

## 前提条件

在开始之前，请确保你的机器已经安装了以下软件：

- Node.js (可以从 [Node.js 官网](https://nodejs.org/) 下载安装)

```bash
# 检查 Node.js 安装
node --version
npm --version
```

### 前端 Vue 3 运行说明

1. 进入 `D3ProVisual` 目录：

```bash
cd D3ProVisual
```

2. 安装依赖：

```bash
npm install
```

3. 运行开发服务器：

```bash
npm run dev
```

这将在本地启动前端开发服务器，默认在 [http://localhost:5173](http://localhost:5173)。即可在网页中查看项目报告
