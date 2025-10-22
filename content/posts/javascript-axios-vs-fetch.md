---
title: "So Sánh Axios và Fetch API trong JavaScript"
date: 2024-02-10
tags: ["JavaScript", "Axios", "Fetch API", "HTTP", "Comparison"]
draft: false
---

Khi làm việc với HTTP requests trong JavaScript, chúng ta có hai lựa chọn chính: Axios và Fetch API. Trong bài viết này, chúng ta sẽ so sánh hai phương pháp này.

## Fetch API

Fetch API là một API native của JavaScript, được hỗ trợ trong tất cả các trình duyệt hiện đại.

### Ưu điểm của Fetch API

- **Native**: Không cần cài đặt thư viện bên ngoài
- **Promise-based**: Hỗ trợ async/await
- **Lightweight**: Nhẹ và nhanh

### Nhược điểm của Fetch API

- **Không tự động parse JSON**: Cần gọi `.json()` thủ công
- **Không có request/response interceptors**
- **Không hỗ trợ timeout mặc định**
- **Không hỗ trợ request cancellation**

## Axios

Axios là một thư viện HTTP client phổ biến, cung cấp nhiều tính năng mạnh mẽ.

### Ưu điểm của Axios

- **Tự động parse JSON**: Tự động chuyển đổi response thành JSON
- **Interceptors**: Hỗ trợ request/response interceptors
- **Request cancellation**: Hỗ trợ hủy requests
- **Timeout**: Hỗ trợ timeout mặc định
- **Error handling**: Xử lý lỗi tốt hơn

### Nhược điểm của Axios

- **Bundle size**: Tăng kích thước bundle
- **Dependency**: Cần cài đặt thư viện bên ngoài

## So sánh chi tiết

### 1. Cú pháp cơ bản

**Fetch API:**
```javascript
// GET request
fetch('https://api.example.com/users')
  .then(response => response.json())
  .then(data => console.log(data))
  .catch(error => console.error('Error:', error));

// POST request
fetch('https://api.example.com/users', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({
    name: 'John Doe',
    email: 'john@example.com'
  })
})
.then(response => response.json())
.then(data => console.log(data));
```

**Axios:**
```javascript
// GET request
axios.get('https://api.example.com/users')
  .then(response => console.log(response.data))
  .catch(error => console.error('Error:', error));

// POST request
axios.post('https://api.example.com/users', {
  name: 'John Doe',
  email: 'john@example.com'
})
.then(response => console.log(response.data));
```

### 2. Error Handling

**Fetch API:**
```javascript
async function fetchWithErrorHandling(url) {
  try {
    const response = await fetch(url);
    
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    
    const data = await response.json();
    return data;
  } catch (error) {
    console.error('Fetch error:', error);
    throw error;
  }
}
```

**Axios:**
```javascript
async function axiosWithErrorHandling(url) {
  try {
    const response = await axios.get(url);
    return response.data;
  } catch (error) {
    if (error.response) {
      // Server responded with error status
      console.error('Server error:', error.response.status);
    } else if (error.request) {
      // Request was made but no response received
      console.error('Network error:', error.message);
    } else {
      // Something else happened
      console.error('Error:', error.message);
    }
    throw error;
  }
}
```

### 3. Interceptors

**Fetch API:** Không hỗ trợ interceptors

**Axios:**
```javascript
// Request interceptor
axios.interceptors.request.use(
  config => {
    // Thêm token vào header
    config.headers.Authorization = `Bearer ${getToken()}`;
    return config;
  },
  error => {
    return Promise.reject(error);
  }
);

// Response interceptor
axios.interceptors.response.use(
  response => {
    return response;
  },
  error => {
    if (error.response?.status === 401) {
      // Redirect to login
      window.location.href = '/login';
    }
    return Promise.reject(error);
  }
);
```

### 4. Request Cancellation

**Fetch API:**
```javascript
const controller = new AbortController();
const signal = controller.signal;

fetch('https://api.example.com/data', { signal })
  .then(response => response.json())
  .then(data => console.log(data))
  .catch(error => {
    if (error.name === 'AbortError') {
      console.log('Request was cancelled');
    }
  });

// Cancel request
controller.abort();
```

**Axios:**
```javascript
const source = axios.CancelToken.source();

axios.get('https://api.example.com/data', {
  cancelToken: source.token
})
.then(response => console.log(response.data))
.catch(error => {
  if (axios.isCancel(error)) {
    console.log('Request was cancelled');
  }
});

// Cancel request
source.cancel('Operation cancelled');
```

### 5. Timeout

**Fetch API:**
```javascript
const controller = new AbortController();
const timeoutId = setTimeout(() => controller.abort(), 5000);

fetch('https://api.example.com/data', { signal: controller.signal })
  .then(response => {
    clearTimeout(timeoutId);
    return response.json();
  })
  .then(data => console.log(data));
```

**Axios:**
```javascript
axios.get('https://api.example.com/data', {
  timeout: 5000
})
.then(response => console.log(response.data));
```

## Khi nào nên sử dụng?

### Sử dụng Fetch API khi:
- Dự án đơn giản, ít HTTP requests
- Muốn giảm bundle size
- Chỉ cần các tính năng cơ bản
- Làm việc với Service Workers

### Sử dụng Axios khi:
- Dự án phức tạp với nhiều HTTP requests
- Cần interceptors
- Cần request cancellation
- Cần timeout mặc định
- Cần error handling tốt hơn

## Kết luận

Cả Fetch API và Axios đều có ưu điểm riêng. Fetch API phù hợp cho các dự án đơn giản, trong khi Axios phù hợp cho các dự án phức tạp cần nhiều tính năng nâng cao. Việc lựa chọn phụ thuộc vào nhu cầu cụ thể của dự án.
