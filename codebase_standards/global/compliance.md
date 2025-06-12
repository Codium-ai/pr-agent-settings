<b>Compliance 1: Contextual Error Logging</b>

is_blocker: True

event: Exception handling in critical system paths

Success Criteria: 
- Comprehensive logging with rich contextual information
- Ability to reconstruct error scenarios

Failure Criteria:
- Minimal or unstructured logging
- Lack of actionable error details

Code example:

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


<b>Compliance 8: Configuration Change Tracking</b>

is_blocker: True

event: System configuration modifications and deployments

Success Criteria:
- Comprehensive logging of configuration changes
- Tracking of who, when, and what was modified
- Ability to reconstruct configuration history

Failure Criteria:
- Undocumented configuration modifications
- Lack of change attribution
- Incomplete change tracking

Code example:

```python
def verify_config_change_compliance(change_entry):
    required_keys = ['user_id', 'timestamp', 'config_key', 'old_value', 'new_value']
    return all(key in change_entry for key in required_keys)

def update_system_configuration(config_key, new_value):
    current_user = get_current_user()
    old_value = get_current_configuration(config_key)
    
    change_entry = {
        'user_id': current_user.id,
        'timestamp': get_current_timestamp(),
        'config_key': config_key,
        'old_value': old_value,
        'new_value': new_value
    }
    
    try:
        if verify_config_change_compliance(change_entry):
            config_logger.info("Configuration changed", extra=change_entry)
        
        apply_configuration_change(config_key, new_value)
    except Exception as e:
        logger.error("Configuration update failed", extra={
            'change_details': change_entry,
            'error': str(e)
        })
        raise
```