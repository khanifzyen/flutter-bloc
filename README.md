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
>       1. [Cara Menggunakan Cubit (Cubit+BlocBuilder)](#cara-menggunakan-cubit)
>       2. [Parsing Cubit ke Halaman Lain (Cubit+BlocProvider)](#parsing-cubit-ke-halaman-lain)
>    2. [BLOC](#bloc)
>       1. [Cara Menggunakan BLOC](#cara-menggunakan-bloc)
>       2. [Membuat MultiBlocProvider dan multi event](#membuat-multiblocprovider-dan-multi-event)
> 3. [Materi Berikutnya: Membuat TodoList dengan Cubit](#materi-berikutnya-membuat-todolist-dengan-cubit)

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

Kali ini kita akan membuat Counter app bawaan yang akan kita ganti menggunakan Cubit. Buat project baru dengan nama project_bloc dengan peritah `flutter create project_bloc`

Pada contoh yang akan kita bahas kali ini, kita hanya menggunakan data sederhana yaitu berupa variabel integer saja. Jangan lupa tambahkan package `flutter_bloc` terlebih dahulu.

Langkah pertama adalah buat file yang extends dengan class `Cubit`, kemudian beri nama file tersebut dengan `counter_cubit.dart`.

file `cubit\counter_cubit.dart`

```dart
import 'package:flutter_bloc/flutter_bloc.dart';

class CounterCubit extends Cubit<int> {
  CounterCubit() : super(0);
  void increment() => emit(state + 1);
  void decrement() {
    if (state > 0) {
      return emit(state - 1);
    }
  }
}
```

Kode diatas berarti kita membuat class CounterCubit yang merupakan turunan cari class Cubit yang memiliki tipe integer. Artinya adalah state ini bertipe integer, kemudian memanggil contructor dari class Cubit yang meminta parameter `initialState` yaitu nilai 0 (`super(0)`). Selanjutnya ada method `increment()` yang me-return `emit(state + 1)` berarti state baru akan ditimpa dengan state lama ditambah 1 kemudian akan di stream / diberitahukan ke semua widget yang menggunakan `CounterCubit`. Kemudian ada method `decrement()` yang mensyaratkan nantinya state tidak ada nilai minus, sehingga jika belum sampai 0, maka state lama dikurangi 1 kemudian akan di-stream/diberitahukan ke semua widget yang menggunakan `CounterCubit`.

Langkah kedua adalah berkaitan dengan UI, yaitu membuat instance `counterCubit` yang berasal dari class `CounterCubit`, dan membungkus `Text` dengan `BlocBuilder` karena kita ingin hanya bagian `Text` ini saja nanti yang akan di-rebuild. Ubah terlebih dahulu class `MyHomePage` dari `StatefulWidget` menjadi `StatelessWidget`.

```dart
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import 'cubit/counter_cubit.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter Demo',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.deepPurple),
        useMaterial3: true,
      ),
      home: MyHomePage(title: 'Flutter Demo Home Page'),
    );
  }
}

class MyHomePage extends StatelessWidget {
  MyHomePage({super.key, required this.title});

  final String title;

  final CounterCubit counterCubit = CounterCubit();

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        backgroundColor: Theme.of(context).colorScheme.inversePrimary,
        title: Text(title),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            const Text(
              'You have pushed the button this many times:',
            ),
            BlocBuilder<CounterCubit, int>(
              bloc: counterCubit,
              builder: (context, state) {
                return Text(
                  '$state',
                  style: Theme.of(context).textTheme.headlineMedium,
                );
              },
            ),
          ],
        ),
      ),
      floatingActionButton: Column(
        mainAxisAlignment: MainAxisAlignment.end,
        children: [
          FloatingActionButton(
            onPressed: counterCubit.increment,
            tooltip: 'Increment',
            child: const Icon(Icons.add),
          ),
          const SizedBox(height: 10),
          FloatingActionButton(
            onPressed: counterCubit.decrement,
            tooltip: 'Decrement',
            child: const Icon(Icons.remove),
          ),
        ],
      ),
    );
  }
}
```

### Parsing Cubit ke Halaman Lain

Ada kalanya kita ingin nilai/state dari counter bisa diakses di halaman lain. Agar ini bisa terjadi maka kita butuh yang namanya **Dependency Injection**. Di dalam `flutter_bloc` selain ada `BlocBuilder`, terdapat juga `BlocProvider` yang berguna agar state tersebut bisa digunakan di halaman/widget manapun, asal masih dalam subtree widget induk dimana `BlocProvider` berada.

Kita akan memodifikasi Counter app diatas dengan memindahkan dua button increment dan decrement menjadi custom widget tersendiri. Secara tidak langsung, kita akan mendaftarkan counterCubit ke dalam BlocProvider. Ingat, kita tidak akan memparsing CounterCubit kedalam constructor custom widget.

Langkah pertama adalah bungkus widget utama setelah `MaterialApp.home` dengan BlocProvider:

```dart
...
  Widget build(BuildContext context) {
    return MaterialApp(
      ...
      home: BlocProvider(
        create: (context) => CounterCubit(),
        child: const MyHomePage(title: 'Flutter Demo Home Page'),
      ),
    );
  }
  ...
```

Setelah itu pindahkan `counterCubit` ke dalam fungsi `build()`, dengan berisi `BlocProvider.of<CounterCubit>(context)`, kemudian cut isi floatingActionButton dan isi dengan `IncDecButton()`.

```dart
class MyHomePage extends StatelessWidget {
  ...
  Widget build(BuildContext context) {
    final counterCubit = BlocProvider.of<CounterCubit>(context);
    return Scaffold(
      appBar: AppBar(...),
      body: ...,
      floatingActionButton: const IncDecButton(), //jangan lupa untuk mengimpor nanti setelah class ini dibuat
```

Langkah kedua yaitu buat file baru, dengan paste isi dari floatingActionButton ke dalam `widgets/inc_dec_button.dart`. Kode lengkapnya sebagai berikut,

```dart
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import '../cubit/counter_cubit.dart';

class IncDecButton extends StatelessWidget {
  const IncDecButton({super.key});

  @override
  Widget build(BuildContext context) {
    final counterCubit = BlocProvider.of<CounterCubit>(context);

    return Column(
      mainAxisAlignment: MainAxisAlignment.end,
      children: [
        FloatingActionButton(
          onPressed: counterCubit.increment,
          tooltip: 'Increment',
          child: const Icon(Icons.add),
        ),
        const SizedBox(height: 10),
        FloatingActionButton(
          onPressed: counterCubit.decrement,
          tooltip: 'Decrement',
          child: const Icon(Icons.remove),
        ),
      ],
    );
  }
}
```

Simpan dan jalankan ulang. Sekilas tampilan masih sama, tetapi sebenarnya konsep Dependency Injection telah dijalankan dengan baik.

## BLOC

`Bloc` adalah class yang lebih lengkap ditambah dengan adanya class `events` untuk mentrigger perubahan pada `state`, tidak seperti `Cubit` yang menggunakan function.

`Bloc` dan `Cubit` keduanya extends dengan `BlocBase` yang memiliki kesamaan dalam hal public API.

Dibandingkan memanggil sebuah function didalam bloc dan langsung mengeksekusi state baru, Bloc lebih terstruktur dengan menerima `events`, dan merubah `event` tersebut menjadi `state` sebagai output.

### Cara Menggunakan BLOC

Sebenarnya, cara menggunakan `Bloc` mirip dengan membuat sebuah `Cubit`. Perbedaannya terletak pada bagaimana cara kita mengelola `state` menggunakan `event`.

Events merupakan masukkan (input) pada `Bloc`. Events biasanya dieksekusi ketika adanya interaksi pengguna seperti menekan tombol.

![Gambar 03. BLOC architecture](img/03%20bloc%20architecture.png)

Gambar 3. Bloc Architecture

Masih di project yang sama, kita akan membuat Counter app tetapi menggunakan Bloc, yang sebelumnya menggunakan Cubit.

`bloc\counter_bloc.dart`

```dart
import 'package:flutter_bloc/flutter_bloc.dart';

class CounterIncremented {}

class CounterBloc extends Bloc<CounterIncremented, int> {
  CounterBloc() : super(0) {
    on<CounterIncremented>((event, emit) {
      emit(state + 1);
    });
  }
}
```

Jika pada `Cubit` hanya menerima satu tipe parameter yaitu `state`, pada `Bloc` menerima dua tipe parameter yaitu yang pertama adalah `Event`, yang kedua adalah `state`. Untuk event sebagai class yang nanti akan ditrigger pada body constructor. Kemudian method `on` yang diikuti parameter `Event` bertindak sebagai event handler, artinya ketika ada event yang cocok, maka yang ada di dalam kurung buka awak akan dieksekusi, dimana ia merima parameter 2 yaitu `event` dan `emit`. Parameter `event` sesuai dengan deklarasi class Event yang masuk pada method `on` tersebut.

Selanjutnya adalah memasukkan `CounterBloc` kedalam `BlocProvider` sehingga bisa diakses ke dalam widget tree. Karena kita ingin `CounterCubit` tetap ada, maka bungkus lagi dengan `BlocProvider` untuk `CounterBloc`.

```dart
...
      home: BlocProvider(
        create: (context) => CounterBloc(),
        child: BlocProvider(
          create: (context) => CounterCubit(),
          child: const MyHomePage(title: 'Flutter Demo Home Page'),
          ...
            BlocBuilder<CounterBloc, int>(
              builder: (context, state) {
                return Text(
                  ...
...
```

Kemudian ubah juga `inc_dec_button.dart`

```dart
...
  Widget build(BuildContext context) {
    // final counterCubit = BlocProvider.of<CounterCubit>(context);
    final counterBloc = BlocProvider.of<CounterBloc>(context);
    ...
        FloatingActionButton(
          onPressed: () {
            counterBloc.add(CounterIncremented());
          },
          tooltip: 'Increment',
          child: const Icon(Icons.add),
        ),
        const SizedBox(height: 10),
        FloatingActionButton(
          //onPressed: counterCubit.decrement,
          onPressed: () {},
          tooltip: 'Decrement',
          child: const Icon(Icons.remove),
          ...
```

Bisa dilihat pada tombol Add, disini ada method `add` yang didapat dari counterBloc, dan diisi dengan nama event. Jadi triggernya adalah event, bukan function seperti pada Cubit.

Simpan dan lakukan ujicoba, seharusnya hasilnya seolah tidak ada yang berubah tapi kita sudah menerapkan konsep bloc, event pada dengan menggunakan counterBloc.

### Membuat MultiBlocProvider dan multi event

Bagaimana jika seumpama kode yang anda buat memiliki puluhan atau ratusan bloc? Tentu nanti nested `BlocProvider` seperti penulisan kode sebelumnya akan semakin dalam. Kita bisa menggunakan `MultiBlocProvider`, seperti berikut:

```dart
...
      home: MultiBlocProvider(
        providers: [
          BlocProvider(create: (_) => CounterBloc()),
          BlocProvider(create: (_) => CounterCubit()),
        ],
        child: const MyHomePage(title: 'Flutter Demo Home Page'),
...
```

Karena ada event untuk increment dan decrement, maka sebaiknya kita pisah ke dalam class baru:

`bloc\counter_event.dart`:

```dart
part of 'counter_bloc.dart';

sealed class CounterEvent {}

final class CounterIncremented extends CounterEvent {}

final class CounterDecremented extends CounterEvent {}
```

`bloc\counter_bloc.dart`:

```dart
import 'package:flutter_bloc/flutter_bloc.dart';

part 'counter_event.dart';

class CounterBloc extends Bloc<CounterEvent, int> {
  CounterBloc() : super(0) {
    on<CounterIncremented>((event, emit) {
      emit(state + 1);
    });
    on<CounterDecremented>((event, emit) {
      emit(state - 1);
    });
  }
}
```

`widgets\inc_dec_button.dart`

```dart
...
  Widget build(BuildContext context) {
  ...
        FloatingActionButton(
          onPressed: () {
            counterBloc.add(CounterIncremented());
          },
          tooltip: 'Increment',
          child: const Icon(Icons.add),
        ),
        const SizedBox(height: 10),
        FloatingActionButton(
          //onPressed: counterCubit.decrement,
          onPressed: () {
            counterBloc.add(CounterDecremented());
          },
          tooltip: 'Decrement',
          child: const Icon(Icons.remove),
...
```

Simpan dan jalankan.

# [Materi Berikutnya: Membuat TodoList dengan Cubit](todolist.md)
