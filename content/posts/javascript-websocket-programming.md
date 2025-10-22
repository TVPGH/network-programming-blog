---
title: "Lập Trình WebSocket với JavaScript"
date: 2024-02-01
tags: ["JavaScript", "WebSocket", "Real-time", "Network"]
draft: false
---

WebSocket cho phép giao tiếp hai chiều real-time giữa client và server. Trong bài viết này, chúng ta sẽ tìm hiểu cách sử dụng WebSocket API trong JavaScript.

## WebSocket là gì?

WebSocket là một giao thức mạng cho phép giao tiếp hai chiều qua một kết nối TCP duy nhất. Nó rất hữu ích cho các ứng dụng real-time như chat, game, và live updates.

## Tạo WebSocket Connection

```javascript
// Kết nối WebSocket
const socket = new WebSocket('ws://localhost:8080');

// Xử lý sự kiện kết nối
socket.onopen = function(event) {
    console.log('Đã kết nối WebSocket');
    // Gửi tin nhắn đầu tiên
    socket.send('Xin chào server!');
};

// Xử lý tin nhắn từ server
socket.onmessage = function(event) {
    console.log('Nhận tin nhắn:', event.data);
    displayMessage(event.data);
};

// Xử lý lỗi
socket.onerror = function(error) {
    console.error('Lỗi WebSocket:', error);
};

// Xử lý đóng kết nối
socket.onclose = function(event) {
    console.log('Kết nối đã đóng:', event.code, event.reason);
};
```

## Chat Application

```javascript
class ChatApp {
    constructor() {
        this.socket = null;
        this.username = null;
        this.messageInput = document.getElementById('messageInput');
        this.messagesContainer = document.getElementById('messages');
        this.connectButton = document.getElementById('connectBtn');
        this.disconnectButton = document.getElementById('disconnectBtn');
        
        this.setupEventListeners();
    }
    
    setupEventListeners() {
        this.connectButton.addEventListener('click', () => this.connect());
        this.disconnectButton.addEventListener('click', () => this.disconnect());
        
        this.messageInput.addEventListener('keypress', (e) => {
            if (e.key === 'Enter') {
                this.sendMessage();
            }
        });
    }
    
    connect() {
        this.username = prompt('Nhập tên của bạn:');
        if (!this.username) return;
        
        this.socket = new WebSocket('ws://localhost:8080');
        
        this.socket.onopen = () => {
            console.log('Đã kết nối chat');
            this.updateUI(true);
        };
        
        this.socket.onmessage = (event) => {
            const message = JSON.parse(event.data);
            this.displayMessage(message);
        };
        
        this.socket.onclose = () => {
            console.log('Đã ngắt kết nối');
            this.updateUI(false);
        };
    }
    
    disconnect() {
        if (this.socket) {
            this.socket.close();
        }
    }
    
    sendMessage() {
        const message = this.messageInput.value.trim();
        if (message && this.socket && this.socket.readyState === WebSocket.OPEN) {
            const messageData = {
                username: this.username,
                message: message,
                timestamp: new Date().toISOString()
            };
            
            this.socket.send(JSON.stringify(messageData));
            this.messageInput.value = '';
        }
    }
    
    displayMessage(messageData) {
        const messageElement = document.createElement('div');
        messageElement.className = 'message';
        messageElement.innerHTML = `
            <strong>${messageData.username}:</strong> 
            ${messageData.message}
            <small>(${new Date(messageData.timestamp).toLocaleTimeString()})</small>
        `;
        
        this.messagesContainer.appendChild(messageElement);
        this.messagesContainer.scrollTop = this.messagesContainer.scrollHeight;
    }
    
    updateUI(connected) {
        this.connectButton.disabled = connected;
        this.disconnectButton.disabled = !connected;
        this.messageInput.disabled = !connected;
    }
}

// Khởi tạo ứng dụng chat
const chatApp = new ChatApp();
```

## Real-time Data Updates

```javascript
class RealTimeDataApp {
    constructor() {
        this.socket = null;
        this.dataContainer = document.getElementById('dataContainer');
        this.connect();
    }
    
    connect() {
        this.socket = new WebSocket('ws://localhost:8080/data');
        
        this.socket.onopen = () => {
            console.log('Đã kết nối real-time data');
            this.requestData();
        };
        
        this.socket.onmessage = (event) => {
            const data = JSON.parse(event.data);
            this.updateDataDisplay(data);
        };
        
        this.socket.onclose = () => {
            console.log('Kết nối đã đóng, thử kết nối lại sau 3 giây...');
            setTimeout(() => this.connect(), 3000);
        };
    }
    
    requestData() {
        if (this.socket && this.socket.readyState === WebSocket.OPEN) {
            this.socket.send(JSON.stringify({
                type: 'request_data',
                timestamp: Date.now()
            }));
        }
    }
    
    updateDataDisplay(data) {
        this.dataContainer.innerHTML = `
            <h3>Dữ liệu Real-time</h3>
            <p>Thời gian: ${new Date(data.timestamp).toLocaleString()}</p>
            <p>Giá trị: ${data.value}</p>
            <p>Trạng thái: ${data.status}</p>
        `;
    }
}
```

## WebSocket với Authentication

```javascript
class AuthenticatedWebSocket {
    constructor(token) {
        this.token = token;
        this.socket = null;
    }
    
    connect() {
        // Thêm token vào URL
        const wsUrl = `ws://localhost:8080/secure?token=${this.token}`;
        this.socket = new WebSocket(wsUrl);
        
        this.socket.onopen = () => {
            console.log('Đã kết nối với xác thực');
        };
        
        this.socket.onmessage = (event) => {
            const data = JSON.parse(event.data);
            if (data.type === 'auth_success') {
                console.log('Xác thực thành công');
            } else if (data.type === 'auth_failed') {
                console.error('Xác thực thất bại:', data.message);
                this.socket.close();
            }
        };
        
        this.socket.onclose = (event) => {
            if (event.code === 1008) {
                console.error('Kết nối bị đóng do lỗi xác thực');
            }
        };
    }
    
    sendSecureMessage(message) {
        if (this.socket && this.socket.readyState === WebSocket.OPEN) {
            this.socket.send(JSON.stringify({
                type: 'secure_message',
                data: message,
                timestamp: Date.now()
            }));
        }
    }
}
```

## Kết luận

WebSocket API trong JavaScript cung cấp một cách mạnh mẽ để xây dựng các ứng dụng real-time. Với khả năng giao tiếp hai chiều và hiệu suất cao, WebSocket là lựa chọn tốt cho các ứng dụng cần cập nhật dữ liệu real-time.
