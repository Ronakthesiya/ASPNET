# URL Parameter Encryption and Decryption

## Overview
This guide explains how to implement URL parameter encryption and decryption in ASP.NET Core to secure sensitive data passed through URLs.

## Step 1: Understanding the UrlEncryptor Class

### 1.1 Class Structure
```csharp
public static class UrlEncryptor
```
- **Static class**: No instantiation needed, methods called directly
- **Purpose**: Centralized encryption/decryption functionality

### 1.2 Encryption Key Setup
```csharp
private static readonly string EncryptionKey = "pjsGLNYrMqU6wny4";
```
- **16-character key**: Required for AES encryption
- **Private readonly**: Prevents modification after initialization
- **Security note**: Change this key in production

### 1.3 Encrypt Method Breakdown
```csharp
public static string Encrypt(string text)
{
    using (var aesAlg = Aes.Create())
    {
        aesAlg.Key = Encoding.UTF8.GetBytes(EncryptionKey);
        aesAlg.IV = new byte[16]; 
        
        var encryptor = aesAlg.CreateEncryptor(aesAlg.Key, aesAlg.IV);
        
        using (var msEncrypt = new MemoryStream())
        {
            using (var csEncrypt = new CryptoStream(msEncrypt, encryptor, CryptoStreamMode.Write))
            using (var swEncrypt = new StreamWriter(csEncrypt))
            {
                swEncrypt.Write(text);
            }
            
            return Convert.ToBase64String(msEncrypt.ToArray());
        }
    }
}
```

**Step-by-step process:**
1. Create AES algorithm instance
2. Set encryption key from UTF8 bytes
3. Initialize IV (Initialization Vector) with 16 zero bytes
4. Create encryptor object
5. Create memory stream to hold encrypted data
6. Create crypto stream for encryption
7. Write plain text through stream writer
8. Convert encrypted bytes to Base64 string
9. Return URL-safe encrypted string

### 1.4 Decrypt Method Breakdown
```csharp
public static string Decrypt(string encryptedText)
{
    using (var aesAlg = Aes.Create())
    {
        aesAlg.Key = Encoding.UTF8.GetBytes(EncryptionKey);
        aesAlg.IV = new byte[16];
        
        var decryptor = aesAlg.CreateDecryptor(aesAlg.Key, aesAlg.IV);
        
        using (var msDecrypt = new MemoryStream(Convert.FromBase64String(encryptedText)))
        using (var csDecrypt = new CryptoStream(msDecrypt, decryptor, CryptoStreamMode.Read))
        using (var srDecrypt = new StreamReader(csDecrypt))
        {
            return srDecrypt.ReadToEnd();
        }
    }
}
```

**Step-by-step process:**
1. Create AES algorithm instance
2. Set same encryption key and IV
3. Create decryptor object
4. Convert Base64 string back to bytes
5. Create memory stream from encrypted bytes
6. Create crypto stream for decryption
7. Read decrypted text through stream reader
8. Return original plain text

## Step 2: Implementing Encryption in Views

### 2.1 View Implementation
```csharp
<a asp-route-ProductID="@UrlEncryptor.Encrypt(dr["ProductID"].ToString())">Delete</a>
```

**Process:**
1. Get ProductID from DataRow
2. Convert to string
3. Pass through UrlEncryptor.Encrypt()
4. Encrypted value becomes URL parameter
5. URL shows encrypted value instead of actual ID

**Example transformation:**
- Original: `ProductID=123`
- Encrypted: `ProductID=XyZ9AbC3DeF6GhI=`

## Step 3: Implementing Decryption in Controllers

### 3.1 Controller Action
```csharp
public IActionResult Delete(string ProductID)
{
    int decryptedProductID = Convert.ToInt32(UrlEncryptor.Decrypt(ProductID));
    // Use decryptedProductID for database operations
}
```

**Process:**
1. Receive encrypted ProductID from URL
2. Pass through UrlEncryptor.Decrypt()
3. Convert decrypted string to integer
4. Use actual ID for database operations

### 3.2 Complete Delete Action Flow
1. **Receive**: Encrypted ProductID parameter
2. **Decrypt**: Convert encrypted string to original ID
3. **Database**: Execute stored procedure with actual ID
4. **Response**: Redirect with success/error message

## Step 4: Security Benefits

### 4.1 URL Obfuscation
- Hides actual database IDs from users
- Prevents ID enumeration attacks
- Makes URLs non-guessable

### 4.2 Data Protection
- AES encryption provides strong security
- Base64 encoding ensures URL compatibility
- Consistent IV simplifies implementation
