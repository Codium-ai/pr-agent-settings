<b>Compliance 4: Data Validation Logging</b>

is_blocker: True

event: Input processing and validation in data-critical systems

Success Criteria:
- Comprehensive input validation
- Logging of validation failures
- Prevention of invalid data processing

Failure Criteria:
- Bypassing validation checks
- Silent failure of validation
- Incomplete logging of validation attempts

Code example:

```python
def validate_input_compliance(validation_results):
    return all(validation_results.values())

def process_user_input(user_data):
    validation_results = {
        'type_check': isinstance(user_data, dict),
        'required_fields': all(field in user_data for field in ['name', 'email'])
    }
    
    if not validate_input_compliance(validation_results):
        logger.warning("Input validation failed", extra=validation_results)
        raise ValidationError("Invalid input data")
    
    return process_data(user_data)
```
