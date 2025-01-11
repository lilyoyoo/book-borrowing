<!-- Login Section -->
<div id="login-section" class="section active">
    <h2>Login</h2>
    <div id="auth-section">
        <label for="username-login">Username</label>
        <input type="text" id="username-login" placeholder="Enter your username" required>
        
        <label for="password-login">Password</label>
        <input type="password" id="password-login" placeholder="Enter your password" required>
        
        <button onclick="login()">Login</button>
        <div id="auth-message"></div>
        <button onclick="showSection('register-section')">Register</button>
    </div>
</div>

<!-- Register Section -->
<div id="register-section" class="section">
    <h2>Register</h2>
    <div id="auth-section">
        <label for="username-register">Username</label>
        <input type="text" id="username-register" placeholder="Choose a username" required>
        
        <label for="password-register">Password</label>
        <input type="password" id="password-register" placeholder="Create a password" required>

        <label for="age">Age</label>
        <input type="number" id="age" placeholder="Enter your age" required>
        
        <label for="gender">Gender</label>
        <select id="gender" required>
            <option value="" disabled selected>Select your gender</option>
            <option value="male">Male</option>
            <option value="female">Female</option>
            <option value="other">Other</option>
        </select>

        <button onclick="register()">Register</button>
        <div id="auth-message"></div>
        <button onclick="showSection('login-section')">Back to Login</button>
    </div>
</div>

<!-- Book Borrowing Section -->
<div id="borrow-section" class="section">
    <h2>Book Borrowing</h2>
    <div id="auth-section">
        <h3>Available Books:</h3>
        <ul id="available-books">
            <!-- Books will be listed here -->
        </ul>

        <h3>Borrow a Book:</h3>
        <select id="book-select">
            <option value="" disabled selected>Select a book</option>
        </select>
        <button onclick="borrowBook()">Borrow</button>

        <h3>Borrowed Books:</h3>
        <ul id="borrowed-books">
            <!-- Borrowed books will be listed here -->
        </ul>

        <button onclick="logout()">Logout</button>
        
        <!-- New Excel Export Button -->
        <button onclick="exportToExcel()">Export Borrowed Books to Excel</button>
    </div>
</div>

<script>
    const users = JSON.parse(localStorage.getItem('users')) || {};
    let currentUser = null;

    const books = [
        "Principles of Marketing",
        "The Sea",
        "The Science Library",
        "Mysteries of Mind Space and Time",
        "The World We Lived In",
        "Discover Science",
        "Integrated Science Philippines 8",
        "Earth Science",
        "Skylab's Astronomy and Space Science"
    ];

    let borrowedBooks = JSON.parse(localStorage.getItem('borrowedBooks')) || {};

    function showSection(sectionId) {
        document.querySelectorAll('.section').forEach(section => section.classList.remove('active'));
        document.getElementById(sectionId).classList.add('active');
    }

    function updateBookLists() {
        const availableBooksList = document.getElementById('available-books');
        const borrowedBooksList = document.getElementById('borrowed-books');
        const bookSelect = document.getElementById('book-select');

        availableBooksList.innerHTML = '';
        borrowedBooksList.innerHTML = '';
        bookSelect.innerHTML = '<option value="" disabled selected>Select a book</option>';

        books.forEach(book => {
            if (borrowedBooks[book]) {
                borrowedBooksList.innerHTML += `<li class="borrowed">${book} (Borrowed by ${borrowedBooks[book].user} on ${borrowedBooks[book].time}) <button onclick="returnBook('${book}')">Return</button></li>`;
            } else {
                availableBooksList.innerHTML += `<li>${book}</li>`;
                bookSelect.innerHTML += `<option value="${book}">${book}</option>`;
            }
        });
    }

    function borrowBook() {
        const selectedBook = document.getElementById('book-select').value;
        if (selectedBook) {
            const borrowTime = new Date().toLocaleString();
            borrowedBooks[selectedBook] = {
                user: currentUser,
                time: borrowTime
            };
            localStorage.setItem('borrowedBooks', JSON.stringify(borrowedBooks));
            updateBookLists();
            alert(`You have borrowed "${selectedBook}" at ${borrowTime}`);
        } else {
            alert("Please select a book to borrow!");
        }
    }

    function returnBook(book) {
        if (borrowedBooks[book] && borrowedBooks[book].user === currentUser) {
            delete borrowedBooks[book];
            localStorage.setItem('borrowedBooks', JSON.stringify(borrowedBooks));
            updateBookLists();
            alert(`You have returned "${book}"`);
        } else {
            alert("This book was not borrowed by you!");
        }
    }

    function register() {
        const username = document.getElementById('username-register').value;
        const password = document.getElementById('password-register').value;
        const age = document.getElementById('age').value;
        const gender = document.getElementById('gender').value;

        if (username && password && age && gender) {
            if (users[username]) {
                alert("User already exists!");
            } else {
                users[username] = { password, age, gender };
                localStorage.setItem('users', JSON.stringify(users));
                alert("Registration successful! Please log in.");
                showSection('login-section');
            }
        } else {
            alert("Please fill in all fields.");
        }
    }

    function login() {
        const username = document.getElementById('username-login').value;
        const password = document.getElementById('password-login').value;

        if (users[username] && users[username].password === password) {
            currentUser = username;
            alert(`Welcome, ${username}!`);
            updateBookLists();
            showSection('borrow-section');
        } else {
            alert("Invalid credentials!");
        }
    }

    function logout() {
        currentUser = null;
        alert("You have been logged out!");
        showSection('login-section');
    }

    // Export the borrowed books to Excel
    function exportToExcel() {
        const borrowedBooksData = [];
        Object.keys(borrowedBooks).forEach(book => {
            const borrowDetails = borrowedBooks[book];
            borrowedBooksData.push([book, borrowDetails.user, borrowDetails.time]);
        });

        if (borrowedBooksData.length > 0) {
            const ws = XLSX.utils.aoa_to_sheet([["Book Title", "Borrowed By", "Date & Time"], ...borrowedBooksData]);
            const wb = XLSX.utils.book_new();
            XLSX.utils.book_append_sheet(wb, ws, "Borrowed Books");

            // Generate and download the Excel file
            XLSX.writeFile(wb, "borrowed_books.xlsx");
        } else {
            alert("No borrowed books to export.");
        }
    }

    // Initialize the app
    updateBookLists();
</script>
