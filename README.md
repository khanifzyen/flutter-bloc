> Daftar Isi:
>
> 1. [State Management](#state-management)
>    1. [Apa Itu State Management](#apa-itu-state-management)
>    2. [Jenis-jenis State Management](#jenis-jenis-state-management-flutter)
>       1. [StatefulWidget](#1-statefulwidget)
>       2. [Provider](#provider)
>       3. [Bloc](#bloc)
> 2. [State Management Bloc](#state-management-bloc)
>    1. [Cubit](#cubit)

# State Management

Seperti yang kita ketahui Flutter adalah Cross-Platform Mobile App SDK (Software Development Kit) untuk membuat aplikasi Android dan iOS dalam satu codebase dengan performa tinggi. Hingga saat ini, flutter digunakan untuk mengembangkan aplikasi Web, Linux, dan MacOS.

Lalu, apa itu state? perlukah kita mengelola state ketika mengembangkan aplikasi dengan flutter? Ketika kamu menjelajahi flutter, akan ada waktu dimana kamu akan menggunakan data yang dapat diakses lebih dari satu halaman, atau kamu ingin terjadi perubahan pada screen (layar) ketika data berubah tanpa harus me-redraw semua komponen halaman tersebut.

## Apa itu State Management?

State management adalah sebuah cara untuk mengatur data / state kita bekerja, bisa juga untuk memisahkan antara logic dan view, dimana logic tersebut juga bisa reusable.

![Gambar 01 ilustrasi state management](img/01%20ilustrasi%20state%20management.gif)

Gambar 1. Ilustrasi State Management

Cara kerja state management seperti provide and listen, artinya adalah kamu bisa memasukan state yang kemungkinan bisa berubah sewaktu waktu, lalu widget yang subscribe (listen) dengan provider yang kita buat akan berubah sesuai dengan state yang berubah.

Setiap bagian dalam flutter terbuat dalam bentuk widget. Terdapat dua widget yang memiliki perlakukan state yang berbeda, yaitu stateful dan stateless widget.

Apa perbedaan keduanya?

- **Stateless** merupakan widget yang dimuat secara statis dimana seluruh konfigurasi yang dimuat didalamnya telah diinisialisasikan sejak awal widget itu dimuat. Artinya, state jenis ini tidak dapat diubah dan tidak akan pernah berubah. Stateless biasa digunakan hanya untuk tampilan saja seperti button, item, box container, dan lain-lain.
- **Stateful** merupakan suatu widget yang sifatnya dinamis atau dapat berubah-ubah, kebalikan dari stateless widget. Pada stateful, kamu dapat menggunakan property initState yang berfungsi untuk menginisialisasi state yang akan pertama kali dijalankan. Stateful widget dapat mengubah atau mengupdate tampilan, menambah widget lainnya, mengubah nilai variabel, icon, warna, dan masih banyak lagi.

**Bersifat Deklaratif**

Flutter bersifat deklaratif, artinya flutter membangun user interfacenya dengan merefleksikan setiap perubahan state. State ini lah yang menjadi trigger untuk me-redraw tampilan user interface. Ketika terjadi perubahan, UI akan merebuild secara otomatis dari awal.

## Jenis-jenis State Management Flutter

State manajement memang merupakan salah satu topik pembahasan yang sangat kompleks apabila kita ingin memperdalam tentang flutter. Namun, untuk menggunakan state management, kamu dapat memanfaat kan beberapa package state management berikut.

### 1. StatefulWidget

`StatefulWidget` adalah mekanisme bawaan Flutter untuk menangani state. Mereka terdiri dari dua class: `StatefulWidget` dan `State`. `StatefulWidget` mewakili bagian yang tidak dapat diubah dari widget, sedangkan objek `State` menyimpan data state yang dapat diubah. Ketika data state berubah, memanggil `setState()` akan memicu pembangunan ulang widget, memperbarui UI dengan memanggil lagi fungsi `build()`.

```dart
class MyStatefulWidget extends StatefulWidget {
  @override
  _MyStatefulWidgetState createState() => _MyStatefulWidgetState();
}

class _MyStatefulWidgetState extends State<MyStatefulWidget> {
  int _counter = 0;

  void _incrementCounter() {
    setState(() {
      _counter++;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Column(
      children: <Widget>[
        Text('Counter: $_counter'),
        ElevatedButton(
          onPressed: _incrementCounter,
          child: Text('Increment'),
        ),
      ],
    );
  }
}
```

### 2. Provider

Salah satu paket state management yang disukai untuk Flutter adalah Provider. Paket ini memungkinkan Anda untuk membuat dan mengelola provider yang menyimpan state aplikasi. Widget kemudian dapat mendengarkan provider ini untuk perubahan state. Provider sangat berguna untuk mengelola state global atau state yang digunakan bersama.

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

class MyWidget extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final counter = Provider.of<Counter>(context);

    return Column(
      children: <Widget>[
        Text('Counter: ${counter.value}'),
        ElevatedButton(
          onPressed: () => counter.increment(),
          child: Text('Increment'),
        ),
      ],
    );
  }
}

class Counter with ChangeNotifier {
  int _value = 0;

  int get value => _value;

  void increment() {
    _value++;
    notifyListeners();
  }
}
```

### 3. BloC

BLoC atau Business Logic Component adalah design pattern yang membantu kamu untuk memisahkan presentation dengan business logic. Sehingga komponen pada project terbagi menjadi presentational component, BLoC, dan backend. Pattern ini memperbolehkan developer untuk fokus dalam mengkonversikan event menjadi state.

BLoC mengelola state dengan menggunakan pendekatan stream atau reactive. Secara umum, data akan bergerak dari BLOC ke UI, atau sebaliknya dalam bentuk streams.

```dart
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';

class CounterCubit extends Cubit<int> {
  CounterCubit() : super(0);

  void increment() => emit(state + 1);
}

class MyWidget extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final counterCubit = context.read<CounterCubit>();

    return Column(
      children: <Widget>[
        BlocBuilder<CounterCubit, int>(
          builder: (context, state) {
            return Text('Counter: $state');
          },
        ),
        ElevatedButton(
          onPressed: () => counterCubit.increment(),
          child: Text('Increment'),
        ),
      ],
    );
  }
}
```

# State Management Bloc

Flutter BLOC adalah state management system pada Flutter yang dibuat dan dikembangkan oleh Felix Angelov. BLOC pattern membantu pengelolaan state dan membuat akses ke data tersentralisasi.

Seperti namanya, BLOC menggunakan pendekatan arsitektur business logic dan presentation (view) yang dibuat secara terpisah. Tujuan dari BLOC sendiri adalah membuat kode program aplikasi kamu menjadi lebih mudah dibaca, terstruktur, maintainable, dan testable.

Ketika membangun atau membuat aplikasi dalam production-ready, manajemen pengelolaan state menjadi sangat penting. Oleh karena itu, architectural pattern atau structured project / codebase diperlukan. BLOC menggunakan pendekatan STREAMS atau REACTIVE. Secara umum, data akan bergerak dari BLOC ke UI, atau dari UI ke BLOC, dalam bentuk streams.

Kamu dapat melihat dokumentasi tentang package Bloc lebih lengkap di Bloclibrary.

Fitur utama yang menjadi core concept dalam package BLOC ada 2, yaitu:

- BLOC itu sendiri, dan
- CUBIT

Kedua fitur diatas memiliki kegunaan masing-masing yang dapat digunakan sesuai kondisi dan kebutuhan. Cubit dinilai memiliki fungsi untuk mengelola state yang lebih sederhana dibanding BLOC.

Karena, di dalam Cubit hanya ada interaksi antara business logic dan state saja, tanpa ada event spesifik yang didefinisikan. Berbeda dengan BLOC yang memiliki struktur lengkap mulai dari business logic, event, dan state.

## Cubit

Cubit adalah class yang dapat kamu gunakan di dalam library bloc yang extends dengan Blocbase dan dapat kamu gunakan untuk mengelola jenis state apapun.

Bloc jenis Cubit akan mengekpos function yang dapat dipanggil sebagai trigger perubahan dalam state.

Cara kerja dari Cubit adalah bloc yang menggunakan konsep function-state. Bisa dikatakan konsep Cubit lebih sederhana dan mudah dipahami ketimbang konsep bloc.

### Cara Menggunakan Cubit

Ketika kamu ingin membuat sebuat Cubit, kamu harus mendefinisikan terlebih dahulu type data dari state yang akan di kelola. Kamu dapat mengelola data berupa variabel primitif maupun custom class jika diperlukan.

![Gambar 02 cubit architecture](img/02%20cubit%20architecture.png)

Gambar 2. Cubit architecture
