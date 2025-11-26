#Inventory System

import csv
import os
from datetime import datetime
import colorama
from colorama import Fore, Style
colorama.init(autoreset=True)  # para automatic nga ma-reset ang color sa console after each display

# para sa pag-define sa file paths ug constants
BASE_DIR = os.path.dirname(os.path.abspath(__file__))
INVENTORY_FILE = os.path.join(BASE_DIR, 'inventory.csv')
SALES_FILE = os.path.join(BASE_DIR, 'sales.csv')
LOW_STOCK_THRESHOLD = 5  # tighatag og alert kung ang stock moubos ana nga number

# para sa pag-load sa inventory gikan sa CSV file
def load_inventory():
    inventory = []
    if os.path.exists(INVENTORY_FILE):
        with open(INVENTORY_FILE, mode='r', newline='') as file:
            reader = csv.DictReader(file)
            for row in reader:
                row['Quantity'] = int(row['Quantity'])
                row['Price'] = float(row['Price'])
                inventory.append(row)
    return inventory

# para mag-save sa inventory didto sa CSV file
def save_inventory(inventory):
    with open(INVENTORY_FILE, mode='w', newline='') as file:
        fieldnames = ['Name', 'Category', 'Quantity', 'Price']
        writer = csv.DictWriter(file, fieldnames=fieldnames)
        writer.writeheader()
        for item in inventory:
            writer.writerow(item)

# para mag-add ug bag-ong item sa inventory
def add_item(inventory):
    name = input("Enter item name: ").strip()
    if not name:
        print(Fore.RED + "Error: Item name can't be empty.")
        return
    category = input("Enter item category (Food, Drink, Material): ").strip()
    if not category:
        print(Fore.RED + "Error: Category can't be empty.")
        return
    try:
        quantity = int(input("Enter quantity: "))
        if quantity < 0:
            print(Fore.RED + "Error: Quantity can't be negative.")
            return
    except ValueError:
        print(Fore.RED + "Error: Quantity must be a number.")
        return
    try:
        price = float(input("Enter price: "))
        if price < 0:
            print(Fore.RED + "Error: Price can't be negative.")
            return
    except ValueError:
        print(Fore.RED + "Error: Price must be a number.")
        return
    
    # mag-check kung existing na ang item
    for item in inventory:
        if item['Name'].lower() == name.lower():
            print(Fore.RED + "Error: Item already exists.")
            return
    
    inventory.append({'Name': name, 'Category': category, 'Quantity': quantity, 'Price': price})
    save_inventory(inventory)
    print(Fore.GREEN + "Item added successfully!")

# para mag-view sa inventory
def view_inventory(inventory):
    if not inventory:
        print(Fore.YELLOW + "No items in inventory.")
        return
    
    print("\n" + "="*55)
    print(Fore.CYAN + f"{'Name':<20} {'Category':<15} {'Quantity':<10} {'Price':<10}")
    print("="*55)
    for item in inventory:
        print(f"{item['Name']:<20} {item['Category']:<15} {item['Quantity']:<10} ₱{item['Price']:<10.2f}")
    print("="*55)
    
    # para ni siya mag-check sa low stock nga item
    low_stock_items = [item for item in inventory if item['Quantity'] < LOW_STOCK_THRESHOLD]
    if low_stock_items:
        print(Fore.MAGENTA + "\nLow Stock Alert:")
        for item in low_stock_items:
            print(Fore.MAGENTA + f"- {item['Name']}: {item['Quantity']} remaining")

# para mag-update sa item
def update_item(inventory):
    name = input("Enter the name of the item to update: ").strip()
    for item in inventory:
        if item['Name'].lower() == name.lower():
            print(f"Current details: Name: {item['Name']}, Category: {item['Category']}, Quantity: {item['Quantity']}, Price: {item['Price']}")
            choice = input("What to update? (name/category/quantity/price): ").strip().lower()
            if choice == 'name':
                new_name = input("Enter new name: ").strip()
                if not new_name:
                    print(Fore.RED + "Error: Name cannot be empty.")
                    return
                item['Name'] = new_name
            elif choice == 'category':
                new_category = input("Enter new category: ").strip()
                if not new_category:
                    print(Fore.RED + "Error: Category cannot be empty.")
                    return
                item['Category'] = new_category
            elif choice == 'quantity':
                try:
                    add_quantity = int(input("Enter quantity to add: "))
                    if add_quantity < 0:
                        print(Fore.RED + "Error: Quantity to add cannot be negative.")
                        return
                    item['Quantity'] += add_quantity
                except ValueError:
                    print(Fore.RED + "Error: Quantity must be a number.")
                    return
            elif choice == 'price':
                try:
                    new_price = float(input("Enter new price: "))
                    if new_price < 0:
                        print(Fore.RED + "Error: Price cannot be negative.")
                        return
                    item['Price'] = new_price
                except ValueError:
                    print(Fore.RED + "Error: Price must be a number.")
                    return
            else:
                print(Fore.RED + "Invalid choice.")
                return
            save_inventory(inventory)
            print(Fore.GREEN + "Item updated successfully!")
            return
    print(Fore.RED + "Item not found.")

# para mag-delete sa item
def delete_item(inventory):
    name = input("Enter the name of the item to delete: ").strip()
    for item in inventory:
        if item['Name'].lower() == name.lower():
            inventory.remove(item)
            save_inventory(inventory)
            print(Fore.GREEN + "Item deleted successfully!")
            return
    print(Fore.RED + "Item not found.")

# para mag-record sa nahalin nga item
def record_sale(inventory):
    name = input("Enter the name of the item sold: ").strip()
    for item in inventory:
        if item['Name'].lower() == name.lower():
            try:
                sold_quantity = int(input("Enter quantity sold: "))
                if sold_quantity <= 0:
                    print(Fore.RED + "Error: Quantity sold must be positive.")
                    return
                if sold_quantity > item['Quantity']:
                    print(Fore.RED + f"Error: Not enough stock. Only {item['Quantity']} remaining.")
                    return
                sales = sold_quantity * item['Price']
                item['Quantity'] -= sold_quantity
                save_inventory(inventory)
                # pag-log sa sales sa sales.csv
                with open(SALES_FILE, mode='a', newline='') as file:
                    writer = csv.writer(file)
                    writer.writerow([datetime.now().strftime('%Y-%m-%d %H:%M:%S'), item['Name'], sold_quantity, sales])
                print(Fore.GREEN + "Sale recorded successfully!")
                if item['Quantity'] < LOW_STOCK_THRESHOLD:
                    print(Fore.MAGENTA + f"Low stock alert: {item['Name']} has {item['Quantity']} remaining.")
                return
            except ValueError:
                print(Fore.RED + "Error: Quantity must be a number.")
                return
    print(Fore.RED + "Item not found.")

# para mag-generate sa report sa inventory ug sales
def generate_report(inventory):
    if not inventory:
        print(Fore.YELLOW + "No items in inventory.")
        return
    
    # para sa pag-load sa sales data gikan sa sales.csv
    sales = []
    if os.path.exists(SALES_FILE):
        with open(SALES_FILE, mode='r') as file:
            reader = csv.reader(file)
            for row in reader:
                sales.append({'date': row[0], 'name': row[1], 'qty': int(row[2]), 'sales': float(row[3])})
    
    # para sa pag-summarize sa sales data gikan sa sales log
    sales_summary = {}
    for s in sales:
        if s['name'] in sales_summary:
            sales_summary[s['name']]['qty'] += s['qty']
            sales_summary[s['name']]['sales'] += s['sales']
        else:
            sales_summary[s['name']] = {'qty': s['qty'], 'sales': s['sales']}
    
    # para makuha ang price sa item gikan sa inventory
    for name in sales_summary:
        for item in inventory:
            if item['Name'] == name:
                sales_summary[name]['price'] = item['Price']
                break
    
    total_daily_sales = sum(s['sales'] for s in sales)
    
    print("\n" + "="*55)
    print(Fore.CYAN + "INVENTORY REPORT")
    print("="*55)
    
    if sales_summary:
        print("\nSold Items:")
        print("-" * 55)
        print(Fore.CYAN + f"{'Item Name':<15} {'Qty Sold':<12} {'Price':<12} {'Sales':<12}")
        print("-" * 55)
        for name, data in sales_summary.items():
            print(f"{name:<15} {data['qty']:<12} ₱{data['price']:<11,.2f} ₱{data['sales']:<12,.2f}")
        print("-" * 55)
        print(f"Total Daily Sales:                        ₱{total_daily_sales:,.2f}")
        print("-" * 55)
        print(f"Report Generated: {datetime.now().strftime('%Y-%m-%d %I:%M %p')}")
    else:
        print("\nSold Items: None")
        print("-" * 55)
        print(f"Report Generated: {datetime.now().strftime('%Y-%m-%d %I:%M %p')}")

# para sa main menu ug pag-handle sa user input
def main():
    inventory = load_inventory()
    while True:
        print("="*55)
        print(Fore.CYAN + "Item Inventory System for PHINMA COC Canteen Stalls")
        print("="*55)
        print(Fore.YELLOW + "1. Add Item")
        print(Fore.YELLOW + "2. View Inventory")
        print(Fore.YELLOW + "3. Update Item")
        print(Fore.YELLOW + "4. Delete Item")
        print(Fore.YELLOW + "5. Record Sale")
        print(Fore.YELLOW + "6. Generate Report")
        print(Fore.RED + "7. Exit")
        print("="*55)
        choice = input("Enter your choice (1-7): ").strip()
        
        if choice == '1':
            add_item(inventory)
        elif choice == '2':
            view_inventory(inventory)
        elif choice == '3':
            update_item(inventory)
        elif choice == '4':
            delete_item(inventory)
        elif choice == '5':
            record_sale(inventory)
        elif choice == '6':
            generate_report(inventory)
        elif choice == '7':
            print(Fore.RED + "Exiting the system. Goodbye!")
            break
        else:
            print(Fore.RED + "Invalid choice. Please enter a number from 1 to 7.")

if __name__ == "__main__":
    main()
