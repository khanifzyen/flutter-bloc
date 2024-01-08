# Membuat TodoList

> Daftar Isi:
>
> 1. [Kode Awal](#kode-awal)
> 2. [Membuat Todo Model](#membuat-model-todo)
> 3. [Menambahkan Cubit](#menambahkan-cubit)
> 4. [Menambahkan BlocProvider](#menambahkan-blocprovider)
> 5. [Menambahkan BlocBuilder](#menambahkan-blocbuilder)
> 6. [Menambahkan onChange dan onError](#menambahkan-onchange-dan-onerror)
> 7. [Materi Berikutnya Membuat Form Login dengan Bloc](#materi-berikutnya-membuat-login-form-dengan-bloc)

Kali ini kita akan membuat Todo app dengan menggunakan Cubit. Buat project baru dengan nama project_todo dengan perintah `flutter create project_todo`.

## Kode awal

Kita akan mengubah isi dari `main.dart` yang nanti akan berisi daftar todo yang sudah masuk (dipisahkan ke `pages/todo_page.dart`) dan ada `floatingActionButton` plus yang nanti akan masuk ke halaman `add_todo_page.dart`.

`main.dart`

```dart
import 'package:flutter/material.dart';
import 'package:project_todo/pages/todo_page.dart';

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
      home: const TodoPage(),
    );
  }
}
```

`pages\todo_page.dart`

```dart
import 'package:flutter/material.dart';

import 'add_todo_page.dart';

class TodoPage extends StatelessWidget {
  const TodoPage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Todo Page'),
      ),
      body: const Center(
        child: Text('List Todo'),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () {
          Navigator.of(context).push(
            MaterialPageRoute(
              builder: (context) => AddTodoPage(),
            ),
          );
        },
        child: const Icon(Icons.add),
      ),
    );
  }
}
```

`pages\add_todo_page.dart`

```dart
import 'package:flutter/material.dart';

class AddTodoPage extends StatelessWidget {
  AddTodoPage({super.key});

  final TextEditingController _titleController = TextEditingController();

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Add Todo'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(10),
        child: Column(
          children: [
            TextField(
                controller: _titleController,
                decoration: const InputDecoration(
                  labelText: 'Enter title',
                  border: OutlineInputBorder(
                    borderRadius: BorderRadius.all(Radius.circular(10.0)),
                    borderSide: BorderSide(color: Colors.blue),
                  ),
                )),
            const SizedBox(height: 10),
            ElevatedButton(
              onPressed: () {},
              child: const Text('Add Todo'),
            ),
          ],
        ),
      ),
    );
  }
}
```

Simpan dan jalankan.

## Membuat Model Todo

Membuat model akan membuat kode kita lebih safety, artinya ketika kita hanya menggunakan pemanggilan index, kode kita rawan typo. Dengan menggunakan data model/object, maka code editor kita akan memberikan eror sebelum dirunning jika tidak ada property yang cocok.

Untuk Todo sendiri hanya memiliki dua data yaitu title dan createdAt, sehingga kodenya adalah sebagai berikut:

`models\todo.dart`

```dart
class Todo {
  final String name;
  final DateTime createdAt;
  Todo({
    required this.name,
    required this.createdAt,
  });
}
```

> **Tips:**
> Anda bisa menggunakan extensions VSCode "Dart Data Class Generator" buatan hzgood untuk mempercepat pembuatan class model.

## Menambahkan Cubit

Tambahkan terlebih dahulu package `flutter_bloc`, dengan cara "Ctrl + Shift + P", ketik "Add Dependency", tekan Enter, kemudian ketik "flutter_bloc".

Kemudian tambahkan class TodoCubit, dengan ada function addTodo dengan parameter title, dan nanti akan mengisi ke model Todo.:

`cubit\todo_cubit.dart`

```dart
import 'package:flutter_bloc/flutter_bloc.dart';
import '../models/todo.dart';

class TodoCubit extends Cubit<List<Todo>> {
  TodoCubit() : super([]);

  void addTodo(String title) {
    final todo = Todo(
      name: title,
      createdAt: DateTime.now(),
    );

    emit([...state, todo]);
  }
}

```

Untuk state disini bukan int ataupun String melainkan List of Todo, artinya bisa berisi lebih dari satu Todo, dan untuk initialData diisi dengan List kosong (`[]`)

Untuk emit sendiri harus berbeda antara state sebelumnya dengan current state, sehingga penambahan index ditulis di dalam emit.

## Menambahkan BlocProvider

Pada main.dart, kita bungkus MaterialApp dengan BlocProvider agar state bisa diakses di semua widget yang ada di subtree MaterialApp.

`main.dart`

```dart
...
  Widget build(BuildContext context) {
    return BlocProvider(
      create: (context) => TodoCubit(),
      child: MaterialApp(...),
        home: const TodoPage(),
...
```

Jangan lupa tambahkan import package untuk flutter_bloc, dan class TodoCubit.

## Menambahkan BlocBuilder

Sekarang kita perlu bagian yang nanti akan meng-update tampilan jika state berubah, maka perlu kita tambahkan BlocBuilder di dalam todo_page.dart

`todo_page.dart`

```dart
...
      body: Column(
        children: [
          const Text('List of Todos:'),
          Expanded(
            child: BlocBuilder<TodoCubit, List<Todo>>(
              builder: (context, todos) {
                return ListView.builder(
                  itemCount: todos.length,
                  itemBuilder: (context, index) {
                    final todo = todos[index];
                    return ListTile(title: Text(todo.name));
                  },
                );
...
```

## Memanggil method addTodo

Sekarang adalah menjalankan onPressed pada button Add Todo pada add_todo_page.dart

`pages\add_todo_page.dart`

```dart
...
            ElevatedButton(
              onPressed: () {
                context.read<TodoCubit>().addTodo(_titleController.text.trim());
                Navigator.of(context).pop();
              },
              child: const Text('Add Todo'),
...
```

> **Catatan :** `context.read<TodoCubit>()` merupakan alternatif dari `BlocProvider.of<TodoCubit>(context)`

## Menambahkan onChange dan onError

Pada class TodoCubit, memiliki method bawaan onChange dan onError. Ini bisa digunakan kita sebagai programmer untuk berbagai macam hal, semisal validator sederhana "title tidak boleh kosong", dan jika isi dari List todo berubah kita mau diinfokan apa.

```dart
class TodoCubit extends Cubit<List<Todo>> {
  TodoCubit() : super([]);

  void addTodo(String title) {
    if (title.isEmpty) {
      addError('Title cannot be empty');
      return;
    }
    final todo = Todo(
      name: title,
      createdAt: DateTime.now(),
    );

    emit([...state, todo]);
  }

  @override
  void onChange(Change<List<Todo>> change) {
    super.onChange(change);
    print(change);
  }

  @override
  void onError(Object error, StackTrace stackTrace) {
    super.onError(error, stackTrace);
    print(error);
  }
}
```

Simpan dan jalankan. Coba title dikosongi kemudian klik Add Todo, maka pada debug console muncul "Title cannot be empty".

> Tugas !
>
> Lengkapi Todo app diatas dengan button Edit dan Hapus yang berfungsi.

# [Materi Berikutnya: Membuat Login Form dengan Bloc](login.md)
