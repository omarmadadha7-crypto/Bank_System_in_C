# Bank Management System (C++)

> A console-based bank management system written in modern C++ (C++20). It manages clients, bank users with role-based permissions, deposits, withdrawals, money transfers with rollback, audit logs, and a currency exchange module.

![C++](https://img.shields.io/badge/C%2B%2B-20-00599C?logo=cplusplus&logoColor=white)
![Platform](https://img.shields.io/badge/platform-Windows-0078D6?logo=windows&logoColor=white)
![IDE](https://img.shields.io/badge/Visual%20Studio-2026-5C2D91?logo=visualstudio&logoColor=white)



## Overview

This project simulates the daily operations of a small bank from the point of view of a bank employee. An employee logs in, and what they can do depends on the permissions assigned to their account. The goal was to practice object-oriented design in C++ (inheritance, abstraction, encapsulation, static factory methods) and to build a real multi-screen application with file-based persistence, without using a database or external libraries.

## Features

**Client management**
- List, add, update, delete, and find clients by account number

**Transactions**
- Deposit and withdraw with balance validation
- View total balances across all clients
- Transfer money between accounts, with validation and an automatic **rollback** if a step fails
- Transfer log recording the date, both accounts, the amount, both balances after the transfer, and the user who performed it

**User management and security**
- Add, update, delete, find, and list system users
- **Permission system using bit flags**: each permission is a power of two, combined into one integer, so any mix of permissions can be stored in a single field (full access is `-1`)
- Login with a **3-attempt lockout**
- Login register showing who logged in and when (restricted by permission)

**Currency exchange**
- Browse 200+ currencies, find a currency by code or country, update exchange rates
- Currency calculator that converts between any two currencies using USD as the base

**Reliability**
- Transfer errors are written to `data/bank_errors.log` while the user sees a clean message
- Rejects non-positive amounts, insufficient balance, and transfers to the same account

## Design

The code is split into three layers, each in its own folder.

```
include/ + src/
├── Core/      Domain classes: clsPerson, clsBankClient, clsUser, clsCurrency, clsUserLog
├── Screen/    One class per screen (Client, User, Transactions, Currencies)
└── Lib/       Reusable helpers: clsString, clsDate, clsUtil, clsInputValidate, clsCheck
data/          Text files used as the database
```

```
InterfaceCommunication          (abstract: SendEmail, SendFax, SendSMS)
        ▲
    clsPerson                   (shared fields: name, email, phone)
     ▲      ▲
clsBankClient   clsUser
```

- **Abstraction:** `InterfaceCommunication` declares the contact methods, `clsPerson` implements them, and both clients and users inherit them.
- **Static factory methods:** objects are created through `Find()`, `GetAddNewClientObject()`, and similar methods, with an explicit mode (`EmptyMode`, `AddNewMode`, `UpdateMode`) so `Save()` knows whether to insert or update.
- **Screen-per-class UI:** every screen is its own class with a shared base (`clsScreen`) that handles headers and permission checks.

## Technical Decisions & Challenges

**Permissions as bit flags.**
Instead of a table of roles, each permission is a bit (`pListClients = 1`, `pAddNewClient = 2`, `pDeleteClient = 4`, and so on). A user's permissions are stored as a single integer, and `CheckAccessPermission` is a bitwise AND. This keeps the user record compact and makes any combination possible.

**Safe transfers without a database.**
A transfer debits the sender, then credits the receiver. If anything fails after the debit, the code credits the amount back to the sender (a compensating rollback) and logs the technical details, while the screen shows a clean message. Validation runs first, so most bad transfers never reach the debit step.

**Flat-file persistence.**
Data is stored in delimited text files (`#//#` separator) and loaded into vectors, then rewritten on save. It is simple and dependency-free, which suits a learning project, but it is not designed for concurrent access or large data.

## Getting Started

### Prerequisites

- Windows
- Visual Studio with the **Desktop development with C++** workload (the project uses the `v145` toolset and C++20, so use a recent Visual Studio)

### Run

```bash
git clone https://github.com/omarmadadha7-crypto/Bank_System_in_C.git
```

1. Open `Bank_System.slnx` in Visual Studio.
2. Choose **Debug** or **Release**, then press **F5**.
3. Run it from Visual Studio so the working directory is the project root, because the program reads the `data/` folder using relative paths.

### Demo login

| Username | Password | Permissions |
|---|---|---|
| `User1` | `1234` | Full access |

Sample clients are included in `data/ClientData.txt` (for example account `A101`).

## Project Structure

```
Bank_System/
├── Bank_System.cpp      Entry point (login loop)
├── Bank_System.slnx     Visual Studio solution
├── include/             Headers (Core, Screen, Lib)
├── src/                 Implementations (Core, Screen, Lib)
└── data/                Clients, users, currencies, and log files
```

## Known Limitations

This is a learning project, and I am honest about what it is not:

- **Not secure:** PINs and passwords are obfuscated with a simple Caesar-style character shift, which is not real encryption. A production system would use a salted hash such as Argon2 or bcrypt.
- **Windows only:** it uses MSVC-specific features (`__declspec(property)`, `system("cls")`) and backslash paths.
- **No automated tests** yet, and no protection against two instances writing the same files.

## Roadmap

- [ ] Replace the character shift with salted password hashing
- [ ] Move storage to SQLite
- [ ] Add unit tests (for example with Catch2 or GoogleTest) for transfers, permissions, and currency conversion
- [ ] Make it cross-platform with CMake and remove MSVC-specific code
- [ ] Use `double` consistently for money values instead of `float` in some places

## Author

**Omar Madadha** · [GitHub](https://github.com/omarmadadha7-crypto) · [LinkedIn](https://www.linkedin.com/in/omar-madadha-0731802b7/) · omarmadadha7@gmail.com
