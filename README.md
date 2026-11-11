Hệ thống Điều khiển Nhiệt độ IoT (IoT Temperature Control System)

Giới thiệu tổng quan

Dự án Thiết kế hệ thống giám sát và điều khiển nhiệt độ IoT là một giải pháp thiết bị biên (Edge Device) hoạt động theo kiến trúc Headless. Hệ thống ứng dụng nhân hệ điều hành thời gian thực FreeRTOS trên vi điều khiển ESP32-S3 để xử lý thuật toán PID rời rạc. Kết hợp với hạ tầng mạng riêng ảo (Mesh VPN) thông qua nền tảng Tailscale, dự án cho phép giám sát và cấu hình từ xa an toàn qua Internet mà không cần can thiệp mở cổng (Port Forwarding) trên Router vật lý.

Tính năng nổi bật

Điều khiển thời gian thực đa lõi (Real-time Multicore Control): Tận dụng cấu trúc 2 lõi CPU của ESP32-S3. Core 0 được ghim cố định cho tác vụ MQTT và Telemetry để xử lý truyền thông, trong khi Core 1 chuyên trách thuật toán PID và đọc cảm biến để đảm bảo độ trễ tối thiểu cho vòng lặp điều khiển nhiệt độ.

Tối ưu hóa điều khiển (Optimized Logic): Hệ thống kết hợp thuật toán PID rời rạc với logic vùng biên để tự động điều chỉnh công suất và đảo chiều dòng điện. Đồng thời, chỉ trích xuất dữ liệu nhiệt độ từ cảm biến SHT31 nhằm tối ưu hóa tốc độ xử lý.

Bảo vệ phần cứng bất đồng bộ (Non-blocking Hardware Protection): Tích hợp cơ chế đếm lùi an toàn ngắt điện áp tải và chờ 5 giây trước khi lật trạng thái NÓNG/LẠNH để chống sốc nhiệt sò Peltier mà không làm nghẽn (block) luồng truyền thông MQTT.

Truyền thông phi tập trung (Decentralized Comms): Giao thức MQTT được thiết lập mức chất lượng dịch vụ QoS 1, đảm bảo gói tin điều khiển (Setpoint) và dữ liệu giám sát được truyền tải tin cậy ngay cả khi mạng chập chờn.

Xuyên thấu NAT & Bảo mật (NAT Traversal): Sử dụng Tailscale (dựa trên nền tảng giao thức mã hóa WireGuard) cài đặt trên Raspberry Pi, cung cấp IP ảo tĩnh (100.x.x.x) để quản trị an toàn từ mọi nơi.

Cấu trúc Phần cứng

Vi điều khiển trung tâm (MCU): ESP32-S3 DevKit (I2C: SDA=GPIO8, SCL=GPIO9; PWM: COOL=GPIO4, HEAT=GPIO5)

Máy chủ (IoT Gateway): Raspberry Pi 4 (Chạy Mosquitto Broker & Node-RED)

Cảm biến (Sensor): SHT31 (Giao tiếp I2C ở địa chỉ 0x44)

Động cơ chấp hành (Actuator): Sò nóng lạnh Peltier (TEC) 5V kết hợp hệ thống quạt và nhôm tản nhiệt

IC Công suất (Driver): Mạch cầu H DRV8833 (Nhận tín hiệu băm xung PWM ở tần số 18kHz, độ phân giải 12-bit để điều tiết dòng cho sò Peltier)

Cấu trúc Phần mềm

Hệ thống được thiết kế theo mô hình phân lớp (Layered Architecture) gồm 6 tầng rõ rệt:

Hardware: Lớp linh kiện vật lý (ESP32-S3, SHT31, DRV8833, Peltier).

HAL: Lớp trừu tượng hóa phần cứng (ESP-IDF I2C & LEDC PWM).

RTOS: Nhân FreeRTOS quản lý đa nhiệm đa lõi (Task_Control trên Core 1 & Task_Telemetry / MQTT_Event_Task trên Core 0).

Application: Xử lý thuật toán điều khiển PID và máy trạng thái bảo vệ đảo chiều nhiệt.

Communication: Ngăn xếp Wi-Fi và MQTT Client đóng gói JSON.

Cloud / Backend: Máy chủ Node-RED, Mosquitto Broker và Tailscale VPN.

Hướng dẫn Cài đặt & Biên dịch

1. Yêu cầu hệ thống

ESP-IDF: Phiên bản v5.x hoặc v6.x (hỗ trợ sẵn môi trường Docker qua .devcontainer).

MQTT Broker: Mosquitto chạy trên Raspberry Pi hoặc máy chủ nội bộ/Tailscale.

2. Cấu hình thông số mạng & MQTT

Mở trình cấu hình ESP-IDF để điền SSID/Password Wi-Fi và địa chỉ MQTT Broker (thay vì sửa trực tiếp vào mã nguồn):

idf.py menuconfig


Cấu hình Wi-Fi tại: Example Connection Configuration -> WiFi SSID / WiFi Password.

Cấu hình MQTT tại: Temp Control System Configuration -> MQTT Broker URL (ví dụ: mqtt://100.64.0.1:1883).

3. Biên dịch và Nạp chương trình

idf.py build
idf.py -p <PORT> flash monitor


4. Cấu trúc Topic MQTT

hethong/temp_control: ESP32 publish dữ liệu giám sát định kỳ với chu kỳ 1 giây/lần theo định dạng JSON:

{"temp": 28.45, "set": 30.0, "pwm": 3100, "mode": "HEAT"}


hethong/set_from_web: ESP32 subscribe để lắng nghe sự kiện thay đổi Setpoint (giá trị từ 10.0 đến 40.0 °C) từ người dùng thao tác qua Web Dashboard.

Giấy phép (License)

Dự án được phân phối dưới giấy phép MIT License.