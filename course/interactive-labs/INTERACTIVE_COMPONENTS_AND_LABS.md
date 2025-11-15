# Interactive Components and Labs

**Last Updated:** 2025-11-15
**Version:** 1.0
**Purpose:** Interactive learning components, hands-on labs, and practical exercises

---

## Interactive Learning Overview

### Interactive Component Categories
1. **Web-Based Simulators** - Browser-based task execution environments
2. **Interactive Workflows** - Visual workflow builders and debuggers
3. **Real-Time Monitoring** - Live dashboards and performance visualizers
4. **Code Playground** - In-browser code editor with Celery execution
5. **Virtual Labs** - Complete development environments with guided exercises

### Learning Objectives
- Provide hands-on experience without local setup
- Enable experimentation with different configurations
- Visualize complex workflows and task relationships
- Facilitate rapid prototyping and testing
- Support collaborative learning and debugging

---

## Web-Based Task Workflow Visualizer

### Component Description
An interactive web application that allows students to build, visualize, and execute Celery workflows in real-time.

### Features
1. **Visual Workflow Builder**
   - Drag-and-drop interface for creating task chains, groups, and chords
   - Real-time connection validation
   - Automatic code generation from visual designs
   - Template library for common patterns

2. **Live Execution Monitoring**
   - Real-time task status updates
   - Execution flow visualization
   - Performance metrics display
   - Error highlighting and debugging

3. **Interactive Debugging**
   - Step-through workflow execution
   - Breakpoint setting at task boundaries
   - Variable inspection and modification
   - Error stack trace visualization

### Technical Implementation
```javascript
// Frontend component for workflow visualization
class WorkflowVisualizer {
    constructor(containerId) {
        this.container = document.getElementById(containerId);
        this.tasks = new Map();
        this.connections = [];
        this.executor = new WorkflowExecutor();
    }

    addTask(taskConfig) {
        const task = new TaskNode(taskConfig);
        this.tasks.set(task.id, task);
        this.render();
    }

    connectTasks(sourceId, targetId) {
        const connection = new Connection(sourceId, targetId);
        this.connections.push(connection);
        this.render();
    }

    async executeWorkflow() {
        const workflow = this.generateWorkflow();
        const execution = this.executor.execute(workflow);
        this.monitorExecution(execution);
    }

    generateWorkflow() {
        // Convert visual representation to Celery workflow code
        const workflow = {
            tasks: Array.from(this.tasks.values()),
            connections: this.connections,
            code: this.generateCeleryCode()
        };
        return workflow;
    }
}
```

### User Interface Design
```html
<div class="workflow-builder">
    <div class="toolbar">
        <button class="task-btn" data-type="chain">Add Chain</button>
        <button class="task-btn" data-type="group">Add Group</button>
        <button class="task-btn" data-type="chord">Add Chord</button>
        <button class="execute-btn">Execute Workflow</button>
    </div>

    <div class="canvas" id="workflow-canvas">
        <!-- Visual workflow representation -->
    </div>

    <div class="code-panel">
        <h3>Generated Code:</h3>
        <pre id="generated-code"></pre>
    </div>

    <div class="monitoring-panel">
        <h3>Execution Status:</h3>
        <div id="execution-status"></div>
    </div>
</div>
```

### Integration with Backend
```python
# Backend API for workflow execution
from flask import Flask, request, jsonify
from celery import chain, group, chord
import json

app = Flask(__name__)

@app.route('/api/execute-workflow', methods=['POST'])
def execute_workflow():
    workflow_data = request.json

    # Parse workflow definition
    if workflow_data['type'] == 'chain':
        workflow = build_chain(workflow_data['tasks'])
    elif workflow_data['type'] == 'group':
        workflow = build_group(workflow_data['tasks'])
    elif workflow_data['type'] == 'chord':
        workflow = build_chord(workflow_data['tasks'], workflow_data['callback'])

    # Execute workflow
    result = workflow.apply_async()

    return jsonify({
        'task_id': result.id,
        'status': result.status
    })

@app.route('/api/workflow-status/<task_id>')
def get_workflow_status(task_id):
    result = AsyncResult(task_id)

    return jsonify({
        'status': result.status,
        'result': result.result if result.ready() else None,
        'traceback': result.traceback if result.failed() else None
    })
```

### Learning Activities
1. **Foundation Exercises:**
   - Build simple task chains
   - Create parallel task groups
   - Implement basic chords
   - Debug failed workflows

2. **Intermediate Challenges:**
   - Design complex multi-stage workflows
   - Implement error handling patterns
   - Optimize workflow performance
   - Handle partial failures

3. **Advanced Projects:**
   - Create workflow templates
   - Implement custom workflow patterns
   - Design auto-scaling workflows
   - Build workflow monitoring systems

---

## Real-Time Monitoring Dashboard

### Component Description
A comprehensive monitoring dashboard that provides real-time insights into Celery task execution, worker performance, and system health.

### Key Features
1. **Live Task Metrics**
   - Active task count and status
   - Task execution time distributions
   - Success/failure rates
   - Queue depth and processing rates

2. **Worker Performance Monitoring**
   - CPU and memory usage per worker
   - Task throughput statistics
   - Worker health status
   - Auto-scaling recommendations

3. **System Health Indicators**
   - Broker connection status
   - Backend availability
   - Network latency measurements
   - Error rate alerts

### Dashboard Architecture
```python
# Real-time monitoring backend
from flask_socketio import SocketIO, emit
from celery.events.state import State
import redis
import json

class MonitoringDashboard:
    def __init__(self, app=None):
        self.app = app
        self.socketio = SocketIO(app)
        self.state = State()
        self.redis_client = redis.Redis()

        if app:
            self.init_app(app)

    def init_app(self, app):
        self.app = app
        self.setup_socketio_handlers()
        self.start_event_capture()

    def setup_socketio_handlers(self):
        @self.socketio.on('connect')
        def handle_connect():
            emit('initial_data', self.get_current_metrics())

        @self.socketio.on('subscribe_metrics')
        def handle_subscription(data):
            # Subscribe to specific metric updates
            room = data.get('room', 'default')
            join_room(room)

    def start_event_capture(self):
        # Start capturing Celery events
        from celery.events import EventReceiver
        from kombu import Connection

        def event_handler(event):
            self.process_event(event)
            self.broadcast_update(event)

        with Connection(self.app.config['CELERY_BROKER_URL']) as conn:
            receiver = EventReceiver(conn, handlers={'*': event_handler})
            receiver.capture()

    def process_event(self, event):
        # Process and store event data
        event_type = event['type']

        if event_type == 'task-sent':
            self.track_task_sent(event)
        elif event_type == 'task-started':
            self.track_task_started(event)
        elif event_type == 'task-succeeded':
            self.track_task_succeeded(event)
        elif event_type == 'task-failed':
            self.track_task_failed(event)

    def get_current_metrics(self):
        return {
            'active_tasks': self.get_active_task_count(),
            'worker_stats': self.get_worker_statistics(),
            'queue_info': self.get_queue_information(),
            'system_health': self.get_system_health()
        }

    def broadcast_update(self, event):
        self.socketio.emit('metric_update', {
            'event': event,
            'metrics': self.get_current_metrics()
        })
```

### Frontend Dashboard Implementation
```javascript
class MonitoringDashboard {
    constructor(containerId) {
        this.container = document.getElementById(containerId);
        this.socket = io();
        this.charts = new Map();
        this.setupSocketHandlers();
        this.initializeCharts();
    }

    setupSocketHandlers() {
        this.socket.on('connect', () => {
            console.log('Connected to monitoring socket');
        });

        this.socket.on('initial_data', (data) => {
            this.updateDashboard(data);
        });

        this.socket.on('metric_update', (data) => {
            this.updateMetrics(data);
        });
    }

    initializeCharts() {
        // Task execution time chart
        this.charts.set('executionTime', new Chart(
            document.getElementById('execution-time-chart'),
            {
                type: 'line',
                data: {
                    labels: [],
                    datasets: [{
                        label: 'Average Execution Time',
                        data: [],
                        borderColor: 'rgb(75, 192, 192)',
                        tension: 0.1
                    }]
                },
                options: {
                    responsive: true,
                    scales: {
                        y: {
                            beginAtZero: true
                        }
                    }
                }
            }
        ));

        // Task success rate chart
        this.charts.set('successRate', new Chart(
            document.getElementById('success-rate-chart'),
            {
                type: 'doughnut',
                data: {
                    labels: ['Success', 'Failure', 'Pending'],
                    datasets: [{
                        data: [0, 0, 0],
                        backgroundColor: [
                            'rgb(75, 192, 192)',
                            'rgb(255, 99, 132)',
                            'rgb(255, 205, 86)'
                        ]
                    }]
                }
            }
        ));
    }

    updateMetrics(data) {
        // Update task counts
        document.getElementById('active-tasks').textContent = data.metrics.active_tasks;

        // Update worker stats
        this.updateWorkerStats(data.metrics.worker_stats);

        // Update charts
        this.updateCharts(data.event);

        // Update health indicators
        this.updateHealthIndicators(data.metrics.system_health);
    }

    updateCharts(event) {
        if (event.type === 'task-succeeded') {
            this.updateExecutionTimeChart(event);
        }

        // Update success/failure rates
        const currentData = this.charts.get('successRate').data.datasets[0].data;
        // Update based on event type
    }
}
```

### Interactive Monitoring Features
1. **Real-Time Filtering**
   - Filter by task type
   - Filter by worker
   - Filter by time range
   - Custom metric filters

2. **Drill-Down Capabilities**
   - Click on tasks for detailed information
   - Worker performance breakdown
   - Queue depth analysis
   - Error investigation tools

3. **Alert Configuration**
   - Custom threshold alerts
   - Rate-based alerting
   - Pattern-based notifications
   - Integration with external systems

### Learning Exercises
1. **Foundation Monitoring:**
   - Set up basic task monitoring
   - Track task execution times
   - Identify failed tasks
   - Monitor worker health

2. **Intermediate Analysis:**
   - Create custom metrics
   - Set up performance alerts
   - Analyze task patterns
   - Optimize based on metrics

3. **Advanced Optimization:**
   - Implement auto-scaling based on metrics
   - Create predictive monitoring
   - Build custom dashboards
   - Integrate with external monitoring systems

---

## Kubernetes Deployment Workshop

### Component Description
A comprehensive interactive workshop that guides students through deploying Celery applications on Kubernetes, from basic concepts to production-ready configurations.

### Workshop Structure

#### Module 1: Kubernetes Fundamentals for Celery (2 hours)
**Learning Objectives:**
- Understand Kubernetes architecture and concepts
- Learn container orchestration basics
- Set up local Kubernetes environment
- Deploy simple applications

**Interactive Exercises:**
1. **Minikube Setup Lab (30 minutes)**
   - Install and configure Minikube
   - Verify cluster functionality
   - Deploy test application
   - Explore cluster components

   ```bash
   # Interactive command simulation
   $ minikube start --cpus=2 --memory=4g
   $ kubectl cluster-info
   $ kubectl get nodes
   $ kubectl run test-app --image=nginx --port=80
   ```

2. **Pod and Service Management (45 minutes)**
   - Create and manage pods
   - Expose services
   - Configure networking
   - Test connectivity

   ```yaml
   # Interactive YAML editor
   apiVersion: v1
   kind: Pod
   metadata:
     name: celery-worker
   spec:
     containers:
     - name: worker
       image: celery/celery:latest
       command: ["celery", "-A", "app", "worker"]
       env:
       - name: CELERY_BROKER_URL
         value: "redis://redis-service:6379"
   ```

3. **Configuration and Secrets (30 minutes)**
   - Manage application configurations
   - Store sensitive data in secrets
   - Environment variable management
   - Configuration validation

#### Module 2: Celery on Kubernetes Architecture (3 hours)
**Learning Objectives:**
- Design Celery deployment patterns
- Implement horizontal pod autoscaling
- Configure resource management
- Set up service discovery

**Interactive Deployment Exercises:**
1. **Celery Worker Deployment (60 minutes)**
   - Create worker deployment manifest
   - Configure resource limits and requests
   - Implement readiness and liveness probes
   - Set up rolling updates

   ```yaml
   # Interactive deployment builder
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: celery-workers
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
           image: my-celery-app:latest
           resources:
             requests:
               cpu: 100m
               memory: 256Mi
             limits:
               cpu: 500m
               memory: 512Mi
           livenessProbe:
             exec:
               command: ["celery", "inspect", "ping"]
             initialDelaySeconds: 30
             periodSeconds: 10
           readinessProbe:
             exec:
               command: ["celery", "inspect", "stats"]
             initialDelaySeconds: 10
             periodSeconds: 5
   ```

2. **Auto-Scaling Configuration (45 minutes)**
   - Configure Horizontal Pod Autoscaler
   - Set scaling metrics and thresholds
   - Test scaling behavior
   - Optimize scaling policies

   ```yaml
   # Interactive HPA configuration
   apiVersion: autoscaling/v2
   kind: HorizontalPodAutoscaler
   metadata:
     name: celery-worker-hpa
   spec:
     scaleTargetRef:
       apiVersion: apps/v1
       kind: Deployment
       name: celery-workers
     minReplicas: 2
     maxReplicas: 10
     metrics:
     - type: Resource
       resource:
         name: cpu
         target:
           type: Utilization
           averageUtilization: 70
     - type: Pods
       pods:
         metric:
           name: celery_queue_length
         target:
           type: AverageValue
           averageValue: "5"
   ```

3. **Service Discovery Configuration (45 minutes)**
   - Set up internal service communication
   - Configure external access
   - Implement load balancing
   - Test connectivity and failover

#### Module 3: Advanced Kubernetes Features (3 hours)
**Learning Objectives:**
- Implement advanced networking
- Configure persistent storage
- Set up monitoring and logging
- Implement security best practices

**Advanced Topics:**
1. **Storage Configuration (60 minutes)**
   - Set up persistent volumes for result backend
   - Configure storage classes
   - Implement data backup strategies
   - Test storage resilience

   ```yaml
   # Interactive storage configuration
   apiVersion: v1
   kind: PersistentVolumeClaim
   metadata:
     name: celery-results-pvc
   spec:
     accessModes:
       - ReadWriteOnce
     resources:
       requests:
         storage: 10Gi
     storageClassName: fast-ssd
   ```

2. **Monitoring Stack Deployment (60 minutes)**
   - Deploy Prometheus for metrics collection
   - Set up Grafana for visualization
   - Configure custom Celery metrics
   - Create alerting rules

   ```yaml
   # Interactive monitoring setup
   apiVersion: v1
   kind: ConfigMap
   metadata:
     name: prometheus-config
   data:
     prometheus.yml: |
       global:
         scrape_interval: 15s
       scrape_configs:
       - job_name: 'celery-workers'
         kubernetes_sd_configs:
         - role: pod
         relabel_configs:
         - source_labels: [__meta_kubernetes_pod_label_app]
           action: keep
           regex: celery-worker
   ```

3. **Security Implementation (60 minutes)**
   - Configure RBAC
   - Set up network policies
   - Implement pod security policies
   - Secure secrets management

### Interactive Lab Environment
```python
# Lab orchestration system
class KubernetesWorkshop:
    def __init__(self):
        self.k8s_client = kubernetes.client.ApiClient()
        self.v1 = kubernetes.client.CoreV1Api()
        self.apps_v1 = kubernetes.client.AppsV1Api()
        self.lab_progress = {}

    def start_lab(self, lab_id, student_id):
        """Initialize lab environment for student"""
        namespace = f"lab-{lab_id}-{student_id}"

        # Create namespace for student
        self.create_namespace(namespace)

        # Deploy initial resources
        self.deploy_lab_resources(namespace, lab_id)

        # Generate access credentials
        kubeconfig = self.generate_kubeconfig(namespace)

        return {
            'namespace': namespace,
            'kubeconfig': kubeconfig,
            'instructions': self.get_lab_instructions(lab_id)
        }

    def validate_lab_step(self, lab_id, step_id, student_id, solution):
        """Validate student's solution"""
        namespace = f"lab-{lab_id}-{student_id}"

        if step_id == 'deploy-worker':
            return self.validate_worker_deployment(namespace, solution)
        elif step_id == 'configure-hpa':
            return self.validate_hpa_configuration(namespace, solution)
        elif step_id == 'setup-monitoring':
            return self.validate_monitoring_setup(namespace, solution)

        return False

    def provide_hints(self, lab_id, step_id, student_id):
        """Provide contextual hints based on student's progress"""
        hints = self.load_hints_database()
        return hints.get(f"{lab_id}_{step_id}", [])

    def auto_grade(self, lab_id, student_id):
        """Automatically grade lab completion"""
        namespace = f"lab-{lab_id}-{student_id}"

        grade_criteria = self.get_grading_criteria(lab_id)
        results = {}

        for criterion in grade_criteria:
            if criterion['type'] == 'deployment_check':
                results[criterion['name']] = self.check_deployment(
                    namespace, criterion['resource']
                )
            elif criterion['type'] == 'metric_check':
                results[criterion['name']] = self.check_metrics(
                    namespace, criterion['metric']
                )
            elif criterion['type'] == 'configuration_check':
                results[criterion['name']] = self.check_configuration(
                    namespace, criterion['config']
                )

        return self.calculate_grade(results)
```

### Workshop Assessment
1. **Practical Evaluation (70%)**
   - Deployment success
   - Configuration correctness
   - Performance optimization
   - Security implementation

2. **Problem-Solving (20%)**
   - Troubleshooting exercises
   - Debugging scenarios
   - Optimization challenges

3. **Documentation (10%)**
   - Deployment documentation
   - Configuration explanations
   - Best practice justification

---

## Performance Profiling and Optimization Lab

### Component Description
An interactive performance profiling lab that helps students identify bottlenecks, optimize Celery applications, and understand performance characteristics of distributed task systems.

### Lab Features
1. **Performance Profiling Tools**
   - CPU and memory profiling
   - Task execution time analysis
   - I/O operation monitoring
   - Network latency measurement

2. **Bottleneck Identification**
   - Visual performance maps
   - Resource utilization graphs
   - Task dependency analysis
   - Queue depth tracking

3. **Optimization Simulator**
   - Configuration optimization
   - Resource allocation testing
   - Algorithm comparison
   - Scaling strategy evaluation

### Interactive Profiling Interface
```python
# Performance profiling system
class CeleryProfiler:
    def __init__(self, app):
        self.app = app
        self.profiles = {}
        self.metrics_collector = MetricsCollector()

    def profile_task(self, task_name, *args, **kwargs):
        """Profile task execution with detailed metrics"""
        def decorator(func):
            @functools.wraps(func)
            def wrapper(*args, **kwargs):
                # Start profiling
                profiler = cProfile.Profile()
                profiler.enable()

                # Collect system metrics before execution
                start_metrics = self.metrics_collector.collect()

                # Execute the task
                start_time = time.time()
                result = func(*args, **kwargs)
                execution_time = time.time() - start_time

                # Stop profiling
                profiler.disable()

                # Collect system metrics after execution
                end_metrics = self.metrics_collector.collect()

                # Analyze results
                profile_data = {
                    'task_name': task_name,
                    'execution_time': execution_time,
                    'cpu_usage': end_metrics['cpu'] - start_metrics['cpu'],
                    'memory_usage': end_metrics['memory'] - start_metrics['memory'],
                    'io_operations': end_metrics['io'] - start_metrics['io'],
                    'profile_stats': profiler.getstats()
                }

                self.profiles[task_name] = profile_data
                return result

            return wrapper
        return decorator

    def generate_performance_report(self, task_name=None):
        """Generate comprehensive performance report"""
        if task_name:
            profiles = [self.profiles.get(task_name)]
        else:
            profiles = list(self.profiles.values())

        report = {
            'summary': self.generate_summary(profiles),
            'bottlenecks': self.identify_bottlenecks(profiles),
            'optimizations': self.suggest_optimizations(profiles),
            'detailed_metrics': profiles
        }

        return report

    def identify_bottlenecks(self, profiles):
        """Identify performance bottlenecks"""
        bottlenecks = []

        for profile in profiles:
            if profile['execution_time'] > 10.0:  # Slow tasks
                bottlenecks.append({
                    'type': 'slow_execution',
                    'task': profile['task_name'],
                    'value': profile['execution_time'],
                    'suggestion': 'Consider breaking down into smaller tasks'
                })

            if profile['cpu_usage'] > 80:  # High CPU usage
                bottlenecks.append({
                    'type': 'high_cpu',
                    'task': profile['task_name'],
                    'value': profile['cpu_usage'],
                    'suggestion': 'Optimize algorithm or increase CPU resources'
                })

            if profile['memory_usage'] > 1000:  # High memory usage
                bottlenecks.append({
                    'type': 'high_memory',
                    'task': profile['task_name'],
                    'value': profile['memory_usage'],
                    'suggestion': 'Optimize memory usage or implement streaming'
                })

        return bottlenecks
```

### Performance Optimization Exercises

#### Exercise 1: Task Granularity Optimization (2 hours)
**Learning Objectives:**
- Understand the impact of task granularity on performance
- Optimize task size for maximum throughput
- Balance overhead and parallelization

**Exercise Steps:**
1. **Baseline Measurement (30 minutes)**
   - Run monolithic task with large dataset
   - Measure execution time and resource usage
   - Analyze bottlenecks in single large task

2. **Task Decomposition (45 minutes)**
   - Break down large task into smaller subtasks
   - Implement proper task coordination
   - Measure performance of decomposed tasks

3. **Optimization Iteration (30 minutes)**
   - Experiment with different task sizes
   - Find optimal granularity
   - Document performance characteristics

4. **Analysis and Documentation (15 minutes)**
   - Compare baseline vs optimized performance
   - Create performance report
   - Justify optimization decisions

#### Exercise 2: Concurrency Model Optimization (2 hours)
**Learning Objectives:**
- Compare different concurrency models
- Optimize worker configuration
- Understand resource allocation impacts

**Exercise Steps:**
1. **Model Comparison (45 minutes)**
   - Test prefork, eventlet, and gevent models
   - Measure performance under different loads
   - Analyze resource utilization patterns

2. **Configuration Optimization (45 minutes)**
   - Optimize worker pool sizes
   - Tune concurrency settings
   - Test memory vs CPU tradeoffs

3. **Load Testing (20 minutes)**
   - Simulate production workloads
   - Test system behavior under stress
   - Identify breaking points

4. **Recommendation Generation (10 minutes)**
   - Document optimal configuration
   - Provide justification for choices
   - Create deployment recommendations

#### Exercise 3: Resource Allocation and Scaling (1.5 hours)
**Learning Objectives:**
- Understand resource allocation strategies
- Implement auto-scaling based on metrics
- Optimize cost-performance ratio

**Exercise Steps:**
1. **Resource Profiling (30 minutes)**
   - Profile resource usage patterns
   - Identify resource bottlenecks
   - Document consumption characteristics

2. **Scaling Strategy Design (30 minutes)**
   - Design scaling triggers and policies
   - Configure auto-scaling parameters
   - Test scaling behavior

3. **Cost Optimization (20 minutes)**
   - Analyze cost implications of different strategies
   - Optimize for cost-performance balance
   - Create scaling recommendations

### Interactive Performance Dashboard
```javascript
class PerformanceDashboard {
    constructor(containerId) {
        this.container = document.getElementById(containerId);
        this.charts = new Map();
        this.realTimeData = [];
        this.initializeDashboard();
        this.connectToPerformanceStream();
    }

    initializeDashboard() {
        this.createExecutionTimeChart();
        this.createResourceUtilizationChart();
        this.createThroughputChart();
        this.createBottleneckChart();
    }

    createExecutionTimeChart() {
        const ctx = document.getElementById('execution-time-chart').getContext('2d');
        this.charts.set('executionTime', new Chart(ctx, {
            type: 'scatter',
            data: {
                datasets: [{
                    label: 'Task Execution Time',
                    data: [],
                    backgroundColor: 'rgba(75, 192, 192, 0.6)'
                }]
            },
            options: {
                responsive: true,
                scales: {
                    x: {
                        title: {
                            display: true,
                            text: 'Task ID'
                        }
                    },
                    y: {
                        title: {
                            display: true,
                            text: 'Execution Time (seconds)'
                        }
                    }
                }
            }
        }));
    }

    updatePerformanceMetrics(metrics) {
        // Update execution time chart
        const execChart = this.charts.get('executionTime');
        execChart.data.datasets[0].data.push({
            x: metrics.task_id,
            y: metrics.execution_time
        });
        execChart.update();

        // Update resource utilization
        this.updateResourceUtilization(metrics);

        // Update bottleneck indicators
        this.updateBottlenecks(metrics);

        // Check for performance alerts
        this.checkPerformanceAlerts(metrics);
    }

    checkPerformanceAlerts(metrics) {
        const alerts = [];

        if (metrics.execution_time > 10) {
            alerts.push({
                level: 'warning',
                message: `Task ${metrics.task_id} execution time exceeded 10 seconds`
            });
        }

        if (metrics.cpu_usage > 80) {
            alerts.push({
                level: 'critical',
                message: `High CPU usage detected: ${metrics.cpu_usage}%`
            });
        }

        if (metrics.memory_usage > 1000) {
            alerts.push({
                level: 'warning',
                message: `High memory usage: ${metrics.memory_usage}MB`
            });
        }

        this.displayAlerts(alerts);
    }

    displayAlerts(alerts) {
        const alertContainer = document.getElementById('performance-alerts');
        alertContainer.innerHTML = '';

        alerts.forEach(alert => {
            const alertElement = document.createElement('div');
            alertElement.className = `alert alert-${alert.level}`;
            alertElement.textContent = alert.message;
            alertContainer.appendChild(alertElement);
        });
    }
}
```

### Learning Outcomes Assessment
1. **Performance Identification (30%)**
   - Ability to identify performance bottlenecks
   - Understanding of performance metrics
   - Proficiency with profiling tools

2. **Optimization Implementation (40%)**
   - Successful optimization implementation
   - Performance improvement measurement
   - Justification of optimization choices

3. **Analysis and Documentation (30%)**
   - Comprehensive performance analysis
   - Clear documentation of findings
   - Actionable recommendations

---

## CI/CD Pipeline Integration Lab

### Component Description
A hands-on lab that teaches students how to integrate Celery applications into CI/CD pipelines, including automated testing, deployment, and monitoring.

### Lab Components

#### Module 1: Automated Testing for Celery (2 hours)
**Learning Objectives:**
- Implement comprehensive testing strategies
- Set up automated test execution
- Test task behavior and workflows
- Performance testing implementation

**Interactive Exercises:**
1. **Unit Testing Implementation (45 minutes)**
   ```python
   # Interactive test creation interface
   class TaskTestCase(TestCase):
       def setUp(self):
           self.app = Celery('test')
           self.app.conf.update(
               task_always_eager=True,
               task_eager_propagates=True,
           )

       def test_add_task(self):
           """Interactive test builder"""
           # Student fills in test implementation
           result = add_task.delay(4, 5)
           self.assertEqual(result.get(), 9)
           self.assertTrue(result.successful())
   ```

2. **Integration Testing Setup (45 minutes)**
   ```python
   # Integration test template
   @override_settings(CELERY_TASK_ALWAYS_EAGER=True)
   def test_workflow_integration(self):
       # Test complete workflow
       workflow = create_sample_workflow()
       result = workflow.apply_async()
       self.assertTrue(result.successful())

       # Verify side effects
       self.assertDatabaseState(expected_state)
   ```

3. **Performance Testing (30 minutes)**
   ```python
   # Performance test framework
   def test_task_performance(self):
       """Measure task performance under load"""
       start_time = time.time()

       # Execute multiple tasks
       tasks = [sample_task.delay(i) for i in range(100)]
       results = [task.get() for task in tasks]

       execution_time = time.time() - start_time

       # Assert performance requirements
       self.assertLess(execution_time, 30)  # Should complete in 30 seconds
   ```

#### Module 2: CI/CD Pipeline Configuration (3 hours)
**Learning Objectives:**
- Configure GitHub Actions for Celery applications
- Implement automated testing and deployment
- Set up environment-specific deployments
- Monitor deployment health

**Interactive Pipeline Builder:**
```yaml
# Interactive CI/CD configuration builder
name: Celery CI/CD Pipeline

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        python-version: [3.9, 3.10, 3.11, 3.12]

    services:
      redis:
        image: redis:6
        ports:
          - 6379:6379
      rabbitmq:
        image: rabbitmq:3-management
        ports:
          - 5672:5672
          - 15672:15672
        env:
          RABBITMQ_DEFAULT_USER: guest
          RABBITMQ_DEFAULT_PASS: guest

    steps:
    - uses: actions/checkout@v3

    - name: Set up Python ${{ matrix.python-version }}
      uses: actions/setup-python@v3
      with:
        python-version: ${{ matrix.python-version }}

    - name: Install dependencies
      run: |
        pip install -r requirements.txt
        pip install pytest pytest-cov

    - name: Run unit tests
      run: |
        pytest tests/unit/ -v --cov=celery_app

    - name: Run integration tests
      run: |
        celery -A celery_app worker --loglevel=info &
        sleep 5
        pytest tests/integration/ -v

    - name: Run performance tests
      run: |
        pytest tests/performance/ -v --benchmark-only

  deploy:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'

    steps:
    - uses: actions/checkout@v3

    - name: Deploy to production
      run: |
        # Deployment script
        docker build -t celery-app:${{ github.sha }} .
        docker push registry.example.com/celery-app:${{ github.sha }}
        kubectl set image deployment/celery-workers celery-worker=registry.example.com/celery-app:${{ github.sha }}
```

#### Module 3: Monitoring and Rollback (2 hours)
**Learning Objectives:**
- Implement deployment monitoring
- Set up automated rollback triggers
- Monitor application health post-deployment
- Handle deployment failures gracefully

### Interactive Deployment Simulator
```python
# Deployment simulation and monitoring
class DeploymentSimulator:
    def __init__(self):
        self.deployments = {}
        self.metrics = MetricsCollector()
        self.alert_manager = AlertManager()

    def simulate_deployment(self, config):
        """Simulate deployment process with monitoring"""
        deployment_id = str(uuid.uuid4())

        # Initialize deployment tracking
        deployment = {
            'id': deployment_id,
            'config': config,
            'status': 'pending',
            'start_time': time.time(),
            'metrics': {},
            'alerts': []
        }

        self.deployments[deployment_id] = deployment

        # Start deployment process
        self.start_deployment_monitoring(deployment_id)

        return deployment_id

    def start_deployment_monitoring(self, deployment_id):
        """Monitor deployment health and performance"""
        def monitor_deployment():
            deployment = self.deployments[deployment_id]

            # Collect deployment metrics
            metrics = self.collect_deployment_metrics(deployment)
            deployment['metrics'] = metrics

            # Check deployment health
            health_status = self.check_deployment_health(metrics)

            if health_status['status'] == 'healthy':
                deployment['status'] = 'success'
                self.alert_manager.send_alert(
                    'deployment_success',
                    f"Deployment {deployment_id} completed successfully"
                )
            elif health_status['status'] == 'degraded':
                deployment['status'] = 'warning'
                self.alert_manager.send_alert(
                    'deployment_warning',
                    f"Deployment {deployment_id} completed with warnings"
                )
            elif health_status['status'] == 'failed':
                deployment['status'] = 'failed'
                self.trigger_rollback(deployment_id)

            return health_status

        # Schedule periodic monitoring
        scheduler.add_job(
            monitor_deployment,
            'interval',
            seconds=30,
            id=f'deployment_monitor_{deployment_id}'
        )

    def trigger_rollback(self, deployment_id):
        """Trigger automated rollback"""
        deployment = self.deployments[deployment_id]

        # Implement rollback logic
        previous_version = deployment['config']['previous_version']

        rollback_config = {
            'version': previous_version,
            'rollback_from': deployment_id,
            'reason': 'Automated rollback due to deployment failure'
        }

        # Execute rollback
        self.execute_rollback(rollback_config)

        self.alert_manager.send_alert(
            'deployment_rollback',
            f"Automated rollback triggered for deployment {deployment_id}"
        )
```

### Lab Assessment and Progress Tracking
```python
# Lab progress and assessment system
class LabAssessment:
    def __init__(self):
        self.student_progress = {}
        self.assessment_criteria = self.load_assessment_criteria()

    def track_progress(self, student_id, lab_id, step_id, completion_data):
        """Track student progress through lab exercises"""
        if student_id not in self.student_progress:
            self.student_progress[student_id] = {}

        if lab_id not in self.student_progress[student_id]:
            self.student_progress[student_id][lab_id] = {}

        self.student_progress[student_id][lab_id][step_id] = {
            'completed_at': time.time(),
            'data': completion_data,
            'score': self.evaluate_step_completion(lab_id, step_id, completion_data)
        }

    def evaluate_step_completion(self, lab_id, step_id, data):
        """Evaluate student completion of lab step"""
        criteria = self.assessment_criteria.get(f"{lab_id}_{step_id}", {})

        score = 0
        max_score = criteria.get('max_score', 100)

        if criteria.get('type') == 'code_review':
            score = self.evaluate_code_quality(data.get('code', ''))
        elif criteria.get('type') == 'configuration_check':
            score = self.validate_configuration(data.get('config', {}), criteria.get('expected'))
        elif criteria.get('type') == 'performance_test':
            score = self.evaluate_performance(data.get('metrics', {}), criteria.get('thresholds'))

        return min(score, max_score)

    def generate_lab_report(self, student_id, lab_id):
        """Generate comprehensive lab completion report"""
        progress = self.student_progress.get(student_id, {}).get(lab_id, {})

        total_score = sum(step['score'] for step in progress.values())
        total_possible = len(progress) * 100  # Assuming 100 points per step
        percentage = (total_score / total_possible * 100) if total_possible > 0 else 0

        report = {
            'student_id': student_id,
            'lab_id': lab_id,
            'completion_percentage': percentage,
            'total_score': total_score,
            'total_possible': total_possible,
            'step_details': progress,
            'recommendations': self.generate_recommendations(progress),
            'next_steps': self.get_next_steps(lab_id, percentage)
        }

        return report
```

---

## Implementation Guidelines

### Technical Requirements
1. **Browser Support:** Modern browsers with WebSocket support
2. **Container Infrastructure:** Docker for isolated environments
3. **Real-time Communication:** WebSocket connections for live updates
4. **Persistent Storage:** User progress and session management
5. **Monitoring Infrastructure:** Metrics collection and visualization

### Security Considerations
1. **Isolation:** Student environments isolated from each other
2. **Resource Limits:** CPU, memory, and network restrictions
3. **Access Control:** Authentication and authorization for lab access
4. **Data Privacy:** No sensitive data in sandbox environments
5. **Audit Logging:** Track all student activities for grading

### Accessibility Support
1. **Keyboard Navigation:** Full keyboard accessibility
2. **Screen Reader Support:** ARIA labels and descriptions
3. **Color Contrast:** WCAG AA compliance
4. **Alternative Input:** Voice control and switch device support
5. **Flexible Timing:** Adjustable time limits for exercises

### Scalability Considerations
1. **Load Balancing:** Distribute lab sessions across multiple servers
2. **Resource Pooling:** Shared resources with efficient allocation
3. **Caching:** Cache common resources and configurations
4. **Auto-scaling:** Dynamic scaling based on user demand
5. **Regional Deployment:** Multi-region support for low latency

This comprehensive interactive component suite provides engaging, hands-on learning experiences that complement the theoretical knowledge presented in the course materials, ensuring students develop practical skills in distributed task processing with Celery.