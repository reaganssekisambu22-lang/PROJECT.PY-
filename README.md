# PROJECT.PY-
import sys

# --- Constants for Formatting ---
CURRENCY = "UGX"

# --- Architectural Principle: Encapsulated Function for Single Expense Handling ---
def log_transaction(budget: float, current_spent: float, transactions: list) -> float:
    """
    Handles the input and processing of a single financial transaction.

    Args:
        budget (float): The initial spending ceiling.
        current_spent (float): The current cumulative total of all expenses.
        transactions (list): The list where all transaction records are stored.

    Returns:
        float: The updated cumulative spending total after the transaction.
    """
    
    # 1. Get Transaction Description
    # Note: Using flush=True for print statements to ensure VSC terminal displays output immediately.
    description = input("Enter transaction description (e.g., 'Rolex', 'Data Bundle'): ").strip()
    if not description:
        print("Description cannot be empty. Transaction skipped.", flush=True)
        return current_spent

    # 2. Get Transaction Value with Error Handling
    while True:
        value_input = input(f"Enter transaction amount in {CURRENCY}: ").strip()
        
        # Check for quit/finish signal
        if value_input.lower() in ('q', 'quit', 'done'):
            return -1 # Special signal to indicate session end

        try:
            value = float(value_input)
            if value <= 0:
                print("⚠️ Expense must be a positive numerical value. Please try again.", flush=True)
            else:
                break # Valid input received
        except ValueError:
            print("⚠️ Invalid input. Please enter a valid number for the transaction amount.", flush=True)
    
    # 3. Process Transaction
    new_spent = current_spent + value
    
    # Store the transaction record
    transactions.append({
        'description': description,
        'value': value,
        'cumulative_spent': new_spent
    })
    
    # 4. Real-Time Fiscal Oversight Check
    print(f"\n--- Update: Total spent is now {CURRENCY} {new_spent:,.2f} ---", flush=True)
    if new_spent > budget:
        # Immediate clear warning notification
        deficit = new_spent - budget
        print(f"🚨 BUDGET BREACH ALERT! You are over budget by {CURRENCY} {deficit:,.2f}!", flush=True)
    elif new_spent == budget:
        print("🛑 BUDGET WARNING! You have reached your spending limit.", flush=True)
    else:
        remaining = budget - new_spent
        print(f"✅ Remaining balance: {CURRENCY} {remaining:,.2f}", flush=True)
    
    print("-" * 40, flush=True)
    return new_spent


def get_budget() -> float:
    """
    Prompts the user for and validates the initial spending budget.
    """
    print("\n" + "=" * 50, flush=True)
    print("      Welcome to Mukono Financial Assistant      ", flush=True)
    print("=" * 50, flush=True)
    print("Let's set your budget for the session.", flush=True)
    
    while True:
        try:
            budget_input = input(f"Enter your total budget in {CURRENCY}: ").strip()
            initial_budget = float(budget_input)
            
            if initial_budget < 0:
                print("⚠️ Budget must be a non-negative value. Please try again.", flush=True)
            else:
                return initial_budget
        except ValueError:
            print("⚠️ Invalid input. Please enter a numerical value for your budget.", flush=True)


def generate_report(initial_budget: float, total_spent: float, transactions: list):
    """
    Generates and prints the final comprehensive financial summary.
    """
    print("\n" + "=" * 50, flush=True)
    print("        FINANCIAL SESSION SUMMARY REPORT        ", flush=True)
    print("=" * 50, flush=True)

    # 1. Financial Position Calculations
    financial_position = initial_budget - total_spent
    
    # 2. Display Core Metrics
    print(f"Initial Budget: {CURRENCY} {initial_budget:,.2f}", flush=True)
    print(f"Total Expenses: {CURRENCY} {total_spent:,.2f}", flush=True)
    
    # 3. Remaining Balance or Deficit
    if financial_position >= 0:
        print(f"\n🎉 Remaining Balance: {CURRENCY} {financial_position:,.2f}", flush=True)
    else:
        deficit = abs(financial_position)
        print(f"\n💔 Deficit Amount: {CURRENCY} {deficit:,.2f} (You overspent)", flush=True)
        
    print("\n" + "-" * 20 + " Itemized Transaction Log " + "-" * 20, flush=True)
    
    # 4. Itemized Transaction List
    if not transactions:
        print("No transactions were recorded during this session.", flush=True)
    else:
        # Calculate padding for aligned output
        max_desc_len = max(len(t['description']) for t in transactions)
        
        # Ensure minimum width for aesthetics if descriptions are very short
        max_desc_len = max(max_desc_len, 11) 
        
        # Print header
        print(f"| {'Description':<{max_desc_len}} | {'Amount':>15} |", flush=True)
        print("-" * (max_desc_len + 21), flush=True)
        
        # Print transactions
        for t in transactions:
            print(f"| {t['description']:<{max_desc_len}} | {CURRENCY} {t['value']:>10,.2f} |", flush=True)
            
    print("-" * 50, flush=True)
    print("Thank you for using the Financial Assistant. Stay financially fit!", flush=True)
    print("=" * 50, flush=True)


def main():
    """
    Main execution logic for the financial assistant utility.
    """
    # Initialize state variables
    transactions = []
    total_spent = 0.0
    MIN_TRANSACTIONS = 5 # Minimum required transactions for testing
    
    # 1. Budget Initialization
    initial_budget = get_budget()
    
    print("\n" + "=" * 50, flush=True)
    print(f"BUDGET SET: {CURRENCY} {initial_budget:,.2f}", flush=True)
    print("Start logging transactions. Type 'quit' or 'done' for the amount to finish.", flush=True)
    print(f"({MIN_TRANSACTIONS} transactions minimum recommended for the assignment.)", flush=True)
    print("=" * 50, flush=True)

    # 2. Transactional Logging Loop (Minimum 5 transactions requirement is handled by warning)
    transaction_count = 0
    while True:
        print(f"\n--- TRANSACTION {transaction_count + 1} ---", flush=True)
        
        # Call the reusable function for separation of concerns
        new_total_spent = log_transaction(initial_budget, total_spent, transactions)
        
        # Check if the function returned the exit signal (-1)
        if new_total_spent == -1:
            break
        
        total_spent = new_total_spent
        transaction_count += 1
        
        # Gentle reminder if minimum transactions haven't been met
        if transaction_count < MIN_TRANSACTIONS:
            print(f"Note: You still need {MIN_TRANSACTIONS - transaction_count} more transactions to meet the minimum test requirement.", flush=True)


    # Final Check before reporting
    if transaction_count < MIN_TRANSACTIONS:
        print("\n" + "=" * 50, flush=True)
        print(f"Warning: Only {transaction_count} transactions recorded. Minimum requirement was {MIN_TRANSACTIONS}.", flush=True)
        print("Proceeding to report generation...", flush=True)
        print("=" * 50, flush=True)
    
    # 3. Final Financial Summary
    generate_report(initial_budget, total_spent, transactions)


if __name__ == "__main__":
    try:
        main()
    except KeyboardInterrupt:
        # Allows user to gracefully exit with Ctrl+C
        print("\n\nProgram interrupted by user. Exiting gracefully.", flush=True)
        sys.exit(0)



        ELECTRICITY_BILL
        electricity_bill():
# Prompt user for units and ensure input is a valid positive number
while True:
try:
units = int(input("Enter the number of units consumed: "))
if units < 0:
print("Units consumed cannot be negative. Please enter a positive value.")
continue
break
except ValueError:
print("Invalid input. Please enter a whole number for units.")

# --- Rate Structure ---
RATE_FIRST_100 = 500 # UGX per unit for the first 100
RATE_ADDITIONAL = 750 # UGX per unit for units above 100
SURCHARGE_THRESHOLD = 100000
SURCHARGE_RATE = 0.05 # 5%

bill_amount = 0

if units <= 100:
# Scenario 1: 100 units or less
bill_amount = units * RATE_FIRST_100

else:
# Scenario 2: More than 100 units

# 1. Cost of the first 100 units
cost_first_100 = 100 * RATE_FIRST_100

# 2. Cost of additional units
additional_units = units - 100
cost_additional = additional_units * RATE_ADDITIONAL

# 3. Total bill before surcharge
bill_amount = cost_first_100 + cost_additional

# --- Surcharge Calculation ---
surcharge = 0
if bill_amount > SURCHARGE_THRESHOLD:
surcharge = bill_amount * SURCHARGE_RATE
print(f"\nBill exceeds {SURCHARGE_THRESHOLD:,.0f} UGX. Applying a 5% surcharge of {surcharge:,.2f} UGX.")

final_bill = bill_amount + surcharge

# --- Display Final Bill Amount ---
print("\n--- Electricity Bill Summary ---")
print(f"Units Consumed: {units}")
print(f"Bill Amount (pre-surcharge): {bill_amount:,.2f} UGX")
print(f"Surcharge Applied: {surcharge:,.2f} UGX")
print(f"**Final Bill Amount: {final_bill:,.2f} UGX**")

# Run the program for part (a)
# calculate_electricity_bill()
def calculate_goal_status(P, R, T, Goal):
"""
Calculates simple interest, total amount, and checks goal status.

P: Principal amount
R: Interest rate per year (as a percentage)
T: Time in years
Goal: Savings goal amount
"""

# Simple Interest Formula: SI = (P * R * T) / 100
simple_interest = (P * R * T) / 100

# Total Amount Formula: Total Amount = P + Simple Interest
total_amount = P + simple_interest

status_message = ""
if total_amount >= Goal:
# Goal was met or exceeded
status_message = f"GOAL MET! The total amount is {total_amount:,.2f} UGX, which meets or exceeds the goal."
else:
# Goal was not met
needed = Goal - total_amount
status_message = f"GOAL NOT MET. You need an additional {needed:,.2f} UGX to reach your goal of {Goal:,.2f} UGX."

# Returns: simple interest, total amount, status message
return simple_interest, total_amount, status_message
def simple_interest_calculator():

# A. Prompt for Principal (P) - Must be positive (P > 0)
while True:
try:
P = float(input("A. Enter the Principal amount (P): "))
if P > 0:
break
print("Invalid: Principal must be a positive number.")
except ValueError:
print("Invalid input. Please enter a numerical value.")

# B. Prompt for Rate (R) - Must be between 1 and 100
while True:
try:
R = float(input("B. Enter the annual interest Rate (R, 1 to 100%): "))
if 1 <= R <= 100:
break
print("Invalid: Rate must be between 1 and 100.")
except ValueError:
print("Invalid input. Please enter a numerical value.")

# C. Prompt for Time (T) - Must be positive integer
while True:
try:
T = int(input("C. Enter the Time in years (T): "))
if T > 0:
break
print("Invalid: Time must be a positive integer.")
except ValueError:
print("Invalid input. Please enter a whole number.")

# D. Prompt for Savings Goal (Goal)
while True:
try
