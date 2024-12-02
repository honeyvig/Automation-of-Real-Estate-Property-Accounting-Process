# Automation-of-Real-Estate-Property-Accounting-Process
We are a real estate management company looking to automate a repetitive accounting process currently managed manually in Excel on a Mac. The final accounting product for each property consists of three files in OneDrive:

1. A PDF file with annual bank statements
2. A PDF file with annual receipts
3. An Excel file containing the actual bookkeeping records

Each entry in the Excel sheet correlates to a bank transaction and typically has a corresponding receipt, though there are exceptions. We need an automation solution that can:

• Automate Excel entries and cross-check each entry in the Excel sheet against transactions in the bank statement PDF and link receipts as needed.
• Verify each Excel entry has the required documentation and flag any discrepancies.
• Use AI-based solutions (e.g., ChatGPT, Copilot, or custom-built AI tools) to handle or suggest categorization and reconciliation tasks.
• Operate efficiently on a Mac environment with seamless integration to OneDrive for file management.

Project Requirements:

• Expertise in AI and automation solutions (preferably with experience in accounting or financial data processing).
• Knowledge of document processing for PDFs and Excel automation.
• Ability to create an intuitive workflow, making it simple for a non-technical user to manage and monitor the process.
• Experience working with OneDrive integration and compatibility with MacOS.

Ideal Skills:

• Familiarity with tools like ChatGPT, Microsoft Copilot, or custom AI models to handle repetitive tasks.
• Advanced proficiency in Excel and PDF document automation.
• Understanding of financial accounting or bookkeeping processes is a plus.
======================
 Python-based solution to automate the accounting process described. It integrates AI for categorization and reconciliation, PDF processing for bank statements and receipts, and Excel automation. The solution also ensures compatibility with MacOS and OneDrive.
Code Implementation
Step 1: Import Necessary Libraries

import os
import pandas as pd
import PyPDF2
from openpyxl import load_workbook
import pytesseract
from pdf2image import convert_from_path
from fuzzywuzzy import fuzz
import shutil
from onedrive_sdk import OneDriveClient
import openai

Step 2: Define AI Categorization and Reconciliation

def ai_categorize_and_reconcile(transaction_description):
    openai.api_key = "your_openai_api_key"
    prompt = f"Categorize the following transaction and suggest reconciliation details:\n\n{transaction_description}"
    response = openai.ChatCompletion.create(
        model="gpt-4",
        messages=[{"role": "user", "content": prompt}]
    )
    return response.choices[0].message['content']

Step 3: Process Bank Statement PDF

def extract_text_from_pdf(pdf_path):
    text = ""
    images = convert_from_path(pdf_path)
    for image in images:
        text += pytesseract.image_to_string(image)
    return text

Step 4: Match Excel Entries with Bank Statement and Receipts

def match_entries(excel_file, bank_statement_text, receipts_folder):
    wb = load_workbook(excel_file)
    sheet = wb.active

    for row in sheet.iter_rows(min_row=2, max_row=sheet.max_row, values_only=False):
        excel_transaction = row[1].value  # Adjust column index as needed
        if excel_transaction in bank_statement_text:
            row[2].value = "Matched"
        else:
            row[2].value = "Not Matched"

        receipt_found = False
        for receipt in os.listdir(receipts_folder):
            if fuzz.partial_ratio(excel_transaction, receipt) > 80:  # Adjust threshold as needed
                receipt_found = True
                row[3].value = receipt

        if not receipt_found:
            row[3].value = "Missing Receipt"

    wb.save(excel_file)

Step 5: Integrate OneDrive for File Management

def download_files_from_onedrive(folder_path, onedrive_client):
    items = onedrive_client.item(drive="me", id="root").children.request().get()
    for item in items:
        file_path = os.path.join(folder_path, item.name)
        with open(file_path, "wb") as file:
            file.write(onedrive_client.item(item.id).content.request().get().content)

Step 6: Main Function to Execute Workflow

def automate_accounting_process():
    # Download files
    onedrive_client = OneDriveClient("your_onedrive_credentials")
    download_files_from_onedrive("local_folder", onedrive_client)

    # Extract text from bank statement PDF
    bank_statement_text = extract_text_from_pdf("local_folder/annual_bank_statement.pdf")

    # Match entries and receipts
    match_entries(
        "local_folder/bookkeeping.xlsx",
        bank_statement_text,
        "local_folder/receipts"
    )

    print("Accounting process completed and discrepancies flagged.")

Instructions to Run

    Install the required libraries:

    pip install pandas PyPDF2 openpyxl pytesseract pdf2image fuzzywuzzy onedrive-sdk openai

    Replace placeholders like your_openai_api_key and your_onedrive_credentials with actual credentials.
    Run the script on a MacOS system with necessary dependencies (e.g., Tesseract OCR).

Notes

    This script assumes a basic folder structure in OneDrive (bookkeeping.xlsx, annual_bank_statement.pdf, and a receipts folder).
    Outputs are flagged in the Excel file with statuses for matches and missing receipts.
    Further customization can handle additional edge cases or user-specific workflows.

Enhancements

    Use a GUI for non-technical users.
    Extend AI's role in dynamic categorization using domain-specific training data.
    Add email notifications for flagged discrepancies.
