# SplitMate

A desktop app for splitting bills with friends, roommates, family and travel groups.
Record who paid for what, see who owes whom, and settle up with the fewest payments.

Built with **Java 17, Swing, SQLite (JDBC), Maven and JUnit 5**. Interface in **Vietnamese and English**, amounts in **Vietnamese dong (₫)**.

![Dashboard](docs/screenshots/03-dashboard.png)

## Features

- Local accounts with hashed passwords; each account has its own private groups
- Groups (trip, home, family, other) with named members
- Expenses with a payer, a category and any subset of members
- Four ways to split: equally, exact amounts, percentages, shares
- Balances that are always recalculated from the saved data, so they are never stale
- "Settle up" suggestions that clear a group in at most *members − 1* payments
- Repayment history, activity feed, search and filters
- Reports by category, month and member, with CSV export
- Light and dark theme, backup and restore

| Expenses | Balances and suggested payments |
|---|---|
| ![Expenses](docs/screenshots/05-expenses.png) | ![Balances](docs/screenshots/06-balances.png) |

| Expense form with live split preview | Reports |
|---|---|
| ![Expense form](docs/screenshots/14-expense-edit.png) | ![Reports](docs/screenshots/11-reports.png) |

## Requirements

- **JDK 17 or newer** (`java -version` to check)
- Internet access the first time you build (Maven downloads itself and the libraries)

You do **not** need to install Maven. The project includes the Maven Wrapper (`mvnw.cmd`).

## Run, test, package

Run the app:

```bash
.\mvnw.cmd exec:java
```

Run the automated tests:

```bash
.\mvnw.cmd test
```

Also run the test that clicks through the real window (it opens the app for about 10 seconds):

```bash
.\mvnw.cmd test "-Dui.tests=true"
```

Build one runnable jar (includes all libraries):

```bash
.\mvnw.cmd package
```

Start that jar:

```bash
java -jar target\splitmate-1.0.0.jar
```

On macOS or Linux use `./mvnw` instead of `.\mvnw.cmd`.

## First steps in the app

1. Click **Tạo tài khoản mới / Create a new account** and register.
2. Create a group. You are added as its first member.
3. Add the other members on the **Members** tab.
4. Click **Add expense**: enter the amount, who paid, and who shares it.
5. Open **Balances** to see who owes whom, and record payments as people pay back.

The language can be changed on the login screen or under **Settings**.

## Where is my data?

Everything is saved automatically in one SQLite file:

```
C:\Users\<you>\.splitmate\splitmate.db
```

Use **Settings → Back up to a file** to copy it somewhere safe. To use a different file (for example for a demo), start the app with `-Dsplitmate.db=path\to\file.db`.

## How the code is organised

```
src/main/java/com/splitmate/
├─ App.java, AppContext.java   start-up and wiring
├─ ui/                         Presentation layer: Swing windows, panels, dialogs
├─ service/                    Business layer: rules, validation, calculations
│  ├─ split/                   the four split methods (Strategy pattern)
│  └─ settle/                  debt simplification
├─ repository/                 Data access layer: interfaces
│  └─ sqlite/                  ...and their JDBC implementations
├─ model/, dto/                plain data classes
├─ security/                   password hashing, session
└─ util/, exception/           helpers
src/main/resources/
├─ db/schema.sql               Database layer: table definitions
└─ i18n/                       Vietnamese and English texts
```

Each layer only talks to the one below it. The business layer contains no Swing code, which is why it can be tested without opening a window.

More detail is in [`docs/`](docs):

| Document | Contents |
|---|---|
| [SRS.md](docs/SRS.md) | Requirements, feature list, business rules, edge cases |
| [ERD.md](docs/ERD.md) | Entity relationship diagram and database schema |
| [class-diagram.md](docs/class-diagram.md) | Classes, interfaces and how they depend on each other |
| [navigation.md](docs/navigation.md) | Map of the screens |
| [test-plan.md](docs/test-plan.md) | Testing strategy and acceptance checklist |

## Two ideas worth knowing

**Money is never a decimal number.** Amounts are whole dong stored in a `long`. When 100.000 ₫ is split three ways, each person first gets 33.333 ₫ and the 1 ₫ left over goes to the first person, so the parts always add up exactly (`service/split/LargestRemainder.java`).

**Balances are never stored.** A member's balance is calculated each time as *paid − own shares + repayments sent − repayments received*. Editing or deleting an expense therefore updates every balance automatically (`service/BalanceService.java`).

## Security notes

- Passwords are stored as salted PBKDF2-HMAC-SHA256 hashes, never as text.
- All SQL uses `PreparedStatement`, so typed text cannot change a query.
- One account cannot read or change another account's groups.
- After 5 wrong passwords, login pauses for 30 seconds.
- **Limit:** the database file itself is not encrypted. Anyone who can open your Windows account can copy the file and read the expenses in it (but not the passwords).

## Credits

Functionally inspired by expense-sharing apps such as Splitwise. Name, icon, layout and code are original.
Libraries: [sqlite-jdbc](https://github.com/xerial/sqlite-jdbc), [FlatLaf](https://www.formdev.com/flatlaf/), [JFreeChart](https://www.jfree.org/jfreechart/), [JUnit 5](https://junit.org/junit5/).
