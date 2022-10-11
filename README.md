# Triển khai chứng thực Radius kết hợp LDAP với các thiết bị Cisco.
#### <a>1. Giới thiệu chung </a>
#### <a>2. Cài đặt </a>
#### <a>3. Vấn đề phát sinh </a>
<hr>

## 1. Giới thiệu chung
### a. Lý thuyết

Dịch vụ chứng thực và phân quyền truy cập mạng (AAA) bao gồm:
- Authentication: xác thực nhằm đảm bảo nhận dạng đúng người dùng.
- Authorization: giới hạn phân quyền người dùng có thể truy cập vào hệ thống
- Accounting: Giám sát những hoạt động của user trên hệ thống/thiết bị.

Có hai giao thức bảo mật chính trong dịch vụ AAA là TACACS(*Terminal Access Controller Access Control System*) và RADIUS(*Remote Authentication Dial-In User Service*). Tacacs do Cisco phát triển có môt vài điểm khác biệt so với Radius. Cả hai đều có thể sử dụng đế chứng thực Wifi, các thiết bị mạng,...

Một số phần mềm:
- Cisco Identity Services Engine: https://www.cisco.com/c/en/us/products/collateral/security/identity-services-engine/guide-c07-656177.html (bản trả phí, yêu cầu cấu hình khá cao tuy nhiên có khá nhiều tính năng).
- Open source: Sử dụng các file config để cấu hình (https://www.pro-bono-publico.de/projects/tac_plus.html)
- Một phiên bản có giao diện của Tacacs (có cả miễn phí và trả phí)https://tacacsgui.com/
- Một ứng dụng open source hỗ trợ Radius: https://freeradius.org/

Ở trong bài này chúng ta sẽ cấu hình bằng FreeRadius.
### b. Yêu cầu
- Các user có thể login đc vào các thiết bị Cisco bằng tài khoản LDAP.
- Phân quyền truy cập dựa trên nhóm các thiết bị và nhóm user trên LDAP.
- Nhật kí ghi login, logout.

Sơ đồ hoạt động:

![](https://www.golinuxcloud.com/wp-content/uploads/wireshark-255.png)


## 2. Cài đặt
#### a. Cấu hình cơ bản
FreeRadius sẽ chạy trên Linux, có thể sử dụng distro bất kì, trong bài viết này dùng CentOS 7.
```
yum install freeradius freeradius-utils freeradius-ldap
```
Các file cấu hình sẽ nằm trong thư mục: **/etc/raddb**

File cấu hình các thông số của radius server ở vị trí: **/etc/raddb/radiusd.conf**

FreeRadius quản lý server giống các web server như nginx và apache, file cấu hình sẽ nằm ở path **/etc/raddb/sites-enable** và đc symlink đến **/etc/raddb/sites-available**

Mặc định sẽ đọc config từ file **/etc/raddb/sites-enable/default**.

Bật tính năng xác thực với ldap

![Bật tính năng xác thực bằng LDAP](attachments/enableLDAPauthen.png)

Cấu hình LDAP authentication:

```
cp etc/raddb/mods-available/ldap etc/raddb/mods-available
```

Lưu ý các thông tin sau:

![Các thông tin](attachments/LDAPinformation.png)

Filter

![Sử dụng filter thích hợp](attachments/search_filter_for_user.png)

Các thông tin về NAS sẽ được lưu trong file **/etc/raddb/clients.conf**

Định nghĩa client mới như ví dụ dưới đây. Lưu ý 2 trường thông tin qua trọng là IP và secret.

![Ví dụ về client](attachments/config_NAS.png)

Chỉ định role network admin cho user authen bằng LDAP bằng cách thêm đoan code sau vào cuối file **/etc/raddb/clients.conf/users**

```
DEFAULT
#	Cisco-AVPair := "shell:priv-lvl=15"
	Cisco-AVPair := "shell:roles=\"network-admin\""
```
#### b. Cấu hình ghi log login logout

### 2.2 Trên các thiết bị
Cấu hình AAA:
```
configure terminal
aaa authentication login console group radius
aaa authentication login default group radius
```
Cấu hình Radius:
```
configure terminal
radius-server host <IP của Radius server> key 0 <Key được định nghĩa trong file clients.conf ở trên>
```
*Lưu ý: số ở phía sau từ **key** nằm trong danh sách sau (clear text (0), mã hóa (6) và (7))*



