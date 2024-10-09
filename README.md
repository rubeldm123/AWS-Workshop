AWS EC2 Workshop: Registration and Login System with MySQL and PHP
Overview
This project provides a step-by-step guide to setting up a simple registration and login system on an AWS EC2 Ubuntu instance using PHP, MySQL, HTML, and CSS. By following this guide, you'll learn how to:

Launch an EC2 instance and connect to it via SSH.
Install the required software (Apache, MySQL, PHP).
Set up a secure MySQL database.
Create user-friendly registration and login forms.
View and manage user data from MySQL.
Prerequisites
Before starting, ensure you have:

An AWS account.
Access to an EC2 Ubuntu instance.
PuTTY (for Windows users) or an SSH client (Linux/macOS).
Installation Guide
1. Launch EC2 Instance
Launch an EC2 Ubuntu instance from your AWS console.
Open your instance's security group and allow the following ports:
SSH (port 22)
HTTP (port 80)
Connect to your instance via SSH:
bash
Copy code
ssh -i your-key.pem ubuntu@your-ec2-public-ip
2. Install Required Software
Once logged into your EC2 instance, install Apache, PHP, and MySQL:

bash
Copy code
sudo apt update
sudo apt install apache2 php libapache2-mod-php php-mysql mysql-server -y
3. Configure MySQL
Secure the MySQL installation:
bash
Copy code
sudo mysql_secure_installation
Set the MySQL root password manually:
bash
Copy code
sudo mysql
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'YourPassword';
FLUSH PRIVILEGES;
EXIT;
4. Create the MySQL Database
Log in to MySQL:
bash
Copy code
sudo mysql -u root -p
Create a new database and table:
sql
Copy code
CREATE DATABASE registrationDB;
USE registrationDB;

CREATE TABLE users (
    id INT(6) UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(30) NOT NULL,
    email VARCHAR(50),
    password VARCHAR(255) NOT NULL
);
5. Set Up Registration and Login Forms
Navigate to the web root directory:
bash
Copy code
cd /var/www/html
Create the following PHP files:
register.php
php
Copy code
<?php
// Connect to MySQL
$conn = new mysqli('localhost', 'root', 'YourPassword', 'registrationDB');

if ($conn->connect_error) {
    die("Connection failed: " . $conn->connect_error);
}

if ($_SERVER['REQUEST_METHOD'] == 'POST') {
    $username = $_POST['username'];
    $email = $_POST['email'];
    $password = password_hash($_POST['password'], PASSWORD_BCRYPT);

    $sql = "INSERT INTO users (username, email, password) VALUES ('$username', '$email', '$password')";
    
    if ($conn->query($sql) === TRUE) {
        header("Location: login.php");
        exit();
    } else {
        echo "Error: " . $sql . "<br>" . $conn->error;
    }
}
?>

<!DOCTYPE html>
<html>
<head>
    <title>Register</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background-color: #f4f7f6;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
        }
        .register-container {
            background-color: #ffffff;
            padding: 20px;
            border-radius: 8px;
            box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
        }
    </style>
</head>
<body>
    <div class="register-container">
        <h2>Register</h2>
        <form method="POST" action="">
            <label for="username">Username:</label>
            <input type="text" name="username" required>
            <label for="email">Email:</label>
            <input type="email" name="email" required>
            <label for="password">Password:</label>
            <input type="password" name="password" required>
            <input type="submit" value="Register">
        </form>
        <div>
            <p>Already have an account? <a href="login.php">Login here</a></p>
        </div>
    </div>
</body>
</html>
login.php
php
Copy code
<?php
session_start();

$conn = new mysqli('localhost', 'root', 'YourPassword', 'registrationDB');

if ($conn->connect_error) {
    die("Connection failed: " . $conn->connect_error);
}

if ($_SERVER['REQUEST_METHOD'] == 'POST') {
    $username = $_POST['username'];
    $password = $_POST['password'];

    $sql = "SELECT * FROM users WHERE username='$username'";
    $result = $conn->query($sql);

    if ($result->num_rows > 0) {
        $row = $result->fetch_assoc();
        if (password_verify($password, $row['password'])) {
            $_SESSION['username'] = $row['username'];
            $_SESSION['email'] = $row['email'];
            header("Location: account.php");
            exit();
        } else {
            $error_message = "Incorrect password.";
        }
    } else {
        $error_message = "No user found with that username.";
    }
}
?>

<!DOCTYPE html>
<html>
<head>
    <title>Login</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background-color: #f4f7f6;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
        }
        .login-container {
            background-color: #ffffff;
            padding: 20px;
            border-radius: 8px;
            box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
        }
    </style>
</head>
<body>
    <div class="login-container">
        <h2>Login</h2>
        <form method="POST" action="">
            <label for="username">Username:</label>
            <input type="text" name="username" required>
            <label for="password">Password:</label>
            <input type="password" name="password" required>
            <input type="submit" value="Login">
        </form>
        <div>
            <p>Don't have an account? <a href="register.php">Register here</a></p>
        </div>
    </div>
</body>
</html>
account.php
php
Copy code
<?php
session_start();

if (!isset($_SESSION['username'])) {
    header("Location: login.php");
    exit();
}

$username = $_SESSION['username'];
$email = $_SESSION['email'];
?>

<!DOCTYPE html>
<html>
<head>
    <title>Account</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background-color: #f4f7f6;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
        }
        .account-container {
            background-color: #ffffff;
            padding: 20px;
            border-radius: 8px;
            box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
        }
    </style>
</head>
<body>
    <div class="account-container">
        <h2>Welcome, <?php echo $username; ?>!</h2>
        <p>Email: <?php echo $email; ?></p>
        <a href="logout.php">Logout</a>
    </div>
</body>
</html>
logout.php
php
Copy code
<?php
session_start();
session_destroy();
header("Location: login.php");
exit();
?>
6. Set Permissions
Make sure Apache has the necessary permissions to access the PHP files:

bash
Copy code
sudo chown www-data:www-data /var/www/html/*.php
sudo chmod 755 /var/www/html/*.php
7. Access the Application
Visit the Registration Page: http://your-ec2-public-ip/register.php
Visit the Login Page: http://your-ec2-public-ip/login.php
Upon successful login, you’ll be redirected to the Account Page, where the user’s name and email will be displayed.
License
This project is licensed under the MIT License.

