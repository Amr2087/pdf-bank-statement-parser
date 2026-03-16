# PDF Bank Statement Parser

[![Downloads](https://static.pepy.tech/badge/pdf-bank-statement-parser)](https://pepy.tech/project/pdf-bank-statement-parser)

Command-line tool for robustly converting PDF bank statements into clean usable CSV. Currently only works for statements from First National Bank (FNB) South Africa (please let me know if you want me to expand the scope).

## Install

```bash
pip install pdf-bank-statement-parser
```

## Example Usage

### Command-line interface

```shell
# parse a single PDF bank statement #
parse-bank-statement-pdf \
  --input_filepath 'bank_statements/2024_03_27 - 2024_06_28.pdf' \
  --output_path 'bank_statements/csv/2024_03_27 - 2024_06_28.csv'

# parse all PDF bank statements in a given directory #
parse-bank-statement-pdf \
  --input_dir 'bank_statements/' \
  --output_path 'bank_statements/csv/' \
  --csv_sep_char ';'
```

### Python API

You can also use this library directly from Python code:

```python
from pdf_bank_statement_parser import (
    extract_transactions_from_fnb_pdf_statement,
    write_transactions_to_csv,
    Transaction,
)

# Parse a single PDF and get a list of Transaction objects
transactions: list[Transaction] = extract_transactions_from_fnb_pdf_statement(
    path_to_pdf_file="bank_statements/2024_03_27 - 2024_06_28.pdf",
    verbose=True,
)

# Each Transaction is a named tuple with these fields:
#   date        – datetime.date
#   description – str
#   amount      – Decimal (negative for debits, positive for credits)
#   balance     – Decimal (running balance after the transaction)
#   bank_fee    – Decimal (associated bank fee, 0 if none)
for transaction in transactions:
    print(transaction.date, transaction.description, transaction.amount)

# Write the parsed transactions to a CSV file
write_transactions_to_csv(
    transactions=transactions,
    output_filepath="bank_statements/csv/2024_03_27 - 2024_06_28.csv",
    csv_sep_char=",",  # optional, default is ","
    verbose=True,       # optional, default is True
)
```

The only format available from FNB for downloading historical bank statements is PDF, which is a useless format for any kind of downstream data task other than reading.

This tool uses [pypdfium2](https://github.com/pypdfium2-team/pypdfium2) for text extraction from PDF and native python for everything else. Transactions are extracted using RegEx.

The parsed results are verified as follows:

1. It is checked (for every transaction extracted) that the balance amount is the sum of the previous balance and the transaction amount.

2. It is checked that the opening balance reported on the statement plus the sum of extracted transactions is equal to the closing balance reported on the statement.

## Security

This library has **no backdoors, no telemetry, and no network access** of any kind. It only reads local PDF files from disk and writes CSV files to disk. Here is a brief summary of what the code does and does not do:

| Concern | Status |
|---|---|
| Network requests | ✅ None — no `requests`, `urllib`, `socket`, or any other networking library is used |
| Remote code execution | ✅ None — `eval()`, `exec()`, and `subprocess` are not used anywhere |
| Data collection / telemetry | ✅ None — the library never transmits any data |
| Third-party dependencies | ✅ Only one: [pypdfium2](https://github.com/pypdfium2-team/pypdfium2), a well-maintained open-source PDF text extraction library |
| File system access | ℹ️ Reads PDF files from paths you provide and writes CSV files to paths you provide — nothing else |

You can verify this yourself by reading the source code; the entire library is under 400 lines of Python.
