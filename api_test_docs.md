# API Testing Documentation

## Base URL
```
http://localhost:8000/api/accounts/
```

## Authentication
This API uses JWT (JSON Web Token) authentication. Include the access token in the Authorization header:
```
Authorization: Bearer <access_token>
```

---

## 📋 Table of Contents
1. User Registration
2. User Login
3. Token Refresh
4. Get/Update Profile
5. Change Password
6. Logout

---

## 1. User Registration

### Endpoint
```
POST /register/
```

### Authentication Required
❌ No

### Request Body
```json
{
    "email": "user@example.com",
    "username": "johndoe",
    "password": "SecurePass123!",
    "password2": "SecurePass123!",
    "phone": "+8801712345678"
}
```

### Field Validation
| Field | Type | Required | Validation |
|-------|------|----------|------------|
| email | string | ✅ Yes | Must be unique |
| username | string | ✅ Yes | Must be unique |
| password | string | ✅ Yes | Django password validation |
| password2 | string | ✅ Yes | Must match password |
| phone | string | ❌ No | Max 15 characters |

### Success Response (201 Created)
```json
{
    "user": {
        "id": 1,
        "email": "user@example.com",
        "username": "johndoe",
        "phone": "+8801712345678"
    },
    "message": "User registered successfully",
    "tokens": {
        "refresh": "eyJ0eXAiOiJKV1QiLCJhbGc...",
        "access": "eyJ0eXAiOiJKV1QiLCJhbGc..."
    }
}
```

### Error Responses

**400 Bad Request - Password Mismatch**
```json
{
    "password": ["Password fields didn't match."]
}
```

**400 Bad Request - Email Already Exists**
```json
{
    "email": ["user with this email already exists."]
}
```

**400 Bad Request - Weak Password**
```json
{
    "password": [
        "This password is too short. It must contain at least 8 characters.",
        "This password is too common."
    ]
}
```

### cURL Example
```bash
curl -X POST http://localhost:8000/api/accounts/register/ \
  -H "Content-Type: application/json" \
  -d '{
    "email": "user@example.com",
    "username": "johndoe",
    "password": "SecurePass123!",
    "password2": "SecurePass123!",
    "phone": "+8801712345678"
  }'
```

---

## 2. User Login

### Endpoint
```
POST /login/
```

### Authentication Required
❌ No

### Request Body
```json
{
    "email": "user@example.com",
    "password": "SecurePass123!"
}
```

### Field Validation
| Field | Type | Required |
|-------|------|----------|
| email | string | ✅ Yes |
| password | string | ✅ Yes |

### Success Response (200 OK)
```json
{
    "refresh": "eyJ0eXAiOiJKV1QiLCJhbGc...",
    "access": "eyJ0eXAiOiJKV1QiLCJhbGc...",
    "user": {
        "id": 1,
        "email": "user@example.com",
        "username": "johndoe",
        "is_verified": false
    }
}
```

### Token Payload (Decoded Access Token)
```json
{
    "token_type": "access",
    "exp": 1705423200,
    "iat": 1705419600,
    "jti": "abc123...",
    "user_id": 1,
    "email": "user@example.com",
    "username": "johndoe",
    "is_verified": false
}
```

### Error Responses

**401 Unauthorized - Invalid Credentials**
```json
{
    "detail": "No active account found with the given credentials"
}
```

### cURL Example
```bash
curl -X POST http://localhost:8000/api/accounts/login/ \
  -H "Content-Type: application/json" \
  -d '{
    "email": "user@example.com",
    "password": "SecurePass123!"
  }'
```

---

## 3. Token Refresh

### Endpoint
```
POST /token/refresh/
```

### Authentication Required
❌ No (but requires valid refresh token)

### Request Body
```json
{
    "refresh": "eyJ0eXAiOiJKV1QiLCJhbGc..."
}
```

### Success Response (200 OK)
```json
{
    "access": "eyJ0eXAiOiJKV1QiLCJhbGc...",
    "refresh": "eyJ0eXAiOiJKV1QiLCJhbGc..."
}
```

### Error Responses

**401 Unauthorized - Invalid/Expired Token**
```json
{
    "detail": "Token is invalid or expired",
    "code": "token_not_valid"
}
```

### cURL Example
```bash
curl -X POST http://localhost:8000/api/accounts/token/refresh/ \
  -H "Content-Type: application/json" \
  -d '{
    "refresh": "eyJ0eXAiOiJKV1QiLCJhbGc..."
  }'
```

---

## 4. Get/Update Profile

### Get Profile

#### Endpoint
```
GET /profile/
```

#### Authentication Required
✅ Yes (Bearer Token)

#### Success Response (200 OK)
```json
{
    "id": 1,
    "email": "user@example.com",
    "username": "johndoe",
    "phone": "+8801712345678",
    "bio": "Software Developer",
    "date_of_birth": "1990-01-15",
    "address": "123 Main Street",
    "city": "Dhaka",
    "country": "Bangladesh",
    "created_at": "2024-01-10T10:30:00Z",
    "updated_at": "2024-01-15T14:20:00Z",
    "is_verified": false
}
```

#### cURL Example
```bash
curl -X GET http://localhost:8000/api/accounts/profile/ \
  -H "Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGc..."
```

---

### Update Profile

#### Endpoint
```
PUT /profile/
```
or
```
PATCH /profile/
```

#### Authentication Required
✅ Yes (Bearer Token)

#### Request Body (All fields optional for PATCH)
```json
{
    "username": "john_doe_updated",
    "phone": "+8801798765432",
    "bio": "Full Stack Developer | Tech Enthusiast",
    "date_of_birth": "1990-01-15",
    "address": "456 New Street, Apartment 5B",
    "city": "Dhaka",
    "country": "Bangladesh"
}
```

#### Read-Only Fields
These fields cannot be updated via this endpoint:
- `email`
- `created_at`
- `updated_at`
- `is_verified`

#### Success Response (200 OK)
```json
{
    "id": 1,
    "email": "user@example.com",
    "username": "john_doe_updated",
    "phone": "+8801798765432",
    "bio": "Full Stack Developer | Tech Enthusiast",
    "date_of_birth": "1990-01-15",
    "address": "456 New Street, Apartment 5B",
    "city": "Dhaka",
    "country": "Bangladesh",
    "created_at": "2024-01-10T10:30:00Z",
    "updated_at": "2024-01-16T09:15:00Z",
    "is_verified": false
}
```

#### Error Responses

**401 Unauthorized - No Token**
```json
{
    "detail": "Authentication credentials were not provided."
}
```

**401 Unauthorized - Invalid Token**
```json
{
    "detail": "Given token not valid for any token type",
    "code": "token_not_valid"
}
```

#### cURL Example
```bash
curl -X PATCH http://localhost:8000/api/accounts/profile/ \
  -H "Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGc..." \
  -H "Content-Type: application/json" \
  -d '{
    "bio": "Full Stack Developer",
    "city": "Dhaka"
  }'
```

---

## 5. Change Password

### Endpoint
```
POST /change-password/
```

### Authentication Required
✅ Yes (Bearer Token)

### Request Body
```json
{
    "old_password": "SecurePass123!",
    "new_password": "NewSecurePass456!",
    "new_password2": "NewSecurePass456!"
}
```

### Field Validation
| Field | Type | Required | Validation |
|-------|------|----------|------------|
| old_password | string | ✅ Yes | Must match current password |
| new_password | string | ✅ Yes | Django password validation |
| new_password2 | string | ✅ Yes | Must match new_password |

### Success Response (200 OK)
```json
{
    "message": "Password changed successfully"
}
```

### Error Responses

**400 Bad Request - Password Mismatch**
```json
{
    "new_password": ["Password fields didn't match."]
}
```

**400 Bad Request - Wrong Old Password**
```json
{
    "old_password": ["Old password is incorrect."]
}
```

**400 Bad Request - Weak New Password**
```json
{
    "new_password": [
        "This password is too short. It must contain at least 8 characters."
    ]
}
```

### cURL Example
```bash
curl -X POST http://localhost:8000/api/accounts/change-password/ \
  -H "Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGc..." \
  -H "Content-Type: application/json" \
  -d '{
    "old_password": "SecurePass123!",
    "new_password": "NewSecurePass456!",
    "new_password2": "NewSecurePass456!"
  }'
```

### Important Notes
⚠️ After changing password:
- User will need to login again
- All existing tokens remain valid until expiry
- Consider implementing token blacklisting for better security

---

## 6. Logout

### Endpoint
```
POST /logout/
```

### Authentication Required
✅ Yes (Bearer Token)

### Request Body
```json
{
    "refresh": "eyJ0eXAiOiJKV1QiLCJhbGc..."
}
```

### Success Response (200 OK)
```json
{
    "message": "Logout successful"
}
```

### Error Responses

**400 Bad Request - Invalid Token**
```json
{
    "error": "Invalid token"
}
```

### cURL Example
```bash
curl -X POST http://localhost:8000/api/accounts/logout/ \
  -H "Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGc..." \
  -H "Content-Type: application/json" \
  -d '{
    "refresh": "eyJ0eXAiOiJKV1QiLCJhbGc..."
  }'
```

### Important Notes
⚠️ **Token Blacklisting Setup Required**

For logout to work properly, you need to:

1. Add `rest_framework_simplejwt.token_blacklist` to INSTALLED_APPS:
```python
INSTALLED_APPS = [
    ...
    'rest_framework_simplejwt.token_blacklist',
]
```

2. Run migrations:
```bash
python manage.py migrate
```

3. Enable token rotation in settings.py:
```python
SIMPLE_JWT = {
    'BLACKLIST_AFTER_ROTATION': True,
    'ROTATE_REFRESH_TOKENS': True,
}
```

---

## Testing Workflow

### 1. Complete User Journey Test
```bash
# Step 1: Register
curl -X POST http://localhost:8000/api/accounts/register/ \
  -H "Content-Type: application/json" \
  -d '{"email":"test@example.com","username":"testuser","password":"Test123!@#","password2":"Test123!@#"}'

# Step 2: Login (if needed)
curl -X POST http://localhost:8000/api/accounts/login/ \
  -H "Content-Type: application/json" \
  -d '{"email":"test@example.com","password":"Test123!@#"}'

# Step 3: Get Profile
curl -X GET http://localhost:8000/api/accounts/profile/ \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN"

# Step 4: Update Profile
curl -X PATCH http://localhost:8000/api/accounts/profile/ \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"bio":"Test bio","city":"Dhaka"}'

# Step 5: Change Password
curl -X POST http://localhost:8000/api/accounts/change-password/ \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"old_password":"Test123!@#","new_password":"NewTest456!@#","new_password2":"NewTest456!@#"}'

# Step 6: Refresh Token
curl -X POST http://localhost:8000/api/accounts/token/refresh/ \
  -H "Content-Type: application/json" \
  -d '{"refresh":"YOUR_REFRESH_TOKEN"}'

# Step 7: Logout
curl -X POST http://localhost:8000/api/accounts/logout/ \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"refresh":"YOUR_REFRESH_TOKEN"}'
```

---

## Security Notes

1. **HTTPS Only**: Always use HTTPS in production
2. **Token Storage**: Store tokens securely (httpOnly cookies recommended)
3. **Token Expiry**: Configure appropriate token lifetimes in settings
4. **Password Policy**: Enforce strong password requirements
5. **Rate Limiting**: Implement rate limiting on accounts endpoints
6. **CORS**: Configure CORS properly for your frontend domain

---

## Environment Variables

```env
SECRET_KEY=your-secret-key-here
DEBUG=False
ALLOWED_HOSTS=yourdomain.com

# JWT Settings
JWT_ACCESS_TOKEN_LIFETIME=5  # minutes
JWT_REFRESH_TOKEN_LIFETIME=1  # days
```

---

## HTTP Status Codes

| Code | Meaning | When Used |
|------|---------|-----------|
| 200 | OK | Successful GET, PUT, PATCH requests |
| 201 | Created | Successful POST (registration) |
| 400 | Bad Request | Validation errors, invalid data |
| 401 | Unauthorized | Invalid/missing credentials or token |
| 404 | Not Found | Resource doesn't exist |
| 500 | Server Error | Unexpected server errors |

---

## Common Issues & Solutions

### Issue: "Authentication credentials were not provided"
**Solution**: Include `Authorization: Bearer <token>` header

### Issue: "Token is invalid or expired"
**Solution**: Use refresh token to get new access token

### Issue: "No active account found with the given credentials"
**Solution**: Check email/password, verify user exists

### Issue: Logout returns "Invalid token"
**Solution**: Ensure token blacklist is set up and refresh token is valid

---

## Notes for Frontend Developers

1. **Store Both Tokens**: Save both access and refresh tokens
2. **Auto-Refresh**: Implement automatic token refresh before expiry
3. **Error Handling**: Handle 401 errors by refreshing token
4. **Logout**: Send both tokens when logging out
5. **Registration**: Tokens are returned after successful registration

---

## Postman Collection Variables

Set these variables in your Postman environment:

```
base_url: http://localhost:8000/api/accounts
access_token: (auto-set from login response)
refresh_token: (auto-set from login response)
```

---

## Support

For issues or questions, contact the development team or create an issue in the repository.