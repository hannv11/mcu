# LED Blink (sáng 400 ms, tắt 800 ms)

## Đề bài

Lập trình vi điều khiển nhấp nháy LED: **sáng 400 ms, tắt 800 ms**, lặp lại liên tục.
Dòng điện chạy qua LED khoảng **10 mA**.

## Thông tin chung

| Mục | Nội dung |
|---|---|
| Vi điều khiển / board | *(ghi chip hoặc board bạn dùng, ví dụ ATmega328P / Arduino Uno)* |
| Điện áp cấp (VCC) | *(5 V hoặc 3.3 V)* |
| Chân điều khiển LED | *(ví dụ PB5 / D13)* |
| Công cụ | *(Arduino IDE / Microchip Studio / PlatformIO ...)* |
| Kiến thức | GPIO output, tạo trễ (delay), tính điện trở hạn dòng |

## Phân tích thời gian

| Thông số | Giá trị |
|---|---|
| Thời gian sáng (T_on) | 400 ms |
| Thời gian tắt (T_off) | 800 ms |
| Chu kỳ (T) | T_on + T_off = 1200 ms |
| Tần số (f) | 1 / 1,2 s ≈ 0,833 Hz |
| Duty cycle | 400 / 1200 ≈ 33,3 % |

## Tính điện trở hạn dòng

Dòng qua LED được quyết định bởi điện trở nối tiếp:

```
R = (VCC - VF) / I
```

- `VCC`: điện áp ở chân xuất ra mức cao (xấp xỉ điện áp cấp của chip)
- `VF`: điện áp thuận của LED (LED đỏ ≈ 2,0 V; xanh lá ≈ 2,1 V; xanh dương/trắng ≈ 3,0 V)
- `I`: dòng mong muốn = 10 mA = 0,01 A

### Trường hợp LED đỏ (VF ≈ 2,0 V)

| VCC | R tính được | Chọn R chuẩn | Dòng thực tế |
|---|---|---|---|
| 5 V | (5 - 2) / 0,01 = 300 Ω | **330 Ω** | 3 / 330 ≈ 9,1 mA |
| 3,3 V | (3,3 - 2) / 0,01 = 130 Ω | **120 Ω** | 1,3 / 120 ≈ 10,8 mA |

Chọn giá trị chuẩn gần nhất trong dãy E12, nên dòng thực tế lệch nhẹ so với 10 mA, vẫn đạt yêu cầu "khoảng 10 mA".

> Nếu dùng LED khác màu, thay `VF` tương ứng (xem datasheet LED hoặc đo bằng đồng hồ) rồi tính lại.

### Kiểm tra an toàn

- Công suất trên điện trở: P = I² x R ≈ (0,01)² x 330 ≈ 0,033 W, nên điện trở 1/4 W là dư.
- Dòng 10 mA nằm trong giới hạn của chân GPIO (ví dụ AVR cho phép tối đa 20 mA mỗi chân, nên 10 mA là an toàn).

## Sơ đồ nối dây

```
 Chân GPIO ----[ R ]----|>|---- GND
                       LED
              (330 Ω @ 5V / 120 Ω @ 3.3V)
```

- Chân dài (anode) của LED nối về phía điện trở, chân ngắn (cathode) nối GND.
- Điện trở có thể đặt trước hoặc sau LED đều được.

![schematic](docs/schematic.png)

## Nguyên lý hoạt động

1. Cấu hình chân LED làm **output**.
2. Lặp vô hạn:
   1. Xuất mức **cao** để LED sáng, rồi chờ **400 ms**.
   2. Xuất mức **thấp** để LED tắt, rồi chờ **800 ms**.

```
LED
 ON  ___         ___         ___
    |   |       |   |       |   |
 OFF|   |_______|   |_______|   |____
    0  400     1200 1600   2400 2800  (ms)
```

## Cấu trúc thư mục

```
led-blink/
├── README.md
├── src/
│   └── main.c        # (hoặc led-blink.ino)
└── docs/
    ├── schematic.png # sơ đồ nối dây
    └── result.gif    # ảnh/video kết quả
```

## Cách build và nạp

*(Điền theo công cụ bạn dùng, ví dụ:)*

1. Mở project trong IDE.
2. Chọn đúng board / chip và cổng nạp.
3. Build, rồi nạp (Upload / Flash).

## Kết quả

![result](docs/result.gif)

- [ ] LED sáng khoảng 0,4 s rồi tắt khoảng 0,8 s, lặp liên tục
- [ ] Đo dòng qua LED bằng đồng hồ vạn năng (chế độ mA, mắc nối tiếp): ≈ ___ mA
- [ ] (Tùy chọn) Đo thời gian bằng logic analyzer / oscilloscope: T_on = ___ ms, T_off = ___ ms

## Ghi chú

- Hàm delay kiểu blocking (`delay_ms`) thường chỉ chính xác tương đối. Muốn chính xác hơn, dùng **Timer** (bài sau trong mục Timer).
- Ghi lại lỗi đã gặp và điều rút ra ở đây.
