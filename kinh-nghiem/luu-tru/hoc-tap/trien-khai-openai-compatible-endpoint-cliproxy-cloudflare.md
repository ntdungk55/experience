# Triển khai OpenAI-compatible endpoint bằng CLIProxyAPI và Cloudflare Tunnel

## Ngày

2026-08-19

## Mục tiêu

Tạo một endpoint HTTPS cố định, tương thích OpenAI Chat Completions, từ CLIProxyAPI đã xác thực OAuth. Endpoint đã triển khai:

```text
https://llm.mcp88.uk/v1
```

Endpoint này có thể được dùng bởi Python, Streamlit, OpenClaw hoặc client tương thích OpenAI API.

## Kiến trúc

```text
Client / OpenClaw
        |
        v
https://llm.mcp88.uk/v1
        |
        v
Cloudflare Tunnel
        |
        v
CLIProxyAPI on Windows :8317
        |
        v
OAuth credential for ChatGPT/Codex account
        |
        v
Model response
```

CLIProxyAPI là lớp proxy. Client dùng proxy API key do CLIProxyAPI quản lý; không dùng trực tiếp OAuth token hoặc API key của OpenAI Platform.

## Điều kiện chuẩn bị

- Windows máy chủ có thể chạy liên tục.
- Source code CLIProxyAPI tại `D:\Project\CLIProxyAPI`.
- Go đã cài để build server.
- Tài khoản ChatGPT/Codex đã hoàn tất OAuth và có quyền sử dụng model.
- Domain quản lý bằng Cloudflare và một Cloudflare Tunnel đã được tạo.
- Một proxy API key riêng. Không ghi key thật, OAuth credential, cookie hoặc Cloudflare token vào tài liệu/source code.

## Bước 1: Build CLIProxyAPI

Trong thư mục source:

```powershell
go build -o cli-proxy-api.exe ./cmd/server
```

Sau khi thay đổi mã Go, dùng:

```powershell
gofmt -w .
go test ./...
go build -o cli-proxy-api.exe ./cmd/server
```

## Bước 2: Cấu hình và xác thực OAuth

Tạo `config.yaml` từ cấu hình mẫu của repository. Các thông số quan trọng:

```yaml
host: 0.0.0.0
port: 8317
auth-dir: auths
api-keys:
  - cpa-REPLACE_WITH_PROXY_KEY
```

Ý nghĩa:

- `host: 0.0.0.0`: nhận kết nối từ mạng LAN; hữu ích khi Cloudflare Tunnel được cấu hình đến IP LAN.
- `port: 8317`: cổng local của proxy.
- `auth-dir: auths`: nơi CLIProxyAPI lưu credential OAuth.
- API key bắt đầu bằng `cpa-...`: key do proxy kiểm tra ở header `Authorization: Bearer ...`.

Thực hiện OAuth device login bằng cơ chế của CLIProxyAPI. Credential thành công sẽ nằm trong `auths/`. Không chia sẻ hoặc commit thư mục này.

## Bước 3: Chạy và test local endpoint

Khởi động server bằng config đã tạo. Base URL local có dạng:

```text
http://127.0.0.1:8317/v1
```

Hoặc, khi cần truy cập từ thiết bị khác cùng LAN:

```text
http://<LAN_IP>:8317/v1
```

Test danh sách model:

```powershell
$headers = @{ Authorization = "Bearer cpa-REPLACE_WITH_PROXY_KEY" }
Invoke-RestMethod -Uri "http://127.0.0.1:8317/v1/models" -Headers $headers
```

Test chat completion:

```powershell
$headers = @{
  Authorization = "Bearer cpa-REPLACE_WITH_PROXY_KEY"
  "Content-Type" = "application/json"
}

$body = @{
  model = "gpt-5.4"
  messages = @(
    @{ role = "user"; content = "Chào bạn" }
  )
} | ConvertTo-Json -Depth 5

Invoke-RestMethod `
  -Uri "http://127.0.0.1:8317/v1/chat/completions" `
  -Method Post `
  -Headers $headers `
  -Body $body
```

Kết quả thành công có object `chat.completion`, model trả lời, và usage tokens. Model `gpt-5.4` đã được xác nhận trả lời trong quá trình triển khai.

## Bước 4: Expose qua Cloudflare Tunnel

Tạo Public Hostname trong Cloudflare Tunnel:

```text
Hostname: llm.mcp88.uk
Service:  http://<LAN_IP>:8317
```

Sau đó URL public của API là:

```text
https://llm.mcp88.uk/v1
```

Ghi chú về origin service:

- Nếu `cloudflared` chạy ngay trên máy proxy, `http://127.0.0.1:8317` thường là lựa chọn an toàn hơn.
- Trong trường hợp giao diện Cloudflare hoặc cách triển khai không nhận localhost, bind proxy bằng `0.0.0.0` và dùng IP LAN của máy, ví dụ `http://192.168.x.x:8317`.
- Không expose port `8317` trực tiếp bằng port forwarding router khi Cloudflare Tunnel đã làm nhiệm vụ này.

## Bước 5: Test public endpoint

Test models qua HTTPS:

```powershell
Invoke-RestMethod `
  -Uri "https://llm.mcp88.uk/v1/models" `
  -Headers @{ Authorization = "Bearer cpa-REPLACE_WITH_PROXY_KEY" }
```

Test chat qua HTTPS bằng cách thay local base URL trong ví dụ trước:

```text
https://llm.mcp88.uk/v1/chat/completions
```

Kết quả thực tế: `GET /v1/models` và chat completion qua public endpoint đã trả về HTTP 200 và có nội dung trả lời.

## Kết nối client

Các client tương thích OpenAI dùng các giá trị sau:

```text
Base URL: https://llm.mcp88.uk/v1
API key:  cpa-... (proxy API key)
Model:    gpt-5.4
```

Ví dụ Python:

```python
from openai import OpenAI

client = OpenAI(
    base_url="https://llm.mcp88.uk/v1",
    api_key="cpa-REPLACE_WITH_PROXY_KEY",
)

response = client.chat.completions.create(
    model="gpt-5.4",
    messages=[{"role": "user", "content": "Chào bạn"}],
)

print(response.choices[0].message.content)
```

Ví dụ cấu hình OpenClaw custom provider:

```json5
{
  env: {
    vars: {
      CLIPROXY_API_KEY: "cpa-REPLACE_WITH_PROXY_KEY"
    }
  },
  agents: {
    defaults: {
      model: { primary: "cliproxy/gpt-5.4" }
    }
  },
  models: {
    mode: "merge",
    providers: {
      cliproxy: {
        baseUrl: "https://llm.mcp88.uk/v1",
        apiKey: "${CLIPROXY_API_KEY}",
        api: "openai-completions",
        models: [
          { id: "gpt-5.4", name: "GPT-5.4 via CLIProxy", input: ["text"] }
        ]
      }
    }
  }
}
```

Sau khi cấu hình, chạy:

```powershell
openclaw doctor
openclaw models list
```

Chỉ bật hoặc khai báo `supportsTools: true` sau khi đã test endpoint có hỗ trợ function/tool calling. Chat API chạy thành công chưa tự động chứng minh MCP tool calling hoạt động.

## Streaming

Để giảm cảm giác chờ lâu, client gửi `stream: true` và đọc Server-Sent Events từng phần. Streaming làm nội dung hiện ra sớm hơn; nó không làm model suy luận nhanh hơn.

## Sự cố đã gặp và cách xử lý

### Cloudflare route không nhận `127.0.0.1:8317`

**Biểu hiện:** Public Hostname không chấp nhận hoặc không kết nối được origin localhost.

**Cách xử lý:** Bind CLIProxyAPI vào `0.0.0.0`, kiểm tra endpoint qua IP LAN, sau đó cấu hình service Cloudflare Tunnel trỏ đến `http://<LAN_IP>:8317`.

### Lỗi font tiếng Việt trong PowerShell

**Biểu hiện:** Nội dung trả lời xuất hiện thành chuỗi như `ChÃ o DÅ©ng`.

**Nguyên nhân:** Encoding của console PowerShell không phải UTF-8.

**Cách xử lý:** Chạy trong PowerShell:

```powershell
[Console]::OutputEncoding = [System.Text.UTF8Encoding]::new()
$OutputEncoding = [Console]::OutputEncoding
```

### PowerShell không nhận `Ask-LLM-Stream`

**Nguyên nhân:** Function chỉ tồn tại trong PowerShell session đã khai báo.

**Cách xử lý:** Chạy lại đoạn định nghĩa function, hoặc lưu thành PowerShell profile/module rồi mở session mới.

### Node.js installer báo không tìm thấy Node hoặc lỗi 1603

**Nguyên nhân thực tế:** Node đã có trên máy nhưng shell cũ chưa nhận PATH; đồng thời ổ C gần đầy làm MSI báo `OutOfDiskSpace` và kết thúc bằng error 1603.

**Cách xử lý:** Mở PowerShell mới, kiểm tra `node --version`, và giải phóng dung lượng ổ C trước khi nâng cấp Node.

## Bảo mật và vận hành

- Không đưa `cpa-...` key, OAuth credential, cookie hay Cloudflare Tunnel token vào Git hoặc ảnh chụp màn hình.
- Tắt hoặc giới hạn management API của proxy khi không cần dùng từ xa.
- Chỉ chia sẻ endpoint cho client tin cậy; API key là lớp bảo vệ chính của proxy.
- Giữ process CLIProxyAPI và Cloudflare Tunnel chạy liên tục. Cân nhắc service manager/Task Scheduler để tự chạy lại sau reboot.
- Khi endpoint lỗi, kiểm tra theo thứ tự: process proxy -> local `/v1/models` -> Tunnel status -> public `/v1/models` -> API key -> OAuth credential.

## Checklist tái triển khai

1. Cài Go và build CLIProxyAPI.
2. Tạo `config.yaml`, chọn port và proxy API key.
3. Hoàn tất OAuth, xác nhận credential nằm trong `auths/`.
4. Khởi động proxy và test local `/v1/models`.
5. Test local `/v1/chat/completions` với Bearer proxy key.
6. Tạo Cloudflare Tunnel và Public Hostname trỏ đến local/LAN service.
7. Test `https://<domain>/v1/models` và chat completion qua HTTPS.
8. Cấu hình client/OpenClaw với base URL, proxy key và model.
9. Test streaming và tool calling riêng trước khi dùng cho automation.
10. Thiết lập log, restart strategy và quy tắc bảo mật.

## Bài học

- Endpoint là một chuỗi gồm proxy local, OAuth credential và Tunnel HTTPS; phải test từng lớp thay vì chỉ test URL public.
- Proxy API key khác hoàn toàn OAuth token của tài khoản.
- Cloudflare Tunnel tạo URL HTTPS cố định mà không cần mở port router.
- System prompt, chat UI, MCP/GameAgentGateway là các lớp độc lập; endpoint chỉ cung cấp năng lực gọi model.
