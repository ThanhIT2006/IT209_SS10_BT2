# Bài 2: Đóng gói ứng dụng Web Backend bằng Single-stage Dockerfile

## 1. Mục tiêu

Xây dựng ứng dụng REST API đơn giản sử dụng Node.js và Express, sau đó đóng gói ứng dụng thành Docker Image bằng Single-stage Dockerfile.

Các yêu cầu chính:

- Sử dụng base image `node:18-alpine`.
- Thiết lập thư mục làm việc `/app`.
- Cài đặt thư viện Express trong quá trình build image.
- Khai báo biến môi trường `PORT=3000`.
- Khai báo cổng ứng dụng bằng `EXPOSE 3000`.
- Khởi chạy ứng dụng bằng lệnh `CMD` ở exec form.
- Build image `hw-backend:v1`.
- Chạy container `app-single-stage` với port mapping `8080:3000`.
- Kiểm tra ứng dụng thông qua HTTP request và xác nhận response JSON.

## 2. Công nghệ sử dụng

- Node.js 18
- Express 4
- Docker Engine
- Alpine Linux

## 3. Cấu trúc thư mục

```text
BT2/
├── Dockerfile
├── .dockerignore
├── package.json
├── server.js
└── README.md
```

## 4. Mã nguồn ứng dụng

Ứng dụng được xây dựng bằng Node.js và Express. File `package.json` khai báo thông tin dự án, script khởi chạy và thư viện Express.

File `server.js` định nghĩa API `GET /`. Khi nhận được yêu cầu, ứng dụng trả về JSON có nội dung:

```json
{
  "status": "success",
  "message": "Hello from Docker Single-stage!"
}
```

Cổng chạy mặc định của ứng dụng là `3000`, có thể được điều chỉnh thông qua biến môi trường `PORT`.

## 5. Nội dung Dockerfile

```dockerfile
FROM node:18-alpine

WORKDIR /app

COPY package*.json ./

RUN npm install --omit=dev

COPY server.js ./

ENV PORT=3000

EXPOSE 3000

CMD ["node", "server.js"]
```

### Giải thích các chỉ thị

| Chỉ thị | Chức năng |
|---|---|
| `FROM` | Sử dụng Node.js 18 trên Alpine Linux làm base image. |
| `WORKDIR` | Thiết lập thư mục làm việc là `/app`. |
| `COPY` | Sao chép file cấu hình và mã nguồn vào image. |
| `RUN` | Cài đặt các thư viện cần thiết, không cài đặt dependencies chỉ dành cho phát triển. |
| `ENV` | Khai báo biến môi trường `PORT=3000`. |
| `EXPOSE` | Khai báo cổng ứng dụng sử dụng trong container. |
| `CMD` | Khởi chạy ứng dụng bằng Node.js theo exec form. |

Dockerfile sử dụng một stage duy nhất. Toàn bộ quá trình cài đặt dependencies và chuẩn bị ứng dụng được thực hiện trong cùng một stage.

## 6. Build Docker Image

Thực hiện lệnh sau tại thư mục chứa Dockerfile:

```bash
docker build -t hw-backend:v1 .
```

Kiểm tra image sau khi build:

```bash
docker images
```

Image được tạo có tên `hw-backend` và tag `v1`.

## 7. Khởi chạy container

Khởi chạy ứng dụng trong chế độ nền:

```bash
docker run -d --name app-single-stage -p 8080:3000 hw-backend:v1
```

Trong đó:

- `-d`: chạy container ở chế độ nền.
- `--name app-single-stage`: đặt tên container.
- `-p 8080:3000`: ánh xạ cổng `8080` trên máy host tới cổng `3000` trong container.
- `hw-backend:v1`: image được sử dụng để khởi chạy container.

## 8. Kiểm tra hoạt động

### 8.1. Kiểm tra trạng thái container

Lệnh kiểm tra:

```bash
docker ps
```

Kết quả thực tế ghi nhận:

```text
CONTAINER ID   IMAGE           COMMAND                  CREATED         STATUS         PORTS                                         NAMES
d1afdadbba57   hw-backend:v1   "docker-entrypoint.s…"   4 seconds ago   Up 3 seconds   0.0.0.0:8080->3000/tcp, [::]:8080->3000/tcp   app-single-stage
```

Container `app-single-stage` có trạng thái `Up`, đồng thời cổng `8080` trên máy host được ánh xạ tới cổng `3000` trong container.

### 8.2. Kiểm tra log

Lệnh kiểm tra:

```bash
docker logs app-single-stage
```

Kết quả thực tế:

```text
Server listening on port 3000
```

Điều này xác nhận ứng dụng Node.js đã khởi động và đang lắng nghe trên cổng `3000`.

### 8.3. Kiểm tra REST API

Gửi HTTP request tới API:

```bash
curl -i http://localhost:8080/
```

Kết quả thực tế ghi nhận:

```text
HTTP/1.1 200 OK
X-Powered-By: Express
Content-Type: application/json; charset=utf-8
Content-Length: 64
ETag: W/"40-upZb7Fps+hszzRU9BoKtiCsmGCQ"
Date: Mon, 12 Oct 2026 10:24:14 GMT
Connection: keep-alive
Keep-Alive: timeout=5

{"status":"success","message":"Hello from Docker Single-stage!"}
```

Kết quả cho thấy API trả về HTTP `200 OK` cùng dữ liệu JSON đúng theo yêu cầu.
