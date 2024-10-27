# The-full-code-of-the-project
The full code of the project
####Starting with the creation of Databse--
CREATE DATABASE library_db;

USE library_db;

CREATE TABLE books (
    id INT PRIMARY KEY AUTO_INCREMENT,
    title VARCHAR(255) NOT NULL,
    author VARCHAR(255) NOT NULL,
    isbn VARCHAR(20) NOT NULL UNIQUE,
    available BOOLEAN DEFAULT TRUE
);

####This is the backend part of our project--

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;

public class DatabaseConnection {
    private static final String URL = "jdbc:mysql://localhost:3306/library_db";
    private static final String USER = "root";
    private static final String PASSWORD = "your_password";

    public static Connection getConnection() throws SQLException {
        return DriverManager.getConnection(URL, USER, PASSWORD);
    }
}

####This is the library servelet of the project--
import jakarta.servlet.*;
import jakarta.servlet.http.*;
import java.io.IOException;
import java.sql.*;
import java.util.ArrayList;
import java.util.List;

// Model class to represent book information
class Book {
    private int id;
    private String title;
    private String author;
    private String isbn;
    private boolean available;

    // Getters and setters omitted for brevity
}

public class LibraryServlet extends HttpServlet {

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        List<Book> books = new ArrayList<>();
        String sql = "SELECT * FROM books";

        try (Connection conn = DatabaseConnection.getConnection();
             Statement stmt = conn.createStatement();
             ResultSet rs = stmt.executeQuery(sql)) {

            while (rs.next()) {
                Book book = new Book();
                book.setId(rs.getInt("id"));
                book.setTitle(rs.getString("title"));
                book.setAuthor(rs.getString("author"));
                book.setIsbn(rs.getString("isbn"));
                book.setAvailable(rs.getBoolean("available"));
                books.add(book);
            }

        } catch (SQLException e) {
            e.printStackTrace();
        }

        request.setAttribute("books", books);
        RequestDispatcher dispatcher = request.getRequestDispatcher("books.jsp");
        dispatcher.forward(request, response);
    }

    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        String title = request.getParameter("title");
        String author = request.getParameter("author");
        String isbn = request.getParameter("isbn");

        String sql = "INSERT INTO books (title, author, isbn, available) VALUES (?, ?, ?, true)";
        try (Connection conn = DatabaseConnection.getConnection();
             PreparedStatement stmt = conn.prepareStatement(sql)) {
            stmt.setString(1, title);
            stmt.setString(2, author);
            stmt.setString(3, isbn);
            stmt.executeUpdate();
        } catch (SQLException e) {
            e.printStackTrace();
        }

        response.sendRedirect("library");
    }
}

####The frontend part of our project in which we have used HTML and CSS--
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Library Management System</title>
    <link rel="stylesheet" href="styles.css">
</head>
<body>
    <h1>Library Management System</h1>

    <form action="library" method="post">
        <h2>Add Book</h2>
        <label for="title">Title:</label>
        <input type="text" id="title" name="title" required>
        
        <label for="author">Author:</label>
        <input type="text" id="author" name="author" required>
        
        <label for="isbn">ISBN:</label>
        <input type="text" id="isbn" name="isbn" required>
        
        <button type="submit">Add Book</button>
    </form>

    <h2>Book List</h2>
    <table>
        <thead>
            <tr>
                <th>Title</th>
                <th>Author</th>
                <th>ISBN</th>
                <th>Available</th>
            </tr>
        </thead>
        <tbody>
            <%-- Book list will be populated here from the servlet --%>
            <c:forEach var="book" items="${books}">
                <tr>
                    <td>${book.title}</td>
                    <td>${book.author}</td>
                    <td>${book.isbn}</td>
                    <td>${book.available ? "Yes" : "No"}</td>
                </tr>
            </c:forEach>
        </tbody>
    </table>
</body>
</html>

####For making our project look attractive on the frontend part we have used CSS for that--
body {
    font-family: Arial, sans-serif;
    max-width: 800px;
    margin: auto;
    padding: 20px;
    background-color: #f4f4f9;
}

h1, h2 {
    text-align: center;
}

form {
    display: flex;
    flex-direction: column;
    gap: 10px;
    margin-bottom: 20px;
}

label {
    font-weight: bold;
}

table {
    width: 100%;
    border-collapse: collapse;
}

table, th, td {
    border: 1px solid #ddd;
}

th, td {
    padding: 10px;
    text-align: center;
}


