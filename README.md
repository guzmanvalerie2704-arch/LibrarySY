# Town Library: Computerized Library System (Admin + Student)

- **Back-end:** Java 17+ REST API (no libraries, no database to install; data is saved in `backend/data/`)
- **Front-end:** PHP + HTML + CSS web interface that talks to the Java API

```
Browser  ->  PHP front-end (port 8000)  ->  Java API (port 8080)  ->  backend/data/*.tsv
```

## How people get in

The first page asks **"Who is signing in?"**

- **I'm a student:** goes to the student sign-in, which also has a link to *Create a student account*.
- **I'm a librarian:** goes to the librarian sign-in. There is no sign-up here; the librarian account is provided in advance.

An account can only sign in through its own door. A student can't sign in on the librarian page, and the other way round.

## Two kinds of users

| | Librarian (admin) | Student |
|---|---|---|
| Sign in | Provided account: **admin / admin123** | Create their own account, or demo: **maria / student123** |
| Books | add, edit, delete, search | search the catalog, see what's on the shelf |
| Members | add, edit, delete | their own profile only |
| Borrowing | issue books directly, return books | **request** a book, cancel pending requests |
| Requests | approve (issues the loan) or decline | see request status and the librarian's note |
| Loans and fines | all loans, overdue list, fines | their own books, due dates, overdue and fines |
| Account | change password | change password |

Students can't open librarian pages, and librarians use their own menu.

### How borrowing works
1. A student finds a book and clicks **Request** (or **Reserve** if no copy is on the shelf).
2. The librarian opens **Requests** and clicks **Approve and issue** (or **Decline**, with an optional reason).
3. Approving issues the loan (14 days by default). The student sees it under **My books** with the due date.
4. At the desk the librarian clicks **Return** in **Loans**. Late books are fined 5 per day.

Rules enforced: 3 books out per member, 5 pending requests per student, no duplicate copies, inactive members can't borrow, and a request can only be approved while a copy is on the shelf.

### Student accounts
Students create their own account on the sign-in page. If the librarian already added them as a member with the **same email**, the new account links to that member record. Otherwise a member record is created automatically. Passwords are stored hashed (PBKDF2), never as plain text.

## Requirements

- JDK 17 or newer (`javac -version`)
- PHP 7.4 or newer (XAMPP's PHP works: `C:\xampp\php\php.exe`)

## Run it

**Terminal 1: Java back-end**

```
cd backend
build.bat         (Mac/Linux: ./build.sh)
run.bat           (Mac/Linux: ./run.sh)
```

**Terminal 2: PHP front-end**

```
cd frontend
C:\xampp\php\php.exe -S localhost:8000     (or run.bat / ./run.sh if PHP is on your PATH)
```

Open **http://localhost:8000** and choose Student or Librarian.

**Upgrading from the first version?** Run `build.bat` again and restart the backend. Your books, members and loans are kept; the accounts file is created on first start.

## Settings

| What | Where |
|---|---|
| First librarian username / password | Environment variables `LIB_ADMIN_USER` / `LIB_ADMIN_PASS` **before the first start** (afterwards use *My account → Change password*) |
| Start over with the default accounts | Stop the backend and delete `backend/data/users.tsv` |
| Backend port / data folder | `run.bat 9000 C:\path\to\data`, then set `LIBRARY_API=http://127.0.0.1:9000/api` for PHP |
| Fine per day, max books, max pending, loan days | Constants at the top of `backend/src/library/Store.java`; rebuild afterwards |
| Library name, currency symbol | `frontend/config.php` |

## Project layout

```
backend/src/library/
  Main.java  Api.java (routes)  Store.java (rules, accounts, saving)
  Book.java  Member.java  Loan.java  User.java  Reservation.java  Json.java  ApiException.java
backend/data/        books, members, loans, users, reservations (.tsv, created on first run)
frontend/
  config.php  includes/ (lib.php, header.php, footer.php)  assets/style.css
  index.php (welcome: student or librarian)  login.php?as=student|librarian  register.php  logout.php  profile.php
  Librarian: dashboard.php  books.php  book_form.php  members.php  member_form.php
             requests.php  issue.php  loans.php
  Student:   home.php  catalog.php  my_requests.php
```

## Notes

- The Java API listens on `127.0.0.1` only and trusts the PHP site, which checks who is signed in and what they may open. Don't expose port 8080 to a network.
- Matching a new account to an existing member by email is convenient but means anyone who knows a member's email could claim that record. Fine for a school or small library; for stricter control, have the librarian create accounts.
- To move to MySQL later, only `Store.java` needs to change.
- Back up the `backend/data` folder to back up the library.

