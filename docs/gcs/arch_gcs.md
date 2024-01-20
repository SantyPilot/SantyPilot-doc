# 软件架构

## 地面站软件采用插件方式开发

`ground/gcs/src`[路径](https://github.com/SantyPilot/SantyPilot/tree/master/ground/gcs/src/plugins)

```
.
├── app  入口函数
├── libs 公用库函数
├── plugins/ 插件合集
|   |── coreplugin 核心插件
|   ├── flightlog 飞行日志
|   └── ...
|   
└── shared 
```

## 自定义插件开发

