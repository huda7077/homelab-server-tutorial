# Audio Passthrough ThinkCentre M910q di Proxmox

Dokumentasi ini menjelaskan langkah-langkah setup audio passthrough pada **Lenovo ThinkCentre M910q** menggunakan **Proxmox VE**, hingga audio dapat digunakan di dalam VM untuk memutar YouTube via terminal.

---

## 1. Tambahkan Audio Device ke VM (PCI Passthrough)

1. Pilih VM yang akan digunakan.
2. **Matikan VM** terlebih dahulu.
3. Masuk ke menu **Hardware**.
4. Klik **Add → PCI Device**.
5. Pilih perangkat audio (biasanya berada di **Raw 00:1f.3**).
6. Centang:

   * `ROM-Bar`
   * `PCI-Express`
7. Klik **Add**.
8. **Start VM**.

---

## 2. Verifikasi Audio Device di VM

Pastikan device audio yang terdeteksi di VM sama dengan yang ada di Proxmox.

### Cek daftar kartu audio

```bash
aplay -l
```

Jika kartu audio muncul, lanjutkan ke tahap berikutnya.

### Buka ALSA Mixer

```bash
alsamixer
```

Jika tampilan volume **tidak muncul**, coba ganti card number:

```bash
alsamixer -c 1
```

(Angka `1` menyesuaikan hasil dari `aplay -l`)

---

## 3. Test Speaker

Gunakan perintah berikut untuk memastikan audio benar-benar keluar:

```bash
speaker-test -c 2 -D hw:1,0
```

Keterangan:

* `-c 2` : stereo
* `hw:1,0` : card dan device (sesuaikan jika berbeda)

Jika suara keluar, setup audio dasar sudah berhasil.

---

## 4. Instalasi Player YouTube (mpv + yt-dlp)

Update repository dan install package:

```bash
sudo apt update
sudo apt install mpv yt-dlp -y
```

### Test play YouTube (audio only)

```bash
mpv --no-video --audio-device=alsa/hw:1,0 "URL_YOUTUBE_DISINI"
```

---

## 5. Perbaikan Error youtube-dl Failed (Symlink)

Beberapa aplikasi masih mencari `youtube-dl`. Buat symlink ke `yt-dlp`:

```bash
sudo ln -s /usr/bin/yt-dlp /usr/local/bin/youtube-dl
```

---

## 6. Membuat Alias agar Lebih Praktis

Karena perintah terlalu panjang, buat alias di `~/.bashrc` atau `~/.bash_aliases`.

Contoh alias sederhana:

```bash
alias ytpl='mpv --no-video --audio-device=alsa/hw:1,0'
```

Penggunaan:

```bash
ytpl "https://youtube.com/watch?v=xxxx"
```

---

## 7. Jika Muncul JavaScript Warning

Install **Node.js** di VM:

```bash
sudo apt install nodejs npm -y
```

---

## 8. Pencarian Lagu Interaktif (yt-dlp + fzf + autoplay)

### Install fzf

```bash
sudo apt update
sudo apt install fzf -y
```

### Alias Search + Autoplay

Tambahkan alias berikut ke `~/.bashrc`:

```bash
alias cari='f() {
    echo "Sedang mencari lagu...";
    SEL=$(yt-dlp --get-title --get-id "ytsearch10:$1" 2>/dev/null | sed "N;s/\n/ -- /" | fzf --height 40% --layout=reverse --border --prompt="Pilih lagu (Autoplay Aktif): ");
    
    ID=$(echo $SEL | awk -F " -- " "{print \$NF}");
    
    if [ ! -z "$ID" ]; then
        echo "Memutar: $SEL";
        echo "Menyiapkan Autoplay/Radio YouTube...";
        mpv --no-video --audio-device=alsa/hw:1,0 --ytdl-raw-options="yes-playlist=" "https://www.youtube.com/watch?v=$ID&list=RD$ID";
    else
        echo "Pencarian dibatalkan.";
    fi;
}; f'
```

### Penggunaan

```bash
cari "judul lagu"
```

* Menampilkan **10 hasil teratas**
* Pilih via `fzf`
* Lagu akan **langsung autoplay (YouTube Radio)**

---

## 9. Catatan

* Card audio (`hw:1,0`) bisa berbeda di tiap sistem.
* Selalu cek dengan `aplay -l` jika audio tidak keluar.
* Dokumentasi ini fokus pada audio passthrough, bukan USB audio.

---

Selesai.
