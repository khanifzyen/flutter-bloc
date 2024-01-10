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
>    4. [Memasang event pada tombol Login]

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
