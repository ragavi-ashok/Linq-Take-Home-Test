## Linq Data Candidate Take-Home Test

🌇 Scenario

In an event-driven system, messages are emitted onto an event bus, and a worker service processes these events in real time, performing key calculations. However, an issue has occurred:
Some events were missed or processed incorrectly due to an error.
We now need to recalculate the results without relying on a traditional database for storing historical event data

## Example Considered: Airline Booking System

To illustrate the solution, consider the Airline Booking System scenario because it may closely match the given situation. <br>


Why This Example? <br>
In an airline booking system, each booking generates multiple events (e.g., payment confirmation, ticket issuance, seat allocation). <br>
Due to a system failure, some "Ticket Issued" events were missed, causing customers to pay for flights but not receive their tickets. <br>
Similar to the given problem, I need to recalculate missing data and trigger the necessary events without relying on a traditional database.<br>


## 1. Explain My Approach
### How would I recover and back-calculate the missing/incorrect data?<br>

* I will check the logs to find payments that were processed but did not trigger a "Ticket Issued" event. <br>
* If the logs are incomplete, I will look at payment records or external sources to confirm successful transactions. <br>
* Once I identify affected bookings, I will reconstruct the missing ticket issuance events and trigger them again. <br>
* If the system allows event replay, I will reprocess past events to fill in the gaps. <br>

### What tools, strategies, or techniques would I use?
* I will use event logs or API calls to identify payments that were processed but didn’t result in ticket issuance.
* I will write a Python script or use another automation tool to scan logs, identify missing events, and generate new "Ticket Issued" events.
* To prevent duplicate tickets, I will ensure the process is idempotent, meaning running it multiple times will not create duplicates.
* Implementing monitoring and alerts to detect similar issues in the future and prevent them from happening again.

### How would I ensure accuracy and consistency in the recalculated results?
* I will cross-check payment records to confirm that every transaction is valid before issuing a missing ticket.
* I will track all fixed records to avoid duplicate ticket generation.
* I will test the solution on a small set of data first to verify that it works correctly before applying it on a larger scale.
* If necessary, I will consult the customer support team to clarify uncertain cases before making final corrections.

## 2. Provide a solution
* Write a small script or code snippet demonstrating your approach.
* You can use **Python, JavaScript, or any language of your choice**.
* The solution doesn’t need to be production-ready but should demonstrate how you would solve the problem.

```python
import json

# Set to track fixed transactions to prevent duplicate processing
fixed_transactions = set()

def load_logs(file_path: str) -> list:
    """
    Load JSON log file containing payment and ticket data.
    
    Parameters:
    file_path (str): Path to the JSON log file.
    
    Returns:
    list: List of JSON objects from the log file.
    """
    try:
        with open(file_path, 'r') as file:
            return [json.loads(line) for line in file]  # Read and parse each line as JSON
    except FileNotFoundError: # Return an error if file is not found
        print(f"Error: {file_path} not found.")
        return []

def find_missing_tickets(payment_logs: list, ticket_logs: list) -> set:
    """
    Compare payments with ticket logs to identify missing tickets.
    
    Parameters:
    payment_logs (list): List of payment log JSON objects.
    ticket_logs (list): List of ticket log JSON objects.
    
    Returns:
    set: Set of transaction IDs where tickets are missing.
    """
    # Extract transaction IDs from payment and ticket logs
    payment_txns = {txn["transaction_id"] for txn in payment_logs}
    ticket_txns = {txn["transaction_id"] for txn in ticket_logs}

    # Identify transactions where a payment exists but no corresponding ticket is found.
    # There can be complex cases but keeping it simple at this time.
    missing_tickets = payment_txns - ticket_txns  
    return missing_tickets

def generate_ticket_event(transaction_id: str) -> dict:
    """
    Create a new 'Ticket Issued' event.
    
    Parameters:
    transaction_id (str): The transaction ID for which to generate the event.
    
    Returns:
    dict: A dictionary representing the 'Ticket Issued' event.
    """
    # Trigger the ticketing events
    # After the ticket issue is successful return the Issued event 
    return {
        "transaction_id": transaction_id,
        "event_type": "Ticket Issued"
    }

def log_fixed_transactions(transaction_id: str) -> None:
    """
    Track fixed transactions to prevent duplicate processing.
    
    Parameters:
    transaction_id (str): The transaction ID to log as fixed.
    
    Returns:
    None
    """
    fixed_transactions.add(transaction_id)  # Add the transaction ID to the set

def is_transaction_fixed(transaction_id: str) -> bool:
    """
    Check if a transaction has already been fixed.
    
    Parameters:
    transaction_id (str): The transaction ID to check.
    
    Returns:
    bool: True if the transaction has been fixed, False otherwise.
    """
    return transaction_id in fixed_transactions  # Return True if the transaction exists in the set

def fix_missing_tickets(payment_logs: list, ticket_logs: list) -> list:
    """
    Identify and fix missing ticket issuance events.
    
    Parameters:
    payment_logs (list): List of payment log JSON objects.
    ticket_logs (list): List of ticket log JSON objects.
    
    Returns:
    list: List of newly generated ticket events.
    """
    missing_tickets = find_missing_tickets(payment_logs, ticket_logs)
    fixed_events = []  # List to store newly generated ticket events

    for txn_id in missing_tickets:
        if not is_transaction_fixed(txn_id):  # Avoid duplicate fixes
            ticket_event = generate_ticket_event(txn_id)  # Create a new ticket event
            fixed_events.append(ticket_event)
            log_fixed_transactions(txn_id)  # Mark transaction as fixed in the set

    return fixed_events

def main() -> None:
    """
    Main function to identify and fix missing ticket issuance events.
    
    Returns:
    None
    """
    # Load payment and ticket logs from JSON files
    payment_logs = load_logs("payments.json")
    ticket_logs = load_logs("tickets.json")

    # Process missing tickets and generate fix events
    missing_ticket_events = fix_missing_tickets(payment_logs, ticket_logs)

    # Output the results
    if missing_ticket_events:
        print("Generated Missing Ticket Events:")
        print(json.dumps(missing_ticket_events, indent=4))
    else:
        print("No missing tickets found.")

if __name__ == "__main__":
    main()
```

## 3. Write-up:
### Summarize your approach and why you chose it.
* This approach identifies and fixes missing "Ticket Issued" events by analyzing event logs. Payments that were successfully processed but did not generate a ticket issuance event are detected and corrected. Instead of using a database, this solution relies on log files to retrieve past transactions.
* To prevent duplicate processing, a set is used to track already fixed transactions. This ensures that the same event is not processed more than once. Using a set provides a lightweight, in-memory solution without external dependencies.
* This approach was chosen because it is simple, effective, and does not require a database or caching system like Redis, making it easy to deploy in environments where database access is restricted.

### Discuss any trade-offs or limitations.
#### Advantages:
- No external dependencies – Works without Redis, SQL, or a message queue.
- Fast in-memory operations – Checking for duplicates is O(1) using a set.
- Simple and easy to implement – Can run on any system without additional setup.
#### Limitations:
- Set storage is temporary – The set only exists during execution and will reset if the script is restarted.
- Not persistent – Since the set is stored in memory, fixed transaction data is lost when the script stops.
- Memory constraints for large data – If processing millions of transactions, a large set may consume excessive memory.

### If you had access to more tools (e.g., a database, logs, etc.), how would your approach change?
If access to persistent storage or distributed processing were available, improvements could be made:
#### Using a Database (PostgreSQL, MongoDB, etc.)
- Instead of checking logs manually, a query could be run to identify missing ticket issuance events efficiently.
- This would reduce the need for file-based scanning, making the process faster and more scalable.
#### Using Redis or a Distributed Cache
- If the set grows too large, Redis or Memcached could be used to persist transaction IDs efficiently.
- This would allow for transaction tracking even after script restarts.
#### Event-Driven Architecture (Kafka, RabbitMQ, AWS SQS)
- Instead of running the script periodically, a real-time event listener could automatically detect missing ticket events and trigger fixes immediately.
- This would make the solution fully automated and responsive.

### If applicable, discuss how your solution would scale if this system processed millions of events per hour.
If handling millions of events per hour, optimizations would be needed:
#### Batch Processing Instead of Single Transactions
- Instead of processing events one by one, transactions could be grouped into batches to improve efficiency.
#### Parallel Processing with Multi-threading
- Multiple threads could process different log segments simultaneously, reducing execution time.
#### Distributed Processing (Apache Spark, Dask, or Ray)
- For very large datasets, a distributed system could process log files across multiple machines to improve performance.
#### Using a Persistent Data Store
- Instead of relying on an in-memory set, a database or a persistent cache could be used to store fixed transactions across script executions.




