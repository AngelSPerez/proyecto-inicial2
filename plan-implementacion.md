# Plan de implementación — RecipeRecive

Este plan detalla cómo construir RecipeRecive desde cero, organizado en fases progresivas: desde lo más básico (que la app compile y muestre algo) hasta lo más complejo (Firebase, recomendaciones, panel admin). Cada fase es funcional por sí misma.

---

## Fase 0 — Fundación del proyecto

**Objetivo:** Tener un proyecto Flutter que compile, con estructura de directorios y dependencias instaladas.

- [ ] `flutter create recipe_recive`
- [ ] Editar `pubspec.yaml`: nombre, descripción, versión, dependencias
- [ ] `flutter pub get`
- [ ] Crear estructura de directorios:
  ```
  lib/l10n/  lib/models/  lib/screens/  lib/services/  lib/utils/  lib/widgets/
  assets/images/
  test/
  ```
- [ ] Colocar `assets/images/image.png` (logo placeholder)
- [ ] Verificar que `flutter analyze` pase (puede fallar por imports faltantes, pero la estructura debe estar)

---

## Fase 1 — Modelos + Servicios base + main.dart mínimo

**Objetivo:** Tener los modelos de datos, servicios sin Firebase (mock), y un `main.dart` que muestre una pantalla vacía.

### Paso 1.1 — Modelos
- [ ] Crear `lib/models/user_model.dart` — clase `AppUser` con uid, name, email, role, toMap/fromMap
- [ ] Crear `lib/models/recipe.dart` — clase `Recipe` con todos los campos bilingües, locale-aware getters, toMap/fromMap/fromFirestore, `ingredientOverlap()` estático

### Paso 1.2 — i18n base
- [ ] Crear `lib/l10n/app_locale.dart` — `AppLocale` con `ValueNotifier<String> language`
- [ ] Crear `lib/l10n/strings.dart` — clase `Str` con mapas EN/ES/PT_BR (iniciar con strings mínimos: appName, ok, cancel, save, delete)

### Paso 1.3 — PreferencesService
- [ ] Crear `lib/services/preferences_service.dart` — wrapper SharedPreferences con get/set para theme mode y language

### Paso 1.4 — main.dart mínimo
- [ ] Crear `lib/main.dart`:
  - `AppTheme` con `ValueNotifier<ThemeMode>`
  - `main()`: WidgetsFlutterBinding, PreferencesService.init(), runApp
  - `RecipeReciveApp`: MaterialApp con tema Material 3, color semilla púrpura (`0xFF9C27B0`), home: Container vacío con texto "RecipeRecive"
- [ ] Verificar que compile y muestre la pantalla

---

## Fase 2 — Navegación básica + Splash + Role Choice + Welcome

**Objetivo:** Flujo de pantallas sin lógica real (solo UI y navegación).

### Paso 2.1 — Shared widgets
- [ ] Crear `lib/widgets/app_widgets.dart`: `AppLogo` (imagen redondeada), `RecipeImagePlaceholder` (container gris con icono)
- [ ] Crear `lib/widgets/locale_aware.dart`: mixin `LocaleAwareState` que escucha `AppLocale.language`

### Paso 2.2 — SplashScreen
- [ ] Crear `lib/screens/splash_screen.dart`: muestra AppLogo, título "Recetario", botón "Continuar" → navega a RoleChoiceScreen
- [ ] Hardcodeado en español. Botón funcional usando `Navigator.push`

### Paso 2.3 — RoleChoiceScreen
- [ ] Crear `lib/screens/role_choice_screen.dart`: dos botones (Usuario → WelcomeScreen, Administrador → LoginScreen con isAdmin: true)
- [ ] Usar `LocaleAwareState`, strings desde `Str`

### Paso 2.4 — WelcomeScreen
- [ ] Crear `lib/screens/welcome_screen.dart`: botones "Comenzar" → RegisterScreen, "Iniciar sesión" → LoginScreen
- [ ] Envolver en `ValueListenableBuilder` para idioma

---

## Fase 3 — Autenticación (AuthService mock + screens)

**Objetivo:** Login y registro funcionales con un AuthService mock (sin Firebase real aún).

### Paso 3.1 — AuthService mock
- [ ] Crear `lib/services/auth_service.dart`:
  - Singleton con `_currentUser` (in-memory, hardcodeado)
  - `login()`: si email es "admin@reciperecive.com" y pass "Admin123!", crea AppUser con role admin; si no, crea user normal
  - `register()`: crea AppUser
  - `logout()`: limpia _currentUser
  - `tryRestoreSession()`: retorna false siempre (no hay persistencia real)
  - `seedAdmin()`: no hace nada (placeholder)

### Paso 3.2 — LoginScreen
- [ ] Crear `lib/screens/login_screen.dart`: campos email/password, botón login
- [ ] Si `isAdmin`: valida role admin, navega a AdminPanelScreen
- [ ] Si no: navega a RecipeListScreen
- [ ] SnackBar en errores, loading indicator

### Paso 3.3 — RegisterScreen
- [ ] Crear `lib/screens/register_screen.dart`: campos nombre/email/password con validación
- [ ] Navega a RecipeListScreen en éxito

---

## Fase 4 — Recipe List + Recipe Detail (datos mock)

**Objetivo:** La pantalla principal de recetas y el detalle funcionando con datos mock (sin Firebase).

### Paso 4.1 — FirebaseService mock
- [ ] Crear `lib/services/firebase_service.dart`:
  - Singleton con lista interna de recetas (`placeholderRecipes` copiadas del modelo Recipe)
  - `streamRecipes()`: retorna `Stream.value(_recipes)` (simula stream)
  - `toggleLike()`: modifica flag in-memory

### Paso 4.2 — RecipeListScreen
- [ ] Crear `lib/screens/recipe_list_screen.dart`:
  - Search bar (TextField, filtra por título)
  - Toggle favoritos (checkbox o switch)
  - Toggle lista/grid (icon button)
  - StreamBuilder de recetas
  - Cada card: imagen (o placeholder), título, tiempo, like icon
  - Tap → RecipeDetailScreen
  - Pull-to-refresh (RefreshIndicator)
  - FAB logout con confirmación

### Paso 4.3 — RecipeDetailScreen
- [ ] Crear `lib/screens/recipe_detail_screen.dart`:
  - AppBar con back + título + like icon (space-between)
  - Hero image
  - Save bookmark
  - Tiempos (prep/cook/total)
  - Dificultad, porciones
  - Ingredientes
  - Action buttons row (share, comment, shopping list) — solo UI, sin funcionalidad
  - Stars rating (solo visual, sin interacción)
  - Pasos numerados
  - Similar recipes (horizontal scroll, placeholder)

---

## Fase 5 — i18n completo

**Objetivo:** Todos los strings de la UI en inglés, español y portugués.

- [ ] Completar `Str` en `strings.dart` con TODOS los textos:
  - Glossary, splash, role choice, welcome, register, login
  - Recipe list, recipe detail, user profile, configuration
  - Admin panel y sub-screens
  - Recommendations
- [ ] Verificar que cada pantalla use `Str` en lugar de texto hardcodeado (excepto Splash que es español fijo)

---

## Fase 6 — Tema y configuración

**Objetivo:** Pantalla de configuración funcional con cambio de idioma y tema.

- [ ] Crear `lib/screens/configuration_screen.dart`:
  - Language tile: RadioDialog EN/ES/PT-BR, persiste en PreferencesService, actualiza AppLocale.language
  - Theme tile: RadioDialog Light/Dark/System, persiste, actualiza AppTheme.mode
  - About Us: AlertDialog
  - Change Password: diálogo con 3 campos (solo UI por ahora, sin Firebase real)
  - Change Email: diálogo con password + email nuevo (solo UI)
- [ ] Conectar AppTheme.mode en main.dart con `ValueListenableBuilder` (ya debería estar)

---

## Fase 7 — Perfil de usuario + UserItemsScreen

**Objetivo:** Pantalla de perfil con acceso a items guardados/liked/comentarios/ratings.

- [ ] Crear `lib/screens/user_profile_screen.dart`:
  - Avatar, nombre, email
  - Lista de _ProfileItem: Saved, Liked, Commented, Rates, Configuration
  - Admin Panel (solo si role == 'admin')
  - Logout

- [ ] Crear `lib/screens/user_items_screen.dart`:
  - `UserSavedScreen`: lista de saved del usuario actual
  - `UserLikedScreen`: lista de liked
  - `UserCommentsScreen`: lista de comentarios
  - `UserRatingsScreen`: lista de ratings
  - Cada sub-screen con `_RecipeCard`: imagen + overlay icon + título, tap → detail

---

## Fase 8 — Firebase real (Auth + Firestore + Storage)

**Objetivo:** Reemplazar los mocks por Firebase real.

### Paso 8.1 — Firebase Auth
- [ ] Agregar `firebase_core`, `firebase_auth` (ya en pubspec.yaml)
- [ ] Crear `lib/firebase_options.dart` con la configuración del proyecto (o usar FlutterFire CLI)
- [ ] Actualizar `main()`: `await Firebase.initializeApp(options: DefaultFirebaseOptions.currentPlatform)`
- [ ] Actualizar `AuthService`:
  - `FirebaseAuth.instance` real
  - `login()`: `signInWithEmailAndPassword`, luego leer desde Firestore collection `users`
  - `register()`: `createUserWithEmailAndPassword`, crear doc en `users/{uid}`
  - `tryRestoreSession()`: leer `users/{uid}` desde Firestore
  - `seedAdmin()`: crear admin en Auth + Firestore + SharedPreferences flag

### Paso 8.2 — Firebase Firestore
- [ ] Agregar `cloud_firestore` (ya en pubspec.yaml)
- [ ] Actualizar `FirebaseService`:
  - Streams reales: `streamRecipes()`, `streamComments()`, etc.
  - CRUD real contra Firestore collections
  - `seedRecipes()` que inserta los 8 recipes iniciales
  - `uploadImage()` usando Firebase Storage

### Paso 8.3 — Firebase Storage
- [ ] Agregar `firebase_storage` + `image_picker` (ya en pubspec.yaml)
- [ ] Implementar `uploadImage()` en FirebaseService

---

## Fase 9 — Funcionalidades completas del detalle de receta

**Objetivo:** Todas las interacciones del detalle de receta funcionando con Firebase.

- [ ] Like toggle real: `FirebaseService.toggleLike()` con colección `liked`
- [ ] Save toggle real: `FirebaseService.toggleSave()` con colección `saved`
- [ ] CommentSheet funcional:
  - Stream de comments para la receta
  - Submit nuevo comment → `addComment()`
  - Rating stars → `addRating()`
- [ ] Average rating en vivo desde `ratings` collection
- [ ] Share modal usando `url_launcher`
- [ ] Shopping list checklist modal
- [ ] Search bar con dropdown de recetas

---

## Fase 10 — Recomendaciones

**Objetivo:** Algoritmo de recomendaciones funcionando.

- [ ] Crear `lib/utils/recommendations.dart`:
  - `getSimilarRecipes()`: Jaccard similarity por ingredientes
  - `getRecommendedRecipes()`: basado en likes del usuario, fallback a mejor calificadas
- [ ] Integrar en RecipeListScreen: sección "Recommended for you" con chips
- [ ] Integrar en RecipeDetailScreen: sección "Similar recipes" horizontal scroll

---

## Fase 11 — Panel de administración (7 screens)

**Objetivo:** CRUD completo para administradores.

### Paso 11.1 — AdminPanelScreen
- [ ] Crear `lib/screens/admin_panel_screen.dart`: grid 2-col con 6 cards que navegan a sub-screens

### Paso 11.2 — AdminRecipesScreen
- [ ] Crear `lib/screens/admin_recipes_screen.dart`:
  - CRUD completo de recetas con formulario bilingüe
  - Image picker subiendo a Storage
  - Search bar

### Paso 11.3 — AdminUsersScreen
- [ ] Crear `lib/screens/admin_users_screen.dart`:
  - Lista de usuarios con toggle admin
  - Edit/delete

### Paso 11.4 — AdminCommentsScreen
- [ ] Crear `lib/screens/admin_comments_screen.dart`:
  - CRUD de comentarios
  - Search, edit, delete

### Paso 11.5 — AdminRatingsScreen
- [ ] Crear `lib/screens/admin_ratings_screen.dart`:
  - CRUD de ratings
  - Interactive stars

### Paso 11.6 — AdminSavedScreen
- [ ] Crear `lib/screens/admin_saved_screen.dart`:
  - Vista de saved items
  - Delete

### Paso 11.7 — AdminLikedScreen
- [ ] Crear `lib/screens/admin_liked_screen.dart`:
  - Vista de liked items
  - Delete

---

## Fase 12 — Edge-to-edge + pulido final

**Objetivo:** Detalles finales de UX/UI.

- [ ] `SystemChrome.setEnabledSystemUIMode(SystemUiMode.edgeToEdge)` en main()
- [ ] `SystemChrome.setSystemUIOverlayStyle()` para barras transparentes
- [ ] `SafeArea` en cuerpos de pantallas
- [ ] Verificar `FadeUpwardsPageTransitionsBuilder` en todas las plataformas
- [ ] Confirmar que todas las pantallas usen los strings de `Str` (excepto Splash)
- [ ] Animar transiciones donde sea apropiado
- [ ] Probar que todos los flows de navegación funcionen

---

## Fase 13 — Smoke test + verificación final

**Objetivo:** Asegurar que la app compile y pase análisis.

- [ ] Escribir `test/widget_test.dart` (smoke test básico)
- [ ] `flutter analyze` — resolver todos los warnings/errors
- [ ] `flutter test` — verificar que pase
- [ ] Verificar `pubspec.yaml` no tenga dependencias innecesarias
- [ ] Revisar que todos los imports sean correctos

---

## Resumen de dependencias entre fases

```
Fase 0 ──► Fase 1 ──► Fase 2 ──► Fase 3 ──► Fase 4
                  │                              │
                  │                              ▼
                  └──► Fase 5 ──► Fase 6 ──► Fase 7
                                                │
                                                ▼
                                          Fase 8 ──► Fase 9 ──► Fase 10
                                                │
                                                ▼
                                          Fase 11 ──► Fase 12 ──► Fase 13
```

Cada fase depende de todas las anteriores. No se debe saltar una fase sin completar la anterior.
