# Docker笔记

# 镜像加速

更新配置文件为

```
{
  "builder": {
    "gc": {
      "defaultKeepStorage": "20GB",
      "enabled": true
    }
  },
  "experimental": false,
  "features": {
    "buildkit": true
  },
  # 加速地址
  "registry-mirrors": [
    "https://reg-mirror.qiniu.com"
  ]
}
```

