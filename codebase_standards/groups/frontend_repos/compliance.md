<b>Compliance 3: Performance and Security Logging</b>

is_blocker: True

event: Critical operations with performance or security implications

Success Criteria:
- Non-blocking logging mechanism
- Minimal performance overhead
- Secure handling of sensitive information

Failure Criteria:
- Logging that significantly impacts system performance
- Exposure of sensitive data
- Blocking main execution thread

Code example:

```python
def verify_logging_performance(log_time, sensitive_data):
    return log_time < 10 and not sensitive_data

def process_sensitive_transaction(transaction):
    start_time = time.time()
    try:
        result = execute_transaction(transaction)
        
        log_entry = {
            "transaction_id": anonymize(transaction.id),
            "sensitive_data": False
        }
        
        log_time = time.time() - start_time
        if verify_logging_performance(log_time, log_entry['sensitive_data']):
            logger.info("Transaction processed", extra=log_entry)
        
        return result
    except TransactionError as e:
        logger.error("Transaction failed", extra={"error": str(e)})
        raise
```
