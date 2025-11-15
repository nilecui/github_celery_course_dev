# Celery Course Exercises and Assessments
**Last Updated:** 2025-11-15
**Version:** 1.0

---

## Learning Path 1: Celery Foundations - Exercises and Quizzes

### Module 1: Introduction to Distributed Task Processing

#### Quiz Questions

**Multiple Choice**
1. Which of the following is the primary benefit of using distributed task processing?
   a) Reduced code complexity
   b) Improved user experience through non-blocking operations
   c) Easier debugging
   d) Reduced database usage

2. A task that takes 30 seconds to complete in a web request will:
   a) Improve user engagement
   b) Cause request timeout and poor UX
   c) Reduce server load
   d) Automatically be processed asynchronously

3. Celery is primarily used for:
   a) Database migrations
   b) Asynchronous task queue management
   c) Web framework routing
   d) Frontend rendering

**True/False**
1. Distributed task processing eliminates the need for message brokers. (False)
2. Email sending is a good candidate for asynchronous processing. (True)
3. Celery can only work with Python applications. (True)

**Fill in the Blanks**
1. The three main components of distributed task processing are ______, ______, and ______.
   Answer: producers, message broker, consumers/workers

2. When a task takes longer than ____ seconds, it should be considered for asynchronous processing.
   Answer: 5-10

#### Exercise: Scenario Analysis

**Task**: Analyze the following scenarios and determine which ones require asynchronous processing:

1. User registration email sending
2. Database query for user profile
3. Generating monthly financial reports
4. Validating user input
5. Processing uploaded images
6. Logging user activities
7. Real-time chat messaging
8. Data backup operations

**Answer Key**:
- Async: 1, 3, 5, 8
- Sync: 2, 4, 6, 7 (with some caveats)

**Discussion Points**:
- Why is image processing async? (Time-consuming, resource-intensive)
- Why is profile query sync? (Fast, needed for immediate response)
- Edge cases for chat messaging (Can be both sync and async depending on requirements)

---

### Module 2: Setting Up Your First Celery Application

#### Quiz Questions

**Multiple Choice**
1. Which message brokers are commonly used with Celery?
   a) MySQL and PostgreSQL
   b) RabbitMQ and Redis
   c) MongoDB and Cassandra
   d) Elasticsearch and Solr

2. The result backend in Celery is used to:
   a) Store task definitions
   b) Cache task results
   c) Manage worker processes
   d) Schedule periodic tasks

3. To start a Celery worker, you use the command:
   a) celery worker
   b) celery run
   c) celery start
   d) celery -A module worker

**True/False**
1. Celery requires both a broker and a result backend to function. (False - broker is required, backend is optional)
2. The CELERY_BROKER_URL configuration can be set in a separate config module. (True)
3. Workers automatically reconnect to the broker if the connection is lost. (True)

#### Practical Exercise: Basic Setup

**Task**: Create a complete Celery application with the following requirements:

```python
# File: celery_app.py
"""
Your task: Complete this Celery application
"""

# TODO: Import necessary Celery components
# TODO: Create Celery app instance with proper configuration
# TODO: Configure broker and result backend
# TODO: Set timezone

# TODO: Define a simple task that adds two numbers
def add_numbers(x, y):
    """Task to add two numbers"""
    pass  # TODO: Add task decorator

# TODO: Define a task that concatenates strings
def concatenate_strings(str1, str2):
    """Task to concatenate two strings"""
    pass  # TODO: Add task decorator

# TODO: Define a task that calculates factorial
def calculate_factorial(n):
    """Task to calculate factorial of n"""
    pass  # TODO: Add task decorator and implementation
```

**Solution**:

```python
from celery import Celery

# Create Celery app
app = Celery('myapp')

# Configure broker and backend
app.conf.update(
    broker_url='redis://localhost:6379/0',
    result_backend='redis://localhost:6379/0',
    timezone='UTC',
    enable_utc=True,
)

@app.task
def add_numbers(x, y):
    """Task to add two numbers"""
    return x + y

@app.task
def concatenate_strings(str1, str2):
    """Task to concatenate two strings"""
    return f"{str1} {str2}"

@app.task
def calculate_factorial(n):
    """Task to calculate factorial of n"""
    if n < 0:
        raise ValueError("Factorial not defined for negative numbers")
    if n == 0 or n == 1:
        return 1
    result = 1
    for i in range(2, n + 1):
        result *= i
    return result
```

#### Lab Exercise: Running and Testing

**Instructions**:
1. Start Redis: `redis-server`
2. Start worker: `celery -A celery_app worker --loglevel=info`
3. Run tasks in Python shell:
   ```python
   from celery_app import add_numbers, concatenate_strings, calculate_factorial

   # Send tasks
   result1 = add_numbers.delay(5, 3)
   result2 = concatenate_strings.delay('Hello', 'World')
   result3 = calculate_factorial.delay(5)

   # Get results
   print(result1.get())  # Should print 8
   print(result2.get())  # Should print "Hello World"
   print(result3.get())  # Should print 120
   ```

---

### Module 3: Task Definition and Execution

#### Quiz Questions

**Multiple Choice**
1. What is the difference between .delay() and .apply_async()?
   a) .delay() is synchronous, .apply_async() is asynchronous
   b) .delay() doesn't accept extra options, .apply_async() does
   c) .delay() is for simple tasks, .apply_async() for complex tasks
   d) There is no difference

2. Task state 'PENDING' means:
   a) The task is waiting to be processed
   b) The task failed
   c) The task is currently running
   d) The task completed successfully

3. To execute a task synchronously for testing, you should:
   a) Use .delay() with wait=True
   b) Use .apply() method
   c) Set CELERY_ALWAYS_EAGER=True
   d) Both b and c

**True/False**
1. Tasks can return any serializable Python object. (True)
2. The task decorator automatically registers the task. (True)
3. Task results are available indefinitely by default. (False - they expire)

#### Exercise: Task Patterns

**Task**: Implement the following task patterns:

```python
"""
Complete the implementations of these task patterns
"""

@app.task(bind=True)
def process_upload(self, file_path):
    """
    Process uploaded file with progress tracking
    - Update progress every 10%
    - Handle file not found errors
    - Return processing summary
    """
    # TODO: Implement with progress updates
    pass

@app.task(max_retries=3, default_retry_delay=60)
def fetch_api_data(self, url):
    """
    Fetch data from API with retry logic
    - Retry on network errors
    - Use exponential backoff
    - Return parsed JSON
    """
    # TODO: Implement with retry logic
    pass

@app.task
def batch_process_items(items, batch_size=10):
    """
    Process items in batches
    - Split items into batches
    - Process each batch in parallel
    - Return all results
    """
    # TODO: Implement batch processing
    pass
```

**Solution**:

```python
import time
import requests
from celery import group

@app.task(bind=True)
def process_upload(self, file_path):
    """Process uploaded file with progress tracking"""
    try:
        # Simulate file processing
        total_steps = 10
        for i in range(total_steps):
            time.sleep(1)  # Simulate work
            progress = int(((i + 1) / total_steps) * 100)
            self.update_state(state='PROGRESS', meta={'progress': progress})

        return {
            'status': 'completed',
            'file': file_path,
            'processed_at': time.time()
        }
    except FileNotFoundError:
        raise self.retry(exc=FileNotFoundError(f"File not found: {file_path}"))
    except Exception as exc:
        raise self.retry(exc=exc)

@app.task(max_retries=3, default_retry_delay=60)
def fetch_api_data(self, url):
    """Fetch data from API with retry logic"""
    try:
        response = requests.get(url, timeout=10)
        response.raise_for_status()
        return response.json()
    except requests.exceptions.RequestException as exc:
        # Exponential backoff
        countdown = self.default_retry_delay * (2 ** self.request.retries)
        raise self.retry(exc=exc, countdown=countdown)

@app.task
def batch_process_items(items, batch_size=10):
    """Process items in batches"""
    # Create batches
    batches = [items[i:i + batch_size] for i in range(0, len(items), batch_size)]

    # Process each batch
    job = group(process_batch.s(batch) for batch in batches)
    result = job.apply_async()

    return result.get()

@app.task
def process_batch(batch):
    """Process a single batch of items"""
    # Simulate processing
    results = []
    for item in batch:
        # Process item
        processed = item * 2  # Example processing
        results.append(processed)
    return results
```

---

### Module 4: Understanding Brokers and Backends

#### Quiz Questions

**Multiple Choice**
1. When should you choose RabbitMQ over Redis?
   a) When you need guaranteed message delivery
   b) When you have a simple setup
   c) When you need fast caching
   d) When budget is limited

2. What happens if you don't configure a result backend?
   a) Tasks won't execute
   b) You can't retrieve task results
   c) Workers won't start
   d) Configuration will fail

3. Which of these is NOT a valid result backend option?
   a) redis://
   b) rpc://
   c) mongodb://
   d) mysql:// (without extension)

**True/False**
1. Redis can be used as both broker and backend simultaneously. (True)
2. Message brokers ensure task execution even if workers restart. (True for RabbitMQ with persistence)
3. RPC backend is suitable for production with many workers. (False - better for development)

#### Exercise: Broker Comparison

**Task**: Create a comparison table and implement configuration for different brokers:

```python
"""
Complete the broker configurations and create comparison
"""

# TODO: Configure for different brokers
BROKER_CONFIGS = {
    'redis': {
        'url': '',  # TODO: Redis URL
        'pros': [],  # TODO: List pros
        'cons': []   # TODO: List cons
    },
    'rabbitmq': {
        'url': '',  # TODO: RabbitMQ URL
        'pros': [],  # TODO: List pros
        'cons': []   # TODO: List cons
    }
}

def create_app_with_broker(broker_type):
    """Create Celery app with specified broker"""
    # TODO: Implement app creation
    pass

def test_broker_performance(app, num_tasks=100):
    """Test broker performance"""
    # TODO: Implement performance test
    pass
```

**Solution**:

```python
BROKER_CONFIGS = {
    'redis': {
        'url': 'redis://localhost:6379/0',
        'pros': [
            'Simple setup',
            'Fast performance',
            'Built-in data structures',
            'Can serve as both broker and backend'
        ],
        'cons': [
            'Potential data loss on crash',
            'Less reliable than RabbitMQ',
            'Limited message acknowledgment'
        ]
    },
    'rabbitmq': {
        'url': 'pyamqp://guest@localhost//',
        'pros': [
            'Reliable message delivery',
            'Advanced routing',
            'Message persistence',
            'Clustering support'
        ],
        'cons': [
            'Complex setup',
            'More resource intensive',
            'Separate backend needed'
        ]
    }
}

def create_app_with_broker(broker_type):
    """Create Celery app with specified broker"""
    config = BROKER_CONFIGS[broker_type]
    app = Celery(f'{broker_type}_app')
    app.conf.update(
        broker_url=config['url'],
        result_backend=f'{broker_type}://localhost' if broker_type == 'redis' else 'redis://localhost'
    )
    return app

@app.task
def dummy_task(x):
    return x * 2

def test_broker_performance(app, num_tasks=100):
    """Test broker performance"""
    import time
    start_time = time.time()

    # Send tasks
    results = [dummy_task.delay(i) for i in range(num_tasks)]

    # Wait for all to complete
    for result in results:
        result.get(timeout=10)

    end_time = time.time()
    return {
        'tasks': num_tasks,
        'time': end_time - start_time,
        'tasks_per_second': num_tasks / (end_time - start_time)
    }
```

---

### Module 5: Basic Task Configuration

#### Quiz Questions

**Multiple Choice**
1. What does the prefetch_multiplier control?
   a) Number of tasks prefetched per worker
   b) Number of worker processes
   c) Task priority levels
   d) Retry attempts

2. To route tasks to specific queues, you need to:
   a) Set task_routes in configuration
   b) Use the @route decorator
   c) Specify queue in .delay()
   d) All of the above

3. The worker_concurrency setting determines:
   a) Number of concurrent tasks per worker
   b) Number of worker instances
   c) Maximum task runtime
   d) Queue processing order

**True/False**
1. Tasks in different queues can be processed by the same worker. (True)
2. Higher priority tasks are always processed first regardless of queue. (False - depends on worker configuration)
3. The timezone setting affects only periodic tasks. (False - affects all task timing)

#### Exercise: Configuration Optimization

**Task**: Optimize Celery configuration for different scenarios:

```python
"""
Complete the configurations for different use cases
"""

def optimize_for_cpu_intensive():
    """Configure for CPU-bound tasks"""
    config = {
        # TODO: Add optimal settings
        'worker_concurrency': None,  # What should this be?
        'worker_prefetch_multiplier': None,
        'task_acks_late': None,
        'worker_disable_rate_limits': None,
    }
    return config

def optimize_for_io_intensive():
    """Configure for I/O-bound tasks"""
    config = {
        # TODO: Add optimal settings
        'worker_concurrency': None,
        'worker_prefetch_multiplier': None,
        'task_acks_late': None,
        'worker_pool': None,  # What pool type?
    }
    return config

def configure_queues():
    """Configure task routing"""
    task_routes = {
        # TODO: Define routes for different task types
        'app.tasks.cpu_intensive': {'queue': None},
        'app.tasks.io_intensive': {'queue': None},
        'app.tasks.urgent': {'queue': None},
    }
    return task_routes
```

**Solution**:

```python
def optimize_for_cpu_intensive():
    """Configure for CPU-bound tasks"""
    config = {
        'worker_concurrency': 4,  # Usually equal to CPU cores
        'worker_prefetch_multiplier': 1,  # Process one at a time
        'task_acks_late': True,  # Ack only after completion
        'worker_disable_rate_limits': True,  # No rate limits
        'worker_pool': 'solo',  # Single thread per process
    }
    return config

def optimize_for_io_intensive():
    """Configure for I/O-bound tasks"""
    config = {
        'worker_concurrency': 100,  # High concurrency
        'worker_prefetch_multiplier': 10,  # Prefetch many tasks
        'task_acks_late': False,  # Ack immediately
        'worker_pool': 'gevent',  # Async I/O pool
    }
    return config

def configure_queues():
    """Configure task routing"""
    from kombu import Queue

    # Define queues
    queues = (
        Queue('cpu', routing_key='cpu'),
        Queue('io', routing_key='io'),
        Queue('urgent', routing_key='urgent'),
    )

    # Define routes
    task_routes = {
        'app.tasks.cpu_intensive': {'queue': 'cpu', 'routing_key': 'cpu'},
        'app.tasks.io_intensive': {'queue': 'io', 'routing_key': 'io'},
        'app.tasks.urgent': {'queue': 'urgent', 'routing_key': 'urgent'},
    }

    return queues, task_routes
```

---

### Module 6: Monitoring and Logging

#### Quiz Questions

**Multiple Choice**
1. Flower provides:
   a) Real-time monitoring dashboard
   b) Task scheduling
   c) Code debugging
   d) Database migration

2. To enable structured JSON logging in Celery, you should:
   a) Set CELERY_TASK_SERIALIZER = 'json'
   b) Configure Python logging with JSONFormatter
   c) Use the --json flag
   d) Install celery-json-logger

3. Worker events are sent to:
   a) The broker
   b) The result backend
   c) A separate event channel
   d) Log files

**True/False**
1. Flower can modify task execution state. (True - can revoke/retry tasks)
2. Celery automatically logs all task failures. (True)
3. Monitoring is optional in production. (False - essential for reliability)

#### Exercise: Monitoring Setup

**Task**: Set up comprehensive monitoring:

```python
"""
Complete the monitoring setup
"""

import logging
from celery.signals import task_prerun, task_postrun, task_failure

# TODO: Configure structured logging
def setup_logging():
    """Setup JSON logging for Celery"""
    pass

# TODO: Set up monitoring callbacks
def monitor_task_prerun(sender=None, task_id=None, task=None, args=None, kwargs=None, **kwds):
    """Called before task execution"""
    pass

def monitor_task_postrun(sender=None, task_id=None, task=None, args=None, kwargs=None, retval=None, **kwds):
    """Called after task execution"""
    pass

def monitor_task_failure(sender=None, task_id=None, exception=None, traceback=None, einfo=None, **kwds):
    """Called on task failure"""
    pass

# TODO: Create custom metrics
class TaskMetrics:
    def __init__(self):
        self.metrics = {
            'tasks_completed': 0,
            'tasks_failed': 0,
            'total_runtime': 0,
        }

    def record_completion(self, runtime):
        """Record task completion"""
        pass

    def record_failure(self):
        """Record task failure"""
        pass

    def get_stats(self):
        """Get current stats"""
        pass
```

**Solution**:

```python
import json
import time
from datetime import datetime

def setup_logging():
    """Setup JSON logging for Celery"""
    from pythonjsonlogger import jsonlogger

    # Create formatter
    formatter = jsonlogger.JsonFormatter(
        '%(asctime)s %(name)s %(levelname)s %(message)s'
    )

    # Configure handlers
    handler = logging.StreamHandler()
    handler.setFormatter(formatter)

    # Configure logger
    logger = logging.getLogger('celery')
    logger.addHandler(handler)
    logger.setLevel(logging.INFO)

    return logger

# Connect signals
task_prerun.connect(monitor_task_prerun)
task_postrun.connect(monitor_task_postrun)
task_failure.connect(monitor_task_failure)

# Global metrics
metrics = TaskMetrics()

def monitor_task_prerun(sender=None, task_id=None, task=None, args=None, kwargs=None, **kwds):
    """Called before task execution"""
    logger = logging.getLogger('celery.monitor')
    logger.info(
        "Task started",
        extra={
            'event': 'task_started',
            'task_id': task_id,
            'task_name': task.name,
            'args': args,
            'kwargs': kwargs,
            'timestamp': datetime.utcnow().isoformat()
        }
    )

def monitor_task_postrun(sender=None, task_id=None, task=None, args=None, kwargs=None, retval=None, **kwds):
    """Called after task execution"""
    runtime = kwds.get('runtime', 0)
    metrics.record_completion(runtime)

    logger = logging.getLogger('celery.monitor')
    logger.info(
        "Task completed",
        extra={
            'event': 'task_completed',
            'task_id': task_id,
            'task_name': task.name,
            'runtime': runtime,
            'result': retval,
            'timestamp': datetime.utcnow().isoformat()
        }
    )

def monitor_task_failure(sender=None, task_id=None, exception=None, traceback=None, einfo=None, **kwds):
    """Called on task failure"""
    metrics.record_failure()

    logger = logging.getLogger('celery.monitor')
    logger.error(
        "Task failed",
        extra={
            'event': 'task_failed',
            'task_id': task_id,
            'exception': str(exception),
            'traceback': traceback,
            'timestamp': datetime.utcnow().isoformat()
        }
    )

class TaskMetrics:
    def __init__(self):
        self.metrics = {
            'tasks_completed': 0,
            'tasks_failed': 0,
            'total_runtime': 0,
            'avg_runtime': 0,
        }

    def record_completion(self, runtime):
        """Record task completion"""
        self.metrics['tasks_completed'] += 1
        self.metrics['total_runtime'] += runtime
        self.metrics['avg_runtime'] = (
            self.metrics['total_runtime'] / self.metrics['tasks_completed']
        )

    def record_failure(self):
        """Record task failure"""
        self.metrics['tasks_failed'] += 1

    def get_stats(self):
        """Get current stats"""
        return {
            **self.metrics,
            'success_rate': (
                self.metrics['tasks_completed'] /
                (self.metrics['tasks_completed'] + self.metrics['tasks_failed'])
                if (self.metrics['tasks_completed'] + self.metrics['tasks_failed']) > 0
                else 0
            )
        }
```

---

## Learning Path 2: Intermediate Celery - Exercises

### Module 12: Advanced Task Patterns

#### Quiz Questions

**Multiple Choice**
1. Task inheritance is useful for:
   a) Sharing common functionality
   b) Improving performance
   c) Reducing memory usage
   d) All of the above

2. The Saga pattern helps with:
   a) Task routing
   b) Distributed transactions
   c) Load balancing
   d) Task scheduling

3. Dynamic task creation can be done using:
   a) @app.task decorator only
   b) app.task() function
   c) Task subclassing
   d) Both b and c

**True/False**
1. Tasks can inherit from other tasks. (True)
2. The producer-consumer pattern requires multiple queues. (True)
3. Task versioning is automatic in Celery. (False - needs custom implementation)

#### Exercise: Advanced Patterns Implementation

**Task**: Implement producer-consumer and pipeline patterns:

```python
"""
Implement advanced task patterns
"""

# TODO: Create abstract base task
class BaseTask(app.Task):
    """Base task with common functionality"""
    # TODO: Add common initialization
    # TODO: Add common error handling
    # TODO: Add logging
    pass

# TODO: Implement producer-consumer pattern
@app.task(base=BaseTask)
def produce_data(queue_name, data):
    """Producer task that adds data to queue"""
    pass

@app.task(base=BaseTask)
def consume_data(queue_name):
    """Consumer task that processes data from queue"""
    pass

# TODO: Implement pipeline pattern
@app.task(base=BaseTask)
def extract_data(source):
    """Extract data from source"""
    pass

@app.task(base=BaseTask)
def transform_data(raw_data):
    """Transform extracted data"""
    pass

@app.task(base=BaseTask)
def load_data(transformed_data, destination):
    """Load transformed data to destination"""
    pass

def create_pipeline(source, destination):
    """Create ETL pipeline"""
    # TODO: Chain extract, transform, load
    pass
```

**Solution**:

```python
import logging
from celery import chain, group
from celery.exceptions import Retry

class BaseTask(app.Task):
    """Base task with common functionality"""
    abstract = True

    def __init__(self):
        self.logger = logging.getLogger(f'celery.task.{self.name}')

    def on_success(self, retval, task_id, args, kwargs):
        """Success handler"""
        self.logger.info(
            f"Task {self.name} completed successfully",
            extra={
                'task_id': task_id,
                'result': retval
            }
        )

    def on_failure(self, exc, task_id, args, kwargs, einfo):
        """Failure handler"""
        self.logger.error(
            f"Task {self.name} failed",
            extra={
                'task_id': task_id,
                'error': str(exc),
                'traceback': str(einfo)
            }
        )

    def on_retry(self, exc, task_id, args, kwargs, einfo):
        """Retry handler"""
        self.logger.warning(
            f"Task {self.name} retrying",
            extra={
                'task_id': task_id,
                'retry_number': self.request.retries,
                'error': str(exc)
            }
        )

# Producer-consumer pattern
@app.task(bind=True, base=BaseTask)
def produce_data(self, queue_name, data):
    """Producer task that adds data to queue"""
    try:
        # Simulate data production
        for item in data:
            # Send to consumer queue
            consume_data.delay(queue_name, item)

        return {
            'status': 'produced',
            'queue': queue_name,
            'items_produced': len(data)
        }
    except Exception as exc:
        raise self.retry(exc=exc)

@app.task(bind=True, base=BaseTask)
def consume_data(self, queue_name, item):
    """Consumer task that processes data from queue"""
    try:
        # Process the item
        processed = {
            'original': item,
            'processed': item * 2,  # Example processing
            'timestamp': time.time()
        }

        # Store result (in real app, would save to DB/backend)
        return processed
    except Exception as exc:
        raise self.retry(exc=exc)

# ETL Pipeline pattern
@app.task(bind=True, base=BaseTask, max_retries=3)
def extract_data(self, source):
    """Extract data from source"""
    try:
        # Simulate data extraction
        if source == 'api':
            raw_data = fetch_from_api()
        elif source == 'file':
            raw_data = read_from_file()
        else:
            raise ValueError(f"Unknown source: {source}")

        return {
            'source': source,
            'data': raw_data,
            'count': len(raw_data)
        }
    except Exception as exc:
        raise self.retry(exc=exc)

@app.task(bind=True, base=BaseTask)
def transform_data(self, raw_data):
    """Transform extracted data"""
    # Extract data from previous step
    data = raw_data['data']

    # Transform data
    transformed = []
    for item in data:
        transformed_item = {
            'id': item.get('id'),
            'value': item.get('value') * 10,
            'processed_at': time.time()
        }
        transformed.append(transformed_item)

    return {
        'original_count': len(data),
        'transformed_count': len(transformed),
        'data': transformed
    }

@app.task(bind=True, base=BaseTask)
def load_data(self, transformed_data, destination):
    """Load transformed data to destination"""
    data = transformed_data['data']

    # Simulate loading
    if destination == 'database':
        save_to_database(data)
    elif destination == 'file':
        save_to_file(data)
    else:
        raise ValueError(f"Unknown destination: {destination}")

    return {
        'destination': destination,
        'loaded_count': len(data),
        'status': 'completed'
    }

def create_pipeline(source, destination):
    """Create ETL pipeline"""
    pipeline = chain(
        extract_data.s(source),
        transform_data.s(),
        load_data.s(destination)
    )
    return pipeline()

# Helper functions (would be implemented in real app)
def fetch_from_api():
    return [{'id': i, 'value': i} for i in range(10)]

def read_from_file():
    return [{'id': i, 'value': i * 2} for i in range(10)]

def save_to_database(data):
    # Simulate DB save
    pass

def save_to_file(data):
    # Simulate file save
    pass
```

---

### Module 14: Canvas: Chains, Groups, and Chords

#### Quiz Questions

**Multiple Choice**
1. A chain executes tasks:
   a) In parallel
   b) Sequentially, passing results
   c) Randomly
   d) Based on priority

2. A chord is useful when you need to:
   a) Execute tasks sequentially
   b) Run tasks in parallel and then process results
   c) Retry failed tasks
   d) Schedule periodic tasks

3. The signature of a task is:
   a) Its unique identifier
   b) Its execution history
   c) Its call representation
   d) Its source code location

**True/False**
1. Groups guarantee execution order. (False)
2. Chords can fail if the callback task fails. (True)
3. Chains can be nested within groups. (True)

#### Exercise: Complex Workflow

**Task**: Build a data processing workflow:

```python
"""
Build a complex workflow using Canvas primitives
"""

@app.task
def fetch_user_data(user_id):
    """Fetch user data from API"""
    # TODO: Implement
    pass

@app.task
def fetch_user_posts(user_id):
    """Fetch user posts from database"""
    # TODO: Implement
    pass

@app.task
def fetch_user_friends(user_id):
    """Fetch user friends list"""
    # TODO: Implement
    pass

@app.task
def process_user_data(user_data, posts, friends):
    """Process all user data together"""
    # TODO: Implement
    pass

@app.task
def calculate_scores(processed_data):
    """Calculate various scores for user"""
    # TODO: Implement
    pass

@app.task
def generate_report(user_id, scores):
    """Generate final user report"""
    # TODO: Implement
    pass

def create_user_analysis_workflow(user_id):
    """
    Create a workflow that:
    1. Fetches user data, posts, and friends in parallel
    2. Processes all data together
    3. Calculates scores
    4. Generates report
    """
    # TODO: Implement using group and chain
    pass

def batch_analyze_users(user_ids):
    """
    Analyze multiple users in parallel
    """
    # TODO: Implement using group
    pass
```

**Solution**:

```python
import requests
from celery import chain, group, chord

# Mock data for demonstration
def mock_fetch_user(user_id):
    return {
        'id': user_id,
        'name': f'User {user_id}',
        'email': f'user{user_id}@example.com',
        'join_date': '2023-01-01'
    }

def mock_fetch_posts(user_id):
    return [
        {'id': i, 'content': f'Post {i} by user {user_id}'}
        for i in range(5)
    ]

def mock_fetch_friends(user_id):
    return [f'friend_{i}' for i in range(3)]

@app.task
def fetch_user_data(user_id):
    """Fetch user data from API"""
    # In real app: requests.get(f'/api/users/{user_id}')
    return mock_fetch_user(user_id)

@app.task
def fetch_user_posts(user_id):
    """Fetch user posts from database"""
    # In real app: Database query
    return mock_fetch_posts(user_id)

@app.task
def fetch_user_friends(user_id):
    """Fetch user friends list"""
    # In real app: Social network API
    return mock_fetch_friends(user_id)

@app.task
def process_user_data(results):
    """Process all user data together

    results comes from chord and contains:
    [user_data, posts, friends] in order
    """
    user_data, posts, friends = results

    processed = {
        'user': user_data,
        'post_count': len(posts),
        'friend_count': len(friends),
        'activity_score': len(posts) * 10 + len(friends) * 5,
        'processed_at': time.time()
    }

    return processed

@app.task
def calculate_scores(processed_data):
    """Calculate various scores for user"""
    scores = {
        'engagement_score': processed_data['activity_score'],
        'social_score': processed_data['friend_count'] * 20,
        'content_score': processed_data['post_count'] * 15,
        'total_score': 0
    }
    scores['total_score'] = (
        scores['engagement_score'] +
        scores['social_score'] +
        scores['content_score']
    )

    return {
        'user_id': processed_data['user']['id'],
        'scores': scores,
        'calculated_at': time.time()
    }

@app.task
def generate_report(results):
    """Generate final user report"""
    # results comes from chain: [processed_data, scores]
    processed_data, scores = results

    report = {
        'user_id': processed_data['user']['id'],
        'user_name': processed_data['user']['name'],
        'summary': {
            'posts': processed_data['post_count'],
            'friends': processed_data['friend_count'],
            'total_score': scores['total_score']
        },
        'details': {
            'engagement': scores['scores']['engagement_score'],
            'social': scores['scores']['social_score'],
            'content': scores['scores']['content_score']
        },
        'generated_at': time.time()
    }

    return report

def create_user_analysis_workflow(user_id):
    """Create workflow using chord and chain"""

    # Create parallel tasks
    parallel_tasks = group(
        fetch_user_data.s(user_id),
        fetch_user_posts.s(user_id),
        fetch_user_friends.s(user_id)
    )

    # Create workflow: parallel -> process -> calculate -> report
    workflow = chain(
        chord(
            parallel_tasks,
            process_user_data.s()
        ),
        calculate_scores.s(),
        generate_report.s()
    )

    return workflow()

def batch_analyze_users(user_ids):
    """Analyze multiple users in parallel"""

    # Create workflow for each user
    workflows = group(
        create_user_analysis_workflow.s(user_id)
        for user_id in user_ids
    )

    return workflows()

# Alternative approach using map for batch processing
def analyze_users_with_map(user_ids):
    """Map operation for batch processing"""
    return create_user_analysis_workflow.map(user_ids)

# Example usage:
# result = create_user_analysis_workflow(123)
# report = result.get()

# batch_result = batch_analyze_users([1, 2, 3, 4, 5])
# reports = batch_result.get()
```

---

## Case Studies

### Case Study 1: E-commerce Order Processing (Learning Path 1 Capstone)

**Scenario**: Build an order processing system for an e-commerce platform

**Requirements**:
1. Process orders asynchronously
2. Send confirmation emails
3. Update inventory
4. Generate shipping labels
5. Handle payment processing
6. Track order status

**Solution Implementation**:

```python
# order_processing.py
from celery import Celery, chain, group
import time
import uuid

app = Celery('order_processor')

# Order states
ORDER_STATES = {
    'PENDING': 'pending',
    'PROCESSING': 'processing',
    'PAID': 'paid',
    'SHIPPED': 'shipped',
    'DELIVERED': 'delivered',
    'CANCELLED': 'cancelled',
    'FAILED': 'failed'
}

@app.task(bind=True, max_retries=3)
def process_payment(self, order_id, amount, payment_method):
    """Process payment with retry logic"""
    try:
        # Simulate payment processing
        time.sleep(2)

        # Simulate occasional failure
        if uuid.uuid4().int % 10 == 0:
            raise Exception("Payment gateway timeout")

        return {
            'order_id': order_id,
            'status': 'success',
            'transaction_id': str(uuid.uuid4()),
            'amount': amount,
            'method': payment_method
        }
    except Exception as exc:
        # Retry with exponential backoff
        countdown = 2 ** self.request.retries
        raise self.retry(exc=exc, countdown=countdown)

@app.task
def update_inventory(items):
    """Update inventory for ordered items"""
    updated = []
    for item in items:
        # In real app: Update database
        updated.append({
            'product_id': item['product_id'],
            'quantity': -item['quantity']
        })
    return updated

@app.task
def send_confirmation_email(order_id, customer_email, order_details):
    """Send order confirmation email"""
    time.sleep(1)  # Simulate email sending
    return {
        'order_id': order_id,
        'email': customer_email,
        'sent_at': time.time(),
        'template': 'order_confirmation'
    }

@app.task
def generate_shipping_label(order_id, shipping_address):
    """Generate shipping label"""
    time.sleep(3)  # Simulate label generation
    return {
        'order_id': order_id,
        'tracking_number': f'TRK{uuid.uuid4().hex[:12].upper()}',
        'label_url': f'https://shipping.example.com/labels/{order_id}',
        'generated_at': time.time()
    }

@app.task
def update_order_status(order_id, status, details=None):
    """Update order status in database"""
    return {
        'order_id': order_id,
        'status': status,
        'details': details,
        'updated_at': time.time()
    }

def process_order(order_data):
    """Create order processing workflow"""
    order_id = order_data['order_id']

    # Create workflow
    workflow = chain(
        # Step 1: Update order to processing
        update_order_status.s(order_id, ORDER_STATES['PROCESSING']),

        # Step 2: Process payment and update inventory in parallel
        group(
            process_payment.s(
                order_data['total'],
                order_data['payment_method']
            ),
            update_inventory.s(order_data['items'])
        ),

        # Step 3: After payment succeeds, continue with fulfillment
        send_confirmation_email.s(
            order_data['customer']['email'],
            order_data
        ),
        generate_shipping_label.s(order_data['shipping_address']),

        # Step 4: Mark as paid
        update_order_status.s(order_id, ORDER_STATES['PAID'])
    )

    return workflow()

# Error handling callback
@app.task
def handle_payment_failure(request, exc, traceback):
    """Handle payment processing failures"""
    order_id = request.args[0] if request.args else 'unknown'

    # Update order status to failed
    update_order_status.delay(order_id, ORDER_STATES['FAILED'], str(exc))

    # Send failure notification
    # send_failure_notification.delay(order_id, exc)

    return f"Handled payment failure for order {order_id}"

# Connect error handler
process_payment.on_failure(handle_payment_failure)
```

**Testing the Case Study**:

```python
# test_order_processing.py
from order_processing import process_order, app

# Test order data
test_order = {
    'order_id': 'ORD-12345',
    'customer': {
        'email': 'customer@example.com',
        'name': 'John Doe'
    },
    'items': [
        {'product_id': 'PROD-001', 'quantity': 2, 'price': 29.99},
        {'product_id': 'PROD-002', 'quantity': 1, 'price': 49.99}
    ],
    'total': 109.97,
    'payment_method': 'credit_card',
    'shipping_address': {
        'street': '123 Main St',
        'city': 'Anytown',
        'state': 'CA',
        'zip': '12345'
    }
}

# Process the order
if __name__ == '__main__':
    result = process_order(test_order)
    print(f"Order processing initiated: {result.id}")

    # Wait for completion (in production, you'd use other methods)
    final_result = result.get(timeout=60)
    print(f"Order processing completed: {final_result}")
```

---

### Case Study 2: Social Media Analytics Pipeline (Learning Path 2 Capstone)

**Scenario**: Build a real-time analytics pipeline for a social media platform

**Requirements**:
1. Process user engagement events
2. Calculate trending topics
3. Generate user recommendations
4. Update analytics dashboards
5. Handle high volume (100K+ events/second)
6. Real-time processing with <5 second latency

**Solution Implementation**:

```python
# analytics_pipeline.py
from celery import Celery, chain, chord, group
from datetime import datetime, timedelta
import time
import hashlib
from collections import defaultdict, Counter
import json

app = Celery('analytics')

# Configuration for high-throughput
app.conf.update(
    broker_url='redis://localhost:6379/1',
    result_backend='redis://localhost:6379/1',
    task_serializer='json',
    worker_prefetch_multiplier=100,
    task_acks_late=False,
    worker_disable_rate_limits=True,
    task_compression='gzip',
    result_compression='gzip',
)

@app.task(bind=True)
def process_event(self, event_data):
    """Process individual engagement event"""
    try:
        # Extract event data
        event_type = event_data['type']
        user_id = event_data['user_id']
        timestamp = event_data['timestamp']
        content_id = event_data.get('content_id')

        # Update metrics
        metrics = {
            'event_type': event_type,
            'user_id': user_id,
            'content_id': content_id,
            'timestamp': timestamp,
            'processed_at': time.time()
        }

        # Route to specialized processors
        if event_type == 'like':
            return process_like_event.delay(metrics)
        elif event_type == 'share':
            return process_share_event.delay(metrics)
        elif event_type == 'comment':
            return process_comment_event.delay(metrics)
        elif event_type == 'view':
            return process_view_event.delay(metrics)

        return metrics
    except Exception as exc:
        # Log error but don't fail - analytics shouldn't block user experience
        app.logger.error(f"Failed to process event: {exc}")
        return None

@app.task
def process_like_event(event_data):
    """Process like event"""
    # Update content likes
    update_content_metrics.delay(
        event_data['content_id'],
        {'likes': 1}
    )

    # Update user activity
    update_user_activity.delay(
        event_data['user_id'],
        'like',
        event_data['timestamp']
    )

    return event_data

@app.task
def calculate_trending_content(time_window=3600):
    """Calculate trending content in the last hour"""
    # This would typically query a time-series database
    # For demo, we'll simulate
    trending = [
        {'content_id': f'CONT-{i}', 'score': 1000 - i * 50}
        for i in range(10)
    ]

    return {
        'timestamp': time.time(),
        'window_seconds': time_window,
        'trending': trending
    }

@app.task
def update_recommendations(user_id):
    """Update user recommendations based on activity"""
    # In real app: Use ML model
    recommendations = [
        {'content_id': f'REC-{i}', 'score': 0.9 - i * 0.1}
        for i in range(5)
    ]

    return {
        'user_id': user_id,
        'recommendations': recommendations,
        'updated_at': time.time()
    }

@app.task
def update_content_metrics(content_id, metrics):
    """Update content engagement metrics"""
    # In real app: Update in database with atomic operations
    return {
        'content_id': content_id,
        'metrics': metrics,
        'updated_at': time.time()
    }

@app.task
def update_user_activity(user_id, activity_type, timestamp):
    """Update user activity tracking"""
    return {
        'user_id': user_id,
        'activity': activity_type,
        'timestamp': timestamp
    }

@app.task
def generate_analytics_report(time_range='hourly'):
    """Generate analytics dashboard data"""
    # Aggregate data from various sources
    report = {
        'time_range': time_range,
        'generated_at': time.time(),
        'metrics': {
            'total_events': 1000000,
            'active_users': 50000,
            'engagement_rate': 0.75,
            'top_content': calculate_trending_content()
        }
    }

    return report

def process_event_batch(events):
    """Process a batch of events efficiently"""
    # Group events by type for batch processing
    event_groups = defaultdict(list)
    for event in events:
        event_groups[event['type']].append(event)

    # Create parallel processing tasks
    tasks = []
    for event_type, event_list in event_groups.items():
        if len(event_list) > 100:  # Large batch
            # Split into smaller batches
            for i in range(0, len(event_list), 100):
                batch = event_list[i:i+100]
                tasks.append(process_event_batch_chunk.s(batch))
        else:
            # Process small batch directly
            for event in event_list:
                tasks.append(process_event.s(event))

    return group(tasks)

@app.task
def process_event_batch_chunk(events):
    """Process a chunk of events as a batch"""
    results = []
    for event in events:
        result = process_event(event)
        results.append(result)
    return results

def setup_periodic_tasks():
    """Setup periodic analytics tasks"""
    from celery.schedules import crontab

    # Every minute: update trending
    app.conf.beat_schedule = {
        'update-trending': {
            'task': 'analytics_pipeline.calculate_trending_content',
            'schedule': 60.0,  # Every 60 seconds
        },
        'hourly-report': {
            'task': 'analytics_pipeline.generate_analytics_report',
            'schedule': crontab(minute=0),  # Every hour
        },
        'update-recommendations': {
            'task': 'analytics_pipeline.update_recommendations',
            'schedule': crontab(minute='*/10'),  # Every 10 minutes
        }
    }

    return app.conf.beat_schedule

# Initialize periodic tasks
setup_periodic_tasks()
```

**Performance Testing**:

```python
# performance_test.py
import time
import random
from analytics_pipeline import process_event_batch

def generate_test_events(num_events):
    """Generate test events"""
    event_types = ['like', 'share', 'comment', 'view']
    events = []

    for i in range(num_events):
        event = {
            'type': random.choice(event_types),
            'user_id': f'USER-{random.randint(1, 10000)}',
            'content_id': f'CONT-{random.randint(1, 1000)}',
            'timestamp': time.time() - random.randint(0, 3600),
            'event_id': f'EVT-{i}'
        }
        events.append(event)

    return events

def run_performance_test(events_per_batch=1000, num_batches=10):
    """Run performance test"""
    print(f"Starting performance test: {events_per_batch} events x {num_batches} batches")

    total_events = 0
    start_time = time.time()

    for batch_num in range(num_batches):
        # Generate events
        events = generate_test_events(events_per_batch)
        total_events += len(events)

        # Process batch
        batch_start = time.time()
        result = process_event_batch(events)
        batch_time = time.time() - batch_start

        print(f"Batch {batch_num + 1}: {len(events)} events in {batch_time:.2f}s")
        print(f"Rate: {len(events) / batch_time:.0f} events/second")

    total_time = time.time() - start_time
    print(f"\nTotal: {total_events} events in {total_time:.2f}s")
    print(f"Average rate: {total_events / total_time:.0f} events/second")

if __name__ == '__main__':
    # Test with 10,000 events per batch
    run_performance_test(events_per_batch=10000, num_batches=5)
```

---

### Case Study 3: High-Frequency Trading System (Learning Path 3 Capstone)

**Scenario**: Build a low-latency trading execution system

**Requirements**:
1. Process market data in real-time
2. Execute trades with microsecond latency
3. Risk management and position limits
4. Audit trail for compliance
5. Handle 10M+ messages/day
6. Sub-millisecond processing for critical paths

**Solution Implementation**:

```python
# trading_system.py
from celery import Celery
import time
import asyncio
import redis
from decimal import Decimal
import json
from dataclasses import dataclass, asdict
from typing import List, Optional
import numpy as np

app = Celery('trading')

# Ultra-low latency configuration
app.conf.update(
    broker_url='redis://localhost:6379/2',
    result_backend='redis://localhost:6379/2',
    task_serializer='json',
    worker_prefetch_multiplier=1,
    task_acks_late=True,
    worker_disable_rate_limits=True,
    broker_connection_retry_on_startup=True,
    broker_connection_max_retries=0,  # Fail fast for trading
)

@dataclass
class MarketData:
    symbol: str
    price: Decimal
    volume: int
    timestamp: float
    exchange: str

@dataclass
class TradeOrder:
    order_id: str
    symbol: str
    side: str  # 'buy' or 'sell'
    quantity: int
    order_type: str  # 'market', 'limit', 'stop'
    price: Optional[Decimal]
    user_id: str
    timestamp: float

@dataclass
class ExecutionReport:
    order_id: str
    status: str
    executed_quantity: int
    executed_price: Optional[Decimal]
    execution_id: str
    timestamp: float

# Redis connection for ultra-fast messaging
redis_client = redis.Redis(host='localhost', port=6379, db=3)

@app.task(bind=True, max_retries=0)  # No retries in trading
def process_market_data(self, market_data_json):
    """Process market data with ultra-low latency"""
    start_time = time.time()

    try:
        # Parse market data
        data = json.loads(market_data_json)
        market_data = MarketData(
            symbol=data['symbol'],
            price=Decimal(str(data['price'])),
            volume=data['volume'],
            timestamp=data['timestamp'],
            exchange=data['exchange']
        )

        # Process with minimal latency
        processing_time = (time.time() - start_time) * 1000  # microseconds

        if processing_time > 100:  # Alert if > 100 microseconds
            log_latency_alert.delay('market_data', processing_time)

        # Update market data cache
        update_market_cache.delay(market_data)

        # Check for trigger orders
        check_trigger_orders.delay(market_data)

        return {
            'symbol': market_data.symbol,
            'processing_time_us': processing_time,
            'timestamp': time.time()
        }

    except Exception as exc:
        # Log error but don't fail the market data feed
        app.logger.error(f"Market data processing error: {exc}")
        return None

@app.task
def execute_order(order_json):
    """Execute trading order"""
    start_time = time.time()

    # Parse order
    data = json.loads(order_json)
    order = TradeOrder(**data)

    # Risk check
    risk_check = perform_risk_check.delay(order)
    if not risk_check.get(timeout=0.001):  # 1ms timeout
        return ExecutionReport(
            order_id=order.order_id,
            status='REJECTED',
            executed_quantity=0,
            executed_price=None,
            execution_id='',
            timestamp=time.time()
        )

    # Execute based on order type
    if order.order_type == 'market':
        execution = execute_market_order(order)
    elif order.order_type == 'limit':
        execution = execute_limit_order.delay(order)
    else:
        execution = ExecutionReport(
            order_id=order.order_id,
            status='REJECTED',
            executed_quantity=0,
            executed_price=None,
            execution_id='',
            timestamp=time.time()
        )

    # Record execution
    record_execution.delay(execution)
    update_position.delay(order, execution)

    processing_time = (time.time() - start_time) * 1000
    return asdict(execution)

@app.task
def perform_risk_check(order: TradeOrder) -> bool:
    """Perform risk management checks"""
    # Check position limits
    current_position = get_current_position.delay(order.user_id, order.symbol).get()

    if order.side == 'buy' and current_position + order.quantity > MAX_POSITION:
        return False

    if order.side == 'sell' and current_position - order.quantity < -MAX_POSITION:
        return False

    # Check user limits
    daily_volume = get_daily_volume.delay(order.user_id).get()
    if daily_volume + order.quantity > USER_DAILY_LIMIT:
        return False

    # Check market hours
    if not is_market_open():
        return False

    return True

@app.task
def execute_market_order(order: TradeOrder) -> ExecutionReport:
    """Execute market order immediately"""
    # Get current market price
    market_price = get_market_price.delay(order.symbol).get()

    # Simulate execution
    executed_quantity = min(order.quantity, get_available_liquidity(order.symbol, order.quantity))

    return ExecutionReport(
        order_id=order.order_id,
        status='FILLED' if executed_quantity == order.quantity else 'PARTIAL_FILLED',
        executed_quantity=executed_quantity,
        executed_price=market_price,
        execution_id=f'EXE-{int(time.time() * 1000000)}',
        timestamp=time.time()
    )

@app.task
def execute_limit_order(order: TradeOrder) -> ExecutionReport:
    """Execute limit order"""
    # Add to order book
    add_to_orderbook.delay(order)

    # Check if can execute immediately
    if can_execute_immediately(order):
        return execute_market_order(order)

    return ExecutionReport(
        order_id=order.order_id,
        status='ACCEPTED',
        executed_quantity=0,
        executed_price=None,
        execution_id='',
        timestamp=time.time()
    )

@app.task
def calculate_portfolio_value(user_id: str):
    """Calculate real-time portfolio value"""
    positions = get_all_positions.delay(user_id).get()
    total_value = Decimal('0')

    for symbol, quantity in positions.items():
        if quantity != 0:
            price = get_market_price.delay(symbol).get()
            total_value += price * quantity

    return {
        'user_id': user_id,
        'total_value': float(total_value),
        'positions': positions,
        'timestamp': time.time()
    }

# Constants
MAX_POSITION = 1000000  # Maximum position size
USER_DAILY_LIMIT = 10000000  # Daily trading limit

# Helper functions (would be implemented)
def get_current_position(user_id, symbol):
    return 0

def get_daily_volume(user_id):
    return 0

def is_market_open():
    return True

def get_market_price(symbol):
    return Decimal('100.00')

def get_available_liquidity(symbol, quantity):
    return quantity

def can_execute_immediately(order):
    return False

# Monitoring tasks
@app.task
def log_latency_alert(operation, latency_us):
    """Log latency alerts"""
    app.logger.warning(f"High latency alert: {operation} took {latency_us}μs")

@app.task
def record_execution(execution):
    """Record execution for compliance"""
    # In real app: Write to audit log
    pass

# Performance monitoring
@app.task
def collect_performance_metrics():
    """Collect system performance metrics"""
    return {
        'timestamp': time.time(),
        'cpu_usage': get_cpu_usage(),
        'memory_usage': get_memory_usage(),
        'message_rate': get_message_rate(),
        'latency_p95': get_latency_percentile(95),
        'latency_p99': get_latency_percentile(99)
    }
```

---

## Comprehensive Quiz Bank

### Foundation Level Quiz (Modules 1-11)

1. **What is the primary purpose of Celery?**
   a) Web framework
   b) Database ORM
   c) Asynchronous task queue
   d) API server

2. **Which component is required for Celery to function?**
   a) Result backend
   b) Message broker
   c) Database
   d) All of the above

3. **A task takes 10 seconds to complete. Should it be run synchronously?**
   a) Yes
   b) No
   c) Depends on the user
   d) Only if it's critical

4. **What does .delay() do?**
   a) Runs task immediately
   b) Sends task to queue
   c) Cancels a task
   d) Retries a task

5. **Which message brokers does Celery support?**
   a) Only RabbitMQ
   b) Only Redis
   c) RabbitMQ, Redis, Amazon SQS
   d) MySQL, PostgreSQL

### Intermediate Level Quiz (Modules 12-22)

1. **What is a chord in Celery?**
   a) A musical task
   b) A task that runs after a group completes
   c) A sequential task chain
   d) A task router

2. **How do you implement task retries?**
   a) Using try/except in task
   b) Using autoretry_for parameter
   c) Both a and b
   d) Retries are automatic

3. **What is the purpose of task routing?**
   a) To encrypt tasks
   b) To direct tasks to specific queues
   c) To schedule tasks
   d) To compress tasks

4. **Which canvas primitive runs tasks in parallel?**
   a) Chain
   b) Group
   c) Chord
   d) Signature

5. **How do you monitor Celery workers?**
   a) Using Flower
   b) Using logs only
   c) Using RabbitMQ UI
   d) Monitoring is not possible

### Advanced Level Quiz (Modules 23-34)

1. **What is a bootstep in Celery?**
   a) A type of task
   b) A worker initialization component
   c) A debugging tool
   d) A monitoring metric

2. **How do you implement custom result backends?**
   a) By inheriting from BaseBackend
   b) By using configuration only
   c) Not possible
   d) Using decorators

3. **What is the Saga pattern used for?**
   a) Task scheduling
   b) Distributed transactions
   c) Load balancing
   d) Caching

4. **How do you optimize Celery for CPU-bound tasks?**
   a) Increase prefetch_multiplier
   b) Use solo pool
   c) Use gevent pool
   d) Disable result backend

5. **What is the purpose of OpenTelemetry integration?**
   a) Data storage
   b) Distributed tracing
   c) Task routing
   d) Message encryption

### Production Level Quiz (Modules 35-47)

1. **How do you achieve zero-downtime deployment with Celery?**
   a) Stop all workers
   b) Use blue-green deployment
   c) Restart workers one by one
   d) Both b and c

2. **What is important for multi-region Celery deployment?**
   a) Local message brokers
   b) Network latency
   c) Data replication
   d) All of the above

3. **How do you handle worker failures in production?**
   a) Manual restart
   b) Supervisor/systemd
   c) Kubernetes
   d) Both b and c

4. **What is crucial for security in production?**
   a) HTTPS only
   b) Authentication
   c) Encryption
   d) All of the above

5. **How do you optimize costs in cloud deployment?**
   a) Spot instances
   b) Auto-scaling
   c) Right-sizing
   d) All of the above

---

## Final Exam

### Part 1: Multiple Choice (30 questions)

1. **Architecture Questions (5)**
   - Broker vs backend differences
   - Worker configurations
   - Message flow understanding
   - Scalability patterns
   - Security considerations

2. **Task Management (5)**
   - Task definition patterns
   - State management
   - Error handling
   - Retry mechanisms
   - Result handling

3. **Canvas Primitives (5)**
   - Chain usage
   - Group patterns
   - Chord implementation
   - Signature creation
   - Complex workflows

4. **Production Operations (5)**
   - Monitoring strategies
   - Performance tuning
   - Troubleshooting
   - Deployment patterns
   - Cost optimization

5. **Advanced Topics (5)**
   - Custom extensions
   - Plugin development
   - Testing strategies
   - Integration patterns
   - Best practices

6. **Scenario-Based (5)**
   - System design questions
   - Problem-solving scenarios
   - Architecture decisions
   - Trade-off analysis
   - Real-world applications

### Part 2: Practical Coding (3 hours)

**Task 1: Build a Document Processing System**
- File upload handling
- Format conversion tasks
- OCR processing
- Notification system
- Progress tracking

**Task 2: Implement a Complex Workflow**
- Multi-stage processing
- Error recovery
- Performance optimization
- Monitoring integration
- Documentation

**Task 3: Debug and Optimize**
- Fix performance issues
- Implement missing features
- Add monitoring
- Write tests
- Document changes

### Part 3: System Design (1 hour)

**Design one of the following:**
1. Real-time analytics platform
2. High-frequency trading system
3. Social media processing pipeline
4. IoT data processing system
5. E-commerce order processing

Requirements:
- Architecture diagram
- Task design
- Error handling
- Scaling strategy
- Monitoring plan

---

## Scoring and Certification

### Passing Requirements
- **Foundation Level**: 80% or higher
- **Intermediate Level**: 80% or higher
- **Advanced Level**: 80% or higher
- **Production Level**: 80% or higher
- **Final Exam**: 85% or higher

### Certification Levels
- **Celery Developer**: Pass all 4 learning paths
- **Celery Expert**: Pass all paths + 90% on final
- **Celery Master**: Pass all paths + 95% on final + complete capstone

### Grading Rubric
- **Quizzes**: Immediate feedback
- **Exercises**: Automated testing
- **Projects**: Manual review
- **Final Exam**: Comprehensive evaluation

---

This comprehensive set of exercises, quizzes, and case studies provides students with hands-on experience and real-world application of Celery concepts, ensuring they graduate with practical skills and a deep understanding of distributed task processing.