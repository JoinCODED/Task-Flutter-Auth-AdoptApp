### Setup

1. In your `models` folder, create a new file called `user` and create a model with the following properties:

```dart
class User {
  int? id;
  String username;
  String? password;

  User({
    this.id,
    required this.username,
    this.password,
  });
}
```

2. Add `json_serializable` package into your project:

```dart
flutter pub add json_serializable
```

3. Import `json_serializable` package into your user model:

```dart
import 'package:json_annotation/json_annotation.dart';
```

4. Add the `part` file:

```dart
part 'user.g.dart';
```

5. Before your class definition, add this:

```dart
@JsonSerializable()
class User {
[...]
```

6. At the end of your class:

```dart
  factory User.fromJson(Map<String, dynamic> json) => _$UserFromJson(json);
  Map<String, dynamic> toJson() => _$UserToJson(this);
```

7. Install the `build_runner` package:

```dart
flutter pub add build_runner
```

8. Run this command:

```dart
flutter pub run build_runner build
```

9. In your `providers` folder create a file called `auth_provider.dart`.

```dart
class AuthProvider extends ChangeNotifier {}
```

10. Add 2 properties for now, a `String` `token` and initialize it with an empty string, and a `User` `user` property and mark it as `late`.

```dart
class AuthProvider extends ChangeNotifier {
  String token = "";
  late User user;
}
```

11.  In your `services` folder, create a new file `client.dart` and initialize a `dio` instance, pass it base url, and create a singleton for it.

```dart
import 'package:dio/dio.dart';

class Client {
  static final Dio dio = Dio(BaseOptions(baseUrl: 'https://coded-pets-api-auth.herokuapp.com'));
}
```

12. Create your `signup` function that returns a future String and takes `user` as an argument.

```dart
  Future<String> signup({required User user}) async {}
```

13. Send a post request to `/signup` and pass the `user` argument using `.toJson` constructor.

```dart
  Future<String> signup({required User user}) async {
    late String token;
    try {
      Response response =
          await Client.dio('/signup', data: user.toJson());
      token = response.data["token"];
    } on DioError catch (error) {
      print(error);
    }
    return token;
  }
```

14. Create the `signup` function in your `auth_provider` with a type of `void` and call our `signup` function from `auth_services.dart`.

```dart
  void signup({required User user}) async {
    await AuthServices().signup(user: user);
  }
```

15. Assign the token coming from the response to your `token` property in the provider, and print the token in the console.

```dart
  void signup({required User user}) async {
    token = await AuthServices().signup(user: user);
    print(token);
  }
```

### Signup

1. In your `pages` folder, create a signup page with two field, username and password.
2. Add this page into your routes in `main.dart`.

```dart
      GoRoute(
        path: '/signup',
        builder: (context, state) => SignupPage(),
      ),
```

3. In your `home_page.dart` create a `Drawer` widget with a header, signup and signin buttons.

```dart
return Scaffold(
      appBar: AppBar(
        title: const Text("Book Store"),
      ),
      drawer: Drawer()
[...]
```

```dart
ListView(
    padding: EdgeInsets.zero,
        children: [
            ListTile(
                title: const Text("Signin"),
                trailing: const Icon(Icons.login),
                onTap: () {},
            ),
            ListTile(
                title: const Text("Signup"),
                trailing: const Icon(Icons.how_to_reg),
                onTap: () {},
            )
            ],
    )
```

4. In your signup button, link it to the `signup` page.

```dart
    ListTile(
            title: const Text("Signup"),
            trailing: const Icon(Icons.how_to_reg),
            onTap: () {
                GoRouter.of(context).push('/signup');
            },
        )
```

5. In `main.dart` change `ChangeNotifierProvider` with `MultiProvider` that takes `providers` array as an argument and add your two providers.

```dart
void main() {
  runApp(
    MultiProvider(
      providers: [
        ChangeNotifierProvider<BooksProvider>(create: (_) => BooksProvider()),
        ChangeNotifierProvider<AuthProvider>(create: (_) => AuthProvider()),
      ],
      child: MyApp(),
    ),
  );
}
```

6. In your Signup page, call the auth provider signup function, and pass it the text fields values, then pop the user to the `home_page` again.

```dart
    ElevatedButton(
        onPressed: () {
            Provider.of<AuthProvider>(context, listen: false).signup(
            user: User(
            username: usernameController.text,
            password: passwordController.text));
        },
        child: const Text("Sign Up"),
    )
```

7. Check your code, you should see the token in the console.

### Signin

1. Create your `signin` function that returns a future `String` and takes `user` as an argument.

```dart
  Future<String> signin({required User user}) async {
    late String token;
    try {
      Response response =
          await Client.dio('/signin', data: user.toJson());
      token = response.data["token"];
    } on DioError catch (error) {
      print(error);
    }
    return token;
  }
```

2. Create the `signin` function in your `auth_provider` with a type of `void` and call our `signin` function from `auth_services.dart`.

```dart
  void signin({required User user}) async {
    token = await AuthServices().signin(user: user);
    print(token);
  }
```

3. Create a Signin page in your `pages` folder and add the page to our routes in `main.dart`.

```dart
      GoRoute(
        path: '/signin',
        builder: (context, state) => SigninPage(),
      ),
```

4. Link the page in the signin button in our `Drawer`.

```dart
ListTile(
    title: const Text("Signin"),
    trailing: const Icon(Icons.login),
        onTap: () {
            GoRouter.of(context).push('/signin');
            },
    ),
```

5. Call the `signin` function on the submit button in your signin page.

```dart
    ElevatedButton(
        onPressed: () {
            Provider.of<AuthProvider>(context, listen: false).signin(
                user: User(
                username: usernameController.text,
                password: passwordController.text));
        },
        child: const Text("Sign In"),
    )
```

6. Check your code, you should see the token in the console.

### Decoding The Token

1. In your `auth_provider.dart`, Create a `bool` getter that tells us if the user is authenticated or not based on the token availability.

```dart
  bool get isAuth {
    if (token.isNotEmpty) {
      return true;
    }
    return false;
  }
```

2. Install the `jwt_decode` package in your project:

```dart
import 'package:jwt_decode/jwt_decode.dart';
```

3. Use this package to check the expiry date of the token in your `isAuth` getter.

```dart
  bool get isAuth {
    if (token.isNotEmpty && Jwt.getExpiryDate(token)!.isAfter(DateTime.now())) {
      return true;
    }
    return false;
  }
```

4. If the token is not expired, decode the token and map the values to the `user` object using `fromJson` constructor.

```dart
  bool get isAuth {
    if (token.isNotEmpty && Jwt.getExpiryDate(token)!.isAfter(DateTime.now())) {
      user = User.fromJson(Jwt.parseJwt(token));
      return true;
    }
    return false;
  }
```

5. In your `Drawer` widget, use a `Consumer` widget and check the `isAuth` getter, if there is a user, show a signout button, else show the signin/up buttons we already have.

```dart
 drawer: Drawer(
        child: Consumer<AuthProvider>(
            builder: (context, authProvider, child) => authProvider.isAuth
                ? ListView(
                    padding: EdgeInsets.zero,
                    children: [
                      DrawerHeader(
                        child: Text("Welcome ${authProvider.user.username}"),
                        decoration: const BoxDecoration(
                          color: Colors.blue,
                        ),
                      ),
                      ListTile(
                        title: const Text("Logout"),
                        trailing: const Icon(Icons.logout),
                        onTap: () {},
                      ),
                    ],
                  )
                : ListView(
                    padding: EdgeInsets.zero,
                    children: [
                      const DrawerHeader(
                        child: Text("Sign in please"),
                        decoration: BoxDecoration(
                          color: Colors.blue,
                        ),
                      ),
                      ListTile(
                        title: const Text("Signin"),
                        trailing: const Icon(Icons.login),
                        onTap: () {
                          GoRouter.of(context).push('/signin');
                        },
                      ),
                      ListTile(
                        title: const Text("Signup"),
                        trailing: const Icon(Icons.how_to_reg),
                        onTap: () {
                          GoRouter.of(context).push('/signup');
                        },
                      )
                    ],
                  ),
                ),
      ),
```

6. Create a signout function in your provider that clears the token from the provider and link it to the signout button.

```dart
  void logout() {
    token = "";
    notifyListeners();
  }
```

```dart
ListTile(
        title: const Text("Logout"),
        trailing: const Icon(Icons.logout),
        onTap: () {
                Provider.of<AuthProvider>(context, listen: false).logout();
                },
    ),
```

### Persistent Login

1. Install `shared_preferences` package into your project.

```dart
flutter pub add shared_preferences
```

2. Import it into your auth provider:

```dart
import 'package:shared_preferences/shared_preferences.dart';
```

3. In your auth provider, create a `setToken` method, that takes a string token as an argument and stores that token in the `shared_preferences`.

```dart
  void setToken(String token) async {
    SharedPreferences prefs = await SharedPreferences.getInstance();
    prefs.setString('token', token);
  }
```

4. Call this method in both `signin` and `signup` function.

```dart
  void signup({required User user}) async {
    token = await AuthServices().signup(user: user);
    setToken(token);
    notifyListeners();
  }

  void signin({required User user}) async {
    token = await AuthServices().signin(user: user);
    setToken(token);
    notifyListeners();
  }
```

5. Create another function `getToken` that gets the token from the `shared_preferences` and assign it to the `token` property in the provider.

```dart
  void getToken() async {
    SharedPreferences prefs = await SharedPreferences.getInstance();
    token = prefs.getString('token') ?? "";
    notifyListeners();
  }
```

6. Call this function in the `isAuth` getter.

```dart
  bool get isAuth {
    getToken();
    if (token.isNotEmpty && Jwt.getExpiryDate(token)!.isAfter(DateTime.now())) {
      user = User.fromJson(Jwt.parseJwt(token));
      return true;
    }
    return false;
  }
```

7. In your `signout` function, remove the token from the `shared_preferences`, and call this function in `isAuth` if the token is expired.

```dart
  void logout() async {
    SharedPreferences prefs = await SharedPreferences.getInstance();
    prefs.remove('token');
    token = "";
    notifyListeners();
  }
```

```dart
  bool get isAuth {
    getToken();
    if (token.isNotEmpty && Jwt.getExpiryDate(token)!.isAfter(DateTime.now())) {
      user = User.fromJson(Jwt.parseJwt(token));
      return true;
    }
    logout();
    return false;
  }
```

### Adding Headers

1. In your `isAuth` getter, after you check for the token validity, add the token to the dio client headers.

```dart
bool get isAuth {
    getToken();
    if (token.isNotEmpty && Jwt.getExpiryDate(token)!.isAfter(DateTime.now())) {
      user = User.fromJson(Jwt.parseJwt(token));
      Client.dio.options.headers = {
        HttpHeaders.authorizationHeader: 'Bearer $token',
      };
      return true;
    }
    logout();
    return false;
  }
```
