# BÀI 4: THIẾT KẾ MEGA-PROMPT SINH MOCK DATA & UNIT TEST

## 1. Nội dung Mega-Prompt duy nhất
```markdown
Đóng vai Senior Automation Test Engineer. 

Nhiệm vụ của bạn là sinh đồng thời bộ dữ liệu test (Mock Data) dạng JSON và mã nguồn kiểm thử JUnit 5 cho một class dịch vụ có tên là `UserAuthentication`. 
Class `UserAuthentication` có phương thức kiểm thử với chữ ký như sau: `public boolean login(String username, String password)`.

Yêu cầu thực hiện chi tiết:
1. Tạo một bộ dữ liệu giả lập (Mock Data) chuẩn định dạng JSON chứa danh sách các trường hợp kiểm thử. Bộ dữ liệu phải bao gồm chính xác 5 kịch bản sau:
   - Kịch bản 1: Thông tin đúng (Tài khoản hoạt động bình thường, thông tin nhập chính xác -> Mong đợi: true)
   - Kịch bản 2: Sai mật khẩu (Tài khoản tồn tại nhưng nhập sai mật khẩu -> Mong đợi: false)
   - Kịch bản 3: Tài khoản bị khóa (Tài khoản hợp lệ nhưng trạng thái tài khoản đang bị khóa -> Mong đợi: false / ném exception tùy thiết kế)
   - Kịch bản 4: Chứa mã độc SQL Injection (Trường username chứa chuỗi injection như "' OR '1'='1" -> Mong đợi: false)
   - Kịch bản 5: Mật khẩu rỗng (Trường password để trống hoặc null -> Mong đợi: false)

2. Viết mã nguồn JUnit 5 (Java 17) để thực hiện kiểm thử phương thức `login(String username, String password)`. 
   - Sử dụng Mockito để mock cơ chế hoạt động của `UserAuthentication` hoặc mô phỏng dữ liệu đầu vào.
   - Viết các test case tương ứng trực quan cho 5 kịch bản trên.
   - Không cần viết mã nguồn thực tế cho class `UserAuthentication`, hãy tự giả định hành vi logic của nó dựa trên các kịch bản test trên.

Hãy định dạng kết quả trả về rõ ràng thành 2 code block riêng biệt:
- Block 1: Mock Data dạng JSON.
- Block 2: Mã nguồn JUnit 5.
```

## 2. Bộ Mock Data (JSON) do AI sinh ra
```json
{
  "testCases": [
    {
      "id": "TC01_VALID_CREDENTIALS",
      "description": "Tài khoản hoạt động bình thường, thông tin nhập chính xác",
      "username": "active_user",
      "password": "SecurePassword123",
      "expectedResult": true
    },
    {
      "id": "TC02_WRONG_PASSWORD",
      "description": "Tài khoản tồn tại nhưng nhập sai mật khẩu",
      "username": "active_user",
      "password": "WrongPassword",
      "expectedResult": false
    },
    {
      "id": "TC03_LOCKED_ACCOUNT",
      "description": "Tài khoản hợp lệ nhưng trạng thái tài khoản đang bị khóa",
      "username": "locked_user",
      "password": "SecurePassword123",
      "expectedResult": false
    },
    {
      "id": "TC04_SQL_INJECTION",
      "description": "Trường username chứa chuỗi injection",
      "username": "admin' OR '1'='1",
      "password": "somepassword",
      "expectedResult": false
    },
    {
      "id": "TC05_EMPTY_PASSWORD",
      "description": "Trường password để trống hoặc null",
      "username": "active_user",
      "password": "",
      "expectedResult": false
    }
  ]
}
```

## 3. Mã nguồn JUnit 5 do AI sinh ra
```java
package com.techshop.auth;

import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.mockito.Mockito;

import static org.junit.jupiter.api.Assertions.*;
import static org.mockito.Mockito.*;

public class UserAuthenticationTest {

    private UserAuthentication authService;

    @BeforeEach
    void setUp() {
        // Khởi tạo Mock cho class UserAuthentication để giả định hành vi login
        authService = mock(UserAuthentication.class);

        // Giả định hành vi (Stubbing) cho kịch bản 1: Thông tin đúng
        when(authService.login("active_user", "SecurePassword123")).thenReturn(true);

        // Giả định hành vi (Stubbing) cho kịch bản 2: Sai mật khẩu
        when(authService.login("active_user", "WrongPassword")).thenReturn(false);

        // Giả định hành vi (Stubbing) cho kịch bản 3: Tài khoản bị khóa
        when(authService.login("locked_user", "SecurePassword123")).thenReturn(false);

        // Giả định hành vi (Stubbing) cho kịch bản 4: SQL Injection
        when(authService.login("admin' OR '1'='1", "somepassword")).thenReturn(false);

        // Giả định hành vi (Stubbing) cho kịch bản 5: Mật khẩu rỗng
        when(authService.login("active_user", "")).thenReturn(false);
        when(authService.login("active_user", null)).thenReturn(false);
    }

    @Test
    @DisplayName("TC01 - Đăng nhập thành công với tài khoản và mật khẩu hợp lệ")
    void testLogin_Success() {
        boolean result = authService.login("active_user", "SecurePassword123");
        assertTrue(result, "Đăng nhập phải thành công với thông tin đúng");
    }

    @Test
    @DisplayName("TC02 - Đăng nhập thất bại do nhập sai mật khẩu")
    void testLogin_WrongPassword() {
        boolean result = authService.login("active_user", "WrongPassword");
        assertFalse(result, "Đăng nhập phải thất bại khi sai mật khẩu");
    }

    @Test
    @DisplayName("TC03 - Đăng nhập thất bại do tài khoản bị khóa")
    void testLogin_LockedAccount() {
        boolean result = authService.login("locked_user", "SecurePassword123");
        assertFalse(result, "Đăng nhập phải thất bại khi tài khoản bị khóa");
    }

    @Test
    @DisplayName("TC04 - Đăng nhập thất bại khi đầu vào có chứa SQL Injection")
    void testLogin_SqlInjectionAttack() {
        boolean result = authService.login("admin' OR '1'='1", "somepassword");
        assertFalse(result, "Đăng nhập phải thất bại để ngăn chặn SQL Injection");
    }

    @Test
    @DisplayName("TC05 - Đăng nhập thất bại khi mật khẩu rỗng hoặc null")
    void testLogin_EmptyPassword() {
        boolean resultEmpty = authService.login("active_user", "");
        boolean resultNull = authService.login("active_user", null);

        assertFalse(resultEmpty, "Đăng nhập phải thất bại khi mật khẩu rỗng");
        assertFalse(resultNull, "Đăng nhập phải thất bại khi mật khẩu null");
    }
}
```
