# Celery Course Interactive Components and Hands-On Labs
**Last Updated:** 2025-11-15
**Version:** 1.0

---

## Overview

This document contains interactive components, hands-on labs, and practical exercises designed to provide students with real-world experience using Celery. Each lab includes setup instructions, step-by-step guidance, and expected outcomes.

## Learning Path 1: Celery Foundations Labs

### Lab 1.1: Interactive Broker Comparison

**Objective**: Experience the differences between Redis and RabbitMQ brokers

**Duration**: 45 minutes

**Setup**:
```bash
# Clone the lab repository
git clone https://github.com/celerycourse/broker-comparison-lab.git
cd broker-comparison-lab

# Setup Python environment
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate
pip install -r requirements.txt

# Start brokers
docker-compose up -d redis rabbitmq
```

**Interactive Exercises**:

1. **Message Persistence Test**
   ```python
   # broker_test.py
   from tasks import simple_task
   import time

   # Test with Redis
   app = make_celery('redis_app', broker='redis://localhost:6379/0')

   # Send tasks
   for i in range(10):
       simple_task.delay(f'redis-{i}')

   # Stop Redis
   docker-compose stop redis

   # Wait and restart
   time.sleep(5)
   docker-compose start redis

   # Check if tasks survived
   # TODO: Add monitoring code
   ```

2. **Throughput Comparison**
   ```python
   # throughput_test.py
   import time
   from tasks import cpu_bound_task

   def test_throughput(broker_url, num_tasks=1000):
       app = make_celery('test', broker=broker_url)
       start = time.time()

       results = [cpu_bound_task.delay(i) for i in range(num_tasks)]

       for result in results:
           result.get()

       duration = time.time() - start
       return num_tasks / duration

   # Test both brokers
   redis_rate = test_throughput('redis://localhost:6379/0')
   rabbit_rate = test_throughput('pyamqp://guest@localhost//')

   print(f"Redis: {redis_rate:.2f} tasks/sec")
   print(f"RabbitMQ: {rabbit_rate:.2f} tasks/sec")
   ```

**Expected Outcomes**:
- Understand message persistence differences
- Measure throughput variations
- Identify use cases for each broker

**Discussion Questions**:
1. When would you choose Redis over RabbitMQ?
2. How does broker choice affect reliability?
3. What trade-offs exist between brokers?

---

### Lab 1.2: Task Execution Playground

**Objective**: Explore different task execution modes and patterns

**Duration**: 60 minutes

**Setup**:
```bash
cd task-playground
pip install -r requirements.txt
celery -A tasks worker --loglevel=info &
```

**Interactive Components**:

1. **Execution Mode Explorer**
   ```python
   # execution_modes.py
   from tasks import *
   import time

   def explore_execution_modes():
       print("=== Synchronous Execution ===")
       start = time.time()
       result1 = add.apply(args=(4, 4))
       print(f"Result: {result1.get()}, Time: {time.time() - start}")

       print("\n=== Asynchronous Execution ===")
       start = time.time()
       result2 = add.delay(4, 4)
       print(f"Task ID: {result2.id}")
       print(f"Result: {result2.get()}, Time: {time.time() - start}")

       print("\n=== Direct Execution ===")
       start = time.time()
       result3 = add.run(4, 4)
       print(f"Result: {result3}, Time: {time.time() - start}")

   explore_execution_modes()
   ```

2. **Task State Monitor**
   ```python
   # state_monitor.py
   from celery import current_app
   import time

   def monitor_task_progress(task_id):
       result = current_app.AsyncResult(task_id)

       states = ['PENDING', 'STARTED', 'RETRY', 'FAILURE', 'SUCCESS']

       while result.state not in ['FAILURE', 'SUCCESS']:
           print(f"Task {task_id}: {result.state}")
           if result.info:
               print(f"  Info: {result.info}")
           time.sleep(1)
           result = current_app.AsyncResult(task_id)

       print(f"Final state: {result.state}")
       if result.successful():
           print(f"Result: {result.result}")
       else:
           print(f"Error: {result.info}")
   ```

**Hands-On Exercise**: Build a task progress tracker

```python
# progress_tracker.py
from celery import Task
import time

class TrackableTask(Task):
    def on_success(self, retval, task_id, args, kwargs):
        print(f"✓ Task {task_id} completed with result: {retval}")

    def on_failure(self, exc, task_id, args, kwargs, einfo):
        print(f"✗ Task {task_id} failed: {exc}")

    def on_retry(self, exc, task_id, args, kwargs, einfo):
        print(f"↻ Task {task_id} retrying...")

@app.task(bind=True, base=TrackableTask)
def long_running_task(self, duration):
    """Task that simulates work and reports progress"""
    steps = 10
    for i in range(steps):
        time.sleep(duration / steps)
        progress = int((i + 1) / steps * 100)
        self.update_state(
            state='PROGRESS',
            meta={'current': i + 1, 'total': steps, 'percent': progress}
        )
    return f"Completed {duration} seconds of work"

# Interactive monitoring
if __name__ == '__main__':
    task = long_running_task.delay(30)
    monitor_task_progress(task.id)
```

---

### Lab 1.3: Error Handling Workshop

**Objective**: Master error handling and retry patterns

**Duration**: 75 minutes

**Setup**:
```bash
cd error-handling-workshop
pip install -r requirements.txt
```

**Interactive Scenarios**:

1. **Retry Patterns Explorer**
   ```python
   # retry_patterns.py
   from celery import Celery
   import random
   import time

   app = Celery('retry_demo')
   app.conf.update(
       broker_url='redis://localhost:6379/0',
       result_backend='redis://localhost:6379/0'
   )

   @app.task(bind=True, max_retries=3, default_retry_delay=1)
   def flaky_task(self, data):
       """Task that randomly fails"""
       if random.random() < 0.7:  # 70% chance of failure
           print(f"Attempt {self.request.retries + 1} failed")
           raise Exception("Random failure occurred")

       return f"Success after {self.request.retries} retries"

   @app.task(bind=True, max_retries=5)
   def exponential_backoff_task(self):
       """Task with exponential backoff"""
       try:
           # Simulate API call
           if random.random() < 0.8:
               raise ConnectionError("API unavailable")
           return "API call successful"
       except ConnectionError as exc:
           countdown = 2 ** self.request.retries
           print(f"Retrying in {countdown} seconds...")
           raise self.retry(exc=exc, countdown=countdown)

   # Interactive testing
   def test_retry_patterns():
       print("Testing flaky task:")
       result = flaky_task.delay("test data")
       print(f"Final result: {result.get(timeout=30)}")

       print("\nTesting exponential backoff:")
       result2 = exponential_backoff_task.delay()
       print(f"Final result: {result2.get(timeout=60)}")
   ```

2. **Dead Letter Queue Handler**
   ```python
   # dlq_handler.py
   from celery import Celery
   import json

   app = Celery('dlq_demo')

   @app.task(bind=True)
   def process_with_dlq(self, data):
       """Task that sends failed items to DLQ"""
       try:
           # Process data
           if not data.get('valid'):
               raise ValueError("Invalid data")
           return f"Processed: {data}"
       except Exception as exc:
           # Send to DLQ
           app.send_task(
               'dlq_handler.process_failed_item',
               args=[data, str(exc)],
               queue='dead_letter'
           )
           raise

   @app.task
   def process_failed_item(data, error):
       """Process items from dead letter queue"""
       print(f"Processing failed item: {data}")
       print(f"Original error: {error}")

       # Manual intervention or automatic recovery logic
       # ... recovery code ...

       return f"Recovered: {data}"
   ```

**Exercise**: Build a resilient data processor

```python
# resilient_processor.py
from celery import chain, group
from functools import wraps

def circuit_breaker(failure_threshold=5, timeout=60):
    """Decorator to implement circuit breaker pattern"""
    def decorator(func):
        func.failure_count = 0
        func.last_failure_time = None
        func.state = 'CLOSED'  # CLOSED, OPEN, HALF_OPEN

        @wraps(func)
        def wrapper(*args, **kwargs):
            now = time.time()

            # Check if we should try again
            if func.state == 'OPEN':
                if now - func.last_failure_time > timeout:
                    func.state = 'HALF_OPEN'
                else:
                    raise Exception("Circuit breaker is OPEN")

            try:
                result = func(*args, **kwargs)

                # Success - reset or close circuit
                if func.state == 'HALF_OPEN':
                    func.state = 'CLOSED'
                func.failure_count = 0
                return result

            except Exception as e:
                func.failure_count += 1
                func.last_failure_time = now

                # Open circuit if threshold reached
                if func.failure_count >= failure_threshold:
                    func.state = 'OPEN'
                    print(f"Circuit breaker OPEN after {func.failure_count} failures")

                raise e

        return wrapper
    return decorator

@app.task(bind=True)
@circuit_breaker(failure_threshold=3, timeout=30)
def external_api_call(self, endpoint):
    """Task with circuit breaker protection"""
    # Simulate API call
    if random.random() < 0.5:
        raise ConnectionError("API unavailable")

    return f"Data from {endpoint}"
```

---

## Learning Path 2: Intermediate Celery Labs

### Lab 2.1: Canvas Workflow Builder

**Objective**: Master Celery's workflow primitives

**Duration**: 90 minutes

**Interactive Tool**: Web-based workflow designer

**Setup**:
```bash
cd canvas-workshop
pip install -r requirements.txt
# Optional: npm install && npm start for web UI
```

**Component 1: Visual Workflow Designer**
```python
# workflow_designer.py
from celery import chain, group, chord, signature
import json

class WorkflowDesigner:
    def __init__(self):
        self.tasks = {}
        self.workflow = None

    def add_task(self, name, task_func, *args, **kwargs):
        """Add a task to the workflow"""
        self.tasks[name] = signature(task_func, args=args, kwargs=kwargs)
        return self

    def chain_tasks(self, *task_names):
        """Create a chain of tasks"""
        signatures = [self.tasks[name] for name in task_names]
        self.workflow = chain(*signatures)
        return self

    def group_tasks(self, *task_names):
        """Create a group of parallel tasks"""
        signatures = [self.tasks[name] for name in task_names]
        self.workflow = group(*signatures)
        return self

    def chord_workflow(self, group_tasks, callback_task):
        """Create a chord workflow"""
        group_sig = group([self.tasks[name] for name in group_tasks])
        callback_sig = self.tasks[callback_task]
        self.workflow = chord(group_sig, callback_sig)
        return self

    def execute(self):
        """Execute the workflow"""
        if self.workflow:
            return self.workflow()
        raise ValueError("No workflow defined")

    def visualize(self):
        """Generate workflow visualization data"""
        # Return JSON for visualization library
        return {
            'tasks': list(self.tasks.keys()),
            'workflow_type': type(self.workflow).__name__,
            'structure': self.get_structure()
        }

# Interactive demo
def build_data_pipeline():
    designer = WorkflowDesigner()

    # Add tasks
    designer.add_task('fetch', fetch_data.s('api'))
    designer.add_task('validate', validate_data.s())
    designer.add_task('transform', transform_data.s())
    designer.add_task('load', load_data.s('database'))
    designer.add_task('notify', send_notification.s())

    # Build pipeline
    workflow = designer.chain_tasks('fetch', 'validate', 'transform', 'load')

    # Execute with notification on completion
    final_workflow = chain(
        workflow,
        notify.s()
    )

    return final_workflow.apply_async()
```

**Component 2: Complex Pattern Explorer**
```python
# pattern_explorer.py
from celery import group, chord, chain
from itertools import product

class PatternExplorer:
    @staticmethod
    def map_reduce_pattern(items, map_task, reduce_task):
        """Implement MapReduce pattern"""
        # Map phase - parallel processing
        map_group = group(map_task.s(item) for item in items)

        # Reduce phase - aggregate results
        workflow = chord(map_group, reduce_task.s())

        return workflow()

    @staticmethod
    def fan_out_fan_in_pattern(data, processors):
        """Fan out to multiple processors, then fan in"""
        # Fan out
        tasks = group(processor.s(data) for processor in processors)

        # Fan in with aggregator
        result = chord(tasks, aggregate_results.s())

        return result()

    @staticmethod
    def pipeline_pattern(data, stages):
        """Sequential processing pipeline"""
        return chain(*[stage.s() for stage in stages])(data)

    @staticmethod
    diamond_pattern(data, task1, task2, join_task):
        """Diamond pattern: split, parallel process, join"""
        parallel_tasks = group(task1.s(data), task2.s(data))
        return chord(parallel_tasks, join_task.s())()

# Interactive pattern testing
def test_patterns():
    explorer = PatternExplorer()

    # Test MapReduce
    numbers = list(range(100))
    map_reduce = explorer.map_reduce_pattern(
        numbers,
        square_task,  # Square each number
        sum_results   # Sum all squares
    )

    # Test fan-out/fan-in
    fan_out = explorer.fan_out_fan_in_pattern(
        "data",
        [process_a, process_b, process_c],
        combine_results
    )

    # Test diamond pattern
    diamond = explorer.diamond_pattern(
        "input",
        transform_a,
        transform_b,
        merge_transforms
    )

    return map_reduce, fan_out, diamond
```

---

### Lab 2.2: Real-time Monitoring Dashboard

**Objective**: Build a comprehensive monitoring solution

**Duration**: 120 minutes

**Setup**:
```bash
cd monitoring-lab
pip install -r requirements.txt
docker-compose up -d redis grafana prometheus
```

**Interactive Dashboard Components**:

1. **WebSocket Monitor**
   ```python
   # realtime_monitor.py
   from flask import Flask, render_template
   from flask_socketio import SocketIO, emit
   from celery.events.state import State
   import redis
   import json

   app = Flask(__name__)
   app.config['SECRET_KEY'] = 'monitor-secret'
   socketio = SocketIO(app, cors_allowed_origins="*")

   # Redis pubsub for real-time events
   redis_client = redis.Redis()
   pubsub = redis_client.pubsub()

   class CeleryMonitor:
       def __init__(self):
           self.state = State()
           self.task_counts = {
               'PENDING': 0,
               'STARTED': 0,
               'SUCCESS': 0,
               'FAILURE': 0,
               'RETRY': 0
           }
           self.worker_stats = {}

       def capture_event(self, event):
           """Capture and process Celery events"""
           self.state.event(event)

           # Update counts
           task_type = event['type']
           if task_type in self.task_counts:
               self.task_counts[task_type] += 1

           # Emit to WebSocket clients
           socketio.emit('task_event', {
               'type': task_type,
               'task_id': event.get('uuid'),
               'worker': event.get('hostname'),
               'timestamp': event.get('timestamp')
           })

           # Update dashboard data
           self.update_dashboard()

       def update_dashboard(self):
           """Send dashboard updates"""
           socketio.emit('dashboard_update', {
               'task_counts': self.task_counts,
               'worker_stats': self.worker_stats,
               'timestamp': time.time()
           })

   monitor = CeleryMonitor()

   @app.route('/')
   def index():
       return render_template('dashboard.html')

   @socketio.on('connect')
   def handle_connect():
       emit('connected', {'data': 'Connected to monitoring'})
       # Send initial state
       emit('dashboard_update', {
           'task_counts': monitor.task_counts,
           'worker_stats': monitor.worker_stats
       })

   if __name__ == '__main__':
       # Start Celery event capture in background
       import threading
       from celery import Celery

       def capture_events():
           app_inspect = Celery().control.inspect()
           while True:
               try:
                   # Get active tasks
                   active = app_inspect.active()
                   if active:
                       for worker, tasks in active.items():
                           monitor.worker_stats[worker] = {
                               'active_tasks': len(tasks),
                               'last_seen': time.time()
                           }
                   time.sleep(1)
               except:
                   pass

       threading.Thread(target=capture_events, daemon=True).start()
       socketio.run(app, debug=True)
   ```

2. **Metrics Collector**
   ```python
   # metrics_collector.py
   from prometheus_client import CollectorRegistry, Gauge, Counter, Histogram, start_http_server
   import time
   import psutil

   class CeleryMetrics:
       def __init__(self):
           self.registry = CollectorRegistry()

           # Define metrics
           self.task_total = Counter(
               'celery_tasks_total',
               'Total number of tasks',
               ['task_name', 'status'],
               registry=self.registry
           )

           self.task_duration = Histogram(
               'celery_task_duration_seconds',
               'Task execution time',
               ['task_name'],
               registry=self.registry
           )

           self.worker_memory = Gauge(
               'celery_worker_memory_bytes',
               'Worker memory usage',
               ['worker_name'],
               registry=self.registry
           )

           self.queue_length = Gauge(
               'celery_queue_length',
               'Number of tasks in queue',
               ['queue_name'],
               registry=self.registry
           )

       def record_task(self, task_name, status, duration=None):
           """Record task metrics"""
           self.task_total.labels(task_name=task_name, status=status).inc()

           if duration is not None:
               self.task_duration.labels(task_name=task_name).observe(duration)

       def update_worker_metrics(self, worker_name):
           """Update worker resource metrics"""
           process = psutil.Process()
           memory = process.memory_info().rss
           self.worker_memory.labels(worker_name=worker_name).set(memory)

       def update_queue_metrics(self, queue_name, length):
           """Update queue length"""
           self.queue_length.labels(queue_name=queue_name).set(length)

   # Start metrics server
   metrics = CeleryMetrics()
   start_http_server(8000, registry=metrics.registry)
   ```

3. **Alerting System**
   ```python
   # alerting_system.py
   from celery.signals import task_failure, task_prerun, task_postrun
   import smtplib
   from email.mime.text import MIMEText
   import json

   class AlertManager:
       def __init__(self):
           self.thresholds = {
               'failure_rate': 0.1,  # 10% failure rate
               'task_duration': 300,  # 5 minutes
               'queue_length': 1000
           }
           self.alert_counts = {}
           self.cooldown_period = 300  # 5 minutes

       def check_failure_rate(self, window=60):
           """Check if failure rate exceeds threshold"""
           # Query metrics from last window
           # Implementation depends on your metrics store
           pass

       def send_alert(self, alert_type, message, severity='warning'):
           """Send alert notification"""
           alert = {
               'type': alert_type,
               'message': message,
               'severity': severity,
               'timestamp': time.time()
           }

           # Rate limiting
           key = f"{alert_type}_{message}"
           now = time.time()

           if key in self.alert_counts:
               if now - self.alert_counts[key]['last_sent'] < self.cooldown_period:
                   return  # Skip due to cooldown

           self.alert_counts[key] = {
               'count': self.alert_counts.get(key, {}).get('count', 0) + 1,
               'last_sent': now
           }

           # Send via email
           self.send_email_alert(alert)

           # Send to Slack
           self.send_slack_alert(alert)

       def send_email_alert(self, alert):
           """Send email notification"""
           msg = MIMEText(f"""
           Alert: {alert['type']}
           Message: {alert['message']}
           Severity: {alert['severity']}
           Time: {alert['timestamp']}
           """)

           msg['Subject'] = f"Celery Alert: {alert['type']}"
           msg['From'] = 'alerts@example.com'
           msg['To'] = 'ops@example.com'

           # Implementation depends on your email setup
           # smtp.send_message(msg)

       def send_slack_alert(self, alert):
           """Send Slack notification"""
           webhook_url = 'YOUR_SLACK_WEBHOOK_URL'
           payload = {
               'text': f"Celery Alert: {alert['type']}",
               'attachments': [{
                   'color': 'danger' if alert['severity'] == 'critical' else 'warning',
                   'fields': [
                       {'title': 'Message', 'value': alert['message']},
                       {'title': 'Severity', 'value': alert['severity']}
                   ]
               }]
           }

           # requests.post(webhook_url, json=payload)

   # Connect signals
   alert_manager = AlertManager()

   @task_failure.connect
   def handle_task_failure(sender, task_id, exception, traceback, einfo, **kwargs):
       alert_manager.send_alert(
           'task_failure',
           f"Task {sender.name} failed: {exception}",
           severity='warning'
       )

   @task_postrun.connect
   def handle_task_completion(sender, task_id, retval, state, **kwargs):
       duration = kwargs.get('runtime', 0)
       if duration > alert_manager.thresholds['task_duration']:
           alert_manager.send_alert(
               'slow_task',
               f"Task {sender.name} took {duration}s",
               severity='warning'
           )
   ```

---

## Learning Path 3: Advanced Celery Labs

### Lab 3.1: Custom Extension Development

**Objective**: Develop Celery plugins and extensions

**Duration**: 150 minutes

**Setup**:
```bash
cd extension-dev-lab
pip install -r requirements.txt
pip install -e .
```

**Extension 1: Task Rate Limiter**
```python
# rate_limit_extension.py
from celery import Task
from celery.utils.log import get_task_logger
import time
import threading
from collections import defaultdict, deque

logger = get_task_logger(__name__)

class RateLimitMixin:
    """Mixin to add rate limiting to tasks"""

    def __init__(self):
        super().__init__()
        self._rate_limiter = None

    def apply_async(self, args=None, kwargs=None, task_id=None, **options):
        # Override to check rate limit before queuing
        if self._should_rate_limit():
            logger.info(f"Task {self.name} rate limited")
            return None  # or schedule for later

        return super().apply_async(args, kwargs, task_id, **options)

    def _should_rate_limit(self):
        """Check if task should be rate limited"""
        return False  # Implement in subclass

class TokenBucketRateLimiter:
    """Token bucket rate limiter implementation"""

    def __init__(self, rate, capacity):
        self.rate = rate  # tokens per second
        self.capacity = capacity
        self.tokens = capacity
        self.last_update = time.time()
        self.lock = threading.Lock()

    def consume(self, tokens=1):
        """Consume tokens if available"""
        with self.lock:
            now = time.time()
            # Add tokens based on elapsed time
            elapsed = now - self.last_update
            self.tokens = min(self.capacity, self.tokens + elapsed * self.rate)
            self.last_update = now

            if self.tokens >= tokens:
                self.tokens -= tokens
                return True
            return False

class DistributedRateLimiter:
    """Rate limiter using Redis for distributed coordination"""

    def __init__(self, redis_client, key_prefix='celery_rate_limit:', default_rate=10):
        self.redis = redis_client
        self.key_prefix = key_prefix
        self.default_rate = default_rate

    def is_allowed(self, identifier, limit=None):
        """Check if action is allowed for identifier"""
        if limit is None:
            limit = self.default_rate

        key = f"{self.key_prefix}{identifier}"
        current = self.redis.incr(key)

        if current == 1:
            self.redis.expire(key, 1)  # Reset after 1 second

        return current <= limit

# Custom task with rate limiting
class RateLimitedTask(Task, RateLimitMixin):
    abstract = True
    rate_limit = None  # Override in subclasses
    rate_window = 1    # Time window in seconds

    def __init__(self):
        super().__init__()
        if self.rate_limit:
            self._rate_limiter = DistributedRateLimiter(
                self.app.broker_connection().channel().client,
                default_rate=self.rate_limit
            )

    def apply_async(self, args=None, kwargs=None, **options):
        # Get identifier for rate limiting (e.g., user_id, IP)
        identifier = self._get_rate_limit_identifier(args, kwargs, options)

        if identifier and self._rate_limiter:
            if not self._rate_limiter.is_allowed(identifier):
                logger.warning(f"Rate limit exceeded for {identifier}")
                return None

        return super().apply_async(args, kwargs, **options)

    def _get_rate_limit_identifier(self, args, kwargs, options):
        """Extract identifier from task arguments"""
        # Override in subclasses
        return None

# Usage example
@app.task(base=RateLimitedTask, rate_limit=5)  # 5 tasks per second
def api_call_task(user_id, endpoint):
    """Task limited to 5 calls per second per user"""
    return f"Called {endpoint} for user {user_id}"
```

**Extension 2: Task Dependency Graph**
```python
# dependency_graph.py
from celery import Task
from celery.utils.log import get_task_logger
import networkx as nx
import matplotlib.pyplot as plt
from functools import wraps

logger = get_task_logger(__name__)

class DependencyGraphTask(Task):
    """Task that tracks and enforces dependencies"""

    abstract = True
    _dependencies = []
    _graph = nx.DiGraph()

    def __init__(self):
        super().__init__()
        self._register_task()

    def _register_task(self):
        """Register task in dependency graph"""
        self._graph.add_node(self.name, task=self)

        # Add dependencies
        for dep in self._dependencies:
            self._graph.add_edge(dep, self.name)

    def run(self, *args, **kwargs):
        """Check dependencies before running"""
        if not self._check_dependencies():
            raise Exception(f"Dependencies not satisfied for {self.name}")

        return super().run(*args, **kwargs)

    def _check_dependencies(self):
        """Check if all dependencies have run successfully"""
        for dep in self._dependencies:
            # Check if dependency has been completed
            # Implementation depends on your tracking mechanism
            pass

    def apply_async(self, args=None, kwargs=None, **options):
        """Schedule dependencies if needed"""
        # Ensure dependencies are scheduled
        for dep in self._dependencies:
            dep_task = self._graph.nodes[dep]['task']
            dep_task.apply_async()

        return super().apply_async(args, kwargs, **options)

    @classmethod
    def visualize_graph(cls):
        """Generate visualization of dependency graph"""
        plt.figure(figsize=(12, 8))
        nx.draw(cls._graph, with_labels=True, node_color='lightblue',
                node_size=1000, arrowsize=20, font_size=10)
        plt.title("Task Dependency Graph")
        plt.savefig('dependency_graph.png')
        return 'dependency_graph.png'

    @classmethod
    def find_execution_order(cls):
        """Find valid execution order using topological sort"""
        try:
            return list(nx.topological_sort(cls._graph))
        except nx.NetworkXError:
            logger.error("Dependency graph contains cycles!")
            return None

# Decorator for adding dependencies
def depends_on(*task_names):
    """Decorator to specify task dependencies"""
    def decorator(task_class):
        if not issubclass(task_class, DependencyGraphTask):
            task_class = type(
                task_class.__name__,
                (DependencyGraphTask, task_class),
                {'__module__': task_class.__module__}
            )

        task_class._dependencies = list(task_names)
        return task_class
    return decorator

# Usage examples
@app.task
@depends_on('fetch_data', 'validate_input')
def process_data():
    """Task that depends on fetch_data and validate_input"""
    return "Processing complete"

@app.task
@depends_on('process_data')
def generate_report():
    """Task that depends on process_data"""
    return "Report generated"
```

---

### Lab 3.2: Performance Profiling and Optimization

**Objective**: Profile and optimize Celery applications

**Duration**: 120 minutes

**Setup**:
```bash
cd performance-lab
pip install -r requirements.txt
pip install py-spy memory-profiler line-profiler
```

**Profiler Components**:

1. **Task Profiler**
   ```python
   # task_profiler.py
   from celery import Task
   from functools import wraps
   import time
   import psutil
   import tracemalloc
   from contextlib import contextmanager

   @contextmanager
   def profile_task(task_name):
       """Context manager for profiling tasks"""
       # Start profiling
       tracemalloc.start()
       start_time = time.time()
       start_memory = psutil.Process().memory_info().rss
       cpu_start = psutil.cpu_percent()

       try:
           yield
       finally:
           # Stop profiling
           end_time = time.time()
           end_memory = psutil.Process().memory_info().rss
           cpu_end = psutil.cpu_percent()

           current, peak = tracemalloc.get_traced_memory()
           tracemalloc.stop()

           # Print profiling results
           print(f"\n=== Profile for {task_name} ===")
           print(f"Duration: {end_time - start_time:.3f}s")
           print(f"Memory delta: {(end_memory - start_memory) / 1024 / 1024:.2f}MB")
           print(f"Peak memory: {peak / 1024 / 1024:.2f}MB")
           print(f"CPU usage: {cpu_end - cpu_start:.1f}%")
           print("=" * 40)

   class ProfiledTask(Task):
       """Task that automatically profiles itself"""
       abstract = True
       profile_enabled = True

       def __call__(self, *args, **kwargs):
           if self.profile_enabled:
               with profile_task(self.name):
                   return super().__call__(*args, **kwargs)
           return super().__call__(*args, **kwargs)

   @app.task(base=ProfiledTask)
   def cpu_intensive_task(n):
       """Example CPU-intensive task"""
       result = sum(i * i for i in range(n))
       return result

   @app.task(base=ProfiledTask)
   def memory_intensive_task(size):
       """Example memory-intensive task"""
       data = [list(range(size)) for _ in range(100)]
       return len(data)
   ```

2. **Benchmark Suite**
   ```python
   # benchmark_suite.py
   from celery import group, chain
   import time
   import statistics
   import pandas as pd
   import matplotlib.pyplot as plt

   class CeleryBenchmark:
       def __init__(self, app):
           self.app = app
           self.results = {}

       def benchmark_throughput(self, task, num_tasks=1000, concurrency=1):
           """Benchmark task throughput"""
           times = []

           for _ in range(5):  # Run 5 trials
               start = time.time()

               # Send tasks
               results = [task.delay() for _ in range(num_tasks)]

               # Wait for completion
               for result in results:
                   result.get()

               duration = time.time() - start
               times.append(duration)

           avg_time = statistics.mean(times)
           throughput = num_tasks / avg_time

           self.results['throughput'] = {
               'tasks_per_second': throughput,
               'avg_duration': avg_time,
               'num_tasks': num_tasks,
               'concurrency': concurrency
           }

           return self.results['throughput']

       def benchmark_scalability(self, task, task_counts=[10, 100, 1000, 5000]):
           """Benchmark scalability with increasing task counts"""
           results = []

           for count in task_counts:
               print(f"Testing with {count} tasks...")
               start = time.time()

               # Use group for parallel execution
               job = group(task.s() for _ in range(count))
               result = job.apply_async()
               result.get()  # Wait for all

               duration = time.time() - start
               throughput = count / duration

               results.append({
                   'task_count': count,
                   'duration': duration,
                   'throughput': throughput
               })

           self.results['scalability'] = results
           return results

       def benchmark_queue_performance(self, task, queue_configs):
           """Benchmark different queue configurations"""
           results = []

           for config in queue_configs:
               print(f"Testing config: {config['name']}")

               # Apply configuration
               self.app.conf.update(config['settings'])

               # Run benchmark
               throughput = self.benchmark_throughput(task, 500)

               results.append({
                   'config': config['name'],
                   'throughput': throughput['tasks_per_second']
               })

           self.results['queue_configs'] = results
           return results

       def generate_report(self):
           """Generate benchmark report"""
           df = pd.DataFrame(self.results)

           # Create plots
           if 'scalability' in self.results:
               scalability_df = pd.DataFrame(self.results['scalability'])
               plt.figure(figsize=(10, 6))
               plt.plot(scalability_df['task_count'], scalability_df['throughput'])
               plt.xlabel('Number of Tasks')
               plt.ylabel('Throughput (tasks/sec)')
               plt.title('Scalability Benchmark')
               plt.grid(True)
               plt.savefig('scalability_benchmark.png')

           return df

   # Example usage
   def run_benchmarks():
       benchmark = CeleryBenchmark(app)

       # Benchmark throughput
       benchmark.benchmark_throughput(cpu_intensive_task, 1000)

       # Benchmark scalability
       benchmark.benchmark_scalability(cpu_intensive_task)

       # Benchmark different configurations
       configs = [
           {'name': 'default', 'settings': {}},
           {'name': 'high_prefetch', 'settings': {'worker_prefetch_multiplier': 100}},
           {'name': 'low_prefetch', 'settings': {'worker_prefetch_multiplier': 1}}
       ]
       benchmark.benchmark_queue_performance(cpu_intensive_task, configs)

       # Generate report
       report = benchmark.generate_report()
       report.to_csv('benchmark_results.csv')

       return benchmark
   ```

---

## Learning Path 4: Production & Integration Labs

### Lab 4.1: Kubernetes Deployment Workshop

**Objective**: Deploy Celery on Kubernetes at scale

**Duration**: 180 minutes

**Setup**:
```bash
cd k8s-workshop
kubectl create namespace celery-lab
kubectl apply -f infrastructure/
```

**Kubernetes Manifests**:

1. **Celery Worker Deployment**
   ```yaml
   # k8s/celery-worker.yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: celery-worker
     namespace: celery-lab
   spec:
     replicas: 3
     selector:
       matchLabels:
         app: celery-worker
     template:
       metadata:
         labels:
           app: celery-worker
       spec:
         containers:
         - name: worker
           image: celerycourse/worker:latest
           env:
           - name: CELERY_BROKER_URL
             valueFrom:
               secretKeyRef:
                 name: celery-secrets
                 key: broker-url
           - name: CELERY_RESULT_BACKEND
             valueFrom:
               secretKeyRef:
                 name: celery-secrets
                 key: result-backend
           - name: WORKER_CONCURRENCY
             value: "4"
           - name: WORKER_QUEUES
             value: "default,high_priority,background"
           resources:
             requests:
               cpu: 100m
               memory: 256Mi
             limits:
               cpu: 500m
               memory: 512Mi
           livenessProbe:
             exec:
               command:
               - celery
               - -A
               - app
               - inspect
               - ping
             initialDelaySeconds: 30
             periodSeconds: 10
           readinessProbe:
             exec:
               command:
               - celery
               - -A
               - app
               - inspect
               - stats
             initialDelaySeconds: 10
             periodSeconds: 5
   ```

2. **Horizontal Pod Autoscaler**
   ```yaml
   # k8s/hpa.yaml
   apiVersion: autoscaling/v2
   kind: HorizontalPodAutoscaler
   metadata:
     name: celery-worker-hpa
     namespace: celery-lab
   spec:
     scaleTargetRef:
       apiVersion: apps/v1
       kind: Deployment
       name: celery-worker
     minReplicas: 2
     maxReplicas: 10
     metrics:
     - type: Resource
       resource:
         name: cpu
         target:
           type: Utilization
           averageUtilization: 70
     - type: Resource
       resource:
         name: memory
         target:
           type: Utilization
           averageUtilization: 80
     - type: Pods
       pods:
         metric:
           name: celery_queue_length
         target:
           type: AverageValue
           averageValue: "100"
   ```

3. **ConfigMap for Configuration**
   ```yaml
   # k8s/configmap.yaml
   apiVersion: v1
   kind: ConfigMap
   metadata:
     name: celery-config
     namespace: celery-lab
   data:
     celery.conf.py: |
       broker_url = os.environ.get('CELERY_BROKER_URL')
       result_backend = os.environ.get('CELERY_RESULT_BACKEND')
       task_serializer = 'json'
       accept_content = ['json']
       result_serializer = 'json'
       timezone = 'UTC'
       enable_utc = True

       task_routes = {
           'app.tasks.cpu_intensive': {'queue': 'cpu_queue'},
           'app.tasks.io_intensive': {'queue': 'io_queue'},
           'app.tasks.urgent': {'queue': 'urgent_queue'}
       }

       worker_prefetch_multiplier = 1
       task_acks_late = True
       worker_disable_rate_limits = False
   ```

4. **Monitoring with Prometheus**
   ```yaml
   # k8s/monitoring.yaml
   apiVersion: v1
   kind: ServiceMonitor
   metadata:
     name: celery-metrics
     namespace: celery-lab
   spec:
     selector:
       matchLabels:
         app: celery-exporter
     endpoints:
     - port: metrics
       interval: 30s
   ```

**Deployment Script**:
```python
# deploy_to_k8s.py
from kubernetes import client, config
import yaml
import time

class CeleryK8sDeployer:
    def __init__(self, namespace='celery-lab'):
        config.load_kube_config()
        self.v1 = client.CoreV1Api()
        self.apps_v1 = client.AppsV1Api()
        self.namespace = namespace

    def deploy_infrastructure(self):
        """Deploy Redis and RabbitMQ"""
        # Deploy Redis
        redis_manifest = yaml.safe_load(open('k8s/redis.yaml'))
        self.apps_v1.create_namespaced_deployment(
            namespace=self.namespace,
            body=redis_manifest
        )

        # Deploy RabbitMQ
        rabbitmq_manifest = yaml.safe_load(open('k8s/rabbitmq.yaml'))
        self.apps_v1.create_namespaced_deployment(
            namespace=self.namespace,
            body=rabbitmq_manifest
        )

        print("Infrastructure deployed successfully")

    def deploy_celery_workers(self, image_tag='latest'):
        """Deploy Celery workers with auto-scaling"""
        # Create ConfigMap
        config_map = yaml.safe_load(open('k8s/configmap.yaml'))
        self.v1.create_namespaced_config_map(
            namespace=self.namespace,
            body=config_map
        )

        # Deploy workers
        worker_manifest = yaml.safe_load(open('k8s/celery-worker.yaml'))
        worker_manifest['spec']['template']['spec']['containers'][0]['image'] = f'celerycourse/worker:{image_tag}'

        self.apps_v1.create_namespaced_deployment(
            namespace=self.namespace,
            body=worker_manifest
        )

        # Setup HPA
        hpa_manifest = yaml.safe_load(open('k8s/hpa.yaml'))
        self.apps_v1.create_namespaced_horizontal_pod_autoscaler(
            namespace=self.namespace,
            body=hpa_manifest
        )

        print("Celery workers deployed with auto-scaling")

    def setup_monitoring(self):
        """Deploy monitoring stack"""
        # Deploy Flower
        flower_manifest = yaml.safe_load(open('k8s/flower.yaml'))
        self.apps_v1.create_namespaced_deployment(
            namespace=self.namespace,
            body=flower_manifest
        )

        # Deploy Prometheus exporter
        exporter_manifest = yaml.safe_load(open('k8s/celery-exporter.yaml'))
        self.apps_v1.create_namespaced_deployment(
            namespace=self.namespace,
            body=exporter_manifest
        )

        print("Monitoring stack deployed")

    def check_deployment_status(self):
        """Check deployment status"""
        while True:
            deployments = self.apps_v1.list_namespaced_deployment(namespace=self.namespace)

            all_ready = True
            for dep in deployments.items:
                if dep.status.ready_replicas != dep.spec.replicas:
                    print(f"Waiting for {dep.metadata.name}...")
                    all_ready = False
                    break

            if all_ready:
                print("All deployments are ready!")
                break

            time.sleep(5)

    def scale_workers(self, replicas):
        """Manually scale workers"""
        body = {'spec': {'replicas': replicas}}
        self.apps_v1.patch_namespaced_deployment_scale(
            name='celery-worker',
            namespace=self.namespace,
            body=body
        )
        print(f"Scaled workers to {replicas}")

# Interactive deployment
def interactive_deployment():
    deployer = CeleryK8sDeployer()

    print("=== Celery Kubernetes Deployment ===")
    print("1. Deploy infrastructure (Redis, RabbitMQ)")
    print("2. Deploy Celery workers")
    print("3. Setup monitoring")
    print("4. Check status")
    print("5. Scale workers")

    while True:
        choice = input("\nEnter choice (1-5): ")

        if choice == '1':
            deployer.deploy_infrastructure()
        elif choice == '2':
            tag = input("Enter image tag (latest): ") or 'latest'
            deployer.deploy_celery_workers(tag)
        elif choice == '3':
            deployer.setup_monitoring()
        elif choice == '4':
            deployer.check_deployment_status()
        elif choice == '5':
            replicas = int(input("Number of replicas: "))
            deployer.scale_workers(replicas)
        elif choice == 'q':
            break

if __name__ == '__main__':
    interactive_deployment()
```

---

### Lab 4.2: CI/CD Pipeline Integration

**Objective**: Build complete CI/CD pipeline for Celery applications

**Duration**: 120 minutes

**Setup**:
```bash
cd cicd-lab
# Configure GitHub Actions or GitLab CI
```

**GitHub Actions Workflow**:
```yaml
# .github/workflows/celery-ci-cd.yml
name: Celery CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]
  release:
    types: [published]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        python-version: [3.9, '3.10', 3.11, 3.12]

    services:
      redis:
        image: redis:7
        ports:
          - 6379:6379
        options: >-
          --health-cmd "redis-cli ping"
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

      rabbitmq:
        image: rabbitmq:3-management
        ports:
          - 5672:5672
          - 15672:15672
        env:
          RABBITMQ_DEFAULT_USER: test
          RABBITMQ_DEFAULT_PASS: test
        options: >-
          --health-cmd "rabbitmq-diagnostics -q check_running"
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

    steps:
    - uses: actions/checkout@v4

    - name: Set up Python
      uses: actions/setup-python@v4
      with:
        python-version: ${{ matrix.python-version }}

    - name: Cache pip dependencies
      uses: actions/cache@v3
      with:
        path: ~/.cache/pip
        key: ${{ runner.os }}-pip-${{ hashFiles('**/requirements.txt') }}

    - name: Install dependencies
      run: |
        python -m pip install --upgrade pip
        pip install -r requirements.txt
        pip install -r requirements-dev.txt

    - name: Lint with flake8
      run: |
        flake8 app/ tests/ --count --select=E9,F63,F7,F82 --show-source --statistics
        flake8 app/ tests/ --count --exit-zero --max-complexity=10 --max-line-length=127 --statistics

    - name: Run type checking
      run: |
        mypy app/

    - name: Run unit tests
      run: |
        pytest tests/unit/ -v --cov=app --cov-report=xml

    - name: Run integration tests
      env:
        CELERY_BROKER_URL: redis://localhost:6379/0
        CELERY_RESULT_BACKEND: redis://localhost:6379/0
      run: |
        pytest tests/integration/ -v

    - name: Test with RabbitMQ
      env:
        CELERY_BROKER_URL: pyamqp://test:test@localhost:5672//
        CELERY_RESULT_BACKEND: rpc://
      run: |
        pytest tests/broker_tests/test_rabbitmq.py -v

    - name: Performance tests
      run: |
        pytest tests/performance/ -v --benchmark-only

    - name: Upload coverage
      uses: codecov/codecov-action@v3
      with:
        file: ./coverage.xml
        flags: unittests
        name: codecov-umbrella

  build:
    needs: test
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write

    steps:
    - uses: actions/checkout@v4

    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v3

    - name: Log in to Container Registry
      uses: docker/login-action@v3
      with:
        registry: ${{ env.REGISTRY }}
        username: ${{ github.actor }}
        password: ${{ secrets.GITHUB_TOKEN }}

    - name: Extract metadata
      id: meta
      uses: docker/metadata-action@v5
      with:
        images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
        tags: |
          type=ref,event=branch
          type=ref,event=pr
          type=sha,prefix={{branch}}-
          type=semver,pattern={{version}}
          type=semver,pattern={{major}}.{{minor}}
          type=semver,pattern={{major}}

    - name: Build and push Docker image
      uses: docker/build-push-action@v5
      with:
        context: .
        push: true
        tags: ${{ steps.meta.outputs.tags }}
        labels: ${{ steps.meta.outputs.labels }}
        cache-from: type=gha
        cache-to: type=gha,mode=max

  deploy-staging:
    needs: build
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/develop'
    environment: staging

    steps:
    - uses: actions/checkout@v4

    - name: Configure kubectl
      uses: azure/k8s-set-context@v3
      with:
        method: kubeconfig
        kubeconfig: ${{ secrets.KUBE_CONFIG }}

    - name: Deploy to staging
      run: |
        helm upgrade --install celery-app ./helm/celery-app \
          --namespace celery-staging \
          --create-namespace \
          --set image.tag=${{ github.sha }} \
          --set environment=staging \
          --set replicas=2 \
          --values ./helm/values-staging.yaml

    - name: Run smoke tests
      run: |
        python scripts/smoke_tests.py --environment=staging

  deploy-production:
    needs: build
    runs-on: ubuntu-latest
    if: github.event_name == 'release'
    environment: production

    steps:
    - uses: actions/checkout@v4

    - name: Configure kubectl
      uses: azure/k8s-set-context@v3
      with:
        method: kubeconfig
        kubeconfig: ${{ secrets.KUBE_CONFIG_PROD }}

    - name: Deploy to production
      run: |
        helm upgrade --install celery-app ./helm/celery-app \
          --namespace celery-prod \
          --create-namespace \
          --set image.tag=${{ github.event.release.tag_name }} \
          --set environment=production \
          --set replicas=5 \
          --values ./helm/values-production.yaml

    - name: Blue-green switch
      run: |
        ./scripts/blue_green_switch.sh

    - name: Verify deployment
      run: |
        python scripts/verify_deployment.py --environment=production
```

**GitLab CI Alternative**:
```yaml
# .gitlab-ci.yml
stages:
  - test
  - build
  - deploy-staging
  - deploy-production

variables:
  DOCKER_DRIVER: overlay2
  DOCKER_TLS_CERTDIR: "/certs"

test:
  stage: test
  image: python:3.11
  services:
    - redis:7
    - rabbitmq:3-management
  variables:
    CELERY_BROKER_URL: redis://redis:6379/0
    CELERY_RESULT_BACKEND: redis://redis:6379/0
  before_script:
    - pip install -r requirements.txt
    - pip install -r requirements-dev.txt
  script:
    - flake8 app/
    - mypy app/
    - pytest tests/ -v --cov=app
  coverage: '/TOTAL.*\s+(\d+%)$/'
  artifacts:
    reports:
      coverage_report:
        coverage_format: cobertura
        path: coverage.xml

build:
  stage: build
  image: docker:24.0.5
  services:
    - docker:24.0.5-dind
  script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
    - docker build -t $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA .
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
    - docker tag $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA $CI_REGISTRY_IMAGE:latest
    - docker push $CI_REGISTRY_IMAGE:latest

deploy-staging:
  stage: deploy-staging
  image: bitnami/kubectl:latest
  script:
    - kubectl config use-context $KUBE_CONTEXT_STAGING
    - helm upgrade --install celery-app ./helm/celery-app
      --namespace celery-staging
      --set image.tag=$CI_COMMIT_SHA
      --set environment=staging
  only:
    - develop

deploy-production:
  stage: deploy-production
  image: bitnami/kubectl:latest
  script:
    - kubectl config use-context $KUBE_CONTEXT_PROD
    - helm upgrade --install celery-app ./helm/celery-app
      --namespace celery-prod
      --set image.tag=$CI_COMMIT_TAG
      --set environment=production
  when: manual
  only:
    - tags
```

---

## Capstone Projects

### Capstone 1: Microservices Event Platform

**Objective**: Build an event-driven microservices platform using Celery

**Duration**: 2 weeks

**Requirements**:
1. Multiple microservices communicating via events
2. Event sourcing for audit trail
3. CQRS pattern implementation
4. Distributed transactions with Saga
5. Real-time event streaming
6. Monitoring and tracing

**Architecture**:
```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   Service   │    │   Service   │    │   Service   │
│     A       │    │     B       │    │     C       │
└─────┬───────┘    └─────┬───────┘    └─────┬───────┘
      │ Event            │ Event            │ Event
      ▼                  ▼                  ▼
┌─────────────────────────────────────────────────────┐
│              Event Bus (Celery + Redis)              │
└─────────────────────────────────────────────────────┘
      │                  │                  │
      ▼                  ▼                  ▼
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   Service   │    │   Service   │    │   Service   │
│     D       │    │     E       │    │     F       │
└─────────────┘    └─────────────┘    └─────────────┘
```

**Implementation Guide**:
```python
# event_platform.py
from celery import Celery
from dataclasses import dataclass
from typing import Dict, Any
import json
import time

# Event store configuration
app = Celery('event_platform')
app.conf.update(
    broker_url='redis://localhost:6379/0',
    result_backend='redis://localhost:6379/0',
    task_serializer='json',
    event_serializer='json',
    accept_content=['json']
)

@dataclass
class DomainEvent:
    """Base class for domain events"""
    event_id: str
    aggregate_id: str
    event_type: str
    data: Dict[str, Any]
    timestamp: float
    version: int = 1

class EventBus:
    """Event bus for publishing and subscribing to events"""

    @staticmethod
    def publish(event: DomainEvent):
        """Publish event to event bus"""
        app.send_task(
            'event_handler.process_event',
            args=[event.__dict__],
            routing_key=f'event.{event.event_type}'
        )

    @staticmethod
    def subscribe(event_type: str, handler):
        """Subscribe to specific event type"""
        app.task(
            name=f'handler.{event_type}',
            bind=True,
            max_retries=3
        )(handler)

# Example microservice
class OrderService:
    @staticmethod
    def create_order(order_data):
        """Create order and publish events"""
        order_id = generate_order_id()

        # Create event
        event = DomainEvent(
            event_id=generate_uuid(),
            aggregate_id=order_id,
            event_type='OrderCreated',
            data={
                'order_id': order_id,
                'customer_id': order_data['customer_id'],
                'items': order_data['items'],
                'total': calculate_total(order_data['items'])
            },
            timestamp=time.time()
        )

        # Store event
        store_event(event)

        # Publish event
        EventBus.publish(event)

        return order_id

# Event handlers
@app.task(bind=True)
def handle_order_created(self, event_data):
    """Handle OrderCreated event"""
    print(f"Processing order: {event_data['aggregate_id']}")

    # Update read model
    update_order_view_model(event_data)

    # Trigger other services
    app.send_task(
        'inventory.reserve_items',
        args=[event_data['aggregate_id'], event_data['data']['items']]
    )

    app.send_task(
        'payment.process_payment',
        args=[event_data['aggregate_id'], event_data['data']['total']]
    )

# Saga orchestrator
class OrderSaga:
    @staticmethod
    def execute(order_data):
        """Execute order processing saga"""
        saga = chain(
            create_order.s(order_data),
            reserve_inventory.s(),
            process_payment.s(),
            confirm_order.s()
        )

        # Configure compensating transactions
        saga.link_error(on_order_saga_failure.s())

        return saga.apply_async()

@app.task
def on_order_saga_failure(request, exc, traceback):
    """Handle saga failure and run compensations"""
    order_id = request.args[0] if request.args else None

    # Run compensating transactions in reverse
    chain(
        release_inventory.s(order_id),
        refund_payment.s(order_id),
        cancel_order.s(order_id)
    ).apply_async()
```

---

## Interactive Web Components

### Web-Based Task Visualizer

```html
<!-- task_visualizer.html -->
<!DOCTYPE html>
<html>
<head>
    <title>Celery Task Visualizer</title>
    <script src="https://d3js.org/d3.v7.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/cytoscape@3.21.0/dist/cytoscape.min.js"></script>
    <style>
        #cy { width: 100%; height: 600px; border: 1px solid #ccc; }
        .controls { padding: 20px; }
    </style>
</head>
<body>
    <div class="controls">
        <h2>Task Workflow Designer</h2>
        <button onclick="addTask('process')">Add Process Task</button>
        <button onclick="addTask('validate')">Add Validate Task</button>
        <button onclick="addChain()">Create Chain</button>
        <button onclick="addGroup()">Create Group</button>
        <button onclick="executeWorkflow()">Execute Workflow</button>
    </div>

    <div id="cy"></div>

    <script>
        const cy = cytoscape({
            container: document.getElementById('cy'),
            elements: [
                { data: { id: 'start', label: 'Start' } }
            ],
            style: [
                {
                    selector: 'node',
                    style: {
                        'background-color': '#666',
                        'label': 'data(label)',
                        'text-valign': 'center',
                        'color': 'white',
                        'width': 80,
                        'height': 80
                    }
                },
                {
                    selector: 'edge',
                    style: {
                        'width': 3,
                        'line-color': '#ccc',
                        'target-arrow-color': '#ccc',
                        'target-arrow-shape': 'triangle'
                    }
                }
            ],
            layout: { name: 'breadthfirst' }
        });

        let taskCounter = 0;

        function addTask(type) {
            taskCounter++;
            const taskId = `task_${taskCounter}`;

            cy.add({
                data: {
                    id: taskId,
                    label: `${type}_${taskCounter}`,
                    type: type
                }
            });

            // Connect to selected node or last node
            const selected = cy.$('node:selected');
            const target = selected.length > 0 ? selected[0] : cy.$('node').last();

            cy.add({
                data: {
                    source: target.id(),
                    target: taskId
                }
            });
        }

        function executeWorkflow() {
            const workflow = exportWorkflow();

            fetch('/execute_workflow', {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify(workflow)
            })
            .then(response => response.json())
            .then(data => {
                console.log('Workflow started:', data);
                monitorWorkflow(data.task_id);
            });
        }

        function exportWorkflow() {
            const nodes = cy.nodes().map(node => ({
                id: node.data('id'),
                type: node.data('type'),
                label: node.data('label')
            }));

            const edges = cy.edges().map(edge => ({
                source: edge.data('source'),
                target: edge.data('target')
            }));

            return { nodes, edges };
        }

        function monitorWorkflow(taskId) {
            const ws = new WebSocket(`ws://localhost:5000/monitor/${taskId}`);

            ws.onmessage = function(event) {
                const data = JSON.parse(event.data);

                // Update node colors based on status
                if (data.node_id && data.status) {
                    const node = cy.getElementById(data.node_id);
                    if (node) {
                        const color = data.status === 'SUCCESS' ? 'green' :
                                     data.status === 'FAILURE' ? 'red' : 'yellow';
                        node.style('background-color', color);
                    }
                }
            };
        }
    </script>
</body>
</html>
```

### Flask Backend for Visualizer

```python
# visualizer_backend.py
from flask import Flask, render_template, request, jsonify
from flask_socketio import SocketIO, emit
import json
import uuid
from celery import Celery
import time

app = Flask(__name__)
app.config['SECRET_KEY'] = 'visualizer-secret'
socketio = SocketIO(app, cors_allowed_origins="*")

# Celery app for executing workflows
celery_app = Celery('workflow_executor')
celery_app.conf.update(
    broker_url='redis://localhost:6379/0',
    result_backend='redis://localhost:6379/0'
)

# Mock tasks for demonstration
@celery_app.task
def process_task(data):
    time.sleep(2)  # Simulate work
    return f"Processed: {data}"

@celery_app.task
def validate_task(data):
    time.sleep(1)
    return True

@celery_app.task
def aggregate_task(results):
    return f"Aggregated {len(results)} results"

@app.route('/')
def index():
    return render_template('task_visualizer.html')

@app.route('/execute_workflow', methods=['POST'])
def execute_workflow():
    workflow = request.json

    # Build Celery workflow from visual representation
    task_map = {
        'process': process_task,
        'validate': validate_task
    }

    # Simple chain execution based on edges
    tasks = []
    for edge in workflow['edges']:
        source = next(n for n in workflow['nodes'] if n['id'] == edge['source'])
        task_class = task_map.get(source['type'], process_task)
        tasks.append(task_class.s(source))

    # Execute workflow
    if len(tasks) == 1:
        result = tasks[0].apply_async()
    else:
        from celery import chain
        result = chain(*tasks).apply_async()

    return jsonify({
        'task_id': result.id,
        'status': 'STARTED'
    })

@socketio.on('connect')
def handle_connect():
    emit('connected', {'data': 'Connected to workflow monitor'})

def monitor_workflow(task_id):
    """Monitor workflow execution and emit updates"""
    while True:
        result = celery_app.AsyncResult(task_id)

        if result.ready():
            emit('workflow_complete', {
                'task_id': task_id,
                'result': result.get() if result.successful() else str(result.info),
                'status': 'SUCCESS' if result.successful() else 'FAILURE'
            })
            break

        # Emit progress updates
        emit('workflow_progress', {
            'task_id': task_id,
            'state': result.state,
            'info': result.info
        })

        time.sleep(1)

if __name__ == '__main__':
    socketio.run(app, debug=True, port=5000)
```

---

## Lab Completion Checklist

### For Each Lab:

**Preparation**:
- [ ] Environment set up
- [ ] Dependencies installed
- [ ] Services running (Redis, RabbitMQ, etc.)
- [ ] Code cloned and reviewed

**Execution**:
- [ ] All exercises completed
- [ ] Code runs without errors
- [ ] Expected outputs verified
- [ ] Performance metrics collected

**Analysis**:
- [ ] Results documented
- [ ] Performance analyzed
- [ ] Bottlenecks identified
- [ ] Optimizations applied

**Clean-up**:
- [ ] Services stopped
- [ ] Resources released
- [ ] Logs saved
- [ ] Code committed

### Progress Tracking

Create a lab progress tracker:

```python
# progress_tracker.py
import json
from datetime import datetime

class LabProgressTracker:
    def __init__(self):
        self.progress = {
            'learning_path_1': {
                'lab_1.1': {'completed': False, 'score': None, 'notes': ''},
                'lab_1.2': {'completed': False, 'score': None, 'notes': ''},
                'lab_1.3': {'completed': False, 'score': None, 'notes': ''}
            },
            'learning_path_2': {
                'lab_2.1': {'completed': False, 'score': None, 'notes': ''},
                'lab_2.2': {'completed': False, 'score': None, 'notes': ''}
            },
            # ... other labs
        }

    def mark_completed(self, lab_path, score=None, notes=''):
        """Mark lab as completed"""
        parts = lab_path.split('.')
        path = parts[0]
        lab = parts[1]

        if path in self.progress and lab in self.progress[path]:
            self.progress[path][lab].update({
                'completed': True,
                'score': score,
                'notes': notes,
                'completed_at': datetime.now().isoformat()
            })

            self.save_progress()

    def get_completion_percentage(self):
        """Calculate overall completion percentage"""
        total = sum(len(path) for path in self.progress.values())
        completed = sum(
            sum(1 for lab in path.values() if lab['completed'])
            for path in self.progress.values()
        )

        return (completed / total) * 100 if total > 0 else 0

    def save_progress(self):
        """Save progress to file"""
        with open('lab_progress.json', 'w') as f:
            json.dump(self.progress, f, indent=2)

    def load_progress(self):
        """Load progress from file"""
        try:
            with open('lab_progress.json', 'r') as f:
                self.progress = json.load(f)
        except FileNotFoundError:
            pass

# Usage
tracker = LabProgressTracker()
tracker.load_progress()

# Mark lab as completed
tracker.mark_completed('learning_path_1.lab_1.1', score=95, notes='Excellent work!')

# Check progress
print(f"Completion: {tracker.get_completion_percentage():.1f}%")
```

---

This comprehensive collection of interactive components, hands-on labs, and practical exercises provides students with immersive learning experiences that build real-world skills in using Celery for distributed task processing.