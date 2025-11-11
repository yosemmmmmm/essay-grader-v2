# 🐍 虚拟环境使用指南

## 🤔 虚拟环境是什么？

虚拟环境（Virtual Environment）是Python的一个独立运行环境，让你的项目依赖与系统Python隔离。

---

## ✅ 使用虚拟环境 vs ❌ 不使用虚拟环境

### 对比表格

| 特性 | 使用虚拟环境 | 不使用虚拟环境 |
|------|-------------|---------------|
| **隔离性** | ✅ 项目独立，互不干扰 | ❌ 所有项目共用依赖 |
| **权限** | ✅ 不需要sudo | ❌ 需要sudo安装 |
| **安全性** | ✅ 不影响系统Python | ❌ 可能破坏系统 |
| **可移植性** | ✅ 容易迁移 | ❌ 难以复制环境 |
| **版本管理** | ✅ 可指定版本 | ❌ 全局版本冲突 |
| **磁盘空间** | ❌ 多占用~100MB | ✅ 节省空间 |
| **部署速度** | ❌ 稍慢（需创建环境）| ✅ 稍快 |

---

## 📊 实际例子

### 场景1: 使用虚拟环境（推荐）

```bash
# 项目A
cd ~/project-a
python3 -m venv venv
source venv/bin/activate
pip install django==3.2  # 安装Django 3.2

# 项目B
cd ~/project-b
python3 -m venv venv
source venv/bin/activate
pip install django==4.2  # 安装Django 4.2

# ✅ 两个项目互不干扰，各用各的版本
```

### 场景2: 不使用虚拟环境

```bash
# 项目A
cd ~/project-a
sudo pip3 install django==3.2

# 项目B
cd ~/project-b
sudo pip3 install django==4.2  # ❌ 覆盖了3.2版本

# ❌ 项目A现在无法运行了！
```

---

## 🚀 两种部署方式

### 方式一：使用虚拟环境（推荐）⭐⭐⭐⭐⭐

```bash
# 部署
sudo bash deploy/deploy-aws.sh

# 特点：
# ✅ 依赖安装在 ~/essay-grader-v2/venv/
# ✅ 不影响系统Python
# ✅ 可以同时运行多个Python项目
# ✅ 符合最佳实践
```

**服务配置：**
```ini
ExecStart=/home/ubuntu/essay-grader-v2/venv/bin/uvicorn main:app ...
```

### 方式二：不使用虚拟环境

```bash
# 部署
sudo bash deploy/deploy-aws-no-venv.sh

# 特点：
# ✅ 部署稍快
# ✅ 节省磁盘空间
# ❌ 依赖安装到系统Python
# ❌ 需要sudo权限
# ❌ 可能与其他项目冲突
```

**服务配置：**
```ini
ExecStart=/usr/bin/python3 -m uvicorn main:app ...
```

---

## 💾 磁盘空间对比

### 使用虚拟环境

```
essay-grader-v2/
├── backend/          ~2MB
├── frontend/dist/    ~5MB
├── venv/            ~100MB  ← 虚拟环境
├── data/            ~10MB
└── 其他              ~5MB
总计: ~122MB
```

### 不使用虚拟环境

```
essay-grader-v2/
├── backend/          ~2MB
├── frontend/dist/    ~5MB
├── data/            ~10MB
└── 其他              ~5MB
总计: ~22MB

系统Python包: ~100MB (全局共享)
```

---

## 🎯 推荐方案

### 适合使用虚拟环境的情况（推荐）

- ✅ 生产环境部署
- ✅ 需要长期维护的项目
- ✅ 服务器上有多个Python项目
- ✅ 需要特定版本的依赖
- ✅ 团队协作项目

**使用脚本：** `deploy/deploy-aws.sh`

### 适合不使用虚拟环境的情况

- ✅ 临时测试
- ✅ 服务器只运行这一个Python项目
- ✅ 磁盘空间极度紧张（<500MB）
- ✅ 快速演示

**使用脚本：** `deploy/deploy-aws-no-venv.sh`

---

## 📋 常见问题

### Q1: 虚拟环境会占用多少空间？

**A**: 约100MB，包含Python解释器副本和所有依赖包。

### Q2: 虚拟环境会影响性能吗？

**A**: 不会。运行时性能完全相同，只是启动时多了激活环境的步骤。

### Q3: 如何查看虚拟环境中安装了什么？

**A**: 
```bash
source ~/essay-grader-v2/venv/bin/activate
pip list
```

### Q4: 如何删除虚拟环境？

**A**: 
```bash
rm -rf ~/essay-grader-v2/venv
```

### Q5: 虚拟环境损坏了怎么办？

**A**: 删除后重新创建：
```bash
cd ~/essay-grader-v2
rm -rf venv
python3 -m venv venv
source venv/bin/activate
pip install -r backend/requirements.txt
```

---

## 🔄 如何切换

### 从虚拟环境切换到系统Python

```bash
# 1. 停止服务
sudo systemctl stop essay-grader

# 2. 删除虚拟环境
rm -rf ~/essay-grader-v2/venv

# 3. 安装依赖到系统
cd ~/essay-grader-v2
sudo pip3 install -r backend/requirements.txt

# 4. 修改服务配置
sudo nano /etc/systemd/system/essay-grader.service
# 修改 ExecStart 为:
# ExecStart=/usr/bin/python3 -m uvicorn main:app --host 0.0.0.0 --port 8000 --workers 2

# 5. 重启服务
sudo systemctl daemon-reload
sudo systemctl start essay-grader
```

### 从系统Python切换到虚拟环境

```bash
# 1. 停止服务
sudo systemctl stop essay-grader

# 2. 创建虚拟环境
cd ~/essay-grader-v2
python3 -m venv venv
source venv/bin/activate
pip install -r backend/requirements.txt

# 3. 修改服务配置
sudo nano /etc/systemd/system/essay-grader.service
# 修改 ExecStart 为:
# ExecStart=/home/ubuntu/essay-grader-v2/venv/bin/uvicorn main:app --host 0.0.0.0 --port 8000 --workers 2

# 4. 重启服务
sudo systemctl daemon-reload
sudo systemctl start essay-grader
```

---

## ✅ 最终建议

### 🎯 我的推荐

**使用虚拟环境！** 理由：

1. ✅ 这是Python社区的最佳实践
2. ✅ 避免99%的依赖冲突问题
3. ✅ 100MB空间对现代服务器来说微不足道
4. ✅ 将来维护更容易

### 📝 部署命令

**推荐（使用虚拟环境）：**
```bash
sudo bash deploy/deploy-aws.sh
```

**备选（不使用虚拟环境）：**
```bash
sudo bash deploy/deploy-aws-no-venv.sh
```

---

## 🎓 学习资源

- [Python官方文档 - 虚拟环境](https://docs.python.org/3/tutorial/venv.html)
- [Real Python - Python虚拟环境入门](https://realpython.com/python-virtual-environments-a-primer/)

---

**总结**: 虚拟环境不是必须的，但强烈推荐使用！ 🐍

