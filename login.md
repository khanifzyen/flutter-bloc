# Membuat Login Form dengan Bloc

> Daftar Isi:
>
> 1. [Kode Awal](#kode-awal)
> 2. [Membuat Bloc](#membuat-bloc)
>    1. [Menambah Event](#menambahkan-event)
>    2. [Menambah State](#menambahkan-state)
>    3. [Menambah Event Handler](#menambahkan-event-handler)
> 3. [Memahami Aliran Data (Fitur Login) dalam bloc](#memahami-aliran-data-fitur-login-dalam-bloc)
>    1. [Melengkapi state AuthFailure dengan String error dan AuthSuccess dengan String uid](#melengkapi-state-authfailure-dengan-string-error-dan-authsuccess-dengan-string-uid)
>    2. [Melengkapi body dari event handler AuthLoginRequested](#melengkapi-body-dari-event-handler-authloginrequested)
>    3. [Menggunakan BlocProvider](#menggunakan-blocprovider)
>    4. [Memasang event pada tombol Login](#memasang-event-pada-tombol-login)
>    5. [Menambahkan Home Page](#menambahkan-homepage)
>    6. [Menambahkan blok try catch](#menambahkan-blok-try-catch)
>    7. [Membungkus BlocListener](#membungkus-bloclistener)
>    8. [Menambahkan state AuthLoading](#menambahkan-state-authloading)
>    9. [Mengganti BlocListener dengan BlocConsumer](#mengganti-bloclistener-dengan-blocconsumer)
>    10. [Menampilkan uid ketika AuthSuccess pada HomePage](#menampilkan-uid-ketika-authsuccess-pada-homepage)
>    11. [Membuat Logout](#membuat-logout)
>    12. []
> 4. [Memahami Bloc Observer](#memahami-bloc-observer)

Kali ini kita akan membuat Login Form dengan menggunakan Bloc. Buat project baru dengan nama project_todo dengan perintah `flutter create project_login_bloc`.

## Kode Awal

Pada `main.dart`, edit menjadi seperti berikut:

```dart
import 'package:flutter/material.dart';

import 'pages/login_page.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      title: 'Flutter Demo',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.deepPurple),
        useMaterial3: true,
      ),
      home: const LoginPage(),
    );
  }
}
```

Kemudian buat file `pages\login_page.dart`. Isikan dengan kode berikut:

```dart
import 'package:flutter/material.dart';

class LoginPage extends StatefulWidget {
  const LoginPage({super.key});

  @override
  State<LoginPage> createState() => _LoginPageState();
}

class _LoginPageState extends State<LoginPage> {
  final TextEditingController _emailController = TextEditingController();
  final TextEditingController _passwordController = TextEditingController();
  bool isVisible = true;

  void toggleVisible() {
    setState(() {
      isVisible = !isVisible;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text("Login Page"),
      ),
      body: Padding(
        padding: const EdgeInsets.all(10.0),
        child: Column(
          children: [
            TextField(
              controller: _emailController,
              decoration: const InputDecoration(
                labelText: "Email",
                hintText: "Enter your email",
                hintStyle: TextStyle(color: Color.fromARGB(255, 175, 175, 175)),
                border: OutlineInputBorder(
                  borderRadius: BorderRadius.all(Radius.circular(10.0)),
                  borderSide: BorderSide(color: Colors.blue),
                ),
                suffixIcon: Icon(Icons.email),
              ),
            ),
            const SizedBox(height: 10),
            TextField(
                controller: _passwordController,
                obscureText: isVisible,
                decoration: InputDecoration(
                  labelText: "Password",
                  hintText: "Enter your password",
                  hintStyle: const TextStyle(
                      color: Color.fromARGB(255, 175, 175, 175)),
                  border: const OutlineInputBorder(
                    borderRadius: BorderRadius.all(Radius.circular(10.0)),
                    borderSide: BorderSide(color: Colors.blue),
                  ),
                  suffixIcon: IconButton(
                      onPressed: toggleVisible,
                      icon: isVisible
                          ? const Icon(Icons.visibility)
                          : const Icon(Icons.visibility_off)),
                )),
            const SizedBox(height: 10),
            ElevatedButton(onPressed: () {}, child: const Text("Login"))
          ],
        ),
      ),
    );
  }
}
```

Simpan dan jalankan.

## Membuat Bloc

Selanjutnya kita akan membuat bloc. Untuk mempersingkat waktu, anda bisa menginstall VS Code extensions, pilih bloc. Kemudian restart vscode, klik kanan pada folder lib, pilih "Bloc: New Bloc"

![Gambar 4 Extension bloc](img/04%20extensions%20bloc.png)

Gambar 4 Extensions bloc

Selanjutnya anda diminta untuk mengisi nama bloc, isikan dengan "auth", maka akan muncul folder dan file baru.

![Gambar 5 Hasil Generate bloc Extension](img/05%20hasil%20generate%20bloc%20extensions.PNG)

Gambar 5. Hasil Generate bloc Extensions

Selanjutnya adalah anda perlu memahami apa yang akan kita lakukan. Pada login, ketika tombol login di klik, maka kita akan memanggil event "onLoginPressed" dimana nanti akan menghasilkan state "success" atau "failure". Failure nanti menampilkan error misal "email tidak boleh kosong", "password karakter minimal 6" atau yang lainnya. Success nanti menampilkan pesan "berhasil login" dengan berpindah ke halaman lain yang menampilkan email yang sedang login.

### Menambahkan event

Maka kita perlu mengubah `auth_event.dart` dengan menambahkan event `AuthLoginRequested`

`bloc\auth_event.dart`

```dart
part of 'auth_bloc.dart';

@immutable
sealed class AuthEvent {}

final class AuthLoginRequested extends AuthEvent {}

```

### Menambahkan state

Kemudian menambahkan state pada `auth_state.dart` untuk state `AuthSuccess` dan `AuthFailure`

`bloc\auth_state.dart`

```dart
part of 'auth_bloc.dart';

@immutable
sealed class AuthEvent {}

final class AuthLoginRequested extends AuthEvent {}
```

### Menambahkan event handler

Selanjutnya menambahkan event handler dari event yang dibuat sebelumnya yaitu AuthLoginRequested

`bloc\auth_bloc.dart`

```dart
import 'package:flutter_bloc/flutter_bloc.dart';
import 'package:flutter/foundation.dart';

part 'auth_event.dart';
part 'auth_state.dart';

class AuthBloc extends Bloc<AuthEvent, AuthState> {
  AuthBloc() : super(AuthInitial()) {
    on<AuthLoginRequested>((event, emit) {});
  }
}
```

## Memahami Aliran Data (Fitur Login) dalam Bloc

Perlu dipahami oleh kita bagaimana alur data (form login) dalam bloc.

1. Form Email dan Password diisi oleh user
2. Klik tombol Login, tombol Login memanggil `onPressed` yang masuk ke dalam event handler `add()`
3. di dalam `add()` memanggil event `AuthLoginRequested` dengan parameter controller dari email dan password
4. Kemudian ke event handler `on` yang sesuai dengan event `AuthLoginRequested` dengan menggunakan event.email dan event.password sebagai parameter ambilan dari form yang disubmit
5. Menjalankan body dari event handler yang sesuai, semisal pengecekan validasi email, password, dan mengubah state menjadi yang sesuai (jika gagal validasi maka emit state yang `AuthFailure` dengan parameter string error, jika sukses maka emit state yang `AuthSuccess` dengan return response dari API misalnya.)

> **Catatan:** Untuk coding real world, sebenarnya urut dari no 1 sampai 5, cuma nanti akan loncat-loncat ke nomor berikutnya sebagian karena belum dibuat kemudian kembali ke nomor sebelumnya. Untuk memudahkan modul ini, maka akan dibuat terbalik, yaitu dari nomor 5 ke nomor 1.

## Melengkapi state AuthFailure dengan String error dan AuthSuccess dengan String uid

`AuthFailure` perlu ditambahkan parameter String untuk memuat eror, dan `AuthSuccess` untuk saat ini sebagai simulasi model, maka kita bisa gunakan String uid dengan penggabungan "email-password" yang nanti akan muncul di halaman homepage / kedua.

`bloc\auth_state.dart`

```dart
part of 'auth_bloc.dart';

@immutable
sealed class AuthState {}

final class AuthInitial extends AuthState {}

final class AuthSuccess extends AuthState {
  final String uid;

  AuthSuccess({required this.uid});
}

final class AuthFailure extends AuthState {
  final String error;

  AuthFailure(this.error);
}
```

## Mengirim email dan password ke Event AuthLoginRequested

Untuk email dan password akan diolah di event, maka kita perlu membuat parameter email dan password pada AuthLoginRequested.

`auth_event.dart`

```dart
part of 'auth_bloc.dart';

@immutable
sealed class AuthEvent {}

final class AuthLoginRequested extends AuthEvent {
  final String email;
  final String password;

  AuthLoginRequested({
    required this.email,
    required this.password,
  });
}
```

## Melengkapi body dari event handler AuthLoginRequested

Event handler letaknya di `auth_bloc.dart`, kemudian isi dari body event handler, letaknya adalah `on(AuthLoginRequested){}` pada kurung kurawal di dalamnya, sehingga menjadi seperti berikut

`bloc\auth_bloc.dart`

```dart
...
class AuthBloc extends Bloc<AuthEvent, AuthState> {
  AuthBloc() : super(AuthInitial()) {
    on<AuthLoginRequested>(
      (event, emit) async {
        final email = event.email;
        final password = event.password;

        //taruh kode untuk format email menggunakan regex
        if (!RegExp(
                r"^[a-zA-Z0-9.a-zA-Z0-9.!#$%&'*+-/=?^_`{|}~]+@[a-zA-Z0-9]+\.[a-zA-Z]+")
            .hasMatch(email)) {
          return emit(AuthFailure('Email tidak valid'));
        }
        //taruh kode untuk password minimal 6 dan terdiri dari huruf besar, huruf kecil, angka, dan simbol
        if (!RegExp(
                r"^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]{6,}$")
            .hasMatch(password)) {
          return emit(AuthFailure(
              'Password minimal 6 huruf, terdiri dari huruf besar, huruf kecil, angka dan simbol'));
        }

        await Future.delayed(const Duration(seconds: 1), () {
          return emit(AuthSuccess(uid: '$email-$password'));
        });
      },
    );
  }
}
```

> **Catatan:** mengapa menggunakan async, karena ini disimulasikan mengambil data dari internet

## Menggunakan BlocProvider

Urusan bloc sudah, sekarang adalah menyediakan bloc ke UI. Tentu kita perlu bungkus `MaterialApp` dengan `BlocProvider`. Arahkan cursor ke MaterialApp, tekan Ctrl + . pilih "Wrap with BlocProvider". Jangan lupa tambahkan package `flutter_bloc`

`main.dart`

```dart
...
  Widget build(BuildContext context) {
    return BlocProvider(
      create: (context) => AuthBloc(),
      child: MaterialApp(
        debugShowCheckedModeBanner: false,
        title: 'Bloc Login Demo',
        theme: ThemeData(
          colorScheme: ColorScheme.fromSeed(seedColor: Colors.deepPurple),
          useMaterial3: true,
        ),
        home: const LoginPage(),
...
```

## Memasang event pada tombol Login

... kepotong dari rumah belum kesubmit, lupa sampai dimana

## Menambahkan HomePage

`pages\home_page.dart`

```dart
import 'package:flutter/material.dart';

class HomePage extends StatelessWidget {
  const HomePage({super.key});

  @override
  Widget build(BuildContext context) {
    return const Scaffold(
        body: Center(
      child: Text("Home Page"),
    ));
  }
}
```

## Menambahkan blok try catch

`auth_bloc.dart`

```dart
...
    on<AuthLoginRequested>(
      (event, emit) async {
        try {
          final email = event.email;
          final password = event.password;

          //taruh kode untuk format email menggunakan regex
          if (!RegExp(
                  r"^[a-zA-Z0-9.a-zA-Z0-9.!#$%&'*+-/=?^_`{|}~]+@[a-zA-Z0-9]+\.[a-zA-Z]+")
              .hasMatch(email)) {
            return emit(AuthFailure('Email tidak valid'));
          }
          //taruh kode untuk password minimal 6 dan terdiri dari huruf besar, huruf kecil, angka, dan simbol
          if (!RegExp(
                  r"^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]{6,}$")
              .hasMatch(password)) {
            return emit(AuthFailure(
                'Password minimal 6 huruf, terdiri dari huruf besar, huruf kecil, angka dan simbol'));
          }

          await Future.delayed(const Duration(seconds: 1), () {
            return emit(AuthSuccess(uid: '$email-$password'));
          });
        } catch (e) {
          return emit(AuthFailure(e.toString()));
        }
      },
    );
...
```

## Membungkus BlocListener

`pages\login_page.dart`

```dart
...
  Widget build(BuildContext context) {
    return BlocListener<AuthBloc, AuthState>(
      listener: (context, state) {
        if (state is AuthFailure) {
          ScaffoldMessenger.of(context).showSnackBar(
            SnackBar(
              content: Text(state.error),
            ),
          );
        }
        if (state is AuthSuccess) {
          Navigator.pushAndRemoveUntil(
              context,
              MaterialPageRoute(builder: (context) => const HomePage()),
              (route) => false);
        }
      },
      child: Scaffold(...)
...
```

## Menambahkan state AuthLoading

`bloc\auth_state.dart`

```dart
...
final class AuthLoading extends AuthState {}
...
```

Ubah state menjadi `AuthLoading()` pada saat event handler awal

`bloc\auth_bloc.dart`

```dart
    on<AuthLoginRequested>(
      (event, emit) async {
        emit(AuthLoading());
        try {
        ...
```

## Mengganti BlocListener dengan BlocConsumer

Untuk loading indicator hanya memanggil widget CircleLoadingIndicator() dan ini masuk ke UI, sehingga CircleLoadingIndicator tidak cocok dimasukkan ke dalam BlocListener. Untuk itu kita ganti menggunakan BlocConsumer.

BlocConsumer memiliki fitur dari BlocListener untuk listening yang mentrigger method sekaligus memiliki fitur BlocBuilder untuk mem-build UI berdasarkan state yang berubah.

`pages\login_page.dart`

```dart
...
  Widget build(BuildContext context) {
    return BlocConsumer<AuthBloc, AuthState>(
      listener: (...) {...}, //kode masih sama dengan sebelumnya
      builder: (context, state) {
        if (state is AuthLoading) {
          return const Scaffold(
            body: Center(
              child: CircularProgressIndicator(),
            ),
          );
        }
        return Scaffold(...); //kode masih sama dengan sebelumnya
      },
    );
  }
...
```

## Menampilkan uid ketika AuthSuccess pada HomePage

`pages\home_page.dart`

```dart
...
  Widget build(BuildContext context) {
    final authState = context.read<AuthBloc>().state as AuthSuccess;
    return Scaffold(
        body: Center(
      child: Text('Welcome ${authState.uid}'),
    ));
  }
...
```

## Membuat Logout

Dibawah uid nanti akan ditampilkan tombol Logout. Untuk logout sendiri kita perlu membuat event `AuthLogoutRequested`, dan tidak perlu membuat state baru cukup state yang dipanggil adalah `AuthInitial`. Tidak ada parameter yang dikirim.

`bloc\auth_event.dart`

```dart
...
final class AuthLoginRequested extends AuthEvent {...} //kode masih sama

final class AuthLogoutRequested extends AuthEvent {}
```

Kemudian buat event handlernya.

`bloc\auth_bloc.dart`

```dart
...
    on<AuthLogoutRequested>(
      (event, emit) async {
        emit(AuthLoading());
        try {
          await Future.delayed(const Duration(seconds: 1), () {
            return emit(AuthInitial());
          });
        } catch (e) {
          return emit(AuthFailure(e.toString()));
        }
      },
    );
...
```

Kemudian membuat tombol logout, beserta BlocConsumer

`pages\home_page.dart`

```dart
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import 'package:project_login_bloc/bloc/auth_bloc.dart';

import 'login_page.dart';

class HomePage extends StatelessWidget {
  const HomePage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: BlocConsumer<AuthBloc, AuthState>(
        listener: (context, state) {
          if (state is AuthInitial) {
            Navigator.pushAndRemoveUntil(
                context,
                MaterialPageRoute(builder: (context) => const LoginPage()),
                (route) => false);
          }
        },
        builder: (context, state) {
          if (state is AuthLoading) {
            return const Center(
              child: CircularProgressIndicator(),
            );
          }
          if (state is AuthSuccess) {
            return Center(
              child: SingleChildScrollView(
                child: Column(
                  children: [
                    Text('Welcome ${state.uid}'),
                    const SizedBox(height: 10),
                    ElevatedButton(
                      onPressed: () {
                        context.read<AuthBloc>().add(AuthLogoutRequested());
                      },
                      child: const Text("Logout"),
                    )
                  ],
                ),
              ),
            );
          }
          return const SizedBox.shrink();
        },
      ),
    );
  }
}
```

Simpan dan jalankan.

## Memindahkan body constructor untuk event handler

Untuk event handler, body constructor kita pindahkan ke method diluar, sehingga ada method baru `_onLoginRequested()` dan `_onLogoutRequested()`. Kodenya menjadi seperti berikut:

```dart
import 'package:flutter_bloc/flutter_bloc.dart';
import 'package:flutter/foundation.dart';

part 'auth_event.dart';
part 'auth_state.dart';

class AuthBloc extends Bloc<AuthEvent, AuthState> {
  AuthBloc() : super(AuthInitial()) {
    on<AuthLoginRequested>(_onAuthLoginRequested);
    on<AuthLogoutRequested>(_onAuthLogoutRequested);
  }

  void _onAuthLoginRequested(
      AuthLoginRequested event, Emitter<AuthState> emit) async {
    emit(AuthLoading());
    try {
      final email = event.email;
      final password = event.password;

      //taruh kode untuk format email menggunakan regex
      if (!RegExp(
              r"^[a-zA-Z0-9.a-zA-Z0-9.!#$%&'*+-/=?^_`{|}~]+@[a-zA-Z0-9]+\.[a-zA-Z]+")
          .hasMatch(email)) {
        return emit(AuthFailure('Email tidak valid'));
      }
      //taruh kode untuk password minimal 6 dan terdiri dari huruf besar, huruf kecil, angka, dan simbol
      if (!RegExp(
              r"^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]{6,}$")
          .hasMatch(password)) {
        return emit(AuthFailure(
            'Password minimal 6 huruf, terdiri dari huruf besar, huruf kecil, angka dan simbol'));
      }

      await Future.delayed(const Duration(seconds: 1), () {
        return emit(AuthSuccess(uid: '$email-$password'));
      });
    } catch (e) {
      return emit(AuthFailure(e.toString()));
    }
  }

  void _onAuthLogoutRequested(
      AuthLogoutRequested event, Emitter<AuthState> emit) async {
    emit(AuthLoading());
    try {
      await Future.delayed(const Duration(seconds: 1), () {
        return emit(AuthInitial());
      });
    } catch (e) {
      return emit(AuthFailure(e.toString()));
    }
  }
}
```

## Memahami Bloc Observer

Jika didalam Cubit ada onChange, di dalam Bloc juga memiliki onChange, ditambah juga ada onTransition. Ia akan mencatat state apa yang berubah, yaitu state sebelumnya dan current state.

Kita bisa membuat satu class khusus yang memuat semua bloc (jika suatu saat aplikasi kita sudah membengkak, memiliki ratusan bloc). Caranya adalah buat sebuah class MyBlocObserver.

`my_bloc_observer.dart`

```dart
import 'package:flutter_bloc/flutter_bloc.dart';

class MyBlocObserver extends BlocObserver {
  @override
  void onCreate(BlocBase bloc) {
    super.onCreate(bloc);
    print('${bloc.runtimeType} Created!');
  }

  @override
  void onChange(BlocBase bloc, Change change) {
    super.onChange(bloc, change);
    print('${bloc.runtimeType} Changed $change');
  }
}
```

Kemudian tambahkan baris berikut di `main.dart`

```dart
...
void main() {
  Bloc.observer = MyBlocObserver();
  runApp(const MyApp());
}
...
```

Simpan dan jalankan. Nanti akan bisa dilihat di debug console.

![Gambar 6 bloc observer](img/06%20bloc%20observer.PNG)
Gambar 6 Bloc Observer

## [Materi Berikutnya: Membuat WeatherApp menggunakan Bloc Pattern](weatherapp.md)
