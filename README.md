
markdown
Copy code
# Flutter Riverpod MVVM Template

A comprehensive Flutter application template using Riverpod for state management and MVVM architecture. This template includes localization, flavor separation, initialization with dart define JSON, helpful extensions, application preferences, and more. It also demonstrates best practices for using GoRouter for navigation, logging, and interceptors with Dio.

## Features

- State Management: Riverpod
- Architecture: MVVM (Model-View-ViewModel)
- Localization
- Flavor Separation (Environments)
- Initialization with dart define JSON
- Helpful Extensions (e.g., date-time formatting)
- Application Preferences
- Theming: Text themes and component colors generated from XML using flutter_gen
- Asset Generation: Using flutter_gen
- App Preparation: Using Melos
- Native Splash Screen Generator: Using flutter_native_splash
- Navigation: GoRouter
- Logging: Logger and observer patterns
- Network Requests: Dio with interceptors

## Getting Started

### Prerequisites

- Flutter SDK
- Dart SDK

### Installation

Clone the repository:

```bash
git clone https://github.com/yourusername/flutter_riverpod_mvvm_template.git
cd flutter_riverpod_mvvm_template
```

Get Project Dependencies And Generate Code:
Note: Check is your melos activated on dart.

```bash
melos prepare
```

## Project Structure

```plaintext
lib/
├── src/
│   ├── auth/
│   ├── router/
│   ├── screen/
│   ├── utils/
│   └── ...
├── main.dart
└── ...
```
## Key Points
### Usage
Localization
This template uses the easy_localization package for localization. Ensure you have added your translations in the appropriate language files.

### Flavor Separation
The template supports different environments (flavors) such as development, staging, and production. Use dart define JSON for initialization.

### Navigation
This template uses GoRouter for navigation. The navigation is managed through a central AppRouter class, ensuring all routes and navigations are handled efficiently.

### Network Requests
Dio is used for making network requests with interceptors for handling errors and logging.

## Feature Wised Implementation Example
Example Work On Google Books Api.

### View>

```dart
import 'package:flutter_mvvm_template/src/router/app_router.dart';
import 'package:flutter_mvvm_template/src/screen/books/data/book_item.dart';
import 'package:flutter_mvvm_template/src/screen/books/data/books_repository.dart';
import 'package:flutter_mvvm_template/src/screen/books/presentation/widgets/book_card.dart';
import 'package:flutter_mvvm_template/src/screen/books/presentation/widgets/search/search_notifier.dart';
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

class BooksScreen extends ConsumerWidget {
  const BooksScreen({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final searchQuery = ref.watch(searchProvider);
    final bookItemsAsyncValue = ref.watch(bookItemsListProvider(searchQuery));
    final favoriteBooks = ref.watch(favoriteBooksProvider);

    ref.listen<AsyncValue<List<BookItem>>>(
      bookItemsListProvider(searchQuery),
      (previous, next) {
        next.whenOrNull(
          error: (error, stack) {
            ScaffoldMessenger.of(context).showSnackBar(
              SnackBar(
                content: Text('Error: $error'),
              ),
            );
          },
        );
      },
    );

    return Scaffold(
      appBar: AppBar(
        title: const Text('Kitaplarım'),
        actions: <Widget>[
          IconButton(
            icon: const Icon(Icons.favorite, color: Colors.white),
            onPressed: () {
              // Navigate to favorite books screen
              ref.read(goRouterProvider).goNamed(AppRoute.favorites.name);
            },
          ),
        ],
        toolbarOpacity: 0.7,
        bottomOpacity: 0.7,
        bottom: PreferredSize(
          preferredSize: const Size.fromHeight(56.0),
          child: Padding(
            padding: const EdgeInsets.symmetric(horizontal: 16.0, vertical: 8),
            child: TextField(
              style: Theme.of(context)
                  .textTheme
                  .bodyMedium
                  ?.copyWith(color: Colors.white),
              onChanged: (query) =>
                  ref.read(searchProvider.notifier).updateQuery(query),
              decoration: InputDecoration(
                hintText: 'Kitap ara...',
                hintStyle: Theme.of(context)
                    .textTheme
                    .bodyMedium
                    ?.copyWith(color: Colors.white),
                prefixIcon: const Icon(
                  Icons.search,
                  color: Colors.white,
                ),
                border: OutlineInputBorder(
                  borderRadius: BorderRadius.circular(8.0),
                  borderSide: BorderSide.none,
                ),
                filled: true,
                fillColor: Colors.white.withOpacity(0.2),
              ),
            ),
          ),
        ),
      ),
      body: Center(
        child: bookItemsAsyncValue.when(
          data: (books) {
            if (books.isEmpty) {
              return Row(
                crossAxisAlignment: CrossAxisAlignment.center,
                mainAxisAlignment: MainAxisAlignment.center,
                children: [
                  const Icon(Icons.book),
                  Text(
                    'No books found',
                    style: Theme.of(context).textTheme.bodyLarge,
                  ),
                ],
              );
            }
            return ListView.builder(
              padding: const EdgeInsets.all(8.0),
              itemCount: books.length,
              itemBuilder: (context, index) {
                final book = books[index];
                final isFavorite =
                    favoriteBooks.any((favBook) => favBook.id == book.id);
                return BookCard(
                    book: book,
                    isFavorite: isFavorite,
                    deleteFavorite: () {
                      ref
                          .read(booksRepositoryProvider.notifier)
                          .removeFavoriteBook(book.id ?? '');
                      ScaffoldMessenger.of(context).showSnackBar(
                        const SnackBar(
                          content: Text('Kitap favorilerinizden kaldırıldı.'),
                        ),
                      );
                    },
                    onFavorite: () {
                      ref
                          .read(booksRepositoryProvider.notifier)
                          .addFavoriteBook(book);
                      ScaffoldMessenger.of(context).showSnackBar(
                        const SnackBar(
                          backgroundColor: Colors.green,
                          content: Text('Kitap favorilerinize eklendi.'),
                        ),
                      );
                    });
              },
            );
          },
          loading: () => const Center(child: CircularProgressIndicator()),
          error: (error, stack) {
            return const Center(child: Text('Hata'));
          },
        ),
      ),
    );
  }
}

```
### View Model
```dart
import 'package:arteria/src/auth/data/auth_repository.dart';
import 'package:arteria/src/screen/books/data/books_repository.dart';
import 'package:riverpod_annotation/riverpod_annotation.dart';

part 'books_screen_controller.g.dart';

@riverpod
class BooksScreenController extends _$BooksScreenController {
  @override
  FutureOr<void> build() {}

  BooksRepository get booksRepository =>
      ref.read(booksRepositoryProvider.notifier);

  fetchBooks(String query) async {
    final currentUser = ref.read(authRepositoryProvider).currentUser;
    if (currentUser == null) {
      throw AssertionError('User can\'t be null');
    }
    state = const AsyncLoading();
    final response = booksRepository.fetchBooks(query);

    state = await AsyncValue.guard(() => response);
    return response;
  }
}

```

### Repository
```dart

import 'package:arteria/src/auth/data/auth_repository.dart';
import 'package:arteria/src/screen/books/data/book_item.dart';
import 'package:arteria/src/screen/books/data/book_response.dart';
import 'package:arteria/src/screen/books/data/books_api_service.dart';
import 'package:arteria/src/tools/hive_boxes.dart';
import 'package:arteria/src/utils/client/dio/dio_providers.dart';
import 'package:arteria/src/utils/env/launch.env.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:hive/hive.dart';
import 'package:riverpod_annotation/riverpod_annotation.dart';

part 'books_repository.g.dart';

class BooksRepository extends StateNotifier<List<BookItem>> {
  BooksRepository(this._apiService, this._hiveBox)
      : super(_hiveBox?.values.toList() ?? []);
  final BooksApiService _apiService;
  final Box<BookItem>? _hiveBox;

  String? _errorMessage;

  Future<BookResponse> fetchBooks(String query) async {
    return _apiService.fetchBooks(query, LaunchEnv.token);
  }

  List<BookItem> getFavoriteBooks() {
    return state;
  }

  Future<void> addFavoriteBook(BookItem book) async {
    await _hiveBox?.put(book.id, book);
    state = _hiveBox?.values.toList() ?? [];
  }

  Future<void> removeFavoriteBook(String bookId) async {
    await _hiveBox?.delete(bookId);
    state = _hiveBox?.values.toList() ?? [];
  }

  String? get errorMessage => _errorMessage;
}

final booksRepositoryProvider =
    StateNotifierProvider<BooksRepository, List<BookItem>>((ref) {
  final dio = ref.watch(dioProvider);
  final apiService = BooksApiService(dio);
  final hiveBox = ref.watch(hiveBoxFavoritesProvider).value;
  return BooksRepository(apiService, hiveBox);
});

@riverpod
Future<List<BookItem>> bookItemsList(BookItemsListRef ref, String query) async {
  final user = ref.watch(firebaseAuthProvider).currentUser;
  if (user == null) {
    throw Exception('User can\'t be null');
  }
  if (query.length > 250) {
    throw Exception('Arama terimi geçersiz. Lütfen daha kısa bir arama yapın.');
  }
  if (query.isEmpty) {
    return [];
  }
  final repository = ref.watch(booksRepositoryProvider.notifier);
  return (await repository.fetchBooks(query)).items ?? [];
}

@riverpod
List<BookItem> favoriteBooks(FavoriteBooksRef ref) {
  return ref.watch(booksRepositoryProvider);
}

```
  


