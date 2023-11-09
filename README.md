# Pets Adoption App Auth 🦄

## Instructions

- If you need a starting point, fork and clone [this repository](https://github.com/JoinCODED/Task-Flutter-CRUD-AdoptApp/tree/main) to your `Development` folder.

## Steps

1. Change your backend baseUrl to this:

```
https://coded-pets-api-auth.eapi.joincoded.com
```

### Setup

1. In your `models` folder, create a new file called `user` and create a model with the following properties:

```dart
  int? id
  String username
  String? password
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
10. Add 2 properties for now, a `String` `token` and initialize it with an empty string, and a `User` `user` property and mark it as `late`.

11. In your `services` folder, create a new file `client.dart`.
12. Initialize a dio instance, pass it the base url and create a singleton for it.
13. Replace every `PetsServices()` in `services/pets.dart` with the singleton you just created.
14. In your `services` folder, create a new file `auth_services.dart`.
15. Create your `signup` function that returns a future String and takes `user` as an argument.
16. Send a post request to `/signup` and pass the `user` argument using `.toJson` constructor.
17. Return the token from the response object.
18. Create the `signup` function in your `auth_provider` with a type of `void` and call our `signup` function from `auth_services.dart`.
19. Assign the token coming from the response to your `token` property in the provider, and print the token in the console.

### Signup

1. In your `pages` folder, create a signup page with two field, username and password.
2. Add this page into your routes in `main.dart`.
3. In your `home_page.dart` create a `Drawer` widget with a header, signup and signin buttons.
4. In your signup button, link it to the `signup` page.
5. In `main.dart` change `ChangeNotifierProvider` with `MultiProvider` that takes `providers` array as an argument and add your two providers.
6. In your Signup page, call the auth provider signup function, and pass it the text fields values, then pop the user to the `home_page` again.
7. Check your code, you should see the token in the console.

### Signin

1. Create your `signin` function that returns a future `String` and takes `user` as an argument.
2. Create the `signin` function in your `auth_provider` with a type of `void` and call our `signin` function from `auth_services.dart`.
3. Create a Signin page in your `pages` folder and add the page to our routes in `main.dart`.
4. Link the page in the signin button in our `Drawer`.
5. Call the `signin` function on the submit button in your signin page.
6. Check your code, you should see the token in the console.

### Decoding The Token

1. In your `auth_provider.dart`, Create a `bool` getter that tells us if the user is authenticated or not based on the token availability.
2. Install the `jwt_decode` package in your project:

```dart
import 'package:jwt_decode/jwt_decode.dart';
```

3. Use this package to check the expiry date of the token in your `isAuth` getter.
4. If the token is not expired, decode the token and map the values to the `user` object using `fromJson` constructor.
5. In your `Drawer` widget, use a `Consumer` widget and check the `isAuth` getter, if there is a user, show a signout button, else show the signin/up buttons we already have.
6. Create a signout function in your provider that clears the token from the provider and link it to the signout button.
7. In the drawer header, show a welcome message with the username.

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
4. Call this method in both `signin` and `signup` function.
5. Create another function `getToken` that gets the token from the `shared_preferences` and assign it to the `token` property in the provider.
6. Call this function in the `isAuth` getter.
7. In your `signout` function, remove the token from the `shared_preferences`, and call this function in `isAuth` if the token is expired.

### Adding Headers

1. In your `isAuth` getter, after you check for the token validity, add the token to the dio client headers.
