# Library---Management---System-

include <iostream>
#include <vector>
#include <string>
#include <algorithm>
#include <iomanip>
#include <ctime>
#include <sstream>
#include <limits>

using namespace std;

// ─────────────────────────────────────────────
//  Utility
// ─────────────────────────────────────────────
string currentDate() {
    time_t now = time(nullptr);
    tm* t = localtime(&now);
    ostringstream oss;
    oss << (t->tm_year + 1900) << "-"
        << setw(2) << setfill('0') << (t->tm_mon + 1) << "-"
        << setw(2) << setfill('0') << t->tm_mday;
    return oss.str();
}

string toLower(const string& s) {
    string out = s;
    transform(out.begin(), out.end(), out.begin(), ::tolower);
    return out;
}

// ─────────────────────────────────────────────
//  Book Class
// ─────────────────────────────────────────────
class Book {
public:
    int     id;
    string  title;
    string  author;
    string  isbn;
    int     totalCopies;
    int     availableCopies;

    Book(int id, const string& title, const string& author,
         const string& isbn, int copies)
        : id(id), title(title), author(author),
          isbn(isbn), totalCopies(copies), availableCopies(copies) {}

    void display() const {
        cout << left
             << setw(5)  << id
             << setw(28) << title
             << setw(22) << author
             << setw(16) << isbn
             << setw(8)  << totalCopies
             << availableCopies << "\n";
    }
};

// ─────────────────────────────────────────────
//  Member Class
// ─────────────────────────────────────────────
class Member {
public:
    int    id;
    string name;
    string email;
    string phone;
    int    booksBorrowed;

    Member(int id, const string& name,
           const string& email, const string& phone)
        : id(id), name(name), email(email),
          phone(phone), booksBorrowed(0) {}

    void display() const {
        cout << left
             << setw(5)  << id
             << setw(22) << name
             << setw(28) << email
             << setw(16) << phone
             << booksBorrowed << "\n";
    }
};

// ─────────────────────────────────────────────
//  BorrowRecord Class
// ─────────────────────────────────────────────
class BorrowRecord {
public:
    int    recordId;
    int    memberId;
    int    bookId;
    string memberName;
    string bookTitle;
    string issueDate;
    string returnDate;   // empty if not yet returned
    bool   isReturned;

    BorrowRecord(int rid, int mid, int bid,
                 const string& mname, const string& btitle,
                 const string& idate)
        : recordId(rid), memberId(mid), bookId(bid),
          memberName(mname), bookTitle(btitle),
          issueDate(idate), returnDate(""), isReturned(false) {}

    void display() const {
        cout << left
             << setw(6)  << recordId
             << setw(6)  << memberId
             << setw(22) << memberName
             << setw(6)  << bookId
             << setw(28) << bookTitle
             << setw(13) << issueDate
             << setw(13) << (isReturned ? returnDate : "Not returned")
             << (isReturned ? "Returned" : "Active") << "\n";
    }
};

// ─────────────────────────────────────────────
//  Library Class
// ─────────────────────────────────────────────
class Library {
    vector<Book>         books;
    vector<Member>       members;
    vector<BorrowRecord> records;

    int nextBookId   = 1;
    int nextMemberId = 1;
    int nextRecordId = 1;

    // ── helpers ──────────────────────────────
    Book* findBook(int id) {
        for (auto& b : books)
            if (b.id == id) return &b;
        return nullptr;
    }

    Member* findMember(int id) {
        for (auto& m : members)
            if (m.id == id) return &m;
        return nullptr;
    }

    void printLine(char c = '-', int n = 100) const {
        cout << string(n, c) << "\n";
    }

    // ── display helpers ───────────────────────
    void printBookHeader() const {
        printLine();
        cout << left
             << setw(5)  << "ID"
             << setw(28) << "Title"
             << setw(22) << "Author"
             << setw(16) << "ISBN"
             << setw(8)  << "Total"
             << "Available\n";
        printLine();
    }

    void printMemberHeader() const {
        printLine();
        cout << left
             << setw(5)  << "ID"
             << setw(22) << "Name"
             << setw(28) << "Email"
             << setw(16) << "Phone"
             << "Borrowed\n";
        printLine();
    }

    void printRecordHeader() const {
        printLine('=', 110);
        cout << left
             << setw(6)  << "RecID"
             << setw(6)  << "MID"
             << setw(22) << "Member"
             << setw(6)  << "BID"
             << setw(28) << "Book"
             << setw(13) << "Issue Date"
             << setw(13) << "Return Date"
             << "Status\n";
        printLine('=', 110);
    }

public:
    // ══════════════════════════════════════════
    //  Book Operations
    // ══════════════════════════════════════════
    void addBook(const string& title, const string& author,
                 const string& isbn, int copies) {
        books.emplace_back(nextBookId++, title, author, isbn, copies);
        cout << "\n  ✔  Book added successfully! ID = " << (nextBookId - 1) << "\n";
    }

    void displayAllBooks() const {
        if (books.empty()) { cout << "\n  No books in the library.\n"; return; }
        cout << "\n  ══ ALL BOOKS ══\n";
        const_cast<Library*>(this)->printBookHeader();
        for (const auto& b : books) b.display();
        printLine();
    }

    // ── Search by title ───────────────────────
    void searchByTitle(const string& keyword) const {
        string kw = toLower(keyword);
        bool found = false;
        cout << "\n  ══ Search Results (Title: \"" << keyword << "\") ══\n";
        const_cast<Library*>(this)->printBookHeader();
        for (const auto& b : books) {
            if (toLower(b.title).find(kw) != string::npos) {
                b.display();
                found = true;
            }
        }
        if (!found) cout << "  No matching books found.\n";
        printLine();
    }

    // ── Search by author ──────────────────────
    void searchByAuthor(const string& keyword) const {
        string kw = toLower(keyword);
        bool found = false;
        cout << "\n  ══ Search Results (Author: \"" << keyword << "\") ══\n";
        const_cast<Library*>(this)->printBookHeader();
        for (const auto& b : books) {
            if (toLower(b.author).find(kw) != string::npos) {
                b.display();
                found = true;
            }
        }
        if (!found) cout << "  No matching books found.\n";
        printLine();
    }

    // ══════════════════════════════════════════
    //  Member Operations
    // ══════════════════════════════════════════
    void addMember(const string& name, const string& email,
                   const string& phone) {
        members.emplace_back(nextMemberId++, name, email, phone);
        cout << "\n  ✔  Member added successfully! ID = " << (nextMemberId - 1) << "\n";
    }

    void displayAllMembers() const {
        if (members.empty()) { cout << "\n  No registered members.\n"; return; }
        cout << "\n  ══ ALL MEMBERS ══\n";
        const_cast<Library*>(this)->printMemberHeader();
        for (const auto& m : members) m.display();
        printLine();
    }

    // ══════════════════════════════════════════
    //  Issue Book
    // ══════════════════════════════════════════
    void issueBook(int memberId, int bookId) {
        Member* m = findMember(memberId);
        if (!m) { cout << "\n  ✘  Member ID " << memberId << " not found.\n"; return; }

        Book* b = findBook(bookId);
        if (!b) { cout << "\n  ✘  Book ID " << bookId << " not found.\n"; return; }

        if (b->availableCopies <= 0) {
            cout << "\n  ✘  No copies of \"" << b->title << "\" are currently available.\n";
            return;
        }

        b->availableCopies--;
        m->booksBorrowed++;

        records.emplace_back(nextRecordId++, memberId, bookId,
                             m->name, b->title, currentDate());

        cout << "\n  ✔  Book issued successfully!\n"
             << "     Member : " << m->name << "\n"
             << "     Book   : " << b->title << "\n"
             << "     Date   : " << currentDate() << "\n"
             << "     Record#: " << (nextRecordId - 1) << "\n";
    }

    // ══════════════════════════════════════════
    //  Return Book
    // ══════════════════════════════════════════
    void returnBook(int recordId) {
        for (auto& rec : records) {
            if (rec.recordId == recordId && !rec.isReturned) {
                rec.isReturned  = true;
                rec.returnDate  = currentDate();

                Book*   b = findBook(rec.bookId);
                Member* m = findMember(rec.memberId);

                if (b) b->availableCopies++;
                if (m && m->booksBorrowed > 0) m->booksBorrowed--;

                cout << "\n  ✔  Book returned successfully!\n"
                     << "     Member : " << rec.memberName << "\n"
                     << "     Book   : " << rec.bookTitle  << "\n"
                     << "     Date   : " << currentDate()  << "\n";
                return;
            }
        }
        cout << "\n  ✘  Active borrow record #" << recordId << " not found.\n";
    }

    // ══════════════════════════════════════════
    //  Display Records
    // ══════════════════════════════════════════
    void displayAllRecords() const {
        if (records.empty()) { cout << "\n  No borrow records yet.\n"; return; }
        cout << "\n  ══ BORROW RECORDS ══\n";
        const_cast<Library*>(this)->printRecordHeader();
        for (const auto& r : records) r.display();
        printLine('=', 110);
    }

    void displayActiveRecords() const {
        cout << "\n  ══ ACTIVE BORROWS ══\n";
        const_cast<Library*>(this)->printRecordHeader();
        bool any = false;
        for (const auto& r : records) {
            if (!r.isReturned) { r.display(); any = true; }
        }
        if (!any) cout << "  No active borrows.\n";
        printLine('=', 110);
    }

    // ══════════════════════════════════════════
    //  Seed demo data
    // ══════════════════════════════════════════
    void seedData() {
        addBook("The Great Gatsby",         "F. Scott Fitzgerald", "978-0743273565", 3);
        addBook("To Kill a Mockingbird",    "Harper Lee",          "978-0061935466", 2);
        addBook("1984",                     "George Orwell",       "978-0451524935", 4);
        addBook("Clean Code",               "Robert C. Martin",    "978-0132350884", 2);
        addBook("The Pragmatic Programmer", "Andrew Hunt",         "978-0201616224", 1);

        addMember("Alice Johnson", "alice@example.com",  "555-1001");
        addMember("Bob Smith",     "bob@example.com",    "555-1002");
        addMember("Carol White",   "carol@example.com",  "555-1003");

        cout << "\n  ── Demo data loaded ──\n";
    }
};

// ─────────────────────────────────────────────
//  Input helpers
// ─────────────────────────────────────────────
int readInt(const string& prompt) {
    int v;
    while (true) {
        cout << prompt;
        if (cin >> v) { cin.ignore(); return v; }
        cout << "  Invalid input. Please enter a number.\n";
        cin.clear();
        cin.ignore(numeric_limits<streamsize>::max(), '\n');
    }
}

string readLine(const string& prompt) {
    string s;
    cout << prompt;
    getline(cin, s);
    return s;
}

// ─────────────────────────────────────────────
//  Menus
// ─────────────────────────────────────────────
void printMainMenu() {
    cout << R"(
  ╔══════════════════════════════════════╗
  ║       LIBRARY MANAGEMENT SYSTEM   ║
  ╠══════════════════════════════════════╣
  ║  1. Book Management                 ║
  ║  2. Member Management               ║
  ║  3. Issue a Book                    ║
  ║  4. Return a Book                   ║
  ║  5. Borrow Records                  ║
  ║  0. Exit                            ║
  ╚══════════════════════════════════════╝
)";
}

void printBookMenu() {
    cout << R"(
  ┌─────────────────────────────┐
  │    BOOK MANAGEMENT          │
  ├─────────────────────────────┤
  │  1. Add Book                │
  │  2. View All Books          │
  │  3. Search by Title         │
  │  4. Search by Author        │
  │  0. Back                    │
  └─────────────────────────────┘
)";
}

void printMemberMenu() {
    cout << R"(
  ┌─────────────────────────────┐
  │    MEMBER MANAGEMENT        │
  ├─────────────────────────────┤
  │  1. Add Member              │
  │  2. View All Members        │
  │  0. Back                    │
  └─────────────────────────────┘
)";
}

void printRecordsMenu() {
    cout << R"(
  ┌─────────────────────────────┐
  │    BORROW RECORDS           │
  ├─────────────────────────────┤
  │  1. All Records             │
  │  2. Active (Not Returned)   │
  │  0. Back                    │
  └─────────────────────────────┘
)";
}

// ─────────────────────────────────────────────
//  main
// ─────────────────────────────────────────────
int main() {
    Library lib;

    cout << "\n  Load demo data? (y/n): ";
    char ch; cin >> ch; cin.ignore();
    if (ch == 'y' || ch == 'Y') lib.seedData();

    int choice;
    do {
        printMainMenu();
        choice = readInt("  Enter choice: ");

        switch (choice) {

        // ── Book Management ───────────────────
        case 1: {
            int bc;
            do {
                printBookMenu();
                bc = readInt("  Enter choice: ");
                switch (bc) {
                case 1: {
                    string title  = readLine("  Title  : ");
                    string author = readLine("  Author : ");
                    string isbn   = readLine("  ISBN   : ");
                    int    copies = readInt ("  Copies : ");
                    lib.addBook(title, author, isbn, copies);
                    break;
                }
                case 2: lib.displayAllBooks(); break;
                case 3: { string kw = readLine("  Keyword: "); lib.searchByTitle(kw);  break; }
                case 4: { string kw = readLine("  Keyword: "); lib.searchByAuthor(kw); break; }
                case 0: break;
                default: cout << "  Invalid option.\n";
                }
            } while (bc != 0);
            break;
        }

        // ── Member Management ─────────────────
        case 2: {
            int mc;
            do {
                printMemberMenu();
                mc = readInt("  Enter choice: ");
                switch (mc) {
                case 1: {
                    string name  = readLine("  Name  : ");
                    string email = readLine("  Email : ");
                    string phone = readLine("  Phone : ");
                    lib.addMember(name, email, phone);
                    break;
                }
                case 2: lib.displayAllMembers(); break;
                case 0: break;
                default: cout << "  Invalid option.\n";
                }
            } while (mc != 0);
            break;
        }

        // ── Issue Book ────────────────────────
        case 3: {
            lib.displayAllMembers();
            lib.displayAllBooks();
            int mid = readInt("  Member ID : ");
            int bid = readInt("  Book ID   : ");
            lib.issueBook(mid, bid);
            break;
        }

        // ── Return Book ───────────────────────
        case 4: {
            lib.displayActiveRecords();
            int rid = readInt("  Borrow Record ID to return: ");
            lib.returnBook(rid);
            break;
        }

        // ── Records ───────────────────────────
        case 5: {
            int rc;
            do {
                printRecordsMenu();
                rc = readInt("  Enter choice: ");
                switch (rc) {
                case 1: lib.displayAllRecords();    break;
                case 2: lib.displayActiveRecords(); break;
                case 0: break;
                default: cout << "  Invalid option.\n";
                }
            } while (rc != 0);
            break;
        }

        case 0:
            cout << "\n  Thank you for using the Library Management System. Goodbye!\n\n";
            break;

        default:
            cout << "  Invalid option. Please try again.\n";
        }

    } while (choice != 0);

    return 0;
}
