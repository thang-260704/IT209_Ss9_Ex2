# Session 09 - Bài 2: Khởi chạy Web Server Nginx với Port Mapping

## 1. Mục tiêu

- Chạy Nginx dạng container ở chế độ ngầm (`-d`) bằng image nhẹ `nginx:alpine`.
- Ánh xạ cổng `8080` của máy chủ vào cổng `80` của container (`-p 8080:80`).
- Kiểm tra bằng `curl`, sau đó dừng và xoá container.

## 2. Môi trường

| Thành phần | Giá trị |
|---|---|
| Máy chủ | VPS Ubuntu 22.04.5 LTS (hostname `wean2607`) |
| Docker | Docker Engine - Community 29.9.0 |
| Image | `nginx:alpine` |
| Container | `my-web` |
| Port mapping | `8080` (host) → `80` (container), cả IPv4 `0.0.0.0` và IPv6 `[::]` |

## 3. Sơ đồ

```text
curl http://localhost:8080
        │
        ▼
 Host wean2607 :8080 ──(docker-proxy / iptables NAT)──► container my-web :80 (nginx)
```

## 4. Các bước thực hiện

### Bước 1 - Khởi chạy container

```bash
docker run -d -p 8080:80 --name my-web nginx:alpine
docker ps
```

| Tham số | Ý nghĩa |
|---|---|
| `-d` | Detached: container chạy ngầm, trả lại dấu nhắc ngay và in ra Container ID |
| `-p 8080:80` | Ánh xạ cổng `8080` của host vào cổng `80` mà Nginx lắng nghe trong container |
| `--name my-web` | Đặt tên dễ nhớ để `stop`/`rm` mà không cần gõ Container ID |
| `nginx:alpine` | Image Nginx dựa trên Alpine Linux, dung lượng nhỏ hơn nhiều so với bản Debian |

```text
Unable to find image 'nginx:alpine' locally
alpine: Pulling from library/nginx
...
Digest: sha256:df221db836e1754089190208cee7eeda94f233197056426eda74a43ab1abeac2
Status: Downloaded newer image for nginx:alpine
2f0b8e9c79a3863d10d95ec25507810094973937e371cc5364a76c1fe008d34f

$ docker ps
CONTAINER ID   IMAGE          COMMAND                  CREATED         STATUS         PORTS                                     NAMES
2f0b8e9c79a3   nginx:alpine   "/docker-entrypoint.…"   5 seconds ago   Up 4 seconds   0.0.0.0:8080->80/tcp, [::]:8080->80/tcp   my-web
```

Image chưa có sẵn nên Docker tự tải `nginx:alpine` từ Docker Hub rồi in ra Container ID. `docker ps` cho thấy `my-web` đang `Up` với cột PORTS `0.0.0.0:8080->80/tcp`.

![docker run và docker ps](images/01-run-ps.png)

### Bước 2 - Kiểm tra trang web

```bash
curl http://localhost:8080
```

```text
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, nginx is successfully installed and working.
Further configuration is required for the web server, reverse proxy,
API gateway, load balancer, content cache, or other features.</p>

<p>For online documentation and support please refer to
<a href="https://nginx.org/">nginx.org</a>.<br/>
To engage with the community please visit
<a href="https://community.nginx.org/">community.nginx.org</a>.<br/>
For enterprise grade support, professional services, additional
security features and capabilities please refer to
<a href="https://f5.com/nginx">f5.com/nginx</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

Phản hồi là trang mặc định của Nginx, có `<title>` và `<h1>` **Welcome to nginx!**, chứng tỏ request vào cổng 8080 của host đã tới được Nginx trong container.

![curl localhost:8080](images/02-curl.png)

### Bước 3 - Dọn dẹp

```bash
docker stop my-web && docker rm my-web
docker ps -a --filter name=my-web
```

```text
my-web
my-web

$ docker ps -a --filter name=my-web
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

- `docker stop` gửi `SIGTERM` để Nginx dừng gọn gàng (quá 10 giây sẽ gửi `SIGKILL`).
- `docker rm` xoá hẳn container đã dừng. Image `nginx:alpine` vẫn còn trong máy để dùng lại.
- `docker ps -a --filter name=my-web` chỉ còn dòng tiêu đề, chứng tỏ container đã bị xoá.

![Dọn dẹp](images/03-cleanup.png)

## 5. Kết luận

- Container `my-web` chạy ngầm từ image `nginx:alpine`, ánh xạ cổng `8080:80`.
- `curl http://localhost:8080` trả về trang HTML có dòng **"Welcome to nginx!"**.
- Container đã được dừng và xoá sau khi thử nghiệm.
