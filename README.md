# Identitas
Nama: Nursyafika  
NIM: H1H024023  
Mata Kuliah: Sistem Mikrokontroler  
Modul: 3 - Protokol Komunikasi  

# Deskripsi Singkat
Pada praktikum Modul 3 ini dilakukan percobaan tentang protokol komunikasi pada Arduino Uno, yaitu UART dan I2C. Percobaan 3A membahas cara mengontrol LED melalui Serial Monitor menggunakan input dari keyboard. Percobaan 3B membahas cara menampilkan nilai potensiometer ke LCD I2C dalam bentuk nilai ADC dan bar level.

# Alat dan Bahan
- Arduino Uno
- Breadboard
- LED
- Resistor
- LCD 16x2 I2C
- Potensiometer
- Kabel jumper
- Laptop/PC
- Arduino IDE

# Konfigurasi Pin

## Percobaan 3A - UART LED

- LED → Pin 12
- GND → GND

## Percobaan 3B - LCD I2C dan Potensiometer

### LCD I2C
- VCC → 5V
- GND → GND
- SDA → A4
- SCL → A5

### Potensiometer
- Kaki kiri → GND
- Kaki tengah → A0
- Kaki kanan → 5V

# Percobaan 3A - Komunikasi Serial UART

## Tujuan
Tujuan percobaan ini adalah untuk memahami cara kerja komunikasi UART pada Arduino Uno. Pada percobaan ini, LED dikendalikan melalui Serial Monitor dengan memberikan input dari keyboard.

## Source Code

```cpp
#include <Arduino.h>

const int PIN_LED = 12;

void setup() {
  Serial.begin(9600);
  Serial.println("Ketik '1' untuk menyalakan LED, '0' untuk mematikan LED");
  pinMode(PIN_LED, OUTPUT);
}

void loop() {
  if (Serial.available() > 0) {
    char data = Serial.read();

    if (data == '1') {
      digitalWrite(PIN_LED, HIGH);
      Serial.println("LED ON");
    }
    else if (data == '0') {
      digitalWrite(PIN_LED, LOW);
      Serial.println("LED OFF");
    }
    else if (data != '\n' && data != '\r') {
      Serial.println("Perintah tidak dikenal");
    }
  }
}

Penjelasan Kode

Program ini digunakan untuk mengontrol LED menggunakan input dari Serial Monitor. LED dihubungkan ke pin 12 Arduino.

Pada bagian awal, PIN_LED dibuat sebagai variabel untuk menentukan pin LED yang digunakan. Pada fungsi setup(), komunikasi serial dijalankan dengan Serial.begin(9600). Angka 9600 menunjukkan baud rate yang digunakan agar Arduino dan komputer dapat berkomunikasi.

Perintah pinMode(PIN_LED, OUTPUT) digunakan agar pin 12 bekerja sebagai output. Setelah itu, pada bagian loop(), Arduino akan mengecek apakah ada data yang masuk dari Serial Monitor menggunakan Serial.available().

Jika ada data yang masuk, data tersebut dibaca menggunakan Serial.read(). Jika pengguna mengetik angka 1, maka LED akan menyala. Jika pengguna mengetik angka 0, maka LED akan mati. Jika input yang dimasukkan selain angka tersebut, maka Serial Monitor akan menampilkan pesan “Perintah tidak dikenal”.

Hasil Percobaan

Berdasarkan praktikum yang dilakukan, LED berhasil dikendalikan menggunakan Serial Monitor. Saat input 1 dikirim, LED menyala. Saat input 0 dikirim, LED mati. Jika memasukkan karakter lain, program menampilkan pesan bahwa perintah tidak dikenal.

Jawaban Pertanyaan Praktikum Percobaan 3A

1. Jelaskan proses dari input keyboard hingga LED menyala atau mati!

Prosesnya dimulai ketika pengguna mengetik angka pada Serial Monitor. Data tersebut dikirim dari komputer ke Arduino melalui kabel USB dengan komunikasi UART. Setelah data masuk, Arduino mengecek data tersebut menggunakan Serial.available(), lalu membaca karakternya menggunakan Serial.read().

Jika karakter yang diterima adalah 1, Arduino memberikan output HIGH ke pin LED sehingga LED menyala. Jika karakter yang diterima adalah 0, Arduino memberikan output LOW sehingga LED mati.

2. Mengapa digunakan Serial.available() sebelum membaca data?

Serial.available() digunakan untuk mengecek apakah ada data yang masuk dari Serial Monitor. Jadi, Arduino tidak langsung membaca data jika belum ada input.

Jika bagian ini dihilangkan, Serial.read() bisa tetap berjalan walaupun tidak ada data yang dikirim. Akibatnya, program bisa membaca data kosong atau data yang tidak sesuai.

3. Modifikasi program agar LED berkedip saat menerima input 2

#include <Arduino.h>

const int PIN_LED = 12;

bool blinkMode = false;
unsigned long waktuSebelumnya = 0;
const long interval = 500;
bool kondisiLED = LOW;

void setup() {
  Serial.begin(9600);
  pinMode(PIN_LED, OUTPUT);
}

void loop() {
  if (Serial.available() > 0) {
    char data = Serial.read();

    if (data == '1') {
      blinkMode = false;
      digitalWrite(PIN_LED, HIGH);
      Serial.println("LED ON");
    }
    else if (data == '0') {
      blinkMode = false;
      digitalWrite(PIN_LED, LOW);
      Serial.println("LED OFF");
    }
    else if (data == '2') {
      blinkMode = true;
      Serial.println("LED BLINK");
    }
    else if (data != '\n' && data != '\r') {
      Serial.println("Perintah tidak dikenal");
    }
  }

  if (blinkMode == true) {
    unsigned long waktuSekarang = millis();

    if (waktuSekarang - waktuSebelumnya >= interval) {
      waktuSebelumnya = waktuSekarang;
      kondisiLED = !kondisiLED;
      digitalWrite(PIN_LED, kondisiLED);
    }
  }
}

Penjelasan singkatnya, program ditambahkan mode blink menggunakan variabel blinkMode. Jika input yang diterima adalah 2, maka LED akan berkedip. Program menggunakan millis() agar Arduino tetap bisa menerima input baru walaupun LED sedang berkedip.

4. Lebih baik menggunakan delay() atau millis()?

Lebih baik menggunakan millis() karena tidak menghentikan proses program. Jika menggunakan delay(), Arduino akan berhenti sementara selama waktu delay tersebut, sehingga input dari Serial Monitor bisa terlambat terbaca. Dengan millis(), LED tetap bisa berkedip dan Arduino tetap bisa menerima perintah baru.

Percobaan 3B - Komunikasi I2C

Tujuan :
Tujuan percobaan ini adalah untuk memahami cara kerja komunikasi I2C antara Arduino Uno dan LCD I2C. Pada percobaan ini, nilai potensiometer dibaca oleh Arduino, lalu ditampilkan pada LCD.

Source Code

#include <Wire.h>
#include <LiquidCrystal_I2C.h>
#include <Arduino.h>

LiquidCrystal_I2C lcd(0x27, 16, 2);

const int pinPot = A0;

void setup() {
  Serial.begin(9600);
  
  lcd.init();
  lcd.backlight();
}

void loop() {
  int nilai = analogRead(pinPot);

  int panjangBar = map(nilai, 0, 1023, 0, 16);

  Serial.print("Nilai ADC : ");
  Serial.println(nilai);

  lcd.setCursor(0, 0);
  lcd.print("ADC: ");
  lcd.print(nilai);
  lcd.print("   ");

  lcd.setCursor(0, 1);
  for (int i = 0; i < 16; i++) {
    if (i < panjangBar) {
      lcd.print((char)255);
    } else {
      lcd.print(" ");
    }
  }

  delay(200);
}

Penjelasan Kode

Program ini digunakan untuk menampilkan nilai potensiometer pada LCD I2C. Potensiometer dihubungkan ke pin A0, sedangkan LCD menggunakan komunikasi I2C dengan alamat 0x27.

Library Wire.h digunakan untuk komunikasi I2C, sedangkan LiquidCrystal_I2C.h digunakan untuk mengatur tampilan LCD. Pada bagian setup(), Serial Monitor dijalankan dengan baud rate 9600. LCD juga diaktifkan menggunakan lcd.init() dan lampu LCD dinyalakan menggunakan lcd.backlight().

Pada bagian loop(), nilai potensiometer dibaca menggunakan analogRead(pinPot). Nilai yang terbaca berupa nilai ADC dengan rentang 0 sampai 1023. Nilai tersebut kemudian diubah menggunakan map() menjadi rentang 0 sampai 16 untuk menentukan panjang bar yang tampil di LCD.

Baris pertama LCD menampilkan nilai ADC. Baris kedua menampilkan bar level. Semakin besar nilai potensiometer, maka bar yang muncul pada LCD juga semakin panjang.

Hasil Percobaan

Berdasarkan praktikum yang dilakukan, LCD berhasil menampilkan nilai ADC dari potensiometer. Saat potensiometer diputar, nilai ADC berubah dan bar pada LCD ikut berubah. Semakin besar nilai potensiometer, tampilan bar pada LCD semakin panjang.

Jawaban Pertanyaan Praktikum Percobaan 3B

1. Jelaskan cara kerja komunikasi I2C antara Arduino dan LCD!

Pada komunikasi I2C, Arduino berperan sebagai master, sedangkan LCD berperan sebagai slave. Arduino mengirimkan data melalui jalur SDA dan menggunakan jalur SCL sebagai clock. LCD akan menerima data jika alamat yang digunakan sesuai, yaitu 0x27.

Dengan I2C, LCD tidak perlu menggunakan banyak pin Arduino. Cukup menggunakan SDA dan SCL, LCD sudah dapat menerima perintah dan menampilkan data.

2. Apakah pin potensiometer harus seperti itu?

Ya, sebaiknya pin potensiometer dipasang sesuai rangkaian. Kaki tengah harus masuk ke pin A0 karena kaki tersebut menghasilkan tegangan yang berubah-ubah saat potensiometer diputar.

Jika kaki kiri dan kanan tertukar, potensiometer tetap bisa terbaca, tetapi arah nilainya menjadi terbalik. Misalnya saat diputar ke kanan nilai ADC malah mengecil, sedangkan saat diputar ke kiri nilai ADC membesar.

3. Modifikasi program dengan menggabungkan UART dan I2C sebagai output

#include <Wire.h>
#include <LiquidCrystal_I2C.h>
#include <Arduino.h>

LiquidCrystal_I2C lcd(0x27, 16, 2);

const int pinPot = A0;

void setup() {
  Serial.begin(9600);

  lcd.init();
  lcd.backlight();
}

void loop() {
  int nilai = analogRead(pinPot);

  float volt = nilai * (5.0 / 1023.0);
  int persen = map(nilai, 0, 1023, 0, 100);
  int panjangBar = map(nilai, 0, 1023, 0, 16);

  Serial.print("ADC: ");
  Serial.print(nilai);
  Serial.print(" Volt: ");
  Serial.print(volt);
  Serial.print(" V Persen: ");
  Serial.print(persen);
  Serial.println("%");

  lcd.setCursor(0, 0);
  lcd.print("ADC: ");
  lcd.print(nilai);
  lcd.print(" ");
  lcd.print(persen);
  lcd.print("%   ");

  lcd.setCursor(0, 1);
  for (int i = 0; i < 16; i++) {
    if (i < panjangBar) {
      lcd.print((char)255);
    } else {
      lcd.print(" ");
    }
  }

  delay(200);
}

Penjelasan singkatnya, program ini menampilkan data ke dua output sekaligus. Data ADC, voltase, dan persentase ditampilkan pada Serial Monitor menggunakan komunikasi UART. Sementara itu, nilai ADC dan bar level ditampilkan pada LCD menggunakan komunikasi I2C.

4. Hasil pengamatan pada Serial Monitor

ADC	Volt(V) Persen(%)
 
1	0.00 V	 0%
21	0.10 V	 2%
49	0.24 V	 5%
74	0.36 V	 7%
96	0.47 V	 9%

Pertanyaan Analisis

1. Keuntungan dan kerugian UART dan I2C

UART lebih sederhana digunakan karena hanya membutuhkan pengaturan baud rate dan cocok untuk komunikasi antara Arduino dengan komputer melalui Serial Monitor. Kekurangannya, UART biasanya lebih cocok untuk komunikasi antar dua perangkat saja.

I2C lebih hemat pin karena hanya menggunakan dua jalur, yaitu SDA dan SCL. Selain itu, beberapa perangkat bisa dihubungkan pada jalur I2C yang sama selama alamatnya berbeda. Kekurangannya, I2C membutuhkan alamat perangkat yang sesuai. Jika alamatnya salah, LCD atau sensor tidak akan terbaca.

2. Peran alamat I2C pada LCD

Alamat I2C berfungsi sebagai identitas perangkat. Pada LCD I2C, alamat yang sering digunakan adalah 0x27 atau 0x20. Jika alamat pada program tidak sesuai dengan alamat LCD yang digunakan, maka LCD tidak akan menampilkan data.

3. Alur kerja jika UART dan I2C digabung

Jika UART dan I2C digabung, Arduino dapat menampilkan data ke dua tempat sekaligus. Data dari potensiometer dibaca oleh Arduino, kemudian hasilnya dikirim ke Serial Monitor melalui UART dan ditampilkan ke LCD melalui I2C.

Arduino bisa mengelola keduanya karena UART dan I2C menggunakan jalur komunikasi yang berbeda. UART menggunakan komunikasi serial melalui USB, sedangkan I2C menggunakan pin SDA dan SCL.

Kesimpulan
1. Pada praktikum ini, komunikasi UART dan I2C berhasil digunakan pada Arduino Uno.
2. Percobaan 3A menunjukkan bahwa LED dapat dikendalikan melalui Serial Monitor menggunakan input 1 dan 0.
3. Percobaan 3B menunjukkan bahwa nilai potensiometer dapat dibaca sebagai nilai ADC dan ditampilkan pada LCD I2C.
4. Komunikasi UART cocok digunakan untuk komunikasi Arduino dengan komputer.
5. Komunikasi I2C cocok digunakan untuk perangkat seperti LCD karena hanya membutuhkan dua pin, yaitu SDA dan SCL.
6. Fungsi Serial.available() penting digunakan agar Arduino hanya membaca data saat ada input yang masuk.
7. Penggunaan millis() lebih baik daripada delay() untuk program blink karena program tetap bisa membaca input baru.
