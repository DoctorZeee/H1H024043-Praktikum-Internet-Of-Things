Nama: Dimas Rafif Zaidan  
NIM: H1H024043  
Shift Awal: D  
Shift Akhir: C  

---

# Pertanyaan Praktikum - Percobaan 1
## 1. Gambarkan diagram alur (flowchart) proses akuisisi data sensor DHT11 pada program di atas!
<img width="2720" height="2608" alt="flowchart_akuisisi_dht22" src="https://github.com/user-attachments/assets/3ad8c826-8861-44ab-bbdb-14bfdc6bdc1c" />

---

## 2. Apa fungsi dari perintah `isnan()` pada program tersebut?
Fungsi `isnan()` digunakan untuk memeriksa apakah suatu nilai bertipe float merupakan NaN (Not a Number), yaitu kondisi ketika sensor gagal memberikan hasil pembacaan yang valid, misalnya akibat koneksi kabel yang longgar, gangguan pada sensor, atau waktu pembacaan yang terlalu cepat. Dengan memeriksa isnan(), program dapat mendeteksi kegagalan pembacaan sebelum data yang tidak valid tersebut ditampilkan atau digunakan untuk pengambilan keputusan, sehingga mencegah kesalahan interpretasi data.

---

## 3. Jelaskan mengapa diperlukan jeda (delay) minimal sekitar 2 detik antar pembacaan sensor DHT11! 
Sensor DHT memiliki keterbatasan pada kecepatan sampling internalnya, yaitu hanya mampu memberikan satu data pembacaan baru setiap kurang lebih 2 detik. Apabila pembacaan dilakukan lebih cepat dari jeda tersebut, sensor berpotensi mengembalikan nilai yang sama dengan pembacaan sebelumnya atau bahkan menghasilkan nilai NaN karena proses konversi sinyal internal sensor belum selesai. Oleh karena itu, `delay(2000)` diperlukan agar pembacaan yang diperoleh benar-benar merupakan data baru yang valid dan sesuai dengan spesifikasi teknis sensor.

---

## 4. Modifikasi program agar data suhu dan kelembaban dirata-ratakan dari 5 kali pembacaan sebelum ditampilkan, dan berikan penjelasan di setiap baris kode yang ditambahkan dalam bentuk README.md!
```cpp
#include <DHT.h>

#define DHTPIN 4
#define DHTTYPE DHT11

DHT dht(DHTPIN, DHTTYPE);

// --- Kode baru: konstanta jumlah pembacaan untuk dirata-ratakan ---
const int JUMLAH_SAMPLING = 5; // menentukan berapa kali pembacaan sensor akan diambil sebelum dirata-ratakan

void setup() {
  Serial.begin(115200);
  dht.begin();
}

void loop() {
  // --- Kode baru: variabel akumulator untuk menjumlahkan hasil pembacaan yang valid ---
  float totalSuhu = 0;       // menyimpan jumlah (akumulasi) nilai suhu dari setiap pembacaan yang valid
  float totalKelembaban = 0; // menyimpan jumlah (akumulasi) nilai kelembaban dari setiap pembacaan yang valid

  // --- Kode baru: penghitung banyaknya pembacaan yang berhasil (valid) ---
  int jumlahValid = 0; // digunakan sebagai pembagi saat menghitung rata-rata, hanya menghitung data yang tidak NaN

  // --- Kode baru: perulangan (loop for) sebanyak JUMLAH_SAMPLING kali ---
  for (int i = 0; i < JUMLAH_SAMPLING; i++) {
    float kelembaban = dht.readHumidity();    // membaca nilai kelembaban pada iterasi ke-i
    float suhu = dht.readTemperature();       // membaca nilai suhu pada iterasi ke-i

    if (isnan(kelembaban) || isnan(suhu)) {
      // --- Kode baru: pesan peringatan khusus per-iterasi jika pembacaan gagal ---
      Serial.println("Gagal membaca data dari sensor DHT22! (pembacaan dilewati)"); 
      // pembacaan yang gagal (NaN) tidak diikutkan ke dalam akumulasi, agar tidak merusak hasil rata-rata
    } else {
      // --- Kode baru: menambahkan nilai yang valid ke akumulator ---
      totalSuhu += suhu;             // menjumlahkan suhu yang valid ke total
      totalKelembaban += kelembaban; // menjumlahkan kelembaban yang valid ke total
      jumlahValid++;                 // menambah penghitung pembacaan valid sebanyak 1
    }

    delay(2000); // tetap diperlukan agar sensor DHT22 punya waktu minimal antar pembacaan (karakteristik sensor)
  }

  // --- Kode baru: proses perhitungan rata-rata setelah 5 kali pembacaan selesai ---
  if (jumlahValid > 0) {
    // hanya menghitung rata-rata jika ada minimal 1 data valid, untuk menghindari pembagian dengan nol
    float rataSuhu = totalSuhu / jumlahValid;             // rata-rata suhu = total suhu dibagi jumlah data valid
    float rataKelembaban = totalKelembaban / jumlahValid; // rata-rata kelembaban = total kelembaban dibagi jumlah data valid

    // --- Kode baru: menampilkan hasil rata-rata, bukan hasil pembacaan tunggal ---
    Serial.print("Rata-rata Suhu: ");
    Serial.print(rataSuhu);
    Serial.print(" °C, Rata-rata Kelembaban: ");
    Serial.print(rataKelembaban);
    Serial.println(" %");
  } else {
    // --- Kode baru: penanganan jika seluruh 5 pembacaan gagal (semua NaN) ---
    Serial.println("Seluruh pembacaan pada siklus ini gagal, rata-rata tidak dapat dihitung.");
  }

  // delay(2000) tambahan di akhir loop tidak diperlukan lagi karena sudah ada delay 2 detik
  // di dalam masing-masing iterasi for di atas
}
```

---

# Pertanyaan Praktikum - Percobaan 2
## 1. Mengapa diperlukan nilai ambang batas (threshold) dalam sistem kendali aktuator berbasis sensor? 
Nilai threshold diperlukan sebagai acuan atau referensi bagi mikrokontroler untuk menentukan kapan suatu aktuator harus diaktifkan atau dinonaktifkan. Tanpa threshold, sistem tidak memiliki kriteria yang jelas untuk mengambil keputusan berdasarkan data sensor yang bersifat kontinu. Threshold memungkinkan sistem merespons kondisi lingkungan secara otomatis dan konsisten, misalnya menyalakan pendingin ketika suhu terlalu tinggi, tanpa memerlukan intervensi manual.

---

## 2. Jelaskan apa yang akan terjadi apabila nilai `suhuThreshold` diturunkan menjadi sangat rendah, misalnya 20.0! 
Jika nilai `suhuThreshold` diturunkan menjadi 20.0°C, maka aktuator (relay/LED) akan menyala hampir sepanjang waktu karena suhu ruangan pada kondisi normal umumnya sudah berada di atas 20°C. Akibatnya aktuator kehilangan fungsinya sebagai indikator kondisi ekstrem, karena akan selalu dalam keadaan ON meskipun suhu lingkungan masih berada pada rentang normal/aman. Pada aplikasi nyata (misalnya relay untuk kipas atau pompa), hal ini dapat menyebabkan aktuator bekerja secara terus-menerus, meningkatkan konsumsi daya, dan mempercepat keausan komponen.

---

## 3. Apa perbedaan antara kendali aktuator secara terus-menerus (kondisi tunggal) dengan kendali menggunakan histerisis (dua ambang batas)?
Pada kendali kondisi tunggal (satu threshold), aktuator akan langsung berganti status setiap kali nilai suhu melewati batas tersebut, sehingga apabila suhu berada tepat di sekitar nilai threshold, aktuator dapat menyala-mati secara berulang dalam waktu singkat (chattering/osilasi). Sebaliknya, pada kendali histerisis digunakan dua nilai ambang batas berbeda: satu batas atas untuk menyalakan aktuator dan satu batas bawah (lebih rendah) untuk mematikannya. Dengan adanya jarak (gap) antara kedua batas ini, perubahan status aktuator menjadi lebih stabil dan tidak mudah berosilasi meskipun suhu berfluktuasi kecil di sekitar titik threshold.

---

## 4. Modifikasi program agar menggunakan dua ambang batas (histerisis), misalnya aktuator menyala pada suhu di atas 30°C dan baru mati pada suhu di bawah 28°C, dan berikan penjelasan di setiap baris kode nya dalam bentuk README.md!
```cpp
#include <DHT.h>

#define DHTPIN 4
#define DHTTYPE DHT11
#define RELAYPIN 26

DHT dht(DHTPIN, DHTTYPE);

// --- Kode baru: dua nilai ambang batas menggantikan satu suhuThreshold ---
const float batasAtas = 30.0;  // batas atas: suhu di atas nilai ini akan MENYALAKAN aktuator
const float batasBawah = 28.0; // batas bawah: suhu di bawah nilai ini akan MEMATIKAN aktuator

// --- Kode baru: variabel status untuk menyimpan kondisi aktuator saat ini ---
bool statusAktuator = false; // false = OFF, true = ON; disimpan secara persisten antar-iterasi loop()

void setup() {
  Serial.begin(115200);
  dht.begin();

  pinMode(RELAYPIN, OUTPUT);
  digitalWrite(RELAYPIN, LOW); // pastikan aktuator mati saat program pertama kali dijalankan
  statusAktuator = false;      // --- Kode baru: sinkronkan variabel status dengan kondisi fisik awal aktuator
}

void loop() {
  float suhu = dht.readTemperature(); // membaca nilai suhu terbaru dari sensor DHT11

  if (isnan(suhu)) {
    // pembacaan gagal: tampilkan pesan, dan JANGAN ubah status aktuator
    Serial.println("Gagal membaca suhu dari sensor DHT22!");
  } else {
    // --- Kode baru: logika histerisis menggantikan if-else tunggal ---
    if (suhu > batasAtas) {
      // jika suhu melebihi batas atas, aktuator dipastikan menyala
      statusAktuator = true; // ubah status menjadi ON
    } else if (suhu < batasBawah) {
      // jika suhu turun di bawah batas bawah, aktuator dipastikan mati
      statusAktuator = false; // ubah status menjadi OFF
    }
    // --- Kode baru: rentang di antara batasBawah dan batasAtas ---
    // jika suhu berada di antara 28°C dan 30°C, tidak ada blok else di sini,
    // sehingga statusAktuator TIDAK diubah dan tetap mempertahankan nilai sebelumnya
    // (inilah inti dari histerisis: mencegah aktuator berganti status berulang kali/osilasi
    // di sekitar satu titik threshold)

    // --- Kode baru: terapkan statusAktuator ke pin fisik relay/LED ---
    digitalWrite(RELAYPIN, statusAktuator ? HIGH : LOW);
    // ekspresi ternary: jika statusAktuator true → HIGH (menyala), jika false → LOW (mati)

    Serial.print("Suhu: ");
    Serial.print(suhu);
    Serial.print(" °C, Aktuator: ");
    // --- Kode baru: status ditampilkan berdasarkan variabel statusAktuator, bukan hasil if seketika ---
    Serial.println(statusAktuator ? "ON" : "OFF");
  }

  delay(2000); // jeda 2 detik sesuai karakteristik minimal sampling sensor DHT22
}
```

---

# Pertanyaan Analisis
## 1. Uraikan hasil tugas pada praktikum yang telah dilakukan pada setiap percobaan!
Pada Percobaan 1A, ESP32 berhasil membaca data suhu dan kelembaban dari sensor DHT11 secara berkala setiap 2 detik dan menampilkannya pada Serial Monitor tanpa error, dengan seluruh data pengamatan berstatus valid (bukan NaN) pada berbagai kondisi pengujian (normal, disentuh tangan, didekatkan ke AC). Pada Percobaan 2A, sistem berhasil mengendalikan aktuator (relay/LED) secara otomatis berdasarkan perbandingan nilai suhu terhadap threshold 29–30°C, dengan pola aktuator OFF pada suhu rendah dan berubah ON saat suhu mendekati/melebihi threshold, sehingga secara umum kedua percobaan telah sesuai dengan spesifikasi yang diharapkan pada modul.

---

## 2. Bagaimana pengaruh akurasi dan waktu tanggap (response time) sensor terhadap kecepatan reaksi aktuator pada sistem IoT?
Akurasi sensor menentukan seberapa tepat nilai yang dibaca mencerminkan kondisi fisik sebenarnya; sensor dengan akurasi rendah dapat menyebabkan aktuator mengambil keputusan yang keliru, misalnya menyala atau mati pada kondisi yang sebenarnya belum memenuhi threshold. Sementara itu, waktu tanggap (response time) sensor menentukan seberapa cepat perubahan kondisi lingkungan dapat terdeteksi; response time yang lambat (seperti pada DHT11 yang memerlukan jeda ± 2 detik) menyebabkan aktuator bereaksi dengan keterlambatan terhadap perubahan suhu yang terjadi secara mendadak. Oleh karena itu, semakin tinggi akurasi dan semakin cepat response time sensor, semakin responsif dan presisi pula reaksi aktuator terhadap perubahan kondisi lingkungan yang sesungguhnya.

---

## 3. Bagaimana cara kerja sistem dalam mengubah data sensor menjadi keputusan kendali aktuator (proses akuisisi hingga aktuasi)? 
Proses dimulai dari sensor DHT11 yang mengukur besaran fisik (suhu dan kelembaban) dan mengubahnya menjadi sinyal digital, kemudian dikirim ke ESP32 melalui protokol komunikasi 1-Wire dan dibaca menggunakan fungsi dht.readTemperature()/dht.readHumidity() dari pustaka DHT.h. Data yang diperoleh divalidasi menggunakan isnan() untuk memastikan tidak terjadi kegagalan pembacaan. Selanjutnya, nilai suhu yang valid dibandingkan dengan nilai threshold yang telah ditentukan melalui struktur kondisi (if-else) pada program. Berdasarkan hasil perbandingan tersebut, ESP32 mengirimkan sinyal digital HIGH atau LOW ke pin GPIO yang terhubung dengan relay/LED, sehingga aktuator melakukan aksi fisik (menyala/mati) sebagai respons terhadap kondisi sensor yang terdeteksi.

---

## 4. Bagaimana kombinasi antara akuisisi data sensor dan kendali aktuator dapat digunakan untuk membangun sistem IoT yang responsif terhadap perubahan kondisi lingkungan, misalnya pada sistem smart farming atau smart home?
Kombinasi akuisisi data sensor dan kendali aktuator merupakan dasar dari sistem closed-loop pada IoT, di mana data dari lingkungan digunakan secara langsung untuk mengambil keputusan otomatis tanpa intervensi manusia. Pada smart farming, misalnya, data suhu dan kelembaban tanah/udara dapat digunakan untuk mengendalikan aktuator seperti pompa irigasi atau kipas ventilasi secara otomatis agar kondisi tanaman tetap optimal. Pada smart home, data yang sama dapat digunakan untuk mengendalikan AC, exhaust fan, atau sistem peringatan dini secara otomatis sesuai kenyamanan dan keamanan penghuni. Dengan menambahkan konektivitas internet (WiFi/IoT platform), sistem semacam ini juga dapat dipantau dan dikendalikan dari jarak jauh serta datanya dapat dianalisis lebih lanjut untuk pengambilan keputusan yang lebih baik di masa depan.
