# 样式指南（中文版）

**最后更新：** 2025-11-15
**版本：** 1.0
**目的：** 确保所有课程材料的一致性和质量

---

## 文档编写标准

### 语气和风格
1. **清晰第一：** 为初学者编写，即使在高级部分
   - 在使用术语之前先定义
   - 提供具体示例，而不仅仅是抽象描述
   - 为每个概念提供代码示例

2. **代码优先：** 始终包含可工作的、可运行的示例
   - 展示完整代码，而不仅仅是片段（除非用于说明）
   - 包含预期输出的注释
   - 演示错误处理

3. **结构：** 遵循一致的标题层次结构
   - 使用统一的标题格式
   - 广泛使用交叉引用
   - 在每个部分包含"另请参阅"部分

4. **代码示例：** 使用真实的、生产就绪的代码
   - 遵循 Python 代码规范（PEP 8）
   - 包含全面的注释
   - 演示最佳实践

### 中文本地化标准
1. **术语一致性：**
   - 使用统一的中文技术术语
   - 在首次使用时提供英文原文对照
   - 建立术语词汇表

2. **文化适应性：**
   - 使用符合中文读者习惯的表达方式
   - 提供本地化的案例和示例
   - 考虑时区和本地化设置

### 代码编写标准

#### 文件组织
- `tasks.py` - 任务定义
- `celeryconfig.py` - 配置（如果需要）
- `myapp.py` 或 `app.py` - 应用程序设置
- `requirements.txt` - 依赖项（如果特定）

#### 代码质量
- **Python 版本：** 支持 Python 3.9+
- **风格：** 遵循 PEP 8
- **注释：** 解释为什么，而不仅仅是什么
- **错误处理：** 包含适当的错误处理
- **导入：** 明确和有组织的（stdlib、第三方、本地）

### 示例文件命名约定
```python
# 示例命名规范
tasks.py                    # 任务定义
celery_app.py              # Celery 应用配置
worker_config.py           # Worker 配置
monitoring_setup.py        # 监控设置
examples/                   # 示例目录
├── basic/                  # 基础示例
├── intermediate/           # 中级示例
└── advanced/               # 高级示例
```

## 代码编写规范

### Python 代码示例

#### 良好示例
```python
"""
示例演示：数据处理任务

这个示例展示了如何创建健壮的数据处理任务，
包括输入验证、错误处理和性能监控。

先决条件：
    - Redis 运行在 localhost:6379
    - Celery worker 正在运行

使用方法：
    # 启动 worker
    celery -A data_processing worker --loglevel=info

    # 运行任务
    python -c "from data_processing import process_user_data; result = process_user_data.delay({'user_id': 123}); print(result.get())"
"""

from celery import Celery
import logging
from datetime import datetime
from typing import Dict, Any

# 配置日志
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

# 创建 Celery 应用
app = Celery(
    'data_processing',
    broker='redis://localhost:6379',
    backend='redis://localhost:6379'
)

# 应用配置
app.conf.update(
    task_serializer='json',
    accept_content=['json'],
    result_serializer='json',
    timezone='UTC',
    enable_utc=True,
)

@app.task(bind=True, max_retries=3)
def process_user_data(self, user_data: Dict[str, Any]) -> Dict[str, Any]:
    """
    处理用户数据的主要任务

    Args:
        user_data: 包含用户信息的字典

    Returns:
        处理后的用户数据

    Raises:
        ValueError: 当输入数据无效时
        KeyError: 当缺少必需字段时

    Example:
        >>> result = process_user_data.delay({'user_id': 123, 'name': '张三'})
        >>> result.get()
        {'user_id': 123, 'processed_at': '2025-11-15T10:30:00Z', 'status': 'success'}
    """
    try:
        # 输入验证
        required_fields = ['user_id', 'name']
        for field in required_fields:
            if field not in user_data:
                raise KeyError(f"缺少必需字段: {field}")

        # 记录任务开始
        start_time = datetime.utcnow()
        logger.info(f"开始处理用户 {user_data['user_id']} 的数据")

        # 处理数据
        processed_data = {
            'user_id': user_data['user_id'],
            'name': user_data['name'],
            'processed_at': start_time.isoformat(),
            'status': 'success'
        }

        # 添加额外处理逻辑
        if 'email' in user_data:
            processed_data['email_domain'] = user_data['email'].split('@')[1]

        # 记录成功完成
        logger.info(f"成功处理用户 {user_data['user_id']} 的数据")

        return processed_data

    except Exception as exc:
        logger.error(f"处理用户数据失败: {str(exc)}")

        # 重试逻辑
        if self.request.retries < self.max_retries:
            countdown = 2 ** self.request.retries
            logger.info(f"将在 {countdown} 秒后重试")
            raise self.retry(countdown=countdown)
        else:
            # 达到最大重试次数
            raise ValueError(f"处理用户数据失败: {str(exc)}")

if __name__ == '__main__':
    # 示例用法
    sample_data = {
        'user_id': 12345,
        'name': '张三',
        'email': 'zhangsan@example.com'
    }

    # 发送任务
    result = process_user_data.delay(sample_data)
    print(f"任务ID: {result.id}")
    print(f"任务状态: {result.state}")
    print(f"任务结果: {result.get(timeout=10)}")
```

#### 不良示例
```python
# 不推荐：缺少注释和错误处理
from celery import Celery

app = Celery('tasks', broker='redis://localhost:6379')

@app.task
def bad_example(data):
    x = data['a'] + data['b']  # 可能的 KeyError
    return x  # 没有说明返回值是什么
```

### 文档字符串标准

#### 函数文档字符串
```python
def complex_calculation(
    base_value: float,
    multiplier: float,
    options: Dict[str, Any] = None
) -> float:
    """
    执行复杂的计算操作

    这个函数执行一系列数学运算，包括乘法、加法和
    可选的转换操作。

    Args:
        base_value: 基础数值，必须为正数
        multiplier: 乘数，范围在 0.1 到 10.0 之间
        options: 可选配置参数
            - 'precision': 结果精度（默认：2）
            - 'round_method': 舍入方法（'up', 'down', 'nearest'）

    Returns:
        计算结果，保留指定精度

    Raises:
        ValueError: 当输入参数超出范围时
        TypeError: 当参数类型不正确时

    Example:
        >>> result = complex_calculation(10.5, 2.0)
        >>> print(result)
        21.0

        >>> result = complex_calculation(
        ...     10.5, 2.0, {'precision': 4}
        ... )
        >>> print(result)
        21.0000
    """
    if options is None:
        options = {}

    # 参数验证
    if base_value <= 0:
        raise ValueError("base_value 必须为正数")

    if not (0.1 <= multiplier <= 10.0):
        raise ValueError("multiplier 必须在 0.1 到 10.0 之间")

    # 计算逻辑
    result = base_value * multiplier

    # 应用精度设置
    precision = options.get('precision', 2)
    round_method = options.get('round_method', 'nearest')

    if round_method == 'up':
        result = round(result + 0.5 * 10 ** (-precision), precision)
    elif round_method == 'down':
        result = round(result - 0.5 * 10 ** (-precision), precision)
    else:
        result = round(result, precision)

    return result
```

## 配置文件标准

### Celery 配置文件
```python
# celeryconfig.py
"""
Celery 应用程序配置

这个文件包含所有 Celery 应用程序的配置选项。
"""

# 基础配置
broker_url = 'redis://localhost:6379/0'
result_backend = 'redis://localhost:6379/0'

# 任务序列化配置
task_serializer = 'json'
accept_content = ['json']
result_serializer = 'json'
result_expires = 3600  # 1小时

# 时区配置
timezone = 'UTC'
enable_utc = True

# Worker 配置
worker_prefetch_multiplier = 1
worker_max_tasks_per_child = 1000

# 任务路由配置
task_routes = {
    'tasks.data_processing.*': {'queue': 'data_processing'},
    'tasks.notifications.*': {'queue': 'notifications'},
    'tasks.reports.*': {'queue': 'reports'},
}

# 任务默认配置
task_default_queue = 'default'
task_default_exchange = 'default'
task_default_routing_key = 'default'

# 错误处理配置
task_reject_on_worker_lost = True
task_ignore_result = False

# 监控配置
worker_send_task_events = True
task_send_sent_event = True

# Beat 调度器配置
beat_schedule = {
    'daily-cleanup': {
        'task': 'tasks.maintenance.cleanup_old_data',
        'schedule': 86400.0,  # 每天一次
    },
    'hourly-metrics': {
        'task': 'tasks.monitoring.collect_metrics',
        'schedule': 3600.0,  # 每小时一次
    },
}
```

### Docker 配置文件
```yaml
# docker-compose.yml
version: '3.8'

services:
  # Redis 消息代理和结果后端
  redis:
    image: redis:7-alpine
    container_name: course-redis
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    command: redis-server --appendonly yes
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 5s
      retries: 3

  # RabbitMQ 消息代理（可选）
  rabbitmq:
    image: rabbitmq:3-management-alpine
    container_name: course-rabbitmq
    ports:
      - "5672:5672"      # AMQP 端口
      - "15672:15672"    # 管理界面端口
    environment:
      RABBITMQ_DEFAULT_USER: guest
      RABBITMQ_DEFAULT_PASS: guest
      RABBITMQ_DEFAULT_VHOST: /
    volumes:
      - rabbitmq_data:/var/lib/rabbitmq
    healthcheck:
      test: ["CMD", "rabbitmq-diagnostics", "ping"]
      interval: 10s
      timeout: 5s
      retries: 3

  # Celery Worker
  worker:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: course-worker
    command: celery -A celery_app worker --loglevel=info --concurrency=4
    volumes:
      - ./app:/app
      - ./logs:/app/logs
    environment:
      - CELERY_BROKER_URL=redis://redis:6379/0
      - CELERY_RESULT_BACKEND=redis://redis:6379/0
      - PYTHONUNBUFFERED=1
    depends_on:
      redis:
        condition: service_healthy
    restart: unless-stopped

  # Celery Beat 调度器
  beat:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: course-beat
    command: celery -A celery_app beat --loglevel=info
    volumes:
      - ./app:/app
      - ./logs:/app/logs
    environment:
      - CELERY_BROKER_URL=redis://redis:6379/0
      - CELERY_RESULT_BACKEND=redis://redis:6379/0
      - PYTHONUNBUFFERED=1
    depends_on:
      redis:
        condition: service_healthy
    restart: unless-stopped

  # Flower 监控
  flower:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: course-flower
    command: celery -A celery_app flower --port=5555 --address=0.0.0.0
    ports:
      - "5555:5555"
    environment:
      - CELERY_BROKER_URL=redis://redis:6379/0
      - CELERY_RESULT_BACKEND=redis://redis:6379/0
      - FLOWER_BASIC_AUTH=admin:password123
    depends_on:
      redis:
        condition: service_healthy
    restart: unless-stopped

volumes:
  redis_data:
  rabbitmq_data:

networks:
  default:
    driver: bridge
```

## 文档结构指南

### 目录组织
```
course/
├── README_CN.md                      # 项目总览和入门
├── COURSE_OUTLINE_CN.md              # 完整课程大纲
├── LEARNING_PATH_GUIDE_CN.md          # 学习路径指南
├── PRACTICAL_EXERCISES_CN.md         # 实践练习集合
├── QUICK_START_CN.md                 # 快速入门指南
├── assessments/                      # 评估和测试
│   └── EXERCISES_AND_ASSESSMENTS_CN.md
├── interactive-labs/                 # 交互式实验室
│   └── INTERACTIVE_COMPONENTS_AND_LABS_CN.md
├── resources/                        # 补充资源
│   └── DEPLOYMENT_GUIDE_CN.md        # 部署指南
├── foundation/                       # 基础路径材料
├── professional/                     # 专业路径材料
├── expert/                          # 专家路径材料
└── specialist/                      # 专业方向材料
```

### 文档命名约定
- 使用中文文件名后缀 `_CN.md` 区分中文版本
- 保持与英文版本相同的文件结构
- 在中文版本中引用其他中文文档

### Markdown 格式指南

#### 标题结构
```markdown
# 主标题（级别 1）
## 章节标题（级别 2）
### 小节标题（级别 3）
#### 子小节标题（级别 4）
##### 详细标题（级别 5）
###### 最小标题（级别 6）
```

#### 代码块
```python
# 代码块标题
from celery import Celery

app = Celery('tasks', broker='redis://localhost:6379')

@app.task
def example_task():
    """示例任务"""
    return "Hello, World!"
```

#### 表格
| 参数 | 类型 | 描述 | 默认值 |
|------|------|------|--------|
| broker_url | str | 消息代理 URL | required |
| result_backend | str | 结果后端 URL | required |
| timezone | str | 时区设置 | 'UTC' |

#### 警告和提示
```markdown
.. note::
    这是重要的提示信息，学生应该注意。

.. warning::
    这是一个关键警告，如果不遵循可能导致问题。

.. versionadded:: 5.0
    这个功能在版本 5.0 中添加。
```

## 质量检查清单

### 代码质量检查
- [ ] 遵循 PEP 8 代码风格
- [ ] 包含类型提示
- [ ] 有全面的文档字符串
- [ ] 包含适当的错误处理
- [ ] 测试覆盖率 > 80%

### 文档质量检查
- [ ] 标题结构一致
- [ ] 术语使用统一
- [ ] 包含必要的交叉引用
- [ ] 代码示例可运行
- [ ] 拼写和语法正确

### 示例质量检查
- [ ] 示例自包含
- [ ] 在不同环境中测试
- [ ] 包含预期输出
- [ ] 演示最佳实践
- [ ] 有清楚的说明

## 更新和维护

### 版本控制
- 使用语义化版本控制
- 在提交信息中说明更改类型
- 保持 ChangeLog 更新

### 定期审查
- 每季度审查代码示例
- 随 Celery 更新更新内容
- 根据学生反馈改进材料

这个样式指南确保所有课程材料的一致性、可读性和高质量，为学生提供最佳的学习体验。