---
title: "Lập Trình Socket với Java - Cơ Bản"
date: 2024-01-15
tags: ["Java", "Socket", "Network Programming"]
draft: false
---

Lập trình socket là một trong những kỹ năng cơ bản nhất trong lập trình mạng. Trong bài viết này, chúng ta sẽ tìm hiểu cách sử dụng Java để tạo các ứng dụng client-server đơn giản.

## Socket là gì?

Socket là một endpoint của kết nối hai chiều giữa hai chương trình chạy trên mạng. Nó được xác định bởi địa chỉ IP và số port.

## Tạo Server Socket

```java
import java.io.*;
import java.net.*;

public class Server {
    public static void main(String[] args) {
        try {
            ServerSocket serverSocket = new ServerSocket(8080);
            System.out.println("Server đang chạy trên port 8080...");
            
            while (true) {
                Socket clientSocket = serverSocket.accept();
                System.out.println("Client đã kết nối: " + clientSocket.getInetAddress());
                
                // Xử lý client trong thread riêng
                new Thread(() -> handleClient(clientSocket)).start();
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    
    private static void handleClient(Socket clientSocket) {
        try {
            BufferedReader in = new BufferedReader(
                new InputStreamReader(clientSocket.getInputStream())
            );
            PrintWriter out = new PrintWriter(
                clientSocket.getOutputStream(), true
            );
            
            String inputLine;
            while ((inputLine = in.readLine()) != null) {
                System.out.println("Nhận từ client: " + inputLine);
                out.println("Server phản hồi: " + inputLine);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

## Tạo Client Socket

```java
import java.io.*;
import java.net.*;

public class Client {
    public static void main(String[] args) {
        try {
            Socket socket = new Socket("localhost", 8080);
            
            BufferedReader in = new BufferedReader(
                new InputStreamReader(socket.getInputStream())
            );
            PrintWriter out = new PrintWriter(
                socket.getOutputStream(), true
            );
            
            // Gửi tin nhắn
            out.println("Xin chào server!");
            
            // Nhận phản hồi
            String response = in.readLine();
            System.out.println("Server phản hồi: " + response);
            
            socket.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

## Kết luận

Lập trình socket với Java cho phép chúng ta tạo ra các ứng dụng mạng mạnh mẽ. Việc hiểu rõ cách hoạt động của socket sẽ giúp bạn phát triển các ứng dụng client-server hiệu quả.
