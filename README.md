# Panduan Backup, Restore, dan Konfigurasi Node Pipe Network

Dokumen ini berisi langkah-langkah untuk melakukan backup, restore, dan konfigurasi node Pipe Network di VPS.

## 1. Backup Node Pipe Network

Sebelum memindahkan node ke VPS baru, kita perlu membuat backup data yang penting.

### Langkah-langkah Backup:

1. Masuk ke VPS lama  
   ```bash
   ssh root@IP_VPS_LAMA
   ```

2. Hentikan service node (jika berjalan di systemd)  
   ```bash
   sudo systemctl stop pop.service
   ```

3. Buat backup file penting  
   ```bash
   tar -czvf /root/backup_pipe.tar.gz -C /root/pipenetwork node_info.json download_cache
   ```

4. Pindahkan file backup ke VPS baru  
   ```bash
   scp /root/backup_pipe.tar.gz root@IP_VPS_BARU:/root/
   ```

## 2. Restore Node di VPS Baru

1. Masuk ke VPS baru  
   ```bash
   ssh root@IP_VPS_BARU
   ```

2. Ekstrak file backup  
   ```bash
   mkdir -p /root/pipenetwork
   tar -xzvf /root/backup_pipe.tar.gz -C /root/pipenetwork/
   ```

3. Pastikan file sudah ada  
   ```bash
   ls -lh /root/pipenetwork/
   ```

## 3. Konfigurasi Node

1. Download versi terbaru Pipe POP  
   ```bash
   cd /root/pipenetwork
   curl -L -o pop "https://dl.pipecdn.app/v0.2.8/pop"
   chmod +x pop
   ```

2. Cek konfigurasi node  
   ```bash
   cat /root/pipenetwork/node_info.json
   ```

3. Konfigurasi firewall  
   ```bash
   sudo ufw allow 8003/tcp
   sudo ufw allow 80/tcp
   sudo ufw allow 443/tcp
   sudo ufw enable
   sudo ufw reload
   ```

4. Jalankan node untuk uji coba  
   ```bash
   cd /root/pipenetwork
   ./pop
   ```
   Jika node berjalan, tekan **Ctrl + C** untuk menghentikannya sebelum membuat service.

## 4. Menjalankan Node sebagai Service (Systemd)

1. Buat file service  
   ```bash
   sudo tee /etc/systemd/system/pop.service << 'EOF'
   [Unit]
   Description=Pipe POP Node Service
   After=network.target
   Wants=network-online.target

   [Service]
   AmbientCapabilities=CAP_NET_BIND_SERVICE
   CapabilityBoundingSet=CAP_NET_BIND_SERVICE
   ExecStart=/root/pipenetwork/pop \
       --ram=isi ram mu \
       --max-disk=isi disk mu \
       --cache-dir=/root/pipenetwork/download_cache \
       --pubKey=isi walletmu
   Restart=always
   RestartSec=5
   LimitNOFILE=65536
   LimitNPROC=4096
   StandardOutput=journal
   StandardError=journal
   SyslogIdentifier=pop-node
   WorkingDirectory=/root/pipenetwork

   [Install]
   WantedBy=multi-user.target
   EOF
   ```

2. Reload systemd dan aktifkan service  
   ```bash
   sudo systemctl daemon-reload
   sudo systemctl enable pop.service
   sudo systemctl start pop.service
   ```

3. Cek status service  
   ```bash
   sudo systemctl status pop.service
   ```
   Jika berhasil, output akan menunjukkan **status active (running)**.

## 5. Memeriksa Status dan Log Node

### Cek Status Node  
   ```bash
   cd /root/pipenetwork
   ./pop --status
   ```

### Cek Log Node (Real-time)  
   ```bash
   journalctl -u pop.service -f
   ```

### Cek Log Node (Sejarah)  
   ```bash
   journalctl -u pop.service --since "1 hour ago"
   ```

## 6. Troubleshooting

Jika mengalami masalah saat menjalankan node, coba langkah berikut:

### Node tidak berjalan?  
   ```bash
   sudo systemctl restart pop.service
   sudo systemctl status pop.service
   ```

### Port 8003 tidak bisa diakses?  
   ```bash
   sudo ufw allow 8003/tcp
   sudo ufw reload
   ```

Panduan ini mencakup semua langkah dari backup, restore, hingga konfigurasi dan monitoring node Pipe Network.
