<b>Compliance 1: Contextual Error Logging</b>

is_blocker: True

event: Exception handling in critical system paths

Success Criteria: 
- Comprehensive logging with rich contextual information
- Ability to reconstruct error scenarios

Failure Criteria:
- Minimal or unstructured logging
- Lack of actionable error details

### Compliance Verification:

```python
def verify_logging_compliance(log_entry):
    return all(key in log_entry for key in ['user_id', 'error_type'])

def process_user_data(user_id, data):
    try:
        return complex_data_processing(data)
    except Exception as e:
        log_entry = {"user_id": user_id, "error_type": type(e).__name__}
        
        if verify_logging_compliance(log_entry):
            get_logger().exception("Failed to process user data", artifact=log_entry)
        
        raise
```
