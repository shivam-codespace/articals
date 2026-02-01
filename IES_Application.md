# 👤 ADMIN USER SETUP GUIDE

## 🎯 PROBLEM SOLVED

You now have **3 ways** to create the initial admin user with BCrypt-encrypted password!

---

## ✅ SOLUTION 1: AUTOMATIC (RECOMMENDED)

### Auto-Create Admin on Startup

**What happens:**
- When you start the application, it automatically checks if admin exists
- If no admin found, it creates one with encrypted password
- No manual work needed!

**How it works:**
- File: `DataInitializer.java` (already created)
- Runs automatically on application startup
- Creates admin only if database is empty

**Steps:**
1. Start your application:
   ```bash
   cd 01-Admin-Api
   ./mvnw spring-boot:run
   ```

2. Check console logs:
   ```
   ✅ Default admin user created successfully!
   📧 Email: admin@ies.com
   🔑 Password: Admin@123
   ⚠️  Please change this password after first login!
   ```

3. Login with these credentials:
   - **Email:** `admin@ies.com`
   - **Password:** `Admin@123`

**That's it!** ✅

---

## ✅ SOLUTION 2: REST API ENDPOINT

### Create Admin via API Call

**Use this if:**
- Application is already running
- You want to create admin manually
- Automatic creation didn't work

### Method 1: Using Postman

**Endpoint:** `POST http://localhost:9091/bootstrap/create-admin`

**Headers:**
```
Content-Type: application/json
```

**No body needed!**

**Success Response:**
```json
{
  "status": "success",
  "message": "Admin user created successfully",
  "email": "admin@ies.com",
  "password": "Admin@123",
  "warning": "Please change this password after first login!"
}
```

### Method 2: Using cURL

```bash
curl -X POST http://localhost:9091/bootstrap/create-admin \
  -H "Content-Type: application/json"
```

### Method 3: Using Browser

Just open this URL in your browser:
```
http://localhost:9091/bootstrap/create-admin
```

---

## ✅ SOLUTION 3: MANUAL SQL INSERT

### Create Admin via Database

**Use this if:**
- You want to insert directly into database
- You need a custom admin user
- Other methods don't work

### Step 1: Generate Encrypted Password

**Option A: Using API**

**Endpoint:** `POST http://localhost:9091/bootstrap/encrypt-password`

**Request Body:**
```json
{
  "password": "YourPassword123"
}
```

**Response:**
```json
{
  "status": "success",
  "plainPassword": "YourPassword123",
  "encryptedPassword": "$2a$10$xyzABC123...",
  "usage": "Use this encrypted password in SQL INSERT statement"
}
```

**Option B: Using cURL**

```bash
curl -X POST http://localhost:9091/bootstrap/encrypt-password \
  -H "Content-Type: application/json" \
  -d '{"password":"Admin@123"}'
```

### Step 2: Insert into Database

Copy the `encryptedPassword` from response and use in SQL:

```sql
INSERT INTO ies_users (
    full_name, 
    email, 
    password, 
    Mobile_No, 
    gender, 
    date_of_birth, 
    SSN, 
    active_status, 
    active_switch, 
    role_id
) VALUES (
    'System Administrator',
    'admin@ies.com',
    '$2a$10$xyzABC123...', -- Use encrypted password from API
    9999999999,
    'Male',
    '1990-01-01',
    999999999,
    'UNLOCKED',
    'Y',
    1
);
```

---

## 🔐 DEFAULT ADMIN CREDENTIALS

After using any method above:

| Field | Value |
|-------|-------|
| **Email** | `admin@ies.com` |
| **Password** | `Admin@123` |
| **Role ID** | `1` (Admin) |
| **Status** | `UNLOCKED` |

---

## 🧪 TESTING ADMIN LOGIN

### Using Postman

**Endpoint:** `POST http://localhost:9091/login`

**Request Body:**
```json
{
  "email": "admin@ies.com",
  "pwd": "Admin@123"
}
```

**Expected Response:**
```json
{
  "status": "Success",
  "email": "admin@ies.com",
  "userId": 1,
  "roleId": 1,
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

### Using cURL

```bash
curl -X POST http://localhost:9091/login \
  -H "Content-Type: application/json" \
  -d '{
    "email": "admin@ies.com",
    "pwd": "Admin@123"
  }'
```

---

## 🔧 BOOTSTRAP ENDPOINTS

All bootstrap endpoints are **public** (no authentication required):

### 1. Create Admin User
```
POST http://localhost:9091/bootstrap/create-admin
```

### 2. Encrypt Password
```
POST http://localhost:9091/bootstrap/encrypt-password
Body: { "password": "YourPassword123" }
```

### 3. Health Check
```
GET http://localhost:9091/bootstrap/health
```

---

## 🎯 RECOMMENDED WORKFLOW

### First Time Setup:

1. **Start Application**
   ```bash
   cd 01-Admin-Api
   ./mvnw spring-boot:run
   ```

2. **Check Logs**
   - Look for: "✅ Default admin user created successfully!"
   - If you see this, admin is ready!

3. **Login**
   - Email: `admin@ies.com`
   - Password: `Admin@123`

4. **Change Password** (Important!)
   - After first login, create a new admin with secure password
   - Or update the default admin password

### If Auto-Creation Fails:

1. **Use API Endpoint**
   ```bash
   curl -X POST http://localhost:9091/bootstrap/create-admin
   ```

2. **Verify**
   ```bash
   curl -X POST http://localhost:9091/login \
     -H "Content-Type: application/json" \
     -d '{"email":"admin@ies.com","pwd":"Admin@123"}'
   ```

---

## 🐛 TROUBLESHOOTING

### Issue 1: "Admin user already exists"

**Cause:** Admin already created

**Solution:** Just login with existing credentials:
- Email: `admin@ies.com`
- Password: `Admin@123`

If you forgot password, use password reset or manually update in database.

---

### Issue 2: "BCrypt error" when inserting via SQL

**Cause:** Password not encrypted

**Solution:** 
1. Use the encrypt-password endpoint to get BCrypt hash
2. Use that hash in your SQL INSERT

**Example:**
```bash
# Step 1: Get encrypted password
curl -X POST http://localhost:9091/bootstrap/encrypt-password \
  -H "Content-Type: application/json" \
  -d '{"password":"Admin@123"}'

# Step 2: Copy the encryptedPassword from response
# Step 3: Use it in SQL INSERT
```

---

### Issue 3: "Cannot find bean PasswordEncoder"

**Cause:** Application not started properly

**Solution:**
1. Clean and rebuild:
   ```bash
   mvn clean install
   ```
2. Restart application

---

### Issue 4: Bootstrap endpoints return 404

**Cause:** SecurityConfig not updated

**Solution:** Already fixed! Bootstrap endpoints are now public.

---

## 🔒 SECURITY NOTES

### ⚠️ IMPORTANT: Production Security

1. **Disable Bootstrap Endpoints in Production**
   
   Add to `application.yml`:
   ```yaml
   bootstrap:
     enabled: false  # Set to false in production
   ```

2. **Change Default Password**
   - Never use `Admin@123` in production
   - Change immediately after first login

3. **Remove DataInitializer**
   - Comment out `@Component` annotation in production
   - Or use profile-specific configuration

4. **Secure Database**
   - Don't expose database to public
   - Use strong database passwords
   - Enable SSL for database connections

---

## 📊 VERIFICATION CHECKLIST

After setup, verify:

- [ ] Application starts without errors
- [ ] Admin user exists in database
- [ ] Password is BCrypt encrypted (starts with `$2a$` or `$2b$`)
- [ ] Login works with admin credentials
- [ ] JWT token is returned
- [ ] Dashboard loads for admin user
- [ ] Admin can create other users

---

## 🎓 UNDERSTANDING BCRYPT

### What is BCrypt?

BCrypt is a password hashing function that:
- Creates one-way encrypted passwords
- Adds random salt for security
- Makes brute-force attacks extremely difficult

### BCrypt Password Format

```
$2a$10$N9qo8uLOickgx2ZMRZoMyeIjZAgcfl7p92ldGxad68LJZdL17lhWy
│  │  │  └─────────────────────────────────────────────┘
│  │  │                     Hash (31 chars)
│  │  └─ Salt (22 chars)
│  └─ Cost factor (10 = 2^10 iterations)
└─ Algorithm version (2a)
```

### Why You Can't Just Copy Passwords

❌ **This won't work:**
```sql
-- Plain text password - LOGIN WILL FAIL!
INSERT INTO ies_users (password) VALUES ('Admin@123');
```

✅ **This will work:**
```sql
-- BCrypt encrypted password - LOGIN WILL SUCCEED!
INSERT INTO ies_users (password) 
VALUES ('$2a$10$N9qo8uLOickgx2ZMRZoMyeIjZAgcfl7p92ldGxad68LJZdL17lhWy');
```

---

## 🚀 QUICK START COMMANDS

### Option 1: Automatic (Just start app)
```bash
cd 01-Admin-Api
./mvnw spring-boot:run
# Admin created automatically!
```

### Option 2: API Endpoint
```bash
# Start app first, then:
curl -X POST http://localhost:9091/bootstrap/create-admin
```

### Option 3: Get Encrypted Password
```bash
curl -X POST http://localhost:9091/bootstrap/encrypt-password \
  -H "Content-Type: application/json" \
  -d '{"password":"Admin@123"}'
# Copy encrypted password and use in SQL
```

---

## ✅ SUCCESS!

If you can login with `admin@ies.com` / `Admin@123` and get a JWT token, you're all set! 🎉

**Next Steps:**
1. Test all endpoints with admin token
2. Create additional users
3. Create plans
4. Test complete workflow

---

**Need help?** Check the troubleshooting section above or review the logs!

