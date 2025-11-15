# Celery 实践练习和实验指南（中文版）

**最后更新：** 2025-11-15
**目标：** 提供全面的动手实践练习，巩固理论知识

---

## 练习结构概览

### 练习类型
1. **基础练习（Foundation）** - 巩固核心概念
2. **进阶练习（Intermediate）** - 提升实战技能
3. **高级练习（Advanced）** - 解决复杂问题
4. **项目练习（Project）** - 综合应用能力
5. **调试练习（Debugging）** - 故障排除能力

### 难度等级
- ⭐ **初级:** 基础概念验证
- ⭐⭐ **中级:** 复杂功能实现
- ⭐⭐⭐ **高级:** 系统级挑战
- ⭐⭐⭐⭐ **专家:** 企业级问题解决

---

## 基础路径练习

### 练习 1.1: Docker 环境搭建 ⭐

**目标:** 搭建完整的 Celery 开发环境

**步骤:**
```bash
# 1. 创建 docker-compose.yml
version: '3.8'
services:
  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"

  rabbitmq:
    image: rabbitmq:3-management
    ports:
      - "5672:5672"
      - "15672:15672"
    environment:
      RABBITMQ_DEFAULT_USER: guest
      RABBITMQ_DEFAULT_PASS: guest

  worker:
    build: .
    command: celery -A app worker --loglevel=info
    volumes:
      - .:/app
    depends_on:
      - redis
      - rabbitmq

  flower:
    build: .
    command: celery -A app flower --port=5555
    ports:
      - "5555:5555"
    depends_on:
      - redis
      - rabbitmq
```

```python
# 2. 创建 app.py
from celery import Celery

app = Celery('tasks', broker='redis://redis:6379')

@app.task
def hello_world():
    return "Hello, Celery!"

if __name__ == '__main__':
    # 启动测试
    result = hello_world.delay()
    print(f"任务ID: {result.id}")
    print(f"任务状态: {result.status}")
    print(f"任务结果: {result.get(timeout=10)}")
```

**验证清单:**
- [ ] 所有容器正常启动
- [ ] Worker 连接到消息代理
- [ ] 可以执行简单任务
- [ ] Flower 监控界面可访问

**扩展挑战:**
- 添加健康检查
- 配置环境变量
- 实现优雅关闭

---

### 练习 1.2: 计算器应用开发 ⭐⭐

**目标:** 创建功能完整的异步计算器

**需求:**
```python
# calculator.py
from celery import Celery
import math
from decimal import Decimal, InvalidOperation

app = Celery('calculator', broker='redis://localhost:6379')

# 基础运算
@app.task
def add(x, y):
    """加法运算"""
    try:
        result = Decimal(str(x)) + Decimal(str(y))
        return float(result)
    except (InvalidOperation, TypeError, ValueError) as e:
        raise ValueError(f"无效的数字输入: {e}")

@app.task
def subtract(x, y):
    """减法运算"""
    try:
        result = Decimal(str(x)) - Decimal(str(y))
        return float(result)
    except (InvalidOperation, TypeError, ValueError) as e:
        raise ValueError(f"无效的数字输入: {e}")

@app.task
def multiply(x, y):
    """乘法运算"""
    try:
        result = Decimal(str(x)) * Decimal(str(y))
        return float(result)
    except (InvalidOperation, TypeError, ValueError) as e:
        raise ValueError(f"无效的数字输入: {e}")

@app.task
def divide(x, y):
    """除法运算"""
    try:
        if y == 0:
            raise ZeroDivisionError("除数不能为零")
        result = Decimal(str(x)) / Decimal(str(y))
        return float(result)
    except (InvalidOperation, TypeError, ValueError) as e:
        raise ValueError(f"无效的数字输入: {e}")
    except ZeroDivisionError:
        raise

# 高级运算
@app.task
def power(base, exponent):
    """幂运算"""
    try:
        result = Decimal(str(base)) ** Decimal(str(exponent))
        return float(result)
    except (InvalidOperation, TypeError, ValueError) as e:
        raise ValueError(f"无效的数字输入: {e}")

@app.task
def factorial(n):
    """阶乘计算"""
    try:
        n = int(n)
        if n < 0:
            raise ValueError("阶乘的输入必须是非负整数")
        if n > 1000:  # 防止过大的计算
            raise ValueError("输入值过大，请使用小于1000的整数")
        return math.factorial(n)
    except (ValueError, TypeError) as e:
        raise ValueError(f"无效的输入: {e}")

@app.task
def fibonacci(n):
    """斐波那契数列第n项"""
    try:
        n = int(n)
        if n < 0:
            raise ValueError("斐波那契数列索引必须是非负整数")
        if n > 100:
            raise ValueError("索引值过大，请使用小于100的整数")

        # 优化算法，避免递归
        if n <= 1:
            return n
        a, b = 0, 1
        for _ in range(2, n + 1):
            a, b = b, a + b
        return b
    except (ValueError, TypeError) as e:
        raise ValueError(f"无效的输入: {e}")

@app.task
def sqrt(number):
    """平方根计算"""
    try:
        num = float(number)
        if num < 0:
            raise ValueError("不能计算负数的平方根")
        return math.sqrt(num)
    except (ValueError, TypeError) as e:
        raise ValueError(f"无效的输入: {e}")
```

**测试代码:**
```python
# test_calculator.py
import time
from calculator import add, subtract, multiply, divide, power, factorial, fibonacci

def test_calculator():
    print("=== 计算器功能测试 ===")

    # 测试基础运算
    print("\n1. 基础运算测试:")
    tests = [
        (add, (5, 3), 8),
        (subtract, (10, 4), 6),
        (multiply, (6, 7), 42),
        (divide, (15, 3), 5)
    ]

    for func, args, expected in tests:
        result = func.delay(*args)
        print(f"{func.__name__}{args} = {result.get(timeout=5)} (期望: {expected})")
        assert result.get(timeout=5) == expected, f"{func.__name__} 计算错误"

    # 测试高级运算
    print("\n2. 高级运算测试:")
    advanced_tests = [
        (power, (2, 8), 256),
        (factorial, (5,), 120),
        (fibonacci, (10,), 55),
        (sqrt, (16,), 4.0)
    ]

    for func, args, expected in advanced_tests:
        result = func.delay(*args)
        print(f"{func.__name__}{args} = {result.get(timeout=5)} (期望: {expected})")
        assert abs(result.get(timeout=5) - expected) < 0.001, f"{func.__name__} 计算错误"

    # 测试错误处理
    print("\n3. 错误处理测试:")
    error_tests = [
        (divide, (10, 0), "除数不能为零"),
        (factorial, (-5,), "阶乘的输入必须是非负整数"),
        (sqrt, (-4,), "不能计算负数的平方根")
    ]

    for func, args, error_msg in error_tests:
        try:
            result = func.delay(*args)
            result.get(timeout=5)
            print(f"❌ {func.__name__}{args} 应该抛出异常但没有")
            assert False, f"{func.__name__} 应该抛出异常"
        except Exception as e:
            print(f"✅ {func.__name__}{args} 正确抛出异常: {str(e)}")

    print("\n=== 所有测试通过! ===")

if __name__ == '__main__':
    test_calculator()
```

**扩展挑战:**
1. 添加三角函数计算
2. 实现计算历史记录
3. 创建 Web 界面
4. 添加批量计算功能

---

### 练习 1.3: 任务重试机制实现 ⭐⭐

**目标:** 掌握任务的错误处理和重试策略

```python
# retry_tasks.py
from celery import Celery
import random
import time
import requests
from requests.exceptions import RequestException

app = Celery('retry_tasks', broker='redis://localhost:6379')

# 简单重试示例
@app.task(bind=True, autoretry_for=(ValueError,), retry_kwargs={'max_retries': 3, 'countdown': 2})
def unreliable_task(self, x):
    """随机失败的任务，演示重试机制"""
    if random.random() < 0.7:  # 70% 概率失败
        raise ValueError("随机任务失败")
    return f"任务成功处理: {x}"

# 网络请求重试
@app.task(bind=True, autoretry_for=(RequestException,), retry_backoff=True, retry_jitter=True)
def fetch_url(self, url):
    """获取网页内容，带有重试机制"""
    try:
        response = requests.get(url, timeout=10)
        response.raise_for_status()
        return {
            'status_code': response.status_code,
            'content_length': len(response.content),
            'url': url
        }
    except RequestException as e:
        self.retry(countdown=60)  # 1分钟后重试

# 指数退避重试
@app.task(bind=True, max_retries=5)
def exponential_backoff_task(self, data):
    """使用指数退避策略的重试任务"""
    try:
        # 模拟可能失败的操作
        if random.random() < 0.5:
            raise Exception("操作失败")

        # 处理数据
        processed_data = [x * 2 for x in data]
        return processed_data

    except Exception as exc:
        # 计算退避时间：2^retry_count 秒
        countdown = 2 ** self.request.retries
        self.retry(exc=exc, countdown=countdown)

# 自定义重试逻辑
@app.task(bind=True)
def custom_retry_task(self, task_data):
    """自定义重试逻辑的任务"""
    max_retries = 3
    retry_count = self.request.retries

    try:
        # 模拟可能失败的操作
        if retry_count < 2 and random.random() < 0.5:
            raise Exception(f"第 {retry_count + 1} 次尝试失败")

        # 成功处理
        return f"任务在第 {retry_count + 1} 次尝试后成功"

    except Exception as exc:
        if retry_count < max_retries:
            # 自定义重试间隔
            countdown = 10 * (retry_count + 1)
            self.retry(exc=exc, countdown=countdown)
        else:
            # 达到最大重试次数，记录错误
            error_msg = f"任务在 {max_retries} 次重试后仍然失败: {str(exc)}"
            # 这里可以发送错误通知或记录到日志
            raise Exception(error_msg)

# 测试重试功能
def test_retry_mechanisms():
    print("=== 重试机制测试 ===")

    # 测试简单重试
    print("\n1. 测试简单重试:")
    result = unreliable_task.delay("测试数据")
    print(f"任务ID: {result.id}")
    print("等待任务完成...")
    try:
        final_result = result.get(timeout=30)
        print(f"最终结果: {final_result}")
    except Exception as e:
        print(f"任务最终失败: {e}")

    # 测试网络请求重试
    print("\n2. 测试网络请求重试:")
    urls = [
        "https://httpbin.org/status/200",
        "https://httpbin.org/status/500",  # 这会触发重试
        "https://httpbin.org/status/404"
    ]

    for url in urls:
        print(f"获取URL: {url}")
        result = fetch_url.delay(url)
        try:
            response = result.get(timeout=60)
            print(f"成功: {response}")
        except Exception as e:
            print(f"失败: {e}")

    # 测试指数退避
    print("\n3. 测试指数退避:")
    test_data = [1, 2, 3, 4, 5]
    result = exponential_backoff_task.delay(test_data)
    try:
        final_result = result.get(timeout=120)
        print(f"处理结果: {final_result}")
    except Exception as e:
        print(f"处理失败: {e}")

if __name__ == '__main__':
    test_retry_mechanisms()
```

**验证要点:**
- [ ] 任务能够正确重试
- [ ] 重试间隔配置正确
- [ ] 达到最大重试次数后停止
- [ ] 重试信息被正确记录

---

## 中级路径练习

### 练习 2.1: 复杂工作流设计 ⭐⭐⭐

**目标:** 设计和实现复杂的多阶段数据处理工作流

```python
# complex_workflows.py
from celery import Celery, chain, group, chord
import time
import random

app = Celery('complex_workflows', broker='redis://localhost:6379')

# 数据处理任务
@app.task
def extract_data(source_config):
    """数据提取任务"""
    print(f"从 {source_config['name']} 提取数据...")
    time.sleep(2)  # 模拟数据提取时间

    # 模拟不同类型的数据源
    if source_config['type'] == 'database':
        data = [f"db_record_{i}" for i in range(100)]
    elif source_config['type'] == 'api':
        data = [f"api_data_{i}" for i in range(50)]
    elif source_config['type'] == 'file':
        data = [f"file_line_{i}" for i in range(200)]
    else:
        data = [f"unknown_data_{i}" for i in range(30)]

    print(f"从 {source_config['name']} 提取了 {len(data)} 条记录")
    return {
        'source': source_config['name'],
        'data': data,
        'count': len(data)
    }

@app.task
def validate_data(extraction_result):
    """数据验证任务"""
    print(f"验证来自 {extraction_result['source']} 的数据...")
    time.sleep(1)

    data = extraction_result['data']
    valid_data = []
    invalid_count = 0

    for item in data:
        # 模拟数据验证逻辑
        if random.random() > 0.05:  # 95% 的数据有效
            valid_data.append(item)
        else:
            invalid_count += 1

    print(f"验证完成: {len(valid_data)} 条有效, {invalid_count} 条无效")

    return {
        'source': extraction_result['source'],
        'valid_data': valid_data,
        'invalid_count': invalid_count,
        'valid_count': len(valid_data)
    }

@app.task
def transform_data(validation_result):
    """数据转换任务"""
    print(f"转换来自 {validation_result['source']} 的数据...")
    time.sleep(3)

    data = validation_result['valid_data']
    transformed_data = []

    for item in data:
        # 模拟数据转换逻辑
        transformed_item = {
            'original': item,
            'transformed': f"transformed_{item}",
            'timestamp': time.time(),
            'metadata': {
                'source': validation_result['source'],
                'processed_at': time.time()
            }
        }
        transformed_data.append(transformed_item)

    print(f"转换完成: {len(transformed_data)} 条记录")

    return {
        'source': validation_result['source'],
        'transformed_data': transformed_data,
        'count': len(transformed_data)
    }

@app.task
def aggregate_results(transformation_results):
    """数据聚合任务"""
    print("聚合所有转换结果...")
    time.sleep(2)

    all_data = []
    total_count = 0
    source_summary = {}

    for result in transformation_results:
        source = result['source']
        data = result['transformed_data']
        count = result['count']

        all_data.extend(data)
        total_count += count

        source_summary[source] = {
            'count': count,
            'sample': data[:3]  # 保存前3条作为样本
        }

    # 创建聚合报告
    aggregation_result = {
        'total_records': total_count,
        'source_count': len(transformation_results),
        'sources': source_summary,
        'aggregated_data': all_data,
        'summary': {
            'average_records_per_source': total_count / len(transformation_results),
            'max_records': max(s['count'] for s in source_summary.values()),
            'min_records': min(s['count'] for s in source_summary.values())
        }
    }

    print(f"聚合完成: {total_count} 条记录来自 {len(transformation_results)} 个数据源")
    return aggregation_result

@app.task
def load_data(aggregation_result):
    """数据加载任务"""
    print("将聚合数据加载到目标系统...")
    time.sleep(2)

    total_records = aggregation_result['total_records']

    # 模拟数据加载过程
    batch_size = 100
    loaded_count = 0

    data = aggregation_result['aggregated_data']

    for i in range(0, len(data), batch_size):
        batch = data[i:i + batch_size]
        # 模拟批次加载
        time.sleep(0.1)
        loaded_count += len(batch)

        if i % 500 == 0:  # 每500条记录打印一次进度
            print(f"已加载 {loaded_count}/{total_records} 条记录...")

    # 创建加载报告
    load_result = {
        'total_loaded': loaded_count,
        'load_time': time.time(),
        'target_system': 'production_database',
        'status': 'completed',
        'details': aggregation_result['summary']
    }

    print(f"数据加载完成: {loaded_count} 条记录")
    return load_result

# 通知任务
@app.task
def send_notification(load_result):
    """发送通知任务"""
    print("发送处理完成通知...")

    message = f"""
    数据处理管道执行完成！

    - 加载记录数: {load_result['total_loaded']}
    - 目标系统: {load_result['target_system']}
    - 状态: {load_result['status']}
    - 完成时间: {time.ctime(load_result['load_time'])}

    详情: {load_result['details']}
    """

    # 这里可以实现实际的邮件发送、Slack通知等
    print("通知内容:")
    print(message)

    return {
        'notification_sent': True,
        'message': message,
        'sent_at': time.time()
    }

# 工作流构建函数
def create_data_pipeline(sources):
    """创建完整的数据处理管道"""

    # 1. 并行数据提取
    extraction_group = group(
        extract_data.s(source) for source in sources
    )

    # 2. 并行数据验证
    validation_group = group(
        validate_data.s() for _ in sources
    )

    # 3. 并行数据转换
    transformation_group = group(
        transform_data.s() for _ in sources
    )

    # 4. 串行处理: 聚合 -> 加载 -> 通知
    processing_chain = chain(
        aggregate_results.s(),
        load_data.s(),
        send_notification.s()
    )

    # 构建完整工作流
    complete_workflow = chain(
        extraction_group,
        validation_group,
        transformation_group,
        processing_chain
    )

    return complete_workflow

# 测试复杂工作流
def test_complex_workflow():
    print("=== 复杂工作流测试 ===")

    # 定义数据源配置
    data_sources = [
        {'name': 'user_database', 'type': 'database'},
        {'name': 'product_api', 'type': 'api'},
        {'name': 'order_files', 'type': 'file'},
        {'name': 'log_files', 'type': 'file'},
        {'name': 'external_api', 'type': 'api'}
    ]

    print(f"\n处理 {len(data_sources)} 个数据源:")
    for source in data_sources:
        print(f"  - {source['name']} ({source['type']})")

    # 创建并执行工作流
    workflow = create_data_pipeline(data_sources)

    print("\n启动工作流...")
    result = workflow.apply_async()

    print(f"工作流ID: {result.id}")
    print("等待工作流完成...")

    try:
        final_result = result.get(timeout=300)  # 5分钟超时
        print("\n=== 工作流执行完成 ===")
        print(f"最终结果: {final_result}")

    except Exception as e:
        print(f"\n工作流执行失败: {e}")
        print("检查 Flower 监控界面获取详细信息")

# 监控工作流进度
def monitor_workflow_progress(workflow_id):
    """监控工作流执行进度"""
    from celery.result import AsyncResult

    result = AsyncResult(workflow_id)

    print(f"工作流 {workflow_id} 状态: {result.state}")

    if result.ready():
        if result.successful():
            print("工作流执行成功!")
            print(f"结果: {result.result}")
        else:
            print("工作流执行失败!")
            print(f"错误: {result.info}")
    else:
        print("工作流仍在执行中...")

if __name__ == '__main__':
    # 执行测试
    test_complex_workflow()

    # 可以在另一个终端中监控进度
    # monitor_workflow_progress(workflow_id)
```

**验证要求:**
- [ ] 所有阶段都能正确执行
- [ ] 并行任务按预期工作
- [ ] 错误能被正确处理
- [ ] 最终结果包含所有数据源的处理结果

**扩展挑战:**
1. 添加工作流失败时的回滚机制
2. 实现工作流进度的实时监控
3. 添加动态任务调度
4. 优化大数据量的处理性能

---

### 练习 2.2: 定时任务调度系统 ⭐⭐⭐

**目标:** 构建一个完整的定时任务调度系统

```python
# scheduled_tasks.py
from celery import Celery
from celery.schedules import crontab
import datetime
import json
import logging

app = Celery('scheduled_tasks', broker='redis://localhost:6379')

# 配置定时任务
app.conf.beat_schedule = {
    'data-cleanup': {
        'task': 'scheduled_tasks.cleanup_old_data',
        'schedule': crontab(minute=0, hour=2),  # 每天凌晨2点
        'args': (30,)  # 保留30天
    },
    'daily-report': {
        'task': 'scheduled_tasks.generate_daily_report',
        'schedule': crontab(minute=0, hour=8),  # 每天早上8点
    },
    'health-check': {
        'task': 'scheduled_tasks.system_health_check',
        'schedule': 300.0,  # 每5分钟
    },
    'user-activity-summary': {
        'task': 'scheduled_tasks.summarize_user_activity',
        'schedule': crontab(minute=0, hour=1, day_of_week=1),  # 每周一凌晨1点
    },
    'cache-refresh': {
        'task': 'scheduled_tasks.refresh_cache',
        'schedule': crontab(minute='*/15'),  # 每15分钟
    }
}

# 定时任务实现
@app.task
def cleanup_old_data(days_to_keep=30):
    """清理旧数据任务"""
    cutoff_date = datetime.datetime.now() - datetime.timedelta(days=days_to_keep)

    print(f"开始清理 {cutoff_date} 之前的数据...")

    # 模拟数据清理
    tables = ['user_sessions', 'audit_logs', 'temp_files', 'cached_results']

    cleanup_results = {}
    total_deleted = 0

    for table in tables:
        # 模拟删除操作
        deleted_count = random.randint(100, 1000)
        total_deleted += deleted_count

        cleanup_results[table] = {
            'deleted_records': deleted_count,
            'status': 'success'
        }

        print(f"表 {table}: 删除了 {deleted_count} 条记录")

    result = {
        'task': 'data_cleanup',
        'executed_at': datetime.datetime.now().isoformat(),
        'cutoff_date': cutoff_date.isoformat(),
        'total_deleted': total_deleted,
        'details': cleanup_results
    }

    print(f"数据清理完成: 总共删除 {total_deleted} 条记录")
    return result

@app.task
def generate_daily_report():
    """生成每日报告任务"""
    print("生成每日报告...")

    # 收集昨日数据
    yesterday = datetime.datetime.now() - datetime.timedelta(days=1)

    # 模拟数据收集
    metrics = {
        'new_users': random.randint(10, 50),
        'active_users': random.randint(100, 500),
        'total_transactions': random.randint(1000, 5000),
        'errors': random.randint(0, 10),
        'server_uptime': random.uniform(99.0, 99.9)
    }

    # 生成报告
    report = {
        'date': yesterday.strftime('%Y-%m-%d'),
        'generated_at': datetime.datetime.now().isoformat(),
        'metrics': metrics,
        'summary': {
            'health_status': 'good' if metrics['errors'] < 5 else 'warning',
            'growth_rate': random.uniform(-5.0, 15.0),
            'performance_score': min(100, metrics['server_uptime'])
        }
    }

    # 保存报告（模拟）
    report_file = f"daily_report_{yesterday.strftime('%Y%m%d')}.json"
    print(f"报告已保存: {report_file}")

    return report

@app.task
def system_health_check():
    """系统健康检查任务"""
    print("执行系统健康检查...")

    # 检查各项系统指标
    health_checks = {
        'database': {
            'status': 'healthy' if random.random() > 0.1 else 'warning',
            'response_time': random.uniform(10, 100),
            'connections': random.randint(50, 200)
        },
        'redis': {
            'status': 'healthy',
            'memory_usage': random.uniform(40, 80),
            'connected_clients': random.randint(10, 50)
        },
        'disk_space': {
            'status': 'healthy' if random.random() > 0.2 else 'critical',
            'usage_percentage': random.uniform(30, 90),
            'available_gb': random.uniform(10, 100)
        },
        'cpu_usage': {
            'status': 'healthy' if random.random() > 0.15 else 'warning',
            'percentage': random.uniform(20, 85),
            'load_average': random.uniform(1.0, 4.0)
        },
        'memory_usage': {
            'status': 'healthy' if random.random() > 0.1 else 'warning',
            'percentage': random.uniform(40, 85),
            'available_gb': random.uniform(2, 16)
        }
    }

    # 计算整体健康状态
    overall_status = 'healthy'
    critical_issues = []
    warnings = []

    for component, checks in health_checks.items():
        if checks['status'] == 'critical':
            overall_status = 'critical'
            critical_issues.append(f"{component}: {checks['status']}")
        elif checks['status'] == 'warning' and overall_status != 'critical':
            overall_status = 'warning'
            warnings.append(f"{component}: {checks['status']}")

    health_report = {
        'timestamp': datetime.datetime.now().isoformat(),
        'overall_status': overall_status,
        'checks': health_checks,
        'critical_issues': critical_issues,
        'warnings': warnings,
        'recommendations': []
    }

    # 添加建议
    if critical_issues:
        health_report['recommendations'].append("立即处理关键问题！")
    if warnings:
        health_report['recommendations'].append("监控警告状态，考虑预防措施")

    print(f"健康检查完成: {overall_status}")
    if critical_issues:
        print(f"关键问题: {critical_issues}")

    return health_report

@app.task
def summarize_user_activity():
    """用户活动汇总任务"""
    print("汇总用户活动数据...")

    # 模拟用户活动数据
    weekly_data = {
        'total_users': random.randint(1000, 5000),
        'active_users': random.randint(500, 2000),
        'new_registrations': random.randint(50, 200),
        'user_retention_rate': random.uniform(60, 90),
        'average_session_duration': random.uniform(5, 30),
        'top_features': [
            {'feature': 'profile_update', 'usage': random.randint(100, 500)},
            {'feature': 'content_upload', 'usage': random.randint(200, 800)},
            {'feature': 'social_sharing', 'usage': random.randint(50, 300)}
        ]
    }

    # 生成用户行为分析
    insights = {
        'engagement_trend': 'increasing' if random.random() > 0.5 else 'decreasing',
        'peak_activity_day': random.choice(['Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday']),
        'most_active_hour': random.randint(9, 22),
        'user_satisfaction_score': random.uniform(7.0, 9.5)
    }

    summary = {
        'week_start': (datetime.datetime.now() - datetime.timedelta(days=7)).strftime('%Y-%m-%d'),
        'generated_at': datetime.datetime.now().isoformat(),
        'metrics': weekly_data,
        'insights': insights,
        'action_items': [
            "优化用户注册流程",
            "改进内容推荐算法",
            "增加用户互动功能"
        ]
    }

    print(f"用户活动汇总完成: {weekly_data['active_users']} 个活跃用户")
    return summary

@app.task
def refresh_cache():
    """刷新缓存任务"""
    print("刷新系统缓存...")

    # 模拟不同类型的缓存刷新
    cache_types = [
        'user_profiles',
        'product_catalog',
        'search_results',
        'api_responses',
        'static_content'
    ]

    refresh_results = {}
    total_items = 0

    for cache_type in cache_types:
        # 模拟缓存刷新
        items_count = random.randint(100, 1000)
        total_items += items_count

        refresh_time = random.uniform(1, 10)

        refresh_results[cache_type] = {
            'items_refreshed': items_count,
            'refresh_time_seconds': refresh_time,
            'status': 'success'
        }

        print(f"刷新 {cache_type}: {items_count} 项，耗时 {refresh_time:.2f}s")

    cache_report = {
        'task': 'cache_refresh',
        'executed_at': datetime.datetime.now().isoformat(),
        'total_items_refreshed': total_items,
        'cache_types': len(cache_types),
        'details': refresh_results,
        'next_refresh': (datetime.datetime.now() + datetime.timedelta(minutes=15)).isoformat()
    }

    print(f"缓存刷新完成: 总共刷新 {total_items} 项")
    return cache_report

# 动态任务调度
@app.task
def schedule_custom_task(task_name, schedule_info, task_args=None):
    """动态调度自定义任务"""
    print(f"调度自定义任务: {task_name}")

    # 这里可以实现动态添加任务到 beat_schedule
    # 实际应用中可能需要重启 beat 或使用动态调度库

    scheduled_info = {
        'task_name': task_name,
        'schedule': schedule_info,
        'args': task_args or [],
        'scheduled_at': datetime.datetime.now().isoformat()
    }

    print(f"任务 {task_name} 已调度")
    return scheduled_info

# 测试定时任务
def test_scheduled_tasks():
    print("=== 定时任务测试 ===")

    # 手动触发各个定时任务进行测试
    print("\n1. 测试数据清理:")
    cleanup_result = cleanup_old_data.delay(7)  # 清理7天前的数据
    print(f"清理任务ID: {cleanup_result.id}")

    print("\n2. 测试每日报告:")
    report_result = generate_daily_report.delay()
    print(f"报告任务ID: {report_result.id}")

    print("\n3. 测试健康检查:")
    health_result = system_health_check.delay()
    print(f"健康检查任务ID: {health_result.id}")

    print("\n4. 测试用户活动汇总:")
    activity_result = summarize_user_activity.delay()
    print(f"活动汇总任务ID: {activity_result.id}")

    print("\n5. 测试缓存刷新:")
    cache_result = refresh_cache.delay()
    print(f"缓存刷新任务ID: {cache_result.id}")

    # 等待任务完成并显示结果
    print("\n等待任务完成...")

    tasks = [
        ('数据清理', cleanup_result),
        ('每日报告', report_result),
        ('健康检查', health_result),
        ('用户活动汇总', activity_result),
        ('缓存刷新', cache_result)
    ]

    for task_name, task_result in tasks:
        try:
            result = task_result.get(timeout=60)
            print(f"\n{task_name} 完成:")
            if isinstance(result, dict):
                for key, value in result.items():
                    if key != 'details':  # 简化显示
                        print(f"  {key}: {value}")
            else:
                print(f"  结果: {result}")
        except Exception as e:
            print(f"\n{task_name} 失败: {e}")

if __name__ == '__main__':
    test_scheduled_tasks()

    print("\n=== 定时任务配置说明 ===")
    print("要启用定时任务，请运行:")
    print("celery -A scheduled_tasks beat --loglevel=info")
    print("\n并在另一个终端运行 worker:")
    print("celery -A scheduled_tasks worker --loglevel=info")
```

**配置说明:**
```bash
# 启动 Beat 调度器
celery -A scheduled_tasks beat --loglevel=info

# 启动 Worker 执行任务
celery -A scheduled_tasks worker --loglevel=info

# 监控任务执行
celery -A scheduled_tasks flower --port=5555
```

**验证要点:**
- [ ] Beat 调度器正常运行
- [ ] 定时任务按计划执行
- [ ] 任务执行结果正确
- [ ] 错误处理和日志记录

---

## 高级路径练习

### 练习 3.1: 微服务架构集成 ⭐⭐⭐⭐

**目标:** 在微服务架构中集成 Celery，实现跨服务的异步任务处理

```python
# microservices_integration.py
from celery import Celery
import requests
import json
import time
from dataclasses import dataclass
from typing import Dict, List, Any

# 主应用配置
app = Celery('microservices', broker='redis://redis:6379')

# 微服务配置
MICROSERVICES = {
    'user_service': 'http://user-service:8001',
    'order_service': 'http://order-service:8002',
    'payment_service': 'http://payment-service:8003',
    'notification_service': 'http://notification-service:8004',
    'inventory_service': 'http://inventory-service:8005',
    'analytics_service': 'http://analytics-service:8006'
}

@dataclass
class OrderData:
    """订单数据结构"""
    order_id: str
    user_id: str
    items: List[Dict[str, Any]]
    total_amount: float
    payment_method: str
    shipping_address: Dict[str, str]

@dataclass
class ServiceResponse:
    """服务响应结构"""
    service: str
    success: bool
    data: Any = None
    error: str = None
    response_time: float = 0.0

# 服务通信工具类
class ServiceCommunicator:
    @staticmethod
    def call_service(service_name: str, endpoint: str, data: Dict = None, method: str = 'POST') -> ServiceResponse:
        """调用微服务API"""
        service_url = MICROSERVICES.get(service_name)
        if not service_url:
            return ServiceResponse(service_name, False, error=f"服务 {service_name} 未配置")

        url = f"{service_url}/{endpoint}"

        try:
            start_time = time.time()

            if method.upper() == 'GET':
                response = requests.get(url, params=data, timeout=10)
            else:
                response = requests.post(url, json=data, timeout=10)

            response_time = time.time() - start_time

            if response.status_code == 200:
                return ServiceResponse(
                    service=service_name,
                    success=True,
                    data=response.json(),
                    response_time=response_time
                )
            else:
                return ServiceResponse(
                    service=service_name,
                    success=False,
                    error=f"HTTP {response.status_code}: {response.text}",
                    response_time=response_time
                )

        except requests.exceptions.RequestException as e:
            return ServiceResponse(
                service=service_name,
                success=False,
                error=f"请求异常: {str(e)}",
                response_time=time.time() - start_time
            )

# 订单处理任务
@app.task(bind=True, max_retries=3)
def process_order(self, order_data: Dict):
    """处理订单的主要任务"""
    try:
        print(f"开始处理订单: {order_data.get('order_id')}")

        # 1. 验证用户信息
        user_validation = validate_user_info.delay(
            order_data['user_id'],
            order_data.get('shipping_address')
        )

        # 2. 检查库存
        inventory_check = check_inventory.delay(order_data['items'])

        # 3. 验证支付方式
        payment_validation = validate_payment_method.delay(
            order_data['payment_method'],
            order_data['total_amount']
        )

        # 等待所有验证完成
        user_result = user_validation.get(timeout=30)
        inventory_result = inventory_check.get(timeout=30)
        payment_result = payment_validation.get(timeout=30)

        # 检查验证结果
        if not all([user_result['valid'], inventory_result['available'], payment_result['valid']]):
            error_msg = "订单验证失败"
            if not user_result['valid']:
                error_msg += f", 用户验证错误: {user_result['error']}"
            if not inventory_result['available']:
                error_msg += f", 库存不足: {inventory_result['unavailable_items']}"
            if not payment_result['valid']:
                error_msg += f", 支付验证错误: {payment_result['error']}"

            raise ValueError(error_msg)

        # 4. 处理支付
        payment_result = process_payment.delay(
            order_data['order_id'],
            order_data['total_amount'],
            order_data['payment_method'],
            order_data['user_id']
        )

        payment_info = payment_result.get(timeout=60)

        if not payment_info['success']:
            raise ValueError(f"支付失败: {payment_info['error']}")

        # 5. 更新库存
        inventory_update = update_inventory.delay(order_data['items'])
        inventory_result = inventory_update.get(timeout=30)

        if not inventory_result['success']:
            # 如果库存更新失败，尝试退款
            refund_payment.delay(
                order_data['order_id'],
                payment_info['transaction_id']
            )
            raise ValueError(f"库存更新失败: {inventory_result['error']}")

        # 6. 创建订单记录
        order_creation = create_order_record.delay(
            order_data,
            payment_info['transaction_id']
        )

        order_record = order_creation.get(timeout=30)

        # 7. 发送确认通知
        send_order_confirmation.delay(
            order_data['user_id'],
            order_record['order_id'],
            order_data
        )

        # 8. 更新分析数据
        update_analytics.delay(
            'order_completed',
            {
                'order_id': order_data['order_id'],
                'user_id': order_data['user_id'],
                'total_amount': order_data['total_amount'],
                'items_count': len(order_data['items'])
            }
        )

        final_result = {
            'order_id': order_data['order_id'],
            'status': 'completed',
            'transaction_id': payment_info['transaction_id'],
            'processed_at': time.time(),
            'order_record_id': order_record['record_id']
        }

        print(f"订单处理完成: {order_data['order_id']}")
        return final_result

    except Exception as exc:
        # 错误处理和重试逻辑
        error_msg = f"订单处理失败: {str(exc)}"
        print(error_msg)

        # 发送错误通知
        send_error_notification.delay(
            'order_processing',
            order_data.get('order_id', 'unknown'),
            str(exc)
        )

        if self.request.retries < self.max_retries:
            # 指数退避重试
            countdown = 2 ** self.request.retries
            raise self.retry(countdown=countdown)
        else:
            # 达到最大重试次数，标记为失败
            mark_order_failed.delay(
                order_data.get('order_id'),
                str(exc)
            )
            raise

# 子任务定义
@app.task
def validate_user_info(user_id: str, shipping_address: Dict = None) -> Dict:
    """验证用户信息"""
    print(f"验证用户信息: {user_id}")

    # 调用用户服务
    response = ServiceCommunicator.call_service(
        'user_service',
        f'users/{user_id}/validate',
        {'shipping_address': shipping_address}
    )

    if response.success:
        return {
            'valid': response.data['valid'],
            'user_info': response.data.get('user_info'),
            'error': None
        }
    else:
        return {
            'valid': False,
            'user_info': None,
            'error': response.error
        }

@app.task
def check_inventory(items: List[Dict]) -> Dict:
    """检查库存"""
    print(f"检查库存: {len(items)} 种商品")

    response = ServiceCommunicator.call_service(
        'inventory_service',
        'check',
        {'items': items}
    )

    if response.success:
        return {
            'available': response.data['all_available'],
            'unavailable_items': response.data.get('unavailable_items', []),
            'reserved_items': response.data.get('reserved_items', [])
        }
    else:
        return {
            'available': False,
            'unavailable_items': [item['id'] for item in items],
            'error': response.error
        }

@app.task
def validate_payment_method(payment_method: str, amount: float) -> Dict:
    """验证支付方式"""
    print(f"验证支付方式: {payment_method}, 金额: {amount}")

    response = ServiceCommunicator.call_service(
        'payment_service',
        'validate',
        {'method': payment_method, 'amount': amount}
    )

    if response.success:
        return {
            'valid': response.data['valid'],
            'payment_details': response.data.get('details'),
            'error': None
        }
    else:
        return {
            'valid': False,
            'payment_details': None,
            'error': response.error
        }

@app.task
def process_payment(order_id: str, amount: float, method: str, user_id: str) -> Dict:
    """处理支付"""
    print(f"处理支付: 订单 {order_id}, 金额 {amount}")

    response = ServiceCommunicator.call_service(
        'payment_service',
        'process',
        {
            'order_id': order_id,
            'amount': amount,
            'method': method,
            'user_id': user_id
        }
    )

    if response.success:
        return {
            'success': True,
            'transaction_id': response.data['transaction_id'],
            'payment_details': response.data,
            'error': None
        }
    else:
        return {
            'success': False,
            'transaction_id': None,
            'payment_details': None,
            'error': response.error
        }

@app.task
def update_inventory(items: List[Dict]) -> Dict:
    """更新库存"""
    print(f"更新库存: {len(items)} 种商品")

    response = ServiceCommunicator.call_service(
        'inventory_service',
        'update',
        {'items': items}
    )

    if response.success:
        return {
            'success': True,
            'updated_items': response.data['updated_items'],
            'error': None
        }
    else:
        return {
            'success': False,
            'updated_items': [],
            'error': response.error
        }

@app.task
def create_order_record(order_data: Dict, transaction_id: str) -> Dict:
    """创建订单记录"""
    print(f"创建订单记录: {order_data['order_id']}")

    response = ServiceCommunicator.call_service(
        'order_service',
        'create',
        {
            'order_data': order_data,
            'transaction_id': transaction_id
        }
    )

    if response.success:
        return {
            'record_id': response.data['record_id'],
            'order_status': response.data['status']
        }
    else:
        raise Exception(f"创建订单记录失败: {response.error}")

@app.task
def send_order_confirmation(user_id: str, order_id: str, order_data: Dict):
    """发送订单确认"""
    print(f"发送订单确认: 用户 {user_id}, 订单 {order_id}")

    response = ServiceCommunicator.call_service(
        'notification_service',
        'send',
        {
            'user_id': user_id,
            'type': 'order_confirmation',
            'data': {
                'order_id': order_id,
                'order_details': order_data
            }
        }
    )

    return {
        'notification_sent': response.success,
        'notification_id': response.data.get('notification_id') if response.success else None
    }

@app.task
def update_analytics(event_type: str, data: Dict):
    """更新分析数据"""
    print(f"更新分析数据: {event_type}")

    response = ServiceCommunicator.call_service(
        'analytics_service',
        'track',
        {
            'event_type': event_type,
            'data': data,
            'timestamp': time.time()
        }
    )

    return {
        'tracked': response.success,
        'analytics_id': response.data.get('event_id') if response.success else None
    }

# 错误处理任务
@app.task
def refund_payment(order_id: str, transaction_id: str):
    """退款处理"""
    print(f"处理退款: 订单 {order_id}, 交易 {transaction_id}")

    response = ServiceCommunicator.call_service(
        'payment_service',
        'refund',
        {
            'order_id': order_id,
            'transaction_id': transaction_id
        }
    )

    return {
        'refund_processed': response.success,
        'refund_id': response.data.get('refund_id') if response.success else None,
        'error': response.error if not response.success else None
    }

@app.task
def mark_order_failed(order_id: str, error_message: str):
    """标记订单失败"""
    print(f"标记订单失败: {order_id}, 原因: {error_message}")

    response = ServiceCommunicator.call_service(
        'order_service',
        f'orders/{order_id}/mark_failed',
        {'error_message': error_message}
    )

    return {
        'marked_failed': response.success,
        'error': response.error if not response.success else None
    }

@app.task
def send_error_notification(component: str, reference_id: str, error_message: str):
    """发送错误通知"""
    print(f"发送错误通知: {component} - {reference_id}")

    # 这里可以实现发送邮件、Slack通知等
    error_report = {
        'component': component,
        'reference_id': reference_id,
        'error_message': error_message,
        'timestamp': time.time(),
        'severity': 'high'
    }

    # 记录错误日志
    print(f"错误报告: {json.dumps(error_report, indent=2)}")

    return error_report

# 测试微服务集成
def test_microservices_integration():
    print("=== 微服务集成测试 ===")

    # 创建测试订单数据
    test_order = {
        'order_id': f'test_order_{int(time.time())}',
        'user_id': 'test_user_123',
        'items': [
            {'id': 'product_1', 'name': '商品A', 'quantity': 2, 'price': 99.99},
            {'id': 'product_2', 'name': '商品B', 'quantity': 1, 'price': 199.99}
        ],
        'total_amount': 399.97,
        'payment_method': 'credit_card',
        'shipping_address': {
            'street': '测试街道123号',
            'city': '测试城市',
            'postal_code': '12345',
            'country': '中国'
        }
    }

    print(f"\n测试订单数据:")
    print(f"订单ID: {test_order['order_id']}")
    print(f"用户ID: {test_order['user_id']}")
    print(f"商品数量: {len(test_order['items'])}")
    print(f"总金额: ¥{test_order['total_amount']}")

    # 启动订单处理流程
    print(f"\n启动订单处理流程...")
    result = process_order.delay(test_order)

    print(f"任务ID: {result.id}")
    print("等待订单处理完成...")

    try:
        final_result = result.get(timeout=300)  # 5分钟超时

        print(f"\n=== 订单处理完成 ===")
        print(f"订单ID: {final_result['order_id']}")
        print(f"状态: {final_result['status']}")
        print(f"交易ID: {final_result['transaction_id']}")
        print(f"处理时间: {time.ctime(final_result['processed_at'])}")

    except Exception as e:
        print(f"\n订单处理失败: {e}")
        print("请检查各个微服务的状态和网络连接")

# 服务健康检查
@app.task
def microservices_health_check():
    """检查所有微服务的健康状态"""
    print("执行微服务健康检查...")

    health_results = {}

    for service_name, service_url in MICROSERVICES.items():
        try:
            response = ServiceCommunicator.call_service(
                service_name,
                'health'
            )

            health_results[service_name] = {
                'status': 'healthy' if response.success else 'unhealthy',
                'response_time': response.response_time,
                'error': response.error if not response.success else None
            }

        except Exception as e:
            health_results[service_name] = {
                'status': 'unreachable',
                'error': str(e)
            }

    # 计算整体健康状态
    healthy_count = sum(1 for result in health_results.values() if result['status'] == 'healthy')
    total_count = len(health_results)

    overall_status = 'healthy' if healthy_count == total_count else 'partial' if healthy_count > 0 else 'unhealthy'

    health_report = {
        'timestamp': time.time(),
        'overall_status': overall_status,
        'healthy_services': healthy_count,
        'total_services': total_count,
        'services': health_results
    }

    print(f"健康检查完成: {healthy_count}/{total_count} 服务健康")

    return health_report

if __name__ == '__main__':
    # 测试微服务集成
    test_microservices_integration()

    # 测试健康检查
    print(f"\n=== 微服务健康检查 ===")
    health_result = microservices_health_check.delay()
    health_data = health_result.get(timeout=30)

    print(f"整体状态: {health_data['overall_status']}")
    for service, status in health_data['services'].items():
        print(f"  {service}: {status['status']} (响应时间: {status.get('response_time', 'N/A')}ms)")
```

**部署配置:**
```yaml
# docker-compose.microservices.yml
version: '3.8'

services:
  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"

  # 主应用
  celery-main:
    build: .
    command: celery -A microservices_integration worker --loglevel=info --queues=main
    environment:
      - CELERY_BROKER_URL=redis://redis:6379
    depends_on:
      - redis

  # 用户服务 (模拟)
  user-service:
    image: python:3.9-slim
    command: python -m flask run --host=0.0.0.0 --port=8001
    working_dir: /app
    volumes:
      - ./mock_services:/app
    environment:
      - FLASK_APP=user_service.py
      - FLASK_ENV=development
    ports:
      - "8001:8001"

  # 订单服务 (模拟)
  order-service:
    image: python:3.9-slim
    command: python -m flask run --host=0.0.0.0 --port=8002
    working_dir: /app
    volumes:
      - ./mock_services:/app
    environment:
      - FLASK_APP=order_service.py
      - FLASK_ENV=development
    ports:
      - "8002:8002"

  # 其他微服务...
  payment-service:
    image: python:3.9-slim
    command: python -m flask run --host=0.0.0.0 --port=8003
    working_dir: /app
    volumes:
      - ./mock_services:/app
    environment:
      - FLASK_APP=payment_service.py
      - FLASK_ENV=development
    ports:
      - "8003:8003"

  # 监控
  flower:
    build: .
    command: celery -A microservices_integration flower --port=5555
    ports:
      - "5555:5555"
    environment:
      - CELERY_BROKER_URL=redis://redis:6379
    depends_on:
      - redis
```

**验证要点:**
- [ ] 所有微服务能够正常通信
- [ ] 订单处理流程完整执行
- [ ] 错误处理和重试机制正常
- [ ] 性能监控和日志记录

---

## 项目练习

### 项目 1: 电商订单处理系统 ⭐⭐⭐

**目标:** 构建完整的电商后端系统

**需求规格:**
```python
# ecommerce_system.py
"""
电商订单处理系统项目要求:

1. 用户管理
   - 用户注册和验证
   - 用户信息管理
   - 用户权限控制

2. 商品管理
   - 商品信息管理
   - 库存管理
   - 价格管理

3. 订单处理
   - 订单创建和验证
   - 库存预留和更新
   - 支付处理
   - 订单状态跟踪

4. 通知系统
   - 邮件通知
   - SMS通知
   - 应用内通知

5. 报表和分析
   - 销售报表
   - 用户行为分析
   - 库存分析

6. 系统管理
   - 系统监控
   - 错误处理
   - 性能优化
"""
```

**实现步骤:**
1. 设计系统架构
2. 实现核心任务
3. 创建工作流
4. 添加监控和日志
5. 性能优化
6. 部署和测试

**评估标准:**
- 功能完整性 (40%)
- 代码质量 (25%)
- 性能表现 (20%)
- 文档和测试 (15%)

---

## 调试练习

### 调试 1: 内存泄漏检测 ⭐⭐⭐

**场景:** Celery worker 内存使用持续增长

**调试代码:**
```python
# memory_debug.py
import psutil
import os
import time
from celery import Celery

app = Celery('memory_debug', broker='redis://localhost:6379')

@app.task
def memory_intensive_task(data_size):
    """内存密集型任务"""
    process = psutil.Process(os.getpid())

    # 记录任务开始时的内存使用
    start_memory = process.memory_info().rss / 1024 / 1024  # MB

    # 分配大量内存
    large_data = [0] * (data_size * 1024 * 1024 // 8)  # 大约 data_size MB

    # 模拟处理时间
    time.sleep(2)

    # 清理数据 (这里可能有问题)
    # large_data = None  # 显式设置为None

    # 记录任务结束时的内存使用
    end_memory = process.memory_info().rss / 1024 / 1024  # MB

    return {
        'start_memory_mb': start_memory,
        'end_memory_mb': end_memory,
        'memory_increase': end_memory - start_memory,
        'data_size_mb': data_size
    }

def debug_memory_leak():
    """调试内存泄漏"""
    print("=== 内存泄漏调试 ===")

    # 监控 worker 内存使用
    for i in range(10):
        result = memory_intensive_task.delay(50)  # 50MB数据
        print(f"任务 {i+1} 结果: {result.get(timeout=30)}")
        time.sleep(1)

if __name__ == '__main__':
    debug_memory_leak()
```

**调试要点:**
1. 识别内存泄漏的原因
2. 分析任务执行前后的内存使用
3. 实现内存优化方案
4. 验证修复效果

---

## 总结

这个实践练习指南提供了从基础到高级的全面练习，涵盖了 Celery 开发的各个方面。通过这些练习，学员将能够:

1. **掌握基础概念** - 通过基础练习理解核心概念
2. **提升实战技能** - 通过进阶练习解决实际问题
3. **解决复杂问题** - 通过高级练习处理系统级挑战
4. **积累项目经验** - 通过项目练习获得完整开发经验
5. **提升调试能力** - 通过调试练习快速定位和解决问题

每个练习都包含详细的代码示例、验证要点和扩展挑战，确保学员能够循序渐进地提升技能水平。