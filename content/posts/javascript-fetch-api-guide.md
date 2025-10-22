---
title: "Hướng Dẫn Sử Dụng Fetch API trong JavaScript"
date: 2024-01-20
tags: ["JavaScript", "Fetch API", "HTTP", "Async"]
draft: false
---

Fetch API là một cách hiện đại và mạnh mẽ để thực hiện các HTTP requests trong JavaScript. Trong bài viết này, chúng ta sẽ tìm hiểu cách sử dụng Fetch API một cách hiệu quả.

## Fetch API là gì?

Fetch API cung cấp một interface JavaScript để truy cập và thao tác các phần của HTTP pipeline, chẳng hạn như requests và responses.

## Cú pháp cơ bản

```javascript
fetch(url, options)
  .then(response => response.json())
  .then(data => console.log(data))
  .catch(error => console.error('Error:', error));
```

## GET Request

```javascript
// Lấy dữ liệu từ API
async function fetchUserData(userId) {
  try {
    const response = await fetch(`https://api.example.com/users/${userId}`);
    
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    
    const userData = await response.json();
    return userData;
  } catch (error) {
    console.error('Lỗi khi lấy dữ liệu user:', error);
    throw error;
  }
}
```

## POST Request

```javascript
// Gửi dữ liệu mới
async function createUser(userData) {
  try {
    const response = await fetch('https://api.example.com/users', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': 'Bearer your-token-here'
      },
      body: JSON.stringify(userData)
    });
    
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    
    const newUser = await response.json();
    return newUser;
  } catch (error) {
    console.error('Lỗi khi tạo user:', error);
    throw error;
  }
}
```

## PUT và DELETE Request

```javascript
// Cập nhật dữ liệu
async function updateUser(userId, userData) {
  const response = await fetch(`https://api.example.com/users/${userId}`, {
    method: 'PUT',
    headers: {
      'Content-Type': 'application/json'
    },
    body: JSON.stringify(userData)
  });
  
  return response.json();
}

// Xóa dữ liệu
async function deleteUser(userId) {
  const response = await fetch(`https://api.example.com/users/${userId}`, {
    method: 'DELETE'
  });
  
  return response.ok;
}
```

## Xử lý lỗi nâng cao

```javascript
async function fetchWithErrorHandling(url) {
  try {
    const response = await fetch(url);
    
    // Kiểm tra status code
    if (response.status === 404) {
      throw new Error('Không tìm thấy tài nguyên');
    }
    
    if (response.status === 500) {
      throw new Error('Lỗi server');
    }
    
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    
    return await response.json();
  } catch (error) {
    if (error.name === 'TypeError') {
      console.error('Lỗi mạng:', error.message);
    } else {
      console.error('Lỗi khác:', error.message);
    }
    throw error;
  }
}
```

## Upload file với FormData

```javascript
async function uploadFile(file) {
  const formData = new FormData();
  formData.append('file', file);
  
  try {
    const response = await fetch('/api/upload', {
      method: 'POST',
      body: formData
    });
    
    if (!response.ok) {
      throw new Error('Upload failed');
    }
    
    const result = await response.json();
    console.log('Upload thành công:', result);
    return result;
  } catch (error) {
    console.error('Lỗi upload:', error);
    throw error;
  }
}
```

## Kết luận

Fetch API cung cấp một cách hiện đại và linh hoạt để thực hiện HTTP requests. Việc hiểu rõ cách sử dụng Fetch API sẽ giúp bạn xây dựng các ứng dụng web tương tác với API một cách hiệu quả.
