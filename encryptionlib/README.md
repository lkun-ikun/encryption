# EncryptionLib

一个轻量级的 Android/Java 加密库，基于 AES/CBC/PKCS5Padding 对称加密算法，使用 PBKDF2WithHmacSHA1 进行密钥派生，内置自定义 Base64 编解码器，提供简洁优雅的加解密 API。

## 主要功能

- **AES 对称加密**：采用 `AES/CBC/PKCS5Padding` 算法，支持自定义密钥、盐值和 IV 向量
- **密钥派生**：通过 `PBKDF2WithHmacSHA1` + `SHA1` 摘要安全派生加密密钥
- **内置 Base64**：自实现 Base64 编解码，不依赖 Android SDK，兼容纯 Java 环境，支持多种模式（默认、无填充、无换行、URL 安全等）
- **Builder 模式**：链式配置密钥长度、字符集、迭代次数、摘要算法、安全随机数等参数，灵活且可读
- **同步/异步双模式**：同步方法直接返回结果，异步方法通过回调处理，不阻塞主线程
- **异常安全封装**：提供 `encryptOrNull` / `decryptOrNull` 方法，异常时返回 `null`，无需 try-catch
- **轻量无依赖**：纯 Java 实现，无任何外部依赖

## 引入项目

### Step 1. 添加 JitPack 仓库

在 **项目级** `settings.gradle` 中配置：

```groovy
dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
    repositories {
        // ... 其他仓库
        maven { url 'https://jitpack.io' }
    }
}
```

或 **项目级** `build.gradle` 中：

```groovy
allprojects {
    repositories {
        // ... 其他仓库
        maven { url 'https://jitpack.io' }
    }
}
```

### Step 2. 添加依赖

在 **模块级** `build.gradle` 中添加：

```groovy
dependencies {
    implementation 'com.github.lkun-ikun:encryption:0.0.5'
}
```

## 使用示例

### 快速开始

```java
String key = "your-secret-key";
String salt = "your-salt";
byte[] iv = new byte[16]; // 16 字节 IV
SecureRandom random = new SecureRandom();
random.nextBytes(iv);

// 创建加密器
Encryption encryption = Encryption.getDefault(key, salt, iv);

// 加密
String encrypted = encryption.encryptOrNull("Hello, EncryptionLib!");

// 解密
String decrypted = encryption.decryptOrNull(encrypted);
```

### 异步加解密

```java
encryption.encryptAsync("Hello, World!", new Encryption.Callback() {
    @Override
    public void onSuccess(String encrypted) {
        // 加密成功
    }

    @Override
    public void onError(Exception e) {
        // 处理异常
    }
});
```

### 自定义配置（Builder）

```java
Encryption encryption = new Encryption.Builder()
    .setKey("my-custom-key")
    .setSalt("my-custom-salt")
    .setIv(iv)
    .setAlgorithm("AES/CBC/PKCS5Padding")
    .setKeyAlgorithm("AES")
    .setKeyLength(256)                    // 256 位密钥
    .setCharsetName("UTF-8")
    .setIterationCount(1000)
    .setDigestAlgorithm("SHA-256")
    .setSecretKeyType("PBKDF2WithHmacSHA256")
    .setBase64Mode(2)                     // 0=DEFAULT, 1=NO_PADDING, 2=NO_WRAP, 8=URL_SAFE
    .setSecureRandomAlgorithm("SHA1PRNG")
    .build();
```

## API 概览

| 方法 | 说明 |
|------|------|
| `encrypt(String data)` | 加密，可能抛出受检异常 |
| `encryptOrNull(String data)` | 加密，异常时返回 `null` |
| `encryptAsync(String data, Callback cb)` | 异步加密，通过回调返回结果 |
| `decrypt(String data)` | 解密，可能抛出受检异常 |
| `decryptOrNull(String data)` | 解密，异常时返回 `null` |
| `decryptAsync(String data, Callback cb)` | 异步解密，通过回调返回结果 |
| `Encryption.getDefault(key, salt, iv)` | 使用默认配置快速创建实例 |

### Base64 模式常量

| 常量 | 值 | 说明 |
|------|----|------|
| `Base64.DEFAULT` | 0 | 标准 Base64，带填充和换行 |
| `Base64.NO_PADDING` | 1 | 无填充字符 `=` |
| `Base64.NO_WRAP` | 2 | 无换行符（推荐用于 URL 传输） |
| `Base64.CRLF` | 4 | 使用 CRLF 换行 |
| `Base64.URL_SAFE` | 8 | URL 安全模式（`+` → `-`, `/` → `_`） |

## 许可证

MIT License
