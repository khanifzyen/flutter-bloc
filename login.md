# Membuat Login Form dengan Bloc

> Daftar Isi:
>
> 1. [Kode Awal](#kode-awal)
> 2. [Membuat Bloc](#membuat-bloc)
>    1. [Menambah Event](#menambahkan-event)
>    2. [Menambah State](#menambahkan-state)
>    3. [Menambah Event Handler](#menambahkan-event-handler)
> 3. [Mengirim email dan password dan mengirim ke Event AuthLoginRequested](#mengirim-email-dan-password-ke-event-authloginrequested)

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

## Mengirim email dan password ke Event AuthLoginRequested
