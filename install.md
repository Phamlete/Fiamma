# Hướng Dẫn Cài Đặt Node Fiamma

## 1. Yêu Cầu Hệ Thống

Trước khi bắt đầu, hãy đảm bảo hệ thống của bạn đáp ứng các yêu cầu sau:

- **CPU:** Quad Core hoặc lớn hơn (AMD hoặc Intel, amd64)
- **RAM:** 32GB
- **Lưu Trữ:** 1TB NVMe SSD (I/O đĩa rất quan trọng)
- **Kết Nối Internet:** 100Mbps bi-directional

## 2. Cài Đặt Go

Fiamma yêu cầu Go để biên dịch phần mềm. Cài đặt Go bằng cách thực hiện các bước sau:

```bash
2. Cài Đặt Các Công Cụ Cần Thiết
Cài đặt các công cụ cần thiết như git, make, và build-essential:
sudo apt update
sudo apt install -y git make build-essential

3. Cài đặt go
cd $HOME
sudo rm -rf go
wget https://golang.org/dl/go1.22.5.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go1.22.5.linux-amd64.tar.gz
echo "export PATH=\$PATH:/usr/local/go/bin" >> ~/.bashrc
source ~/.bashrc 

4. Clone Repository và Cài Đặt Fiamma
git clone https://github.com/fiamma-chain/fiamma.git
cd fiamma
git checkout v0.2.1
make install

5. Khởi Tạo Thư Mục Node
Khởi tạo thư mục cấu hình cho node Fiamma:
# Thay thế $NODENAME bằng tên node của bạn
cd $HOME
fiammad init $NODENAME --chain-id fiamma-testnet-1
Tải xuống tệp genesis và đặt nó vào thư mục cấu hình node:
wget https://raw.githubusercontent.com/fiamma-chain/networks/main/fiamma-testnet-1/genesis.json -O ~/.fiamma/config/genesis.json

6. Cập Nhật Cổng Mặc Định
Bạn có thể thay đổi cổng bằng cách sử dụng biến hoặc trực tiếp thay đổi số cổng. Dưới đây là các lệnh sed để thay đổi cổng:
# Thay thế các cổng mặc định trong config.toml
sed -i 's/^laddr = "tcp:\/\/0.0.0.0:26656"/laddr = "tcp:\/\/0.0.0.0:37656"/' ~/.fiamma/config/config.toml
sed -i 's/^laddr = "tcp:\/\/0.0.0.0:26657"/laddr = "tcp:\/\/0.0.0.0:37657"/' ~/.fiamma/config/config.toml
sed -i 's/^laddr = "tcp:\/\/0.0.0.0:26658"/laddr = "tcp:\/\/0.0.0.0:37658"/' ~/.fiamma/config/config.toml
sed -i 's/^laddr = "tcp:\/\/0.0.0.0:6060"/laddr = "tcp:\/\/0.0.0.0:37060"/' ~/.fiamma/config/config.toml

# Cập nhật các cổng trong app.toml
sed -i 's/^laddr = "tcp:\/\/0.0.0.0:1317"/laddr = "tcp:\/\/0.0.0.0:37317"/' ~/.fiamma/config/app.toml
sed -i 's/^laddr = "tcp:\/\/0.0.0.0:9090"/laddr = "tcp:\/\/0.0.0.0:37090"/' ~/.fiamma/config/app.toml

# Cập nhật các cổng trong client.toml
sed -i 's/^laddr = "tcp:\/\/0.0.0.0:26657"/laddr = "tcp:\/\/0.0.0.0:37657"/' ~/.fiamma/config/client.toml

7. Cập Nhật Cấu Hình Peers và Seed Nodes
Sửa các thuộc tính seeds và persistent_peers trong tệp ~/.fiamma/config/config.toml:
sed -i 's/^seeds = .*/seeds = "5d6828849a45cf027e035593d8790bc62aca9cef@18.182.20.173:37656,526d13f3ce3e0b56fa3ac26a48f231e559d4d60c@35.73.202.182:37656"/' ~/.fiamma/config/config.toml
sed -i 's/^persistent_peers = .*/persistent_peers = "5d6828849a45cf027e035593d8790bc62aca9cef@18.182.20.173:37656,526d13f3ce3e0b56fa3ac26a48f231e559d4d60c@35.73.202.182:37656"/' ~/.fiamma/config/config.toml

8. Cập Nhật Giá Gas Tối Thiểu
Sửa thuộc tính minimum-gas-prices trong tệp ~/.fiamma/config/app.toml:
sed -i 's/^minimum-gas-prices = .*/minimum-gas-prices = "0.00001ufia"/' ~/.fiamma/config/app.toml

9. Cài Đặt và Cấu Hình Cosmovisor
Cosmovisor giúp tự động quản lý các bản cập nhật của ứng dụng. Cài đặt Cosmovisor và cấu hình nó:
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@latest

mkdir -p ~/.fiamma/cosmovisor
mkdir -p ~/.fiamma/cosmovisor/genesis/bin
mkdir -p ~/.fiamma/cosmovisor/upgrades

cp $GOPATH/bin/fiammad ~/.fiamma/cosmovisor/genesis/bin/fiammad

10. Tạo dịch vụ hệ thống cho Cosmovisor:
sudo tee /etc/systemd/system/fiamma.service > /dev/null <<EOF
[Unit]
Description=Fiamma daemon
After=network-online.target

[Service]
User=$USER
ExecStart=$(which cosmovisor) run start --x-crisis-skip-assert-invariants
Restart=always
RestartSec=3
LimitNOFILE=infinity

Environment="DAEMON_NAME=fiammad"
Environment="DAEMON_HOME=${HOME}/.fiamma"
Environment="DAEMON_RESTART_AFTER_UPGRADE=true"
Environment="DAEMON_ALLOW_DOWNLOAD_BINARIES=false"

[Install]
WantedBy=multi-user.target
EOF

11. Khởi Động Node
Khởi động dịch vụ Fiamma và kiểm tra trạng thái:
sudo systemctl daemon-reload
sudo systemctl enable fiamma
sudo systemctl start fiamma

12. Kiểm tra trạng thái của node:
systemctl status fiamma

13. Xem nhật ký của dịch vụ:
journalctl -u fiamma -f



