<b>Compliance 2: Granular Exception Handling</b>

is_blocker: False

event: Multi-step operations with potential failure modes

Success Criteria:
- Type-specific exception handling
- Differentiated error responses
- Actionable error information

Failure Criteria:
- Catching all exceptions with a single, generic handler
- Losing specific error context

Code example:

```python
def check_exception_compliance(exceptions):
    return len(exceptions) > 1

def retrieve_user_data(user_id):
    exceptions = []
    try:
        return database.get_user(user_id)
    except ConnectionError as e:
        exceptions.append(e)
        logger.error("Connection failed")
    except NotFoundError as e:
        exceptions.append(e)
        logger.warning("User not found")
    
    if not check_exception_compliance(exceptions):
        logger.warning("Compliance check failed")
```
