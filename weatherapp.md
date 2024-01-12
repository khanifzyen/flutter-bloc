# Weather App Menggunakan Bloc Pattern

> Daftar Isi:
>
> 1. [Perkenalan](#perkenalan)
> 2. [Desain Awal dan Struktur Data](#desain-awal-dan-struktur-data)
> 3. [Membuat WeatherDataProvider](#membuat-weatherdataprovider)
> 4. [Membuat WeatherRepository](#membuat-weatherrepository)
> 5. [Membuat WeatherModel](#membuat-weathermodel)
> 6. [Membuat Bloc](#membuat-bloc)
> 7. [Menambahkan BlocProvider dan RepositoryProvider](#menambahkan-blocprovider-dan-repositoryprovider)
> 8. [Mentrigger event](#mentrigger-event)

## Perkenalan

Bloc disini adalah kepanjangan dari _Business Logic Component_. Ini adalah pola arsitektur yang digunakan di Flutter untuk mengelola aliran data, state, dan logika bisnis dalam sebuah aplikasi. Ini memberikan cara yang terstruktur untuk memisahkan lapisan presentasi (UI) dari lapisan logika bisnis. Akibatnya, ini membantu dalam mengembangkan aplikasi Flutter yang lebih mudah dipelihara, diuji, dan dapat diskalakan. Arsitektur BLoC sering digunakan untuk membuat aplikasi yang reaktif dan berkinerja tinggi.

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

Selanjutnya adalah memanggil raw data api dari openweathermap. Kita bisa mendapatkan URL api dari dokumentasi. Bisa diakses di https://openweathermap.org/current#geocoding. Endpoint URL API: https://api.openweathermap.org/data/2.5/forecast?q={city name}&appid={API key}

Kita tes terlebih dahulu melalui Thunder Client atau Postman

![Gambar 09 API Request menggunakan Thunder client](img/09%20API%20Requeset%20menggunakan%20Thunder%20client.PNG)

Gambar 10 API Request Current Weather menggunakan Thunder client

Kemudian bisa kita lihat struktur jsonnya adalah sebagai berikut:

```json
{
  "cod": "200",
  "message": 0,
  "cnt": 40,
  "list": [
    {
      "dt": 1705060800,
      "main": {
        "temp": 299.4,
        "feels_like": 299.4,
        "temp_min": 298.31,
        "temp_max": 299.4,
        "pressure": 1008,
        "sea_level": 1008,
        "grnd_level": 1006,
        "humidity": 86,
        "temp_kf": 1.09
      },
      "weather": [
        {
          "id": 804,
          "main": "Clouds",
          "description": "overcast clouds",
          "icon": "04n"
        }
      ],
      "clouds": {
        "all": 100
      },
      "wind": {
        "speed": 2.44,
        "deg": 211,
        "gust": 6.85
      },
      "visibility": 10000,
      "pop": 0.36,
      "sys": {
        "pod": "n"
      },
      "dt_txt": "2024-01-12 12:00:00"
    }
  ],
  "city": {
    "id": 1642549,
    "name": "Jepara",
    "coord": {
      "lat": -5.1695,
      "lon": 105.7044
    },
    "country": "ID",
    "population": 1000,
    "timezone": 25200,
    "sunrise": 1705013581,
    "sunset": 1705058207
  }
}
```

Dari desain UI dan response json, berarti kira-kira yang dibutuhkan adalah

- ["list"][0]["main"]["temp"]
- ["list"][0]["weather"][0]["main"]
- ["list"][0]["main"]["humidity"]
- ["list"][0]["wind"]["speed"]
- ["list"][0]["main"]["pressure"]

Maka bisa langsung kita ambil dari key ["list"][0]

## Membuat WeatherDataProvider

Sesuai dokumentasi, didapatkan URL api adalah https://api.openweathermap.org/data/2.5/forecast?q=$cityName&APPID=$openWeatherAPIKey, maka kita bisa buat http get diikuti dengan URL apinya. Tambahkan terlebih dahulu package `http` di `pubspec.yaml`. Setelah itu kita buat file di `data\data_provider\weather_data_provider.dart`. Ingat disini hanya mendapatkan raw string, belum diolah ke model.

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

      if (data['cod'] != "200") {
        throw 'An unexpected error occured';
      }
      return data;
    } catch (e) {
      throw e.toString();
    }
  }
}
```

## Membuat WeatherModel

Selanjutnya adalah mapping dari weatherData yang didapat dari weatherRepository ke weatherModel. Buat filenya terlebih dahulu di `models\weather_model.dart`

```dart
class WeatherModel {
  final num currentTemp;
  final String currentSky;
  final num currentPressure;
  final num currentWindSpeed;
  final num currentHumidity;
}
```

Setelah itu klik kanan pilih "Generate Data Class", maka WeatherModel akan tergenerate constructor dan method-method tambahan. Pada method `fromMap()` ubah bodynya sesuai dengan index data mapping dari json sebelumnya, sehingga menjadi

`models\weather_model.dart`

```dart
...
  factory WeatherModel.fromMap(Map<String, dynamic> map) {
    final currentWeatherData = map["list"][0];

    return WeatherModel(
      currentTemp: currentWeatherData["main"]["temp"],
      currentSky: currentWeatherData["weather"][0]["main"],
      currentPressure: currentWeatherData["main"]["pressure"],
      currentWindSpeed: currentWeatherData["wind"]["speed"],
      currentHumidity: currentWeatherData["main"]["humidity"],
    );
  }
...
```

Sekarang WeatherModel sudah jadi, kembali ke `weather_repository.dart`, dan ganti baris menjadi seperti berikut:

```dart
...
  Future<WeatherModel> getCurrentWeather() async {
    try {
      ...
      return WeatherModel.fromMap(data);
    } catch (e) {
      throw e.toString();
    }
  }
...
```

## Membuat Bloc

Tambahkan package `flutter_bloc` terlebih dahulu pada `pubspec.yaml`. Selanjutnya, jalankan perintah "Bloc: New Bloc", dan masukkan "weather" maka akan muncul folder bloc yang didalamnya ada 3 file (weather_bloc.dart, weather_state.dart, weather_event.dart).

Isikan `weather_state.dart` menjadi seperti berikut:

```dart
part of 'weather_bloc.dart';

@immutable
sealed class WeatherState {}

final class WeatherInitial extends WeatherState {}

final class WeatherSuccess extends WeatherState {
  final WeatherModel weatherModel;
  WeatherSuccess({required this.weatherModel});
}

final class WeatherFailure extends WeatherState {
  final String error;
  WeatherFailure(this.error);
}

final class WeatherLoading extends WeatherState {}
```

`WeatherSuccess` nanti akan dipanggil setelah berhasil mendapatkan kembalian dari `WeatherRepository` yang berupa `WeatherModel`. Sedangkan `WeatherFailure` akan dipanggil jika terjadi error, dengan nanti akan mengembalikan berupa String dari eror yang muncul.

Kemudian lanjut isikan `weather_event.dart` menjadi seperti berikut:

```dart
part of 'weather_bloc.dart';

@immutable
sealed class WeatherEvent {}

final class WeatherFetchEvent extends WeatherEvent {}
```

Untuk `WeatherFetchEvent` karena tidak ada yang diolah maka isi body class kosong (cityName masih hardcoded).

Kemudian lanjut isi ke `weather_bloc.dart`, dimana kita akan mulai memasang event handler untuk `WeatherFetchEvent`

```dart
import 'package:flutter_bloc/flutter_bloc.dart';
import 'package:flutter/foundation.dart';

import '../data/repository/weather_repository.dart';
import '../models/weather_model.dart';

part 'weather_event.dart';
part 'weather_state.dart';

class WeatherBloc extends Bloc<WeatherEvent, WeatherState> {
  final WeatherRepository weatherRepository;
  WeatherBloc(this.weatherRepository) : super(WeatherInitial()) {
    on<WeatherFetchEvent>(_getCurrentWeather);
  }

  void _getCurrentWeather(
      WeatherFetchEvent event, Emitter<WeatherState> emit) async {
    emit(WeatherLoading());
    try {
      final weather = await weatherRepository.getCurrentWeather();
      emit(WeatherSuccess(weatherModel: weather));
    } catch (e) {
      emit(WeatherFailure(e.toString()));
    }
  }
}
```

## Menambahkan BlocProvider dan RepositoryProvider

> **Catatan:** pada real world, biasanya pembuatan tampilan dilakukan pada tahapan awal. Untuk mempermudah penjelasan modul, maka disusun sesuai urutan.

Tahap berikutnya adalah melakukan _dependency injection_, yaitu dengan menggunakan `BlocProvider` agar `WeatherBloc` bisa diakses disemua bagian widget / UI. `WeatherBloc` butuh parameter `WeatherRepository`, akan tetapi nanti tidak kita masukkan secara langsung, tetapi lewat `RepositoryProvider`.

Ada satu fitur lagi pada package `flutter_bloc` yaitu `RepositoryProvider`. Ia memiliki fungsi yang hampir mirip seperti `BlocProvider`, hanya saja ia dikhususkan untuk `Repository`, sehingga `WeatherRepository` akan kita inject menggunakan `RepositoryProvider`. Pada `main.dart`, isikan dengan kode berikut:

```dart
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';

import 'bloc/weather_bloc.dart';
import 'data/data_provider/weather_data_provider.dart';
import 'data/repository/weather_repository.dart';
import 'presentation/screens/weather_screen.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return RepositoryProvider(
      create: (context) => WeatherRepository(WeatherDataProvider()),
      child: BlocProvider(
        create: (context) => WeatherBloc(context.read<WeatherRepository>()),
        child: MaterialApp(
          title: 'Weather App',
          theme: ThemeData(
            colorScheme: ColorScheme.fromSeed(seedColor: Colors.deepPurple),
            useMaterial3: true,
          ),
          home: const WeatherScreen(),
        ),
      ),
    );
  }
}
```

Jika kita memiliki banyak Repository, maka kita bisa menggunakan `MultiRepositoryProvider`, sehingga sangat mirip juga seperti `MultiBlocProvider`. Pada tahap ini yang kita lakukan adalah meng-inject Bloc dan Repository saja, belum mentrigger/memanggil events.

## Mentrigger event

Selanjutnya pada `presentation\screens\weather_screen.dart`, isikan dengan kode berikut

```dart
import 'dart:ui';
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';

import '../../bloc/weather_bloc.dart';
import '../widgets/additional_info_item.dart';

class WeatherScreen extends StatefulWidget {
  const WeatherScreen({super.key});

  @override
  State<WeatherScreen> createState() => _WeatherScreenState();
}

class _WeatherScreenState extends State<WeatherScreen> {
  late Future<Map<String, dynamic>> weather;

  @override
  void initState() {
    super.initState();
    context.read<WeatherBloc>().add(WeatherFetchEvent());
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text(
          'Weather App',
          style: TextStyle(
            fontWeight: FontWeight.bold,
          ),
        ),
        centerTitle: true,
        actions: [
          IconButton(
            onPressed: () {
              context.read<WeatherBloc>().add(WeatherFetchEvent());
            },
            icon: const Icon(Icons.refresh),
          ),
        ],
      ),
      body: BlocBuilder<WeatherBloc, WeatherState>(
        builder: (context, state) {
          if (state is WeatherFailure) {
            return Center(
              child: Text(state.error),
            );
          }

          if (state is! WeatherSuccess) {
            return const Center(
              child: CircularProgressIndicator.adaptive(),
            );
          }

          final data = state.weatherModel;

          return Padding(
            padding: const EdgeInsets.all(16.0),
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                // main card
                SizedBox(
                  width: double.infinity,
                  child: Card(
                    elevation: 10,
                    shape: RoundedRectangleBorder(
                      borderRadius: BorderRadius.circular(16),
                    ),
                    child: ClipRRect(
                      borderRadius: BorderRadius.circular(16),
                      child: BackdropFilter(
                        filter: ImageFilter.blur(
                          sigmaX: 10,
                          sigmaY: 10,
                        ),
                        child: Padding(
                          padding: const EdgeInsets.all(16.0),
                          child: Column(
                            children: [
                              Text(
                                '${data.currentTemp} K',
                                style: const TextStyle(
                                  fontSize: 32,
                                  fontWeight: FontWeight.bold,
                                ),
                              ),
                              const SizedBox(height: 16),
                              Icon(
                                data.currentSky == 'Clouds' ||
                                        data.currentSky == 'Rain'
                                    ? Icons.cloud
                                    : Icons.sunny,
                                size: 64,
                              ),
                              const SizedBox(height: 16),
                              Text(
                                data.currentSky,
                                style: const TextStyle(
                                  fontSize: 20,
                                ),
                              ),
                            ],
                          ),
                        ),
                      ),
                    ),
                  ),
                ),
                const SizedBox(height: 20),
                const Text(
                  'Hourly Forecast',
                  style: TextStyle(
                    fontSize: 24,
                    fontWeight: FontWeight.bold,
                  ),
                ),
                const SizedBox(height: 8),
                // SizedBox(
                //   height: 120,
                //   child: ListView.builder(
                //     itemCount: 5,
                //     scrollDirection: Axis.horizontal,
                //     itemBuilder: (context, index) {
                //       final hourlyForecast = data['list'][index + 1];
                //       final hourlySky =
                //           data['list'][index + 1]['weather'][0]['main'];
                //       final hourlyTemp =
                //           hourlyForecast['main']['temp'].toString();
                //       final time = DateTime.parse(hourlyForecast['dt_txt']);
                //       return HourlyForecastItem(
                //         time: DateFormat.j().format(time),
                //         temperature: hourlyTemp,
                //         icon: hourlySky == 'Clouds' || hourlySky == 'Rain'
                //             ? Icons.cloud
                //             : Icons.sunny,
                //       );
                //     },
                //   ),
                // ),

                const SizedBox(height: 20),
                const Text(
                  'Additional Information',
                  style: TextStyle(
                    fontSize: 24,
                    fontWeight: FontWeight.bold,
                  ),
                ),
                const SizedBox(height: 8),
                Row(
                  mainAxisAlignment: MainAxisAlignment.spaceAround,
                  children: [
                    AdditionalInfoItem(
                      icon: Icons.water_drop,
                      label: 'Humidity',
                      value: data.currentHumidity.toString(),
                    ),
                    AdditionalInfoItem(
                      icon: Icons.air,
                      label: 'Wind Speed',
                      value: data.currentWindSpeed.toString(),
                    ),
                    AdditionalInfoItem(
                      icon: Icons.beach_access,
                      label: 'Pressure',
                      value: data.currentPressure.toString(),
                    ),
                  ],
                ),
              ],
            ),
          );
        },
      ),
    );
  }
}
```

Selanjutnya pada `presentation\widgets\weather_screen.dart`, isikan dengan kode berikut:

```dart
import 'package:flutter/material.dart';

class AdditionalInfoItem extends StatelessWidget {
  final IconData icon;
  final String label;
  final String value;
  const AdditionalInfoItem({
    super.key,
    required this.icon,
    required this.label,
    required this.value,
  });

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        Icon(
          icon,
          size: 32,
        ),
        const SizedBox(height: 8),
        Text(label),
        const SizedBox(height: 8),
        Text(
          value,
          style: const TextStyle(
            fontSize: 16,
            fontWeight: FontWeight.bold,
          ),
        )
      ],
    );
  }
}
```
