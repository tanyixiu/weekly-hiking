# 🏔️ 周末爬山抽签器 Weekly Hiking

一个简单有趣的网页小游戏，帮助家庭随机选择本周要爬的山。

## ✨ 功能特性

- 🎲 随机抽取山峰
- 🎨 流畅的动画效果（快速闪现 → 减速 → 停止）
- 📜 历史记录（存储在浏览器本地）
- 📊 显示山峰信息（位置、海拔、难度）
- 📝 支持 CSV 文件导入山的列表

## 🚀 快速开始

```bash
# 安装依赖
npm install

# 启动开发服务器
npm run dev

# 构建生产版本
npm run build
```

## 📁 项目结构

```
client/
├── src/
│   ├── components/
│   │   └── MountainPicker.vue    # 主游戏组件
│   ├── assets/
│   │   └── mountains.csv         # 山的列表数据
│   ├── App.vue
│   └── main.js
└── package.json
```

## 🎯 技术栈

- **Vue 3** - 渐进式前端框架
- **Vite** - 现代化构建工具
- **PapaParse** - CSV 解析库
- **LocalStorage** - 本地数据存储

## 📝 自定义山的列表

编辑 `src/assets/mountains.csv` 文件，格式如下：

```csv
name,difficulty,location,elevation
紫金山,简单,南京,448
栖霞山,简单,南京,286
```

## 🔮 未来扩展

- [ ] 多设备数据同步
- [ ] 家庭成员共享
- [ ] 天气信息集成
- [ ] 照片上传和记录
- [ ] 难度筛选功能

---

Made with ❤️ for family hiking adventures
