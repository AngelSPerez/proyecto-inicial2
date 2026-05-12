# RecipeRecive — Prompt para reconstruir desde cero

Eres un desarrollador Flutter experto. Debes construir una aplicación completamente funcional llamada **RecipeRecive** — un libro de recetas digital para cocina casera. Sigue CADA UNO de los siguientes pasos en orden, sin saltarte ninguno. Lee todo el prompt antes de empezar.

---

## 1. Configuración inicial del proyecto

1. Crea un nuevo proyecto Flutter llamado `recipe_recive` con `flutter create recipe_recive`
2. **SDK constraint**: `>=3.0.0 <4.0.0`
3. Edita `pubspec.yaml`:
   - name: `recipe_recive`
   - description: `RecipeRecive — digital recipe book for home cooking`
   - version: `1.0.0+1`
   - Dependencias REQUERIDAS:
     ```yaml
     dependencies:
       flutter:
         sdk: flutter
       cupertino_icons: ^1.0.6
       firebase_core: ^2.24.2
       firebase_auth: ^4.16.0
       cloud_firestore: ^4.14.0
       firebase_storage: ^11.6.0
       image_picker: ^1.0.7
       shared_preferences: ^2.2.2
       url_launcher: ^6.2.2
     ```
   - Dev dependencies:
     ```yaml
     dev_dependencies:
       flutter_test:
         sdk: flutter
       flutter_lints: ^3.0.1
     ```
   - Assets: descomenta `assets/images/`
   - Font: NO descomentes las fuentes .ttf (el código usa `fontFamily: 'Roboto'` pero caerá en sistema por defecto)
4. `flutter pub get`
5. Crea la estructura completa de directorios:
   ```
   lib/
     l10n/
     models/
     screens/
     services/
     utils/
     widgets/
   assets/
     images/
   ```
6. Coloca una imagen placeholder en `assets/images/image.png` (puede ser cualquier PNG, 512x512 aprox)

---

## 2. Modelos de datos

### `lib/models/user_model.dart`
Clase `AppUser` con:
- `String uid`
- `String name`
- `String email`
- `String role` ('user' o 'admin')
- `Map<String, dynamic> toMap()`
- `factory AppUser.fromMap(Map<String, dynamic> map)`

### `lib/models/recipe.dart`
Clase `Recipe` con:
- `String id` (Firestore doc ID)
- `int idx` (índice numérico)
- `String titleEn`, `String titleEs` (título bilingüe)
- `String description`
- `String difficulty` ('easy', 'medium', 'hard')
- `int prepTime`, `int cookTime`, `int totalTime` (minutos)
- `String image` (URL)
- `List<String> ingredientsEn`, `List<String> ingredientsEs`
- `List<String> stepsEn`, `List<String> stepsEs`
- `String category`
- `int servings`
- `double rating` (promedio)
- `bool isLiked`, `bool isSaved`
- Métodos de acceso con locale-aware: `get title => AppLocale.language.value == 'es' ? titleEs : titleEn` (igual para ingredients y steps)
- `Map<String, dynamic> toMap()`
- `factory Recipe.fromMap(Map<String, dynamic> map)`
- `factory Recipe.fromFirestore(DocumentSnapshot doc)`
- `static double ingredientOverlap(Recipe a, Recipe b)` — Jaccard similarity entre ingredientes (intersección / unión)

---

## 3. Servicios

### `lib/services/preferences_service.dart`
Wrapper de `shared_preferences`:
- `static late SharedPreferences _prefs`
- `static Future<void> init()` 
- `static Future<void> setThemeMode(String mode)` — guarda 'light', 'dark', o 'system'
- `static String getThemeMode()` — devuelve 'system' por defecto
- `static Future<void> setLanguage(String lang)` — guarda 'en', 'es', o 'pt_BR'
- `static String getLanguage()` — devuelve 'en' por defecto
- Singleton NO needed, usar métodos estáticos

### `lib/l10n/app_locale.dart`
```dart
import 'package:flutter/material.dart';

class AppLocale {
  static ValueNotifier<String> language = ValueNotifier<String>('en');
}
```

### `lib/l10n/strings.dart`
Clase `Str` con TODOS los textos de la UI en 3 idiomas. Organizar por secciones en métodos getter estáticos:

**Secciones REQUERIDAS (contenido aproximado):**

- **Glossary**: `appName` ("RecipeRecive"), `ok`, `cancel`, `save`, `delete`, `edit`, `search`, `loading`, `error`, `noData`
- **Splash**: `splashTitle` ("Recetario"), `splashSubtitle` ("Todas tus recetas favoritas en un solo lugar"), `splashButton` ("Continuar"). Siempre en español fijo.
- **Role choice**: `roleTitle` ("Elige tu rol"), `roleUser` ("Usuario"), `roleAdmin` ("Administrador"), `roleUserDesc`, `roleAdminDesc`
- **Welcome**: `welcomeTitle` ("¡Bienvenido!"), `welcomeSubtitle`, `getStarted` ("Comenzar"), `logIn` ("Iniciar sesión"), `alreadyAccount`
- **Register**: `registerTitle` ("Crear cuenta"), `nameLabel`, `emailLabel`, `passwordLabel`, `registerButton`, `nameRequired`, `emailInvalid`, `passwordLength`
- **Login**: `loginTitle`, `emailLabel`, `passwordLabel`, `loginButton`, `forgotPassword`, `noAccount`
- **Recipe list**: `searchHint`, `allRecipes`, `recommended`, `favorites`, `noRecipes`, `listView`, `gridView`, `logout`, `confirmLogout`, `pullToRefresh`
- **Recipe detail**: `ingredients`, `steps`, `prepTime`, `cookTime`, `totalTime`, `servings`, `difficulty`, `comments`, `addComment`, `yourRating`, `averageRating`, `share`, `shoppingList`, `similar`, `saveRecipe`, `likeRecipe`
- **User profile**: `profile`, `saved`, `liked`, `commented`, `rates`, `configuration`, `adminPanel`, `logout`, `myProfile`
- **Configuration**: `language`, `theme`, `about`, `changePassword`, `changeEmail`, `darkMode`, `lightMode`, `systemMode`, `english`, `spanish`, `portuguese`, `currentPassword`, `newPassword`, `confirmPassword`, `passwordChanged`, `emailChanged`
- **Admin panel**: `adminPanel`, `manageRecipes`, `manageUsers`, `manageComments`, `manageRatings`, `manageSaved`, `manageLiked`, `addRecipe`, `editRecipe`, `deleteRecipe`, `addUser`, `toggleAdmin`
- **Recommendations**: `recommendedForYou`, `similarRecipes`, `basedOnYourLikes`

Implementación: método `static String get(String key, {List<String>? args})` o getters directos. El idioma se determina con `AppLocale.language.value`. Tres Map internos: `_en`, `_es`, `_pt_BR`.

### `lib/services/auth_service.dart`
Singleton `AuthService`:
- `static final AuthService instance = AuthService._();`
- `final FirebaseAuth _auth = FirebaseAuth.instance;`
- `AppUser? _currentUser;` con getter `AppUser? get currentUser`
- `Future<void> init()` — escucha cambios de auth state, carga usuario desde Firestore cuando cambia
- `Future<void> login(String email, String password)` — `signInWithEmailAndPassword`, luego busca en colección `users` y setea `_currentUser`. Si el email es admin@reciperecive.com y role no es admin, lanza error
- `Future<void> register(String name, String email, String password)` — `createUserWithEmailAndPassword`, luego actualiza `displayName`, crea documento en `users/{uid}` con role='user'
- `Future<void> logout()` — `_auth.signOut()`, limpia `_currentUser`
- `Future<void> changePassword(String currentPassword, String newPassword)` — reautentica con credencial, luego `user.updatePassword()`
- `Future<void> changeEmail(String newEmail, String password)` — reautentica con `EmailAuthProvider.credential`, luego `user.updateEmail(newEmail)`, actualiza en Firestore
- `Future<bool> isLoggedIn()` — `_auth.currentUser != null`
- `Future<bool> tryRestoreSession()` — si `_auth.currentUser` existe, busca en `users/{uid}` para rehidratar `_currentUser`, retorna true si encontró el doc
- `Future<void> seedAdmin()` — verifica si ya se sembró (SharedPreferences flag `admin_seeded`). Si no, crea usuario `admin@reciperecive.com` con password `Admin123!` en Auth, crea documento en `users/{uid}` con role='admin', name='Admin', marca flag en SharedPreferences, luego hace logout (para que el usuario empiece desde splash)
- Método privado: `Future<void> _reauthenticate(String password)` — crea `EmailAuthProvider.credential` con email actual y password, llama `user.reauthenticateWithCredential()`

### `lib/services/firebase_service.dart`
Singleton `FirebaseService`:
- `final FirebaseFirestore _db = FirebaseFirestore.instance;`
- `final FirebaseStorage _storage = FirebaseStorage.instance;`

**Streams (todos retornan `Stream<List<T>>`):**
- `streamRecipes()` — snapshot de `recipes` collection, mapea con `Recipe.fromFirestore`
- `streamComments({String? recipeId})` — snapshot de `comments`, opcionalmente filtrado por `recipeId`
- `streamRatings({String? recipeId})` — snapshot de `ratings`
- `streamSaved({String? user})` — snapshot de `saved`
- `streamLiked({String? user})` — snapshot de `liked`
- `streamUsers()` — snapshot de `users`

**Métodos CRUD:**
- `Future<void> addRecipe(Recipe recipe)` — `_db.collection('recipes').add(recipe.toMap())`
- `Future<void> updateRecipe(String id, Map<String, dynamic> data)`
- `Future<void> deleteRecipe(String id)`
- `Future<void> addComment({required String user, required String recipeId, String? text, int? rating})` — doc en `comments` con timestamp
- `Future<void> updateComment(String docId, Map<String, dynamic> data)`
- `Future<void> deleteComment(String docId)`
- `Future<void> addRating({required String user, required String recipeId, required int rating})` — doc en `ratings`
- `Future<void> updateRating(String docId, Map<String, dynamic> data)`
- `Future<void> deleteRating(String docId)`
- `Future<DocumentSnapshot?> getLikedByUserAndRecipe(String user, String recipeId)` — query `liked` where user y recipeId, limit 1
- `Future<DocumentSnapshot?> getSavedByUserAndRecipe(String user, String recipeId)` — igual para `saved`
- `Future<List<DocumentSnapshot>> getRatingsForRecipe(String recipeId)` — query `ratings` where recipeId
- `Future<void> toggleLike(String user, String recipeId, bool isLiked)` — si isLiked elimina, si no agrega
- `Future<void> toggleSave(String user, String recipeId, bool isSaved)` — igual

**Seed data (`_seedData`):** Lista de 8 recetas bilingües (Apple Pie, Chicken, Cheesecake, Cookies, Wings, Pasta, Tacos, Pancakes). Cada una con: título EN/ES, ingredientes EN/ES, pasos EN/ES, tiempo de preparación/cocción/total, dificultad, categoría, porciones. Método `Future<void> seedRecipes()` que las inserta en Firestore si la colección está vacía.

**Subida de imágenes:**
- `Future<String> uploadImage(String localPath)` — sube a `recipe_images/{uuid}.jpg`, retorna URL de descarga

---

## 4. Widgets compartidos

### `lib/widgets/app_widgets.dart`
- **`AppLogo`**: StatelessWidget, parámetro opcional `double size` (default 140). Container circular con borde redondeado (radius 20), imagen `Image.asset('assets/images/image.png')` con fit BoxFit.cover
- **`RecipeImagePlaceholder`**: StatelessWidget, Container gris con icono de restaurante (Icons.restaurant), usado como fallback cuando no hay imageUrl

### `lib/widgets/locale_aware.dart`
Mixin `LocaleAwareState<T extends StatefulWidget>` on `State<T>`:
- Sobrescribe `initState()` para añadir listener a `AppLocale.language`
- El listener llama `setState(() {})` para reconstruir el widget cuando cambia el idioma
- `void dispose()` quita el listener

---

## 5. Tema y entrada principal

### `lib/main.dart`
Estructura completa:

```dart
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';
// ... demás imports

// ValueNotifier global para el modo de tema
class AppTheme {
  static ValueNotifier<ThemeMode> mode = ValueNotifier(ThemeMode.system);
}

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  
  // Edge-to-edge en Android
  SystemChrome.setEnabledSystemUIMode(SystemUiMode.edgeToEdge);
  SystemChrome.setSystemUIOverlayStyle(const SystemUiOverlayStyle(
    systemNavigationBarColor: Colors.transparent,
    systemNavigationBarDividerColor: Colors.transparent,
    systemNavigationBarContrastEnforced: false,
  ));
  
  // Inicializar Firebase
  await Firebase.initializeApp(options: DefaultFirebaseOptions.currentPlatform);
  
  // Inicializar SharedPreferences
  await PreferencesService.init();
  
  // Restaurar tema guardado
  final savedTheme = PreferencesService.getThemeMode();
  AppTheme.mode.value = _parseThemeMode(savedTheme);
  
  // Restaurar idioma guardado
  final savedLang = PreferencesService.getLanguage();
  AppLocale.language.value = savedLang;
  
  // Inicializar AuthService
  await AuthService.instance.init();
  
  // Seed admin en primer lanzamiento
  await AuthService.instance.seedAdmin();
  
  runApp(const RecipeReciveApp());
}

ThemeMode _parseThemeMode(String mode) {
  switch (mode) {
    case 'light': return ThemeMode.light;
    case 'dark': return ThemeMode.dark;
    default: return ThemeMode.system;
  }
}

class RecipeReciveApp extends StatefulWidget {
  const RecipeReciveApp({super.key});
  @override
  State<RecipeReciveApp> createState() => _RecipeReciveAppState();
}

class _RecipeReciveAppState extends State<RecipeReciveApp> {
  @override
  void initState() {
    super.initState();
    AppTheme.mode.addListener(() => setState(() {}));
    AppLocale.language.addListener(() => setState(() {}));
  }
  
  @override
  Widget build(BuildContext context) {
    return ValueListenableBuilder<ThemeMode>(
      valueListenable: AppTheme.mode,
      builder: (context, themeMode, _) {
        return MaterialApp(
          title: 'RecipeRecive',
          debugShowCheckedModeBanner: false,
          themeMode: themeMode,
          theme: ThemeData(
            useMaterial3: true,
            colorScheme: ColorScheme.fromSeed(
              seedColor: const Color(0xFF9C27B0),
              brightness: Brightness.light,
            ),
            fontFamily: 'Roboto',
            pageTransitionsTheme: const PageTransitionsTheme(
              builders: {
                TargetPlatform.android: FadeUpwardsPageTransitionsBuilder(),
                TargetPlatform.iOS: FadeUpwardsPageTransitionsBuilder(),
                TargetPlatform.windows: FadeUpwardsPageTransitionsBuilder(),
                TargetPlatform.macos: FadeUpwardsPageTransitionsBuilder(),
                TargetPlatform.linux: FadeUpwardsPageTransitionsBuilder(),
              },
            ),
          ),
          darkTheme: ThemeData(
            useMaterial3: true,
            colorScheme: ColorScheme.fromSeed(
              seedColor: const Color(0xFF9C27B0),
              brightness: Brightness.dark,
            ),
            fontFamily: 'Roboto',
            pageTransitionsTheme: const PageTransitionsTheme(
              builders: {
                TargetPlatform.android: FadeUpwardsPageTransitionsBuilder(),
                TargetPlatform.iOS: FadeUpwardsPageTransitionsBuilder(),
                TargetPlatform.windows: FadeUpwardsPageTransitionsBuilder(),
                TargetPlatform.macos: FadeUpwardsPageTransitionsBuilder(),
                TargetPlatform.linux: FadeUpwardsPageTransitionsBuilder(),
              },
            ),
          ),
          home: const SplashScreen(),
        );
      },
    );
  }
}
```

- El body de MaterialApp usa `ValueListenableBuilder<ThemeMode>` para reconstruir cuando cambia el tema
- Escucha cambios en `AppTheme.mode` y `AppLocale.language` en el State
- Home siempre es `SplashScreen`
- NOTA: La importación de `firebase_options.dart` se genera con FlutterFire CLI, pero debes crearla manualmente con la configuración de tu proyecto Firebase. Coloca un placeholder que será reemplazado.

---

## 6. Pantallas (17 screens en orden de flujo)

### 6.1 SplashScreen (`lib/screens/splash_screen.dart`)
- **Siempre en español** (hardcoded, no usa AppLocale)
- Muestra: `AppLogo` grande, título "Recetario", subtítulo "Todas tus recetas favoritas en un solo lugar", descripción, botón "Continuar"
- Al iniciar: `initState()` llama `_tryAutoLogin()` después de 800ms de delay
- `_tryAutoLogin()`: llama `AuthService.instance.tryRestoreSession()`. Si retorna true, navega a `RecipeListScreen` (o `AdminPanelScreen` si role='admin') con `pushAndRemoveUntil`. Si no, se queda esperando interacción del usuario
- Botón "Continuar" navega a `RoleChoiceScreen`

### 6.2 RoleChoiceScreen (`lib/screens/role_choice_screen.dart`)
- Extiende StatefulWidget con `LocaleAwareState`
- Muestra `AppLogo`, instrucciones, botón "Usuario" → navega a `WelcomeScreen()`, botón "Administrador" → navega a `LoginScreen(isAdmin: true)`

### 6.3 WelcomeScreen (`lib/screens/welcome_screen.dart`)
- Extiende StatefulWidget con `LocaleAwareState`, envuelto en `ValueListenableBuilder<String>` para idioma
- Muestra logo, título bienvenida, subtítulo, botón "Comenzar" → `RegisterScreen`, botón "Iniciar sesión" → `LoginScreen()`

### 6.4 RegisterScreen (`lib/screens/register_screen.dart`)
- Extiende StatefulWidget con `LocaleAwareState`
- Campos: nombre, email, contraseña con validación (email formato, password >= 6 caracteres)
- Botón registrarse → llama `AuthService.instance.register()` → navega a `RecipeListScreen` (pushAndRemoveUntil)
- Errors: muestra SnackBar con mensaje de error

### 6.5 LoginScreen (`lib/screens/login_screen.dart`)
- Extiende StatefulWidget con `LocaleAwareState`
- Parámetro `bool isAdmin = false`
- Campos: email, contraseña
- Si `isAdmin`: valida que el role del usuario sea 'admin' después del login, navega a `AdminPanelScreen`. Si no, lanza error "No autorizado"
- Si no es admin: navega a `RecipeListScreen`
- Botón "Forgot password" → `AuthService.instance.sendPasswordResetEmail()`
- Loading spinner durante login
- Errors: SnackBar

### 6.6 RecipeListScreen (`lib/screens/recipe_list_screen.dart`)
**La pantalla principal.** Extiende StatefulWidget con `LocaleAwareState`.

Estructura del build:
- **AppBar**: título dinámico según idioma, actions: botón perfil (navigates to `UserProfileScreen`)
- **Cuerpo**: Column con:
  1. Search bar (TextField con icono de búsqueda, filtro por título)
  2. Fila de botones toggle: favoritos (solo liked), vista lista/grid
  3. Sección "Recommended for you" (solo si `_isRecommendedExpanded` es true):
     - Título con flecha toggle (expande/colapsa)
     - Chips horizontales scrollables con nombres de recetas recomendadas
  4. StreamBuilder<List<Recipe>> de `FirebaseService.instance.streamRecipes()`
     - Muestra lista o grid según toggle
     - Cada card: `RecipeImagePlaceholder` o imagen, título, tiempo total
     - Like icon toggle por receta
     - Tap → `RecipeDetailScreen(recipe: recipe)`
- **FloatingActionButton**: logout con confirmación

**Lógica clave:**
- `_isRecommendedExpanded` bool, toggle con flecha
- `_showFavorites` bool, filtra solo recetas con isLiked
- `_isListView` bool, toggle entre ListView.builder y GridView.builder
- Cada recipe card llama `FirebaseService.instance.toggleLike()` al tocar corazón
- Pull-to-refresh con `RefreshIndicator`
- El stream de recetas escucha cambios en tiempo real desde Firestore

### 6.7 RecipeDetailScreen (`lib/screens/recipe_detail_screen.dart`)
**La pantalla más compleja.** Extiende StatefulWidget. NO usa LocaleAwareState (no necesita).

**AppBar fila personalizada:** back arrow + título + like icon, con `space-between`

**Cuerpo (SingleChildScrollView):**
1. Hero image (Container con imagen o placeholder, altura 200)
2. Save bookmark icon sobre la imagen
3. Título (más grande)
4. `_TimeRow` widget: prepTime, cookTime, totalTime con iconos de reloj
5. Dificultad, porciones
6. Descripción
7. Sección ingredientes con `_ActionButton` row: share (comparte texto), comment (abre bottom sheet), shopping list (abre modal con checklist)
8. Interactive star rating: 5 estrellas tappables que abren el CommentSheet existente
9. Average rating display con conteo (calculado desde `ratings` collection con `getRatingsForRecipe`)
10. Lista numerada de pasos
11. `_SimilarRecipesSection`: horizontal scroll con recetas similares (usando Recommendations util)

**`_CommentSheet` (showModalBottomSheet):**
- Rating stars interactivas
- Lista de comentarios existentes (stream desde Firestore)
- Campo de texto para nuevo comentario
- Botón enviar → llama `addComment()` y `addRating()`
- Rating wrapped en try/catch

**`_ShareOption` modal:** lista de apps para compartir (WhatsApp, Telegram, etc.) usando `url_launcher` para abrir URLs con texto de la receta

**Search bar al final:** con dropdown de sugerencias de recetas, al seleccionar navega a detail de esa receta

### 6.8 UserProfileScreen (`lib/screens/user_profile_screen.dart`)
- Extiende StatefulWidget con `LocaleAwareState`
- Avatar: CircleAvatar con icono person
- Nombre y email del usuario actual
- Lista de `_ProfileItem`:
  - Saved → `UserSavedScreen`
  - Liked → `UserLikedScreen`
  - Commented → `UserCommentsScreen`
  - Rates → `UserRatingsScreen`
  - Configuration → `ConfigurationScreen`
  - Admin Panel → `AdminPanelScreen` (solo visible si `user.role == 'admin'`)
- Botón logout con confirmación

### 6.9 UserItemsScreen (`lib/screens/user_items_screen.dart`)
Contiene 4 sub-screens en el mismo archivo, cada una es un StatefulWidget independiente:

1. **UserSavedScreen**: StreamBuilder de saved items del usuario actual. Muestra lista con `_RecipeCard` (imagen + overlay bookmark + título)
2. **UserLikedScreen**: igual pero con overlay heart
3. **UserCommentsScreen**: StreamBuilder de comments del usuario. Muestra receta + rating stars + texto del comentario
4. **UserRatingsScreen**: StreamBuilder de ratings del usuario. Matching por recipeId primero, fallback a recipeIdx. Muestra receta + rating stars

`_RecipeCard` widget compartido: Container con ClipRRect de imagen, icono overlay, título abajo. Tap → `RecipeDetailScreen`

### 6.10 ConfigurationScreen (`lib/screens/configuration_screen.dart`)
- Extiende StatefulWidget con `LocaleAwareState`
- Escucha `AppTheme.mode` para reflejar cambios
- Tiles de configuración:
  1. **Language**: muestra idioma actual, tap abre RadioDialog con EN/ES/PT-BR. Al seleccionar: guarda en PreferencesService, actualiza AppLocale.language
  2. **Theme**: muestra modo actual, tap abre RadioDialog con Light/Dark/System. Al seleccionar: guarda en PreferencesService, actualiza AppTheme.mode
  3. **About Us**: AlertDialog con info de la app
  4. **Change Password**: AlertDialog con 3 campos (current password, new password, confirm). Valida coincidencia, llama `changePassword()`
  5. **Change Email**: AlertDialog pidiendo current password primero (para reautenticar), luego nuevo email. Llama `changeEmail()`

### 6.11 AdminPanelScreen (`lib/screens/admin_panel_screen.dart`)
- Extiende StatefulWidget con `LocaleAwareState`
- Grid 2-columnas de `_AdminCard` tiles:
  1. Recipes (Icons.book) → AdminRecipesScreen
  2. Users (Icons.person) → AdminUsersScreen
  3. Comments (Icons.chat) → AdminCommentsScreen
  4. Ratings (Icons.star) → AdminRatingsScreen
  5. Saved (Icons.bookmark) → AdminSavedScreen
  6. Liked (Icons.favorite) → AdminLikedScreen
- AppBar con título y back button

### 6.12 AdminRecipesScreen (`lib/screens/admin_recipes_screen.dart`)
- Extiende StatefulWidget con `LocaleAwareState`
- Search bar
- StreamBuilder de recetas
- Cada item: imagen, título EN/ES, tiempo, rating. Long-press o more-vert → bottom sheet con Edit/Delete
- FAB + para nueva receta
- Recipe form dialog con:
  - Campos bilingües (EN y ES para título, ingredientes, pasos)
  - Image URL + galería (image_picker → upload a Storage → auto-completar URL)
  - Prep/cook/total time
  - Ingredientes (textarea separado por comas)
  - Pasos (textarea separado por líneas)
  - Dificultad (dropdown: easy/medium/hard)
  - Categoría, porciones
- Edit pre-poppula el formulario
- Delete con confirmación

### 6.13 AdminUsersScreen (`lib/screens/admin_users_screen.dart`)
- Search bar
- StreamBuilder de usuarios
- List: avatar (primera letra del nombre), nombre, badge admin si aplica, email
- More-vert: Edit (nombre/email/role), Toggle Admin, Delete
- Dismissible swipe-to-delete con confirmación
- Form dialog: nombre, email, role dropdown

### 6.14 AdminCommentsScreen (`lib/screens/admin_comments_screen.dart`)
- Search bar (filtra por usuario, receta, texto)
- StreamBuilder de comments
- Card: CircleAvatar usuario, nombre, texto, recipe badge, timestamp
- Edit/delete icons por comment
- Add button → form con dropdowns de usuario/receta + text field
- Dismissible swipe-to-delete

### 6.15 AdminRatingsScreen (`lib/screens/admin_ratings_screen.dart`)
- Search bar
- StreamBuilder de ratings
- Card: usuario, recipe title, star display, edit/delete
- Add button → form con dropdowns + 5-star interactive rating
- Matching recipes por índice

### 6.16 AdminSavedScreen (`lib/screens/admin_saved_screen.dart`)
- Search bar
- StreamBuilder de saved
- Card: bookmark icon, recipe title, user name, save time
- Add form con dropdowns
- More-vert → delete

### 6.17 AdminLikedScreen (`lib/screens/admin_liked_screen.dart`)
- Search bar
- StreamBuilder de liked
- Card: heart icon, recipe title, user name
- Add form con dropdowns
- More-vert → delete

---

## 7. Utilidades

### `lib/utils/recommendations.dart`
Clase `Recommendations`:
- `static List<Recipe> getSimilarRecipes(Recipe target, List<Recipe> allRecipes)` — ordena por `ingredientOverlap` descendente, excluye target, máximo 4 resultados
- `static List<Recipe> getRecommendedRecipes(List<Recipe> likedRecipes, List<Recipe> allRecipes)` — si hay likes, encuentra recetas con ingredientes similares usando overlaps, máximo 6. Si no hay likes, retorna las mejor calificadas

---

## 8. Firebase Options

### `lib/firebase_options.dart`
Archivo generado por FlutterFire CLI. Debe contener configuración por plataforma. Si no puedes ejecutar el CLI, crea un archivo placeholder con la estructura para tu proyecto Firebase específico:
- `firebase_options.dart` exporta `DefaultFirebaseOptions` con getter `currentPlatform` que retorna la configuración para android/ios/web/macos/windows. Linux lanza UnsupportedError.

---

## 9. Tests

### `test/widget_test.dart`
Smoke test mínimo: 
```dart
void main() {
  testWidgets('App renders without crashing', (WidgetTester tester) async {
    await tester.pumpWidget(const RecipeReciveApp());
    expect(find.byType(RecipeReciveApp), findsOneWidget);
  });
}
```
NOTA: Este test fallará sin Firebase mockeado. Debe ser considerado como prueba de concepto.

---

## 10. Consideraciones finales y detalles importantes

1. **Navegación**: Siempre usa `Navigator.push(MaterialPageRoute(...))` o `pushReplacement`. No uses packages de routing.
2. **Edge-to-edge**: Solo Android. Usar `SystemChrome.setEnabledSystemUIMode(SystemUiMode.edgeToEdge)` al inicio. Envolver cuerpos de pantalla en `SafeArea`.
3. **Like/Save**: El campo `user` en las colecciones usa `AuthService.instance.currentUser?.name`. Para usuarios sin displayName, usa el prefijo del email antes de @.
4. **recipeId**: Siempre usar el Firestore document ID para relaciones entre colecciones, no el idx numérico.
5. **Admin seed**: En `main()`, después de inicializar AuthService, llamar `AuthService.instance.seedAdmin()`. Esto solo corre en el primer launch gracias a SharedPreferences flag.
6. **i18n**: No uses Flutter intl. Usa el sistema descrito con `Str` class y `AppLocale.language` ValueNotifier.
7. **Tema**: Usar `ColorScheme.fromSeed(seedColor: Color(0xFF9C27B0))` para ambos temas.
8. **Transiciones**: Usar `FadeUpwardsPageTransitionsBuilder` en TODAS las plataformas.
9. **Font**: `fontFamily: 'Roboto'` está hardcodeado pero los assets .ttf están comentados en pubspec.yaml, así que caerá en fuente sistema.
10. **Firebase config**: Los archivos `google-services.json` y `GoogleService-Info.plist` NO se incluyen en el repo. Debes generarlos desde Firebase Console.
11. **minSdkVersion**: Android requiere minSdkVersion 23 en `android/app/build.gradle`.
12. **Imágenes**: La app usa `Image.asset('assets/images/image.png')` para el logo. Asegúrate de que el archivo exista.
13. **SnackBars**: Todos los errores de operaciones asíncronas deben mostrar un SnackBar con el mensaje de error.
14. **Pull-to-refresh**: Implementar en RecipeListScreen y admin screens donde sea apropiado.
15. **Responsive**: La app debe funcionar en dispositivos móviles en orientación retrato. No se requiere soporte tablet/landscape.
16. **Seguridad**: No exponer API keys ni secrets. Los archivos de configuración de Firebase están en .gitignore.
17. **Timestamp**: Todas las operaciones que guardan timestamp usar `FieldValue.serverTimestamp()`.
18. **Dismissible**: Admin screens usan swipe-to-delete con confirmación dialog.
19. **Confirmación**: Todas las operaciones destructivas (delete, logout) requieren confirmación del usuario.
20. **No tests unitarios**: Solo existe el smoke test. No es necesario crear tests adicionales.

---

## Resumen de archivos a crear (26 archivos Dart + assets):

```
lib/
├── main.dart
├── firebase_options.dart
├── l10n/
│   ├── app_locale.dart
│   └── strings.dart
├── models/
│   ├── recipe.dart
│   └── user_model.dart
├── screens/
│   ├── admin_comments_screen.dart
│   ├── admin_liked_screen.dart
│   ├── admin_panel_screen.dart
│   ├── admin_ratings_screen.dart
│   ├── admin_recipes_screen.dart
│   ├── admin_saved_screen.dart
│   ├── admin_users_screen.dart
│   ├── configuration_screen.dart
│   ├── login_screen.dart
│   ├── recipe_detail_screen.dart
│   ├── recipe_list_screen.dart
│   ├── register_screen.dart
│   ├── role_choice_screen.dart
│   ├── splash_screen.dart
│   ├── user_items_screen.dart
│   ├── user_profile_screen.dart
│   └── welcome_screen.dart
├── services/
│   ├── auth_service.dart
│   ├── firebase_service.dart
│   └── preferences_service.dart
├── utils/
│   └── recommendations.dart
└── widgets/
    ├── app_widgets.dart
    └── locale_aware.dart
assets/
└── images/
    └── image.png
test/
└── widget_test.dart
```
