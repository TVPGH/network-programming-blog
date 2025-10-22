---
title: "Lập Trình Mạng với Java NIO"
date: 2024-02-05
tags: ["Java", "NIO", "Non-blocking", "Network Programming"]
draft: false
---

Java NIO (New I/O) cung cấp một cách hiệu quả hơn để xử lý I/O operations. Trong bài viết này, chúng ta sẽ tìm hiểu cách sử dụng NIO cho lập trình mạng.

## NIO là gì?

NIO (New I/O) là một API mới trong Java cho phép xử lý I/O operations một cách non-blocking và hiệu quả hơn so với I/O truyền thống.

## Các thành phần chính của NIO

- **Channels**: Kênh giao tiếp
- **Buffers**: Vùng đệm dữ liệu
- **Selectors**: Bộ chọn để quản lý nhiều channels

## Server NIO cơ bản

```java
import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.*;
import java.util.Iterator;
import java.util.Set;

public class NIOServer {
    private Selector selector;
    private ServerSocketChannel serverChannel;
    
    public void start(int port) throws IOException {
        // Tạo selector
        selector = Selector.open();
        
        // Tạo server socket channel
        serverChannel = ServerSocketChannel.open();
        serverChannel.configureBlocking(false);
        serverChannel.bind(new InetSocketAddress(port));
        
        // Đăng ký server channel với selector
        serverChannel.register(selector, SelectionKey.OP_ACCEPT);
        
        System.out.println("NIO Server đang chạy trên port " + port);
        
        // Vòng lặp chính
        while (true) {
            // Chờ các events
            int readyChannels = selector.select();
            
            if (readyChannels == 0) continue;
            
            // Lấy các keys đã sẵn sàng
            Set<SelectionKey> selectedKeys = selector.selectedKeys();
            Iterator<SelectionKey> keyIterator = selectedKeys.iterator();
            
            while (keyIterator.hasNext()) {
                SelectionKey key = keyIterator.next();
                
                if (key.isAcceptable()) {
                    handleAccept(key);
                } else if (key.isReadable()) {
                    handleRead(key);
                } else if (key.isWritable()) {
                    handleWrite(key);
                }
                
                keyIterator.remove();
            }
        }
    }
    
    private void handleAccept(SelectionKey key) throws IOException {
        ServerSocketChannel serverChannel = (ServerSocketChannel) key.channel();
        SocketChannel clientChannel = serverChannel.accept();
        clientChannel.configureBlocking(false);
        
        // Đăng ký client channel để đọc
        clientChannel.register(selector, SelectionKey.OP_READ);
        
        System.out.println("Client đã kết nối: " + clientChannel.getRemoteAddress());
    }
    
    private void handleRead(SelectionKey key) throws IOException {
        SocketChannel clientChannel = (SocketChannel) key.channel();
        ByteBuffer buffer = ByteBuffer.allocate(1024);
        
        int bytesRead = clientChannel.read(buffer);
        
        if (bytesRead > 0) {
            buffer.flip();
            byte[] data = new byte[buffer.remaining()];
            buffer.get(data);
            String message = new String(data);
            
            System.out.println("Nhận từ client: " + message);
            
            // Gửi phản hồi
            String response = "Server phản hồi: " + message;
            ByteBuffer responseBuffer = ByteBuffer.wrap(response.getBytes());
            clientChannel.write(responseBuffer);
        } else if (bytesRead == -1) {
            // Client đã đóng kết nối
            System.out.println("Client đã ngắt kết nối");
            clientChannel.close();
        }
    }
    
    private void handleWrite(SelectionKey key) throws IOException {
        // Xử lý ghi dữ liệu
        SocketChannel channel = (SocketChannel) key.channel();
        // Logic ghi dữ liệu ở đây
    }
    
    public static void main(String[] args) {
        try {
            new NIOServer().start(8080);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

## Client NIO

```java
import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.SocketChannel;
import java.util.Scanner;

public class NIOClient {
    public static void main(String[] args) {
        try {
            SocketChannel socketChannel = SocketChannel.open();
            socketChannel.connect(new InetSocketAddress("localhost", 8080));
            
            Scanner scanner = new Scanner(System.in);
            
            while (true) {
                System.out.print("Nhập tin nhắn (hoặc 'quit' để thoát): ");
                String message = scanner.nextLine();
                
                if ("quit".equals(message)) {
                    break;
                }
                
                // Gửi tin nhắn
                ByteBuffer buffer = ByteBuffer.wrap(message.getBytes());
                socketChannel.write(buffer);
                
                // Nhận phản hồi
                ByteBuffer responseBuffer = ByteBuffer.allocate(1024);
                int bytesRead = socketChannel.read(responseBuffer);
                
                if (bytesRead > 0) {
                    responseBuffer.flip();
                    byte[] data = new byte[responseBuffer.remaining()];
                    responseBuffer.get(data);
                    String response = new String(data);
                    System.out.println("Server phản hồi: " + response);
                }
            }
            
            socketChannel.close();
            scanner.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

## Non-blocking File Transfer

```java
import java.io.*;
import java.nio.ByteBuffer;
import java.nio.channels.*;
import java.nio.file.*;

public class NIOFileTransfer {
    public static void transferFile(String sourcePath, String destPath) throws IOException {
        Path source = Paths.get(sourcePath);
        Path destination = Paths.get(destPath);
        
        try (FileChannel sourceChannel = FileChannel.open(source, StandardOpenOption.READ);
             FileChannel destChannel = FileChannel.open(destination, 
                 StandardOpenOption.CREATE, StandardOpenOption.WRITE)) {
            
            ByteBuffer buffer = ByteBuffer.allocate(8192);
            long transferred = 0;
            long fileSize = sourceChannel.size();
            
            while (transferred < fileSize) {
                int bytesRead = sourceChannel.read(buffer);
                if (bytesRead == -1) break;
                
                buffer.flip();
                destChannel.write(buffer);
                buffer.clear();
                
                transferred += bytesRead;
                System.out.println("Đã chuyển: " + (transferred * 100 / fileSize) + "%");
            }
            
            System.out.println("Chuyển file hoàn tất!");
        }
    }
    
    public static void main(String[] args) {
        try {
            transferFile("input.txt", "output.txt");
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

## Asynchronous NIO

```java
import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.AsynchronousServerSocketChannel;
import java.nio.channels.AsynchronousSocketChannel;
import java.nio.channels.CompletionHandler;
import java.util.concurrent.CountDownLatch;

public class AsyncNIOServer {
    public static void main(String[] args) throws Exception {
        AsynchronousServerSocketChannel server = 
            AsynchronousServerSocketChannel.open();
        server.bind(new InetSocketAddress(8080));
        
        System.out.println("Async NIO Server đang chạy...");
        
        server.accept(null, new CompletionHandler<AsynchronousSocketChannel, Void>() {
            @Override
            public void completed(AsynchronousSocketChannel client, Void attachment) {
                // Chấp nhận kết nối mới
                server.accept(null, this);
                
                // Xử lý client
                handleClient(client);
            }
            
            @Override
            public void failed(Throwable exc, Void attachment) {
                exc.printStackTrace();
            }
        });
        
        // Giữ server chạy
        new CountDownLatch(1).await();
    }
    
    private static void handleClient(AsynchronousSocketChannel client) {
        ByteBuffer buffer = ByteBuffer.allocate(1024);
        
        client.read(buffer, null, new CompletionHandler<Integer, Void>() {
            @Override
            public void completed(Integer bytesRead, Void attachment) {
                if (bytesRead > 0) {
                    buffer.flip();
                    byte[] data = new byte[buffer.remaining()];
                    buffer.get(data);
                    String message = new String(data);
                    
                    System.out.println("Nhận: " + message);
                    
                    // Gửi phản hồi
                    String response = "Async Server phản hồi: " + message;
                    ByteBuffer responseBuffer = ByteBuffer.wrap(response.getBytes());
                    client.write(responseBuffer);
                }
            }
            
            @Override
            public void failed(Throwable exc, Void attachment) {
                exc.printStackTrace();
            }
        });
    }
}
```

## Kết luận

Java NIO cung cấp một cách hiệu quả và scalable để xử lý I/O operations. Với khả năng non-blocking và asynchronous, NIO là lựa chọn tốt cho các ứng dụng mạng hiệu suất cao.
