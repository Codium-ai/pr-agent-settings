<b>Compliance 6: Resource Consumption Monitoring</b>

is_blocker: False

event: Long-running operations and resource-intensive processes

Success Criteria:
- Tracking of resource utilization
- Logging of potential performance bottlenecks
- Early detection of inefficient operations

Failure Criteria:
- Lack of resource consumption tracking
- Missing performance metrics
- Inability to identify resource-intensive operations

Code example:

```python
def check_resource_consumption_compliance(resource_metrics):
    return (
        resource_metrics['cpu_usage'] is not None and
        resource_metrics['memory_usage'] is not None
    )

def execute_complex_operation(operation):
    resource_metrics = {
        'cpu_usage': measure_cpu_usage(),
        'memory_usage': measure_memory_usage()
    }
    
    try:
        result = operation()
        
        if not check_resource_consumption_compliance(resource_metrics):
            logger.warning("Resource monitoring incomplete", extra=resource_metrics)
        
        return result
    except Exception as e:
        logger.error("Operation failed", extra={
            'error': str(e),
            'resource_metrics': resource_metrics
        })
        raise
```
