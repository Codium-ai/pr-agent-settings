<b>Compliance 7: Distributed System Tracing</b>

is_blocker: True

event: Microservices and distributed system interactions

Success Criteria:
- End-to-end request tracing
- Correlation of actions across multiple services
- Comprehensive distributed context preservation

Failure Criteria:
- Loss of request context between services
- Inability to track request flow
- Incomplete distributed system visibility

Code example:

```python
def verify_distributed_tracing(trace_context):
    required_keys = ['trace_id', 'parent_span_id', 'service_name']
    return all(key in trace_context for key in required_keys)

def process_distributed_request(request):
    trace_context = {
        'trace_id': generate_trace_id(),
        'parent_span_id': get_current_span_id(),
        'service_name': 'user-service'
    }
    
    try:
        if verify_distributed_tracing(trace_context):
            logger.info("Distributed request processing", extra=trace_context)
        
        result = forward_request_to_next_service(request, trace_context)
        return result
    except Exception as e:
        logger.error("Distributed request failed", extra={
            'trace_context': trace_context,
            'error': str(e)
        })
        raise
```
