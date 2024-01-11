# Weather App Menggunakan Bloc Pattern

Bloc disini adalah kepanjangan dari Business Logic Component. Ini adalah pola arsitektur yang digunakan di Flutter untuk mengelola aliran data, state, dan logika bisnis dalam sebuah aplikasi. Ini memberikan cara yang terstruktur untuk memisahkan lapisan presentasi (UI) dari lapisan logika bisnis. Akibatnya, ini membantu dalam mengembangkan aplikasi Flutter yang lebih mudah dipelihara, diuji, dan dapat diskalakan. Arsitektur BLoC sering digunakan untuk membuat aplikasi yang reaktif dan berkinerja tinggi.

![Gambar 7 bloc pattern ](img/07%20bloc%20pattern.webp)

Gambar 7. Bloc Pattern

Untuk alur dari bloc pattern kurang lebih seperti ini:

1. User mengakses halaman
2. Bloc di request, sesuai dengan event yang sesuai
3. Bloc event handler menjalankan sesuai event yang cocok
4. Bloc meminta data matang dari repository sesuai dengan struktur data yang ditentukan oleh model
5. Repository meminta data mentah dari raw api mendapatkan kembalian string / json
6. Repository mengolah/serialisasi string/json menjadi data yang sesuai denga struktur data model
7. Data matang dari repository diterima oleh Bloc
8. Bloc mengupdate state sesuai dengan data yang diterima
9. UI mengupdate data setelah ada trigger dari perubahan state dari Bloc, dan dilihat hasil tampilan terbaru oleh user

Sedangkan struktur folder bloc pattern nanti kurang lebih seperti ini.

![Gambar 8 Struktur Folder Bloc Pattern ](img/08%20struktur%20folder%20bloc%20pattern.PNG)

Gambar 8. Struktur Folder Bloc Pattern

Buat project baru dengan nama `project_weather_bloc`. Kita akan berlatih menggunakan bloc pattern dengan mendapatkan data cuaca yang didapatkan dari api dari openweathermap. Langkah pertama silahkan mendaftar ke https://openweathermap.org, kemudian lihat api key di https://home.openweathermap.org/api_keys, kemudian masukkan ke file `utils\constant.dart`

```dart
const openWeatherAPIKey = 'MASUKKAN-API-KEY-ANDA';
```

## Membuat WeatherDataProvider

Selanjutnya adalah memanggil raw data api dari openweathermap. Kita bisa mendapatkan URL api dari dokumentasi. Bisa diakses di https://openweathermap.org/current#geocoding. Kita tes terlebih dahulu melalui Thunder Client atau Postman
