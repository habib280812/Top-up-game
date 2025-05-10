# Top-up-game
CREATE TABLE topup_data (
    id INT AUTO_INCREMENT PRIMARY KEY,
    game VARCHAR(100),
    player_id VARCHAR(100),
    jumlah_diamond INT,
    harga VARCHAR(20),
    waktu TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

<?php
$host = 'localhost';
$user = 'root'; // Ganti dengan user database kamu
$pass = '';     // Ganti dengan password database kamu
$db = 'topup_db';

$conn = new mysqli($host, $user, $pass, $db);

if ($conn->connect_error) {
    die('Koneksi gagal: ' . $conn->connect_error);
}
?>

<?php
include 'db.php';

$game = $_POST['game'];
$playerId = $_POST['playerId'];
$jumlah = $_POST['jumlah'];
$harga = $_POST['harga'];

$sql = "INSERT INTO topup_data (game, player_id, jumlah_diamond, harga)
        VALUES ('$game', '$playerId', '$jumlah', '$harga')";

if ($conn->query($sql) === TRUE) {
    echo "Data berjaya disimpan!";
} else {
    echo "Ralat: " . $conn->error;
}

$conn->close();
?>

<script>
  function topUp() {
    const game = document.getElementById('game').value;
    const playerId = document.getElementById('playerId').value;
    const amount = document.getElementById('amount').value;

    if (!playerId) {
      alert('Sila masukkan ID pemain!');
      return;
    }

    const harga = {
      "10": "RM1.50",
      "50": "RM7.00",
      "100": "RM13.00",
      "200": "RM25.00"
    };

    const formData = new FormData();
    formData.append('game', game);
    formData.append('playerId', playerId);
    formData.append('jumlah', amount);
    formData.append('harga', harga[amount]);

    fetch('simpan.php', {
      method: 'POST',
      body: formData
    })
    .then(response => response.text())
    .then(data => {
      document.getElementById('result').innerHTML = `
        <h3>Butiran Top-Up:</h3>
        <p>Game: ${game}</p>
        <p>ID Pemain: ${playerId}</p>
        <p>Jumlah: ${amount} Diamond</p>
        <p>Harga: ${harga[amount]}</p>
        <p>Status: ${data}</p>
      `;
    })
    .catch(error => {
      console.error('Ralat:', error);
    });
  }
</script>

CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(100) UNIQUE,
    password VARCHAR(255)
);

<?php session_start(); ?>
<!DOCTYPE html>
<html lang="id">
<head>
  <meta charset="UTF-8">
  <title>Login</title>
</head>
<body>

  <h1>Login Pengguna</h1>

  <form action="login_process.php" method="POST">
    <label for="username">Username:</label>
    <input type="text" name="username" id="username" required>

    <label for="password">Password:</label>
    <input type="password" name="password" id="password" required>

    <button type="submit">Login</button>
  </form>

  <?php
  if (isset($_SESSION['error'])) {
    echo "<p style='color: red;'>{$_SESSION['error']}</p>";
    unset($_SESSION['error']);
  }
  ?>

</body>
</html>

<?php
session_start();
include 'db.php';

$username = $_POST['username'];
$password = $_POST['password'];

// Periksa apakah pengguna ada dalam database
$sql = "SELECT * FROM users WHERE username = '$username'";
$result = $conn->query($sql);

if ($result->num_rows > 0) {
    $row = $result->fetch_assoc();
    // Verifikasi password
    if (password_verify($password, $row['password'])) {
        $_SESSION['user_id'] = $row['id'];
        $_SESSION['username'] = $row['username'];
        header('Location: index.php'); // Arahkan ke halaman utama setelah login
        exit();
    } else {
        $_SESSION['error'] = 'Password salah!';
        header('Location: login.php');
        exit();
    }
} else {
    $_SESSION['error'] = 'Pengguna tidak ditemukan!';
    header('Location: login.php');
    exit();
}

$conn->close();
?>

<?php
session_start();
session_unset();
session_destroy();
header('Location: login.php');
exit();
?>

<?php
session_start();
if (!isset($_SESSION['user_id'])) {
    header('Location: login.php');
    exit();
}

include 'db.php';

// Ambil riwayat transaksi pengguna
$user_id = $_SESSION['user_id'];
$sql = "SELECT * FROM topup_data WHERE user_id = '$user_id' ORDER BY waktu DESC";
$result = $conn->query($sql);

?>

<!DOCTYPE html>
<html lang="id">
<head>
  <meta charset="UTF-8">
  <title>Top-Up Game</title>
</head>
<body>

  <h1>Selamat Datang, <?php echo $_SESSION['username']; ?>!</h1>
  <a href="logout.php">Logout</a>

  <h2>Riwayat Transaksi:</h2>
  <table border="1">
    <tr>
      <th>Game</th>
      <th>ID Pemain</th>
      <th>Jumlah Diamond</th>
      <th>Harga</th>
      <th>Waktu</th>
    </tr>

    <?php while ($row = $result->fetch_assoc()) { ?>
      <tr>
        <td><?php echo $row['game']; ?></td>
        <td><?php echo $row['player_id']; ?></td>
        <td><?php echo $row['jumlah_diamond']; ?></td>
        <td><?php echo $row['harga']; ?></td>
        <td><?php echo $row['waktu']; ?></td>
      </tr>
    <?php } ?>
  </table>

  <h2>Top-Up Game</h2>
  <!-- Form top-up seperti sebelumnya di sini -->

</body>
</html>

<?php
$conn->close();
?>

ALTER TABLE topup_data ADD COLUMN user_id INT;

<?php
session_start();
include 'db.php';

if (!isset($_SESSION['user_id'])) {
    die("Sila login terlebih dahulu!");
}

$game = $_POST['game'];
$playerId = $_POST['playerId'];
$jumlah = $_POST['jumlah'];
$harga = $_POST['harga'];
$user_id = $_SESSION['user_id'];

$sql = "INSERT INTO topup_data (game, player_id, jumlah_diamond, harga, user_id)
        VALUES ('$game', '$playerId', '$jumlah', '$harga', '$user_id')";

if ($conn->query($sql) === TRUE) {
    echo "Data berjaya disimpan!";
} else {
    echo "Ralat: " . $conn->error;
}

$conn->close();
?>
