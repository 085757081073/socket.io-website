```javascript
const express = require('express');
const http = require('http');
const socketIO = require('socket.io');

const app = express();
const server = http.createServer(app);
const io = socketIO(server);

// Menyimpan daftar pengguna yang terhubung
let connectedUsers = [];

// Menangani koneksi socket baru
io.on('connection', (socket) => {
  console.log('User terhubung');

  // Menambahkan pengguna baru ke daftar pengguna terhubung
  socket.on('join', (username) => {
    const user = {
      id: socket.id,
      username: username
    };
    connectedUsers.push(user);
    console.log(`${username} telah bergabung`);

    // Mengirim daftar pengguna terhubung ke semua pengguna
    io.emit('userList', connectedUsers);
  });

  // Mengirim pesan ke grup chat
  socket.on('message', (message) => {
    console.log(`Pesan baru: ${message.text}`);

    // Mengirim pesan ke semua pengguna
    io.emit('message', message);
  });

  // Menghapus pengguna dari daftar pengguna terhubung saat socket terputus
  socket.on('disconnect', () => {
    connectedUsers = connectedUsers.filter(user => user.id !== socket.id);
    console.log('User terputus');

    // Mengirim daftar pengguna terhubung yang diperbarui ke semua pengguna
    io.emit('userList', connectedUsers);
  });
});

server.listen(3000, () => {
  console.log('Server berjalan di http://localhost:3000');
});
```

2. Client-side (menggunakan Socket.io pada JavaScript di sisi klien):
```html
<!DOCTYPE html>
<html>
<head>
  <title>Chat Grup Tanpa Akun</title>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/socket.io/2.3.0/socket.io.js"></script>
</head>
<body>
  <h1>Chat Grup Tanpa Akun</h1>

  <div id="userList"></div>

  <div id="chatContainer"></div>

  <div>
    <input type="text" id="messageInput" placeholder="Tulis pesan" />
    <button onclick="sendMessage()">Kirim</button>
  </div>

  <script>
    const socket = io();

    socket.on('connect', () => {
      const username = prompt("Masukkan nama pengguna:");
      socket.emit('join', username);
    });

    socket.on('userList', (users) => {
      const userList = document.getElementById('userList');
      userList.innerHTML = 'Daftar Pengguna Terhubung:';
      users.forEach(user => {
        const listItem = document.createElement('li');
        listItem.textContent = user.username;
        userList.appendChild(listItem);
      });
    });

    socket.on('message', (message) => {
      const chatContainer = document.getElementById('chatContainer');

      const messageElement = document.createElement('div');
      messageElement.textContent = `${message.sender}: ${message.text}`;

      chatContainer.appendChild(messageElement);
    });

    function sendMessage() {
      const messageInput = document.getElementById('messageInput');
      const message = messageInput.value;

      if (message) {
        const newMessage = {
          sender: 'User',
          text: message
        };

        socket.emit('message', newMessage);

        // Clear pesan input setelah pengiriman
        messageInput.value = '';
      }
    }
  </script>
</body>
</html>
```
