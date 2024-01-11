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

## Desain Awal dan Struktur Data

Ada baiknya kita perlu mendesain tampilan dari Weather App sebelum mulai coding. Berikut desain yang akan digunakan:

![Gambar 10 Desain awal](img/10%20Desain%20awal.PNG)

Gambar 9. Desain awal

Selanjutnya adalah memanggil raw data api dari openweathermap. Kita bisa mendapatkan URL api dari dokumentasi. Bisa diakses di https://openweathermap.org/current#geocoding. Kita tes terlebih dahulu melalui Thunder Client atau Postman

![Gambar 09 API Requeset menggunakan Thunder client](img/09%20API%20Requeset%20menggunakan%20Thunder%20client.PNG)

Gambar 10 API Request menggunakan Thunder client

Kemudian bisa kita lihat struktur jsonnya adalah sebagai berikut:

```json
{
  "coord": {
    "lon": 105.7044,
    "lat": -5.1695
  },
  "weather": [
    {
      "id": 804,
      "main": "Clouds",
      "description": "overcast clouds",
      "icon": "04d"
    }
  ],
  "base": "stations",
  "main": {
    "temp": 298.67,
    "feels_like": 299.71,
    "temp_min": 298.67,
    "temp_max": 298.67,
    "pressure": 1008,
    "humidity": 93,
    "sea_level": 1008,
    "grnd_level": 1005
  },
  "visibility": 10000,
  "wind": {
    "speed": 2.51,
    "deg": 170,
    "gust": 5.49
  },
  "clouds": {
    "all": 99
  },
  "dt": 1704965961,
  "sys": {
    "country": "ID",
    "sunrise": 1704927153,
    "sunset": 1704971787
  },
  "timezone": 25200,
  "id": 1642549,
  "name": "Jepara",
  "cod": 200
}
```

## Membuat WeatherDataProvider

Sesuai dokumentasi, didapatkan URL api adalah https://api.openweathermap.org/data/2.5/forecast?q=$cityName&APPID=$openWeatherAPIKey, maka kita bisa buat http get diikuti dengan URL apinya. Tambahkan terlebih dahulu package http di pubspec.yaml. Setelah itu kita buat file di `data\data_provider\weather_data_provider.dart`. Ingat disini hanya mendapatkan raw string, belum diolah ke model.

```dart
import 'package:http/http.dart' as http;
import '../../utils/constant.dart';

class WeatherDataProvider {
  Future<String> getCurrentWeather(String cityName) async {
    try {
      final res = await http.get(
        Uri.parse(
          'https://api.openweathermap.org/data/2.5/forecast?q=$cityName&APPID=$openWeatherAPIKey',
        ),
      );

      return res.body;
    } catch (e) {
      throw e.toString();
    }
  }
}
```

## Membuat WeatherRepository

Kita perlu membuat WeatherRepository dengan constructor WeatherDataProvider, maka kodenya adalah sebagai berikut

`data\repository\weather_repository.dart`

```dart
import 'dart:convert';

import '../data_provider/weather_data_provider.dart';

class WeatherRepository {
  final WeatherDataProvider weatherDataProvider;
  WeatherRepository(this.weatherDataProvider);

  Future<Map<String, dynamic>> getCurrentWeather() async {
    try {
      String cityName = 'Jepara';
      final weatherData = await weatherDataProvider.getCurrentWeather(cityName);

      final data = jsonDecode(weatherData);

      if (data['cod'] != 200) {
        throw 'An unexpected error occured';
      }
      return data;
    } catch (e) {
      throw e.toString();
    }
  }
}
```
