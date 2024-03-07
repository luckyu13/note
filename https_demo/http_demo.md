# 基于 C++ 的 HTTPS 服务器案例
## 创建公私密钥
* 生成私钥
```
openssl genrsa -out server.key 2048
```
> **openssl**: 这是 OpenSSL 工具的名称，用于执行各种加密操作。
>  **genrsa**: 这是生成 RSA 密钥对的命令选项。它告诉 OpenSSL 要生成一个 RSA 密钥对。
> **-out server.key**: 这是输出私钥的选项。server.key 是要生成的私钥文件的名称。您可以根据需要更改文件名。
> **2048**: 这是密钥的长度。在这种情况下，2048 指定了生成的 RSA 密钥的位数。2048 位密钥是目前普遍认为是足够安全的标准长度。

* 生成证书签名请求 (CSR)，使用生成的私钥生成证书签名请求文件 (server.csr)，命令如下
```
openssl req -new -key server.key -out server.csr
```

* 生成自签名证书, 使用生成的 CSR 文件和私钥生成一个自签名证书 (server.crt)
```
openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.crt
```

## Cpp Https Server Demo
code as follow:
```
#include <iostream>
#include <string>
#include <openssl/ssl.h>
#include <openssl/err.h>

#define SERVER_CERT_FILE "server.crt"
#define SERVER_KEY_FILE "server.key"
#define SERVER_CA_FILE "ca.crt"
#define SERVER_PORT 4433

void init_openssl() {
    SSL_load_error_strings();
    OpenSSL_add_ssl_algorithms();
}

void cleanup_openssl() {
    EVP_cleanup();
}

SSL_CTX* create_context() {
    const SSL_METHOD *method;
    SSL_CTX *ctx;

    method = SSLv23_server_method();

    ctx = SSL_CTX_new(method);
    if (!ctx) {
        perror("Unable to create SSL context");
        ERR_print_errors_fp(stderr);
        exit(EXIT_FAILURE);
    }

    return ctx;
}

void configure_context(SSL_CTX* ctx) {
    SSL_CTX_set_ecdh_auto(ctx, 1);

    // Load the server certificate into the SSL context
    if (SSL_CTX_use_certificate_file(ctx, SERVER_CERT_FILE, SSL_FILETYPE_PEM) <= 0) {
        ERR_print_errors_fp(stderr);
        exit(EXIT_FAILURE);
    }

    // Load the private key into the SSL context
    if (SSL_CTX_use_PrivateKey_file(ctx, SERVER_KEY_FILE, SSL_FILETYPE_PEM) <= 0) {
        ERR_print_errors_fp(stderr);
        exit(EXIT_FAILURE);
    }
}

int main() {
    SSL_CTX *ctx;
    int server_fd;
    struct sockaddr_in server_addr;
    socklen_t server_len = sizeof(server_addr);
    int client_fd;
    struct sockaddr_in client_addr;
    socklen_t client_len = sizeof(client_addr);
    char buffer[1024];
    int bytes;

    init_openssl();
    ctx = create_context();
    configure_context(ctx);

    server_fd = socket(AF_INET, SOCK_STREAM, 0);
    server_addr.sin_family = AF_INET;
    server_addr.sin_addr.s_addr = INADDR_ANY;
    server_addr.sin_port = htons(SERVER_PORT);

    if (bind(server_fd, (struct sockaddr*)&server_addr, sizeof(server_addr)) != 0) {
        perror("Unable to bind address");
        exit(EXIT_FAILURE);
    }

    if (listen(server_fd, 10) != 0) {
        perror("Unable to listen");
        exit(EXIT_FAILURE);
    }

    while (1) {
        printf("Waiting for connection...\n");
        client_fd = accept(server_fd, (struct sockaddr*)&client_addr, &client_len);
        printf("Client connected.\n");

        SSL *ssl = SSL_new(ctx);
        SSL_set_fd(ssl, client_fd);
        if (SSL_accept(ssl) <= 0) {
            ERR_print_errors_fp(stderr);
        } else {
            SSL_read(ssl, buffer, sizeof(buffer));
            std::cout << "Received: " << buffer << std::endl;
            std::string response = "HTTP/1.1 200 OK\nContent-Type: text/plain\nContent-Length: 12\n\nHello World!";
            SSL_write(ssl, response.c_str(), response.length());
        }

        SSL_shutdown(ssl);
        SSL_free(ssl);
        close(client_fd);
    }

    close(server_fd);
    cleanup_openssl();
}
```