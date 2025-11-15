# Celery 快速入门指南（中文版）

**最后更新：** 2025-11-15
**目标：** 帮助初学者快速上手 Celery 课程

---

## 🚀 快速开始

### 系统要求
- Python 3.9 或更高版本
- 8GB RAM（推荐 16GB）
- 20GB 可用磁盘空间
- 稳定的网络连接

### 5分钟快速体验

```bash
# 1. 克隆课程仓库
git clone <repository-url>
cd github_celery_course_dev

# 2. 启动开发环境
docker-compose up -d

# 3. 运行第一个 Celery 任务
cd celery/examples/tutorial
celery -A tasks worker --loglevel=info &

# 4. 在新终端执行任务
python -c "from tasks import add; print(add.delay(4, 4).get())"

# 5. 访问监控界面
# http://localhost:5555 (Flower)
```

---

## 📚 选择你的学习路径

### 🔰 零基础入门
如果你：
- 是 Python 开发新手
- 没有分布式系统经验
- 想要从基础开始学习

**推荐路径：** 基础路径 → 专业路径 → 专家路径

### 🎯 有经验开发者
如果你：
- 有 2-3 年 Python 开发经验
- 了解基本编程概念
- 想要快速掌握分布式任务处理

**推荐路径：** 专业路径（可选修基础模块）

### 🏗️ 架构师/技术负责人
如果你：
- 有 5+ 年开发经验
- 负责系统架构设计
- 需要深入理解企业级应用

**推荐路径：** 专家路径 → 专业方向

---

## 🛠️ 环境搭建指南

### 方法 1：Docker（推荐）

```bash
# 启动完整开发环境
docker-compose up -d

# 查看服务状态
docker-compose ps

# 查看日志
docker-compose logs -f celery

# 停止服务
docker-compose down
```

### 方法 2：本地安装

```bash
# 安装 Celery
pip install celery redis

# 安装 Redis（Ubuntu/Debian）
sudo apt-get install redis-server

# 启动 Redis
redis-server

# 验证 Redis 连接
redis-cli ping
```

### 验证安装

```python
# test_celery.py
from celery import Celery

app = Celery('test', broker='redis://localhost:6379')

@app.task
def hello():
    return "Hello, Celery!"

if __name__ == '__main__':
    result = hello.delay()
    print(f"任务ID: {result.id}")
    print(f"任务结果: {result.get(timeout=5)}")
```

---

## 📖 课程导航

### 文档结构

```
course/
├── README.md                    # 项目总览
├── COURSE_OUTLINE_CN.md         # 完整课程大纲
├── LEARNING_PATH_GUIDE_CN.md     # 学习路径指南
├── PRACTICAL_EXERCISES_CN.md     # 实践练习
├── COURSE_CONTENT_SPECIFICATIONS.md  # 详细内容规范
├── assessments/                  # 评估和测试
├── interactive-labs/             # 交互式实验室
├── foundation/                   # 基础课程材料
├── professional/                 # 专业课程材料
├── expert/                      # 专家课程材料
└── resources/                    # 学习资源
```

### 学习顺序建议

1. **第一步：** 阅读 `COURSE_OUTLINE_CN.md` 了解整体课程结构
2. **第二步：** 查看 `LEARNING_PATH_GUIDE_CN.md` 选择适合的路径
3. **第三步：** 按照路径开始学习，配合 `PRACTICAL_EXERCISES_CN.md` 中的练习
4. **第四步：** 完成 `assessments/` 中的评估获得认证

---

## ⚡ 常用命令速查

### Celery 命令

```bash
# 启动 Worker
celery -A your_app worker --loglevel=info

# 启动 Beat 调度器
celery -A your_app beat --loglevel=info

# 启动 Flower 监控
celery -A your_app flower --port=5555

# 查看活动任务
celery -A your_app inspect active

# 查看统计信息
celery -A your_app inspect stats

# 清理队列
celery -A your_app purge
```

### Docker 命令

```bash
# 查看所有容器
docker ps

# 查看容器日志
docker logs <container_name>

# 进入容器
docker exec -it <container_name> bash

# 重启服务
docker-compose restart celery

# 构建镜像
docker-compose build celery
```

### 调试命令

```python
# Python 调试
import pdb; pdb.set_trace()

# 查看任务状态
from celery.result import AsyncResult
result = AsyncResult('task-id')
print(result.state)

# 取消任务
result.revoke(terminate=True)
```

---

## 🔧 常见问题解决

### Q1: Worker 启动失败
**症状：** `ImportError: No module named 'celery'`

**解决方案：**
```bash
# 检查 Python 环境
python --version
pip list | grep celery

# 重新安装
pip install --upgrade celery
```

### Q2: 连接 Redis 失败
**症状：** `ConnectionError: Error 111 connecting to localhost:6379`

**解决方案：**
```bash
# 检查 Redis 是否运行
redis-cli ping

# 启动 Redis
redis-server

# 检查配置
celery -A your_app inspect conf
```

### Q3: 任务不执行
**症状：** 任务状态一直是 `PENDING`

**解决方案：**
```python
# 检查 Worker 是否运行
celery -A your_app inspect active

# 检查任务定义
@app.task  # 确保装饰器正确
def my_task():
    return "Hello"

# 检查队列配置
app.conf.task_routes = {
    'my_task': {'queue': 'default'}
}
```

### Q4: 内存使用过高
**症状：** Worker 内存持续增长

**解决方案：**
```python
# 限制任务内存使用
@app.task(max_retries=3)
def memory_intensive_task(data):
    try:
        # 处理数据
        result = process_data(data)
        return result
    except MemoryError:
        # 清理内存
        import gc
        gc.collect()
        raise self.retry(countdown=60)

# 配置 Worker 限制
# celery worker --max-tasks-per-child=1000
```

---

## 💡 学习技巧

### 1. 理论与实践结合
- 先阅读概念文档
- 运行示例代码
- 修改参数实验
- 完成练习项目

### 2. 循序渐进
```python
# 学习进度建议
学习计划 = {
    "第1-2周": "模块1-2：基础概念和环境搭建",
    "第3-4周": "模块3-4：任务定义和配置",
    "第5-6周": "模块5-6：结果处理和模式",
    "第7-8周": "模块7-8：工作流和安全"
}
```

### 3. 善用工具
- **Flower:** 可视化监控
- **调试器:** pdb 和 IDE 调试
- **日志:** 详细记录执行过程
- **测试:** 验证任务正确性

### 4. 参与社区
- 论坛讨论问题
- 分享学习心得
- 参与开源项目
- 关注技术动态

---

## 🎯 学习目标检查

### 基础目标（4周）
- [ ] 理解分布式任务处理概念
- [ ] 搭建 Celery 开发环境
- [ ] 创建和执行基本任务
- [ ] 配置消息代理和结果后端
- [ ] 使用 Flower 监控任务

### 进阶目标（8周）
- [ ] 设计复杂任务工作流
- [ ] 实现定时任务调度
- [ ] 处理错误和重试机制
- [ ] 优化任务性能
- [ ] 部署到生产环境

### 专家目标（12周）
- [ ] 设计企业级架构
- [ ] 实施高可用性方案
- [ ] 优化系统性能
- [ ] 集成微服务架构
- [] 掌握安全最佳实践

---

## 📞 获取帮助

### 技术支持
- **文档搜索:** 在 `celery/docs/` 目录查找详细信息
- **代码示例:** 查看 `celery/examples/` 目录
- **社区论坛:** 在课程论坛提问
- **导师答疑:** 参加定期答疑时间

### 报告问题
```bash
# 收集环境信息
python --version
celery --version
pip list | grep celery

# 记录错误信息
python -c "
import traceback
try:
    # 你的代码
    pass
except Exception as e:
    traceback.print_exc()
"
```

### 联系方式
- **问题反馈:** GitHub Issues
- **学习讨论:** 课程论坛
- **紧急联系:** support@example.com

---

## 🚀 下一步

1. **立即开始:** 运行 `docker-compose up -d` 启动开发环境
2. **选择路径:** 查看 `LEARNING_PATH_GUIDE_CN.md` 选择适合的学习路径
3. **加入社区:** 注册课程论坛，与其他学员交流
4. **制定计划:** 根据个人情况制定学习时间表

## 💭 学习建议

> "学习分布式任务处理就像学习开车——你需要理解理论，但更重要的是实际练习。从简单的任务开始，逐步增加复杂度，不要害怕犯错，每个错误都是学习的机会。" — 课程导师

**开始你的 Celery 学习之旅吧！** 🎉

记住：最好的学习方式就是动手实践。现在就开始编写你的第一个 Celery 任务吧！