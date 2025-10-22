---
title: "Hướng Dẫn Sử Dụng HTTP Client trong Java 11+"
date: 2024-01-25
tags: ["Java", "HTTP Client", "REST API", "Network"]
draft: false
---

Java 11 đã giới thiệu HTTP Client API mới, thay thế cho HttpURLConnection cũ. Trong bài viết này, chúng ta sẽ tìm hiểu cách sử dụng HTTP Client API hiện đại này.

## HTTP Client API là gì?

HTTP Client API cung cấp một cách hiện đại và linh hoạt để thực hiện HTTP requests trong Java. Nó hỗ trợ HTTP/2, WebSocket và các tính năng async.

## Cú pháp cơ bản

```java
import java.net.http.HttpClient;
import java.net.http.HttpRequest;
import java.net.http.HttpResponse;
import java.net.URI;

public class BasicHttpClient {
    public static void main(String[] args) throws Exception {
        HttpClient client = HttpClient.newHttpClient();
        
        HttpRequest request = HttpRequest.newBuilder()
            .uri(URI.create("https://api.example.com/users"))
            .build();
            
        HttpResponse<String> response = client.send(request, 
            HttpResponse.BodyHandlers.ofString());
            
        System.out.println("Status: " + response.statusCode());
        System.out.println("Body: " + response.body());
    }
}
```

## GET Request

```java
import java.net.http.*;
import java.net.URI;
import java.time.Duration;

public class GetRequestExample {
    public static void main(String[] args) throws Exception {
        HttpClient client = HttpClient.newBuilder()
            .connectTimeout(Duration.ofSeconds(10))
            .build();
            
        HttpRequest request = HttpRequest.newBuilder()
            .uri(URI.create("https://jsonplaceholder.typicode.com/posts/1"))
            .header("Accept", "application/json")
            .GET()
            .build();
            
        HttpResponse<String> response = client.send(request, 
            HttpResponse.BodyHandlers.ofString());
            
        if (response.statusCode() == 200) {
            System.out.println("Dữ liệu nhận được:");
            System.out.println(response.body());
        } else {
            System.err.println("Lỗi: " + response.statusCode());
        }
    }
}
```

## POST Request với JSON

```java
import java.net.http.*;
import java.net.URI;
import java.nio.charset.StandardCharsets;

public class PostRequestExample {
    public static void main(String[] args) throws Exception {
        HttpClient client = HttpClient.newHttpClient();
        
        String jsonData = """
            {
                "title": "Bài viết mới",
                "body": "Nội dung bài viết",
                "userId": 1
            }
            """;
            
        HttpRequest request = HttpRequest.newBuilder()
            .uri(URI.create("https://jsonplaceholder.typicode.com/posts"))
            .header("Content-Type", "application/json")
            .POST(HttpRequest.BodyPublishers.ofString(jsonData))
            .build();
            
        HttpResponse<String> response = client.send(request, 
            HttpResponse.BodyHandlers.ofString());
            
        System.out.println("Status: " + response.statusCode());
        System.out.println("Response: " + response.body());
    }
}
```

## Async HTTP Requests

```java
import java.net.http.*;
import java.net.URI;
import java.util.concurrent.CompletableFuture;

public class AsyncHttpExample {
    public static void main(String[] args) throws Exception {
        HttpClient client = HttpClient.newHttpClient();
        
        // Tạo nhiều requests đồng thời
        CompletableFuture<HttpResponse<String>> future1 = client.sendAsync(
            HttpRequest.newBuilder()
                .uri(URI.create("https://jsonplaceholder.typicode.com/posts/1"))
                .build(),
            HttpResponse.BodyHandlers.ofString()
        );
        
        CompletableFuture<HttpResponse<String>> future2 = client.sendAsync(
            HttpRequest.newBuilder()
                .uri(URI.create("https://jsonplaceholder.typicode.com/posts/2"))
                .build(),
            HttpResponse.BodyHandlers.ofString()
        );
        
        // Chờ tất cả requests hoàn thành
        CompletableFuture.allOf(future1, future2).join();
        
        System.out.println("Post 1: " + future1.get().body());
        System.out.println("Post 2: " + future2.get().body());
    }
}
```

## Xử lý lỗi và timeout

```java
import java.net.http.*;
import java.net.URI;
import java.time.Duration;
import java.util.concurrent.TimeoutException;

public class ErrorHandlingExample {
    public static void main(String[] args) {
        HttpClient client = HttpClient.newBuilder()
            .connectTimeout(Duration.ofSeconds(5))
            .build();
            
        HttpRequest request = HttpRequest.newBuilder()
            .uri(URI.create("https://httpstat.us/500"))
            .timeout(Duration.ofSeconds(10))
            .build();
            
        try {
            HttpResponse<String> response = client.send(request, 
                HttpResponse.BodyHandlers.ofString());
                
            if (response.statusCode() >= 200 && response.statusCode() < 300) {
                System.out.println("Thành công: " + response.body());
            } else {
                System.err.println("Lỗi HTTP: " + response.statusCode());
            }
        } catch (TimeoutException e) {
            System.err.println("Request timeout");
        } catch (Exception e) {
            System.err.println("Lỗi: " + e.getMessage());
        }
    }
}
```

## Authentication với Bearer Token

```java
import java.net.http.*;
import java.net.URI;

public class AuthenticatedRequest {
    public static void main(String[] args) throws Exception {
        HttpClient client = HttpClient.newHttpClient();
        
        String token = "your-jwt-token-here";
        
        HttpRequest request = HttpRequest.newBuilder()
            .uri(URI.create("https://api.example.com/protected-resource"))
            .header("Authorization", "Bearer " + token)
            .header("Accept", "application/json")
            .GET()
            .build();
            
        HttpResponse<String> response = client.send(request, 
            HttpResponse.BodyHandlers.ofString());
            
        System.out.println("Status: " + response.statusCode());
        System.out.println("Response: " + response.body());
    }
}
```

## Kết luận

HTTP Client API trong Java 11+ cung cấp một cách hiện đại và mạnh mẽ để thực hiện HTTP requests. Với hỗ trợ async, HTTP/2 và các tính năng nâng cao, nó là lựa chọn tốt nhất cho việc phát triển ứng dụng mạng trong Java.
