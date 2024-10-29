要在 Envoy 中实现对请求的 RSA 加密，您需要对 Envoy 的过滤器（filter）进行二次开发。通常，您可以编写自定义的网络或 HTTP 过滤器来对请求数据进行加密处理。以下是实现该功能的基本步骤：

## 1. 创建自定义过滤器
Envoy 支持使用 C++ 编写自定义过滤器。您需要创建一个新的过滤器来对请求进行 RSA 加密。

步骤：
选择过滤器类型：根据您的需求，可以选择创建一个网络过滤器或 HTTP 过滤器。

网络过滤器：适用于需要对原始 TCP 流数据进行加密的场景。
HTTP 过滤器：适用于需要对 HTTP 请求或响应体进行加密的场景。
创建过滤器文件：

在 source/extensions/filters/http/（或 network/）目录下创建一个新的子目录，比如 rsa_encryption.
在新目录下创建必要的源文件，比如 rsa_encryption_filter.h 和 rsa_encryption_filter.cc。
实现过滤器逻辑：

在过滤器的实现文件中，使用 OpenSSL 或其他加密库来进行 RSA 加密。
重写过滤器的回调函数，如 decodeHeaders、decodeData 等（如果是 HTTP 过滤器），以便在处理请求数据时进行 RSA 加密。
## 2. 使用 OpenSSL 库进行 RSA 加密
在您的过滤器实现中，使用 OpenSSL 库的 API 来执行 RSA 加密。例如：

```
#include <openssl/rsa.h>
#include <openssl/pem.h>
#include <openssl/err.h>

// 示例函数，用于对数据进行 RSA 加密
std::string rsaEncrypt(const std::string& data, const std::string& public_key_pem) {
    RSA* rsa = RSA_new();
    BIO* bio = BIO_new_mem_buf(public_key_pem.c_str(), -1);
    PEM_read_bio_RSA_PUBKEY(bio, &rsa, nullptr, nullptr);

    std::vector<unsigned char> encrypted(RSA_size(rsa));
    int result = RSA_public_encrypt(data.size(),
                                    reinterpret_cast<const unsigned char*>(data.c_str()),
                                    encrypted.data(),
                                    rsa,
                                    RSA_PKCS1_PADDING);
    
    RSA_free(rsa);
    BIO_free(bio);

    if (result == -1) {
        // 处理加密错误
        char err[130];
        ERR_load_crypto_strings();
        ERR_error_string(ERR_get_error(), err);
        throw std::runtime_error(std::string("RSA encryption failed: ") + err);
    }

    return std::string(encrypted.begin(), encrypted.end());
}
```

## 3. 注册并配置过滤器
在 source/extensions/filters/http/BUILD 文件中添加您的过滤器以便在构建时编译。
在 envoy.yaml 配置文件中，添加您的自定义过滤器到需要应用加密的监听器或路由中。
```
http_filters:
- name: envoy.filters.http.rsa_encryption
  typed_config:
    "@type": type.googleapis.com/envoy.extensions.filters.http.rsa_encryption.v3.RSAEncryption
```

## 4. 重新编译 Envoy
完成过滤器的实现后，使用 Bazel 重新编译 Envoy：
```
bazel build -c opt //source/exe:envoy-static
```
## 5. 验证和测试
启动 Envoy 并加载修改后的配置文件。
测试加密功能是否按预期工作。

# 总结
通过自定义过滤器，您可以在 Envoy 中实现对请求的 RSA 加密。需要编写新的过滤器代码，使用 OpenSSL 或其他库进行加密处理，然后将过滤器注册到 Envoy 配置中并重新编译。这样，您就可以实现对传入请求的动态加密功能。