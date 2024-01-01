---
label: Flutter Examples
icon: <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" id="flutter"><path d="M13.9 2.01 3.9 12l3.09 3.09 2.71-2.7L20.09 2l-6.19.01zm.82 14.6 5.39-5.38h-5.93c-.11 0-.26 0-.34.07l-2.23 2.23-3.09 3.07 3.09 3.1 2.15 2.15c.07.07.14.17.26.15h6.07z"></path></svg>
order: 1
---

Here we discuss about a flutter todo application using Nitrite database. It uses nitrite as a file based storage engine. It also uses Riverpod for state management. It demonstrates the use of Nitrite database in a flutter application. The full source code is available [here](https://github.com/nitrite/nitrite-flutter/tree/main/examples/nitrite_demo). This tutorial assumes that you have basic knowledge of flutter and riverpod.

## Setup

Add the following Nitrite dependencies in your `pubspec.yaml` file along with `path_provider` and `riverpod_generator`:

```yaml
dependencies:
  nitrite: ^[latest-version]
  nitrite_hive_adapter: ^[latest-version]
  path_provider: ^2.0.15
  riverpod_annotation: ^2.1.1

dev_dependencies:
  build_runner: ^2.4.6
  riverpod_generator: ^2.2.3
  nitrite_generator: ^[latest-version]
```

## Entity Classes

Define a simple `Todo` entity class to hold your todo data:

```dart
import 'package:nitrite/nitrite.dart';

part 'models.no2.dart';

@Entity(name: 'todo', indices: [
  Index(fields: ['title'], type: IndexType.fullText),
])
@Convertable()
class Todo with _$TodoEntityMixin {
  @Id(fieldName: 'id')
  final String id;
  final String title;
  bool completed = false;

  Todo({
    required this.id,
    required this.title,
    required this.completed,
  });

  @override
  bool operator ==(Object other) =>
      identical(this, other) ||
      other is Todo &&
          runtimeType == other.runtimeType &&
          id == other.id &&
          title == other.title &&
          completed == other.completed;

  @override
  int get hashCode => id.hashCode ^ title.hashCode ^ completed.hashCode;

  @override
  String toString() {
    return 'Todo{id: $id, title: $title, completed: $completed}';
  }
}
```

And run the following command to generate the `_$TodoEntityMixin`:

```bash
dart run build_runner build
```

This command will generate the `models.no2.dart` file in the same directory.

## Database Initialization

Next create a Nitrite database provider using riverpod:

```dart
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:nitrite/nitrite.dart';
import 'package:nitrite_hive_adapter/nitrite_hive_adapter.dart';
import 'package:path_provider/path_provider.dart';
import 'package:riverpod_annotation/riverpod_annotation.dart';

part 'providers.g.dart';

@riverpod
Future<Nitrite> db(DbRef ref) async {
  var docPath = await getApplicationDocumentsDirectory();
  var dbDir = await Directory('${docPath.path}${Platform.pathSeparator}db')
      .create(recursive: true);
  var storeModule =
      HiveModule.withConfig().crashRecovery(true).path(dbDir.path).build();

  var db = await Nitrite.builder()
      .loadModule(storeModule)
      .fieldSeparator('.')
      .registerEntityConverter(TodoConverter())
      .openOrCreate(username: 'demo', password: 'demo123');

  return db;
}
```

Now using the `dbProvider`, create a todo `ObjectRepository` provider:

```dart
@riverpod
Future<ObjectRepository<Todo>> todoRepository(TodoRepositoryRef ref) async {
  var db = await ref.read(dbProvider.future);
  return await db.getRepository<Todo>();
}
```

And other required providers for listing and searching:

```dart
@riverpod
class Todos extends _$Todos {
  Future<List<Todo>> _fetchTodo() async {
    var repository = await ref.read(todoRepositoryProvider.future);
    var filter = ref.watch(filterProvider);
    var findOptions = ref.watch(findOptionStateProvider);
    return repository.find(filter: filter, findOptions: findOptions).toList();
  }

  @override
  Future<List<Todo>> build() async {
    return _fetchTodo();
  }

  Future<void> addTodo(Todo todo) async {
    state = const AsyncValue.loading();

    try {
      var repository = await ref.read(todoRepositoryProvider.future);
      await repository.insert(todo);
      state = AsyncValue.data(await _fetchTodo());
    } catch (e, s) {
      state = AsyncValue.error(e, s);
    }
  }

  Future<void> removeTodo(String todoId) async {
    state = const AsyncValue.loading();
    try {
      var repository = await ref.read(todoRepositoryProvider.future);
      await repository.remove(where('id').eq(todoId));
      state = AsyncValue.data(await _fetchTodo());
    } catch (e, s) {
      state = AsyncValue.error(e, s);
    }
  }

  Future<void> toggle(String todoId) async {
    state = const AsyncValue.loading();
    try {
      var repository = await ref.read(todoRepositoryProvider.future);
      var byId = await repository.getById(todoId);
      if (byId != null) {
        byId.completed = !byId.completed;
        await repository.updateOne(byId);
      }
      state = AsyncValue.data(await _fetchTodo());
    } catch (e, s) {
      state = AsyncValue.error(e, s);
    }
  }
}

@riverpod
class FindOptionState extends _$FindOptionState {
  @override
  FindOptions build() {
    return FindOptions(
      orderBy: SortableFields.from([('title', SortOrder.ascending)]),
      skip: 0,
      limit: 10,
    );
  }
}

final filterProvider = StateProvider<Filter>((ref) => all);
final todoTextProvider = StateProvider<String>((ref) => '');

@riverpod
int pendingCounter(PendingCounterRef ref) {
  var todos = ref.watch(todosProvider);
  return todos.when(
    data: (todoList) => todoList.where((todo) => !todo.completed).length,
    loading: () => 0,
    error: (err, stack) => 0,
  );
}

@riverpod
int completedCounter(CompletedCounterRef ref) {
  var todos = ref.watch(todosProvider);
  return todos.when(
    data: (todoList) => todoList.where((todo) => todo.completed).length,
    loading: () => 0,
    error: (err, stack) => 0,
  );
}
```

Now again run the following command to generate the providers:

```bash
dart run build_runner build
```

## UI

Now define a todo widget to display a single time and define different actions on it:

```dart
class TodoWidget extends ConsumerWidget {
  final Todo todo;

  const TodoWidget({super.key, required this.todo});

  bool _isDesktop() =>
      Platform.isLinux || Platform.isMacOS || Platform.isWindows;

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    if (_isDesktop()) {
      return Card(
        color: Theme.of(context).colorScheme.surface,
        child: ListTile(
          title: Text(
            todo.title,
            style: TextStyle(
              decoration: todo.completed ? TextDecoration.lineThrough : null,
            ),
          ),
          trailing: Row(
            mainAxisSize: MainAxisSize.min,
            children: [
              IconButton(
                onPressed: () =>
                    ref.read(todosProvider.notifier).toggle(todo.id),
                icon: Icon(
                  todo.completed ? Icons.task_alt : Icons.check_box_outlined,
                  color: todo.completed ? Colors.green : Colors.grey,
                ),
              ),
              IconButton(
                onPressed: () =>
                    ref.read(todosProvider.notifier).removeTodo(todo.id),
                icon: const Icon(
                  Icons.delete,
                  color: Colors.red,
                ),
              ),
            ],
          ),
        ),
      );
    } else {
      return Slidable(
        key: ValueKey(todo.id),
        startActionPane: ActionPane(
          motion: const ScrollMotion(),
          children: [
            SlidableAction(
              borderRadius: BorderRadius.circular(5),
              spacing: 10,
              onPressed: (context) =>
                  ref.read(todosProvider.notifier).toggle(todo.id),
              backgroundColor: Colors.green,
              foregroundColor: Colors.white,
              icon: todo.completed ? Icons.task_alt : Icons.check_box_outlined,
            ),
          ],
        ),
        endActionPane: ActionPane(
          motion: const ScrollMotion(),
          children: [
            SlidableAction(
              borderRadius: BorderRadius.circular(5),
              spacing: 10,
              onPressed: (context) =>
                  ref.read(todosProvider.notifier).removeTodo(todo.id),
              backgroundColor: Colors.red,
              foregroundColor: Colors.white,
              icon: Icons.delete,
            ),
          ],
        ),
        child: ListTile(
          leading: const Icon(Icons.adjust, color: Colors.black26),
          shape: RoundedRectangleBorder(
            borderRadius: BorderRadius.circular(5),
          ),
          title: Text(
            todo.title,
            style: TextStyle(
              decoration: todo.completed ? TextDecoration.lineThrough : null,
            ),
          ),
        ),
      );
    }
  }
}
```

And a todo list widget to display the list of todos:

```dart
class TodoList extends ConsumerWidget {
  const TodoList({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // Get the todos from the provider.
    var todos = ref.watch(todosProvider);

    return todos.when(
      data: (todoList) => ListView.builder(
        itemCount: todoList.length,
        itemBuilder: (context, index) {
          Todo todo = todoList[index];
          return Padding(
            padding: const EdgeInsets.only(left: 30.0, right: 30.0),
            child: TodoWidget(todo: todo),
          );
        },
      ),
      error: (err, stack) => Text('Error: $err\n$stack'),
      loading: () => const Center(
        child: CircularProgressIndicator(),
      ),
    );
  }
}
```

Now you can use these widgets in your app and other widgets to add new todo, search, etc.
