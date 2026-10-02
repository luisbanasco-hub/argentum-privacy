# Verificación — qué respalda cada afirmación del documento

Cada línea de `index.html` y `account-deletion.html` que afirma algo sobre los
datos tiene que poder apoyarse en una línea de código. Este archivo es esa
lista. Se escribió el **17 de septiembre de 2026**, leyendo la rama `main` de
los repos de las apps y del backend, y sirve para que la próxima revisión
compare en vez de empezar de cero.

**Cómo leerlo.** Si una fila deja de ser cierta, el documento miente desde ese
momento. Si se agrega una afirmación al documento, se agrega una fila acá.

**Rama `main` de cada repo al momento de verificar:**

| repo | commit | fecha |
|---|---|---|
| `argentum-ios` | `eb268e7` | 2026-09-16 |
| `argentum-android` | `4c2366b` | 2026-09-16 |
| `argentum-news-feed` | `4a01298` | 2026-08-19 |
| `argentum-agent-service` | `eda450d` | 2026-08-29 |

Los repos se leyeron **sin tocar su working tree** (`git show main:<archivo>`,
`git grep <patrón> main`), porque había otras sesiones trabajando en ellos con
ramas propias.

---

## 1. Qué sale del dispositivo

| Afirmación del documento | Dónde se verifica |
|---|---|
| Los símbolos de la cartera van a Yahoo Finance desde el teléfono | iOS `PortfolioViewModel.swift:156` → `LiveQuotes.swift:93` → `MarketDataRepository.swift:310` · Android `PortfolioViewModel.kt:237` → `LiveQuotes.kt:71` → `MarketDataRepository.kt:359` |
| Los símbolos de la watchlist también, de forma continua | Android `TickerViewModel.kt:47-54` (universo `watchlist ∪ intake`) → `QuotePoller.kt:68,102` · iOS `AssetDetailView.swift:68,89` |
| Los símbolos de cripto van a CoinGecko | iOS `MarketDataRepository.swift:141,328,354` · Android `MarketDataRepository.kt:208,386,411` |
| Lo que escribís al buscar un símbolo va a Yahoo y CoinGecko | iOS `MarketDataRepository.swift:471,491`, `IntakeView.swift:117` · Android `MarketDataRepository.kt:157,178` |
| Yahoo deja una cookie de sesión que la app guarda y reenvía | iOS `MarketDataRepository.swift:426-441` · Android `MarketDataRepository.kt:495-503` |
| A esos dos **no** les llega la cantidad, sólo el símbolo | Las firmas que se usan reciben tickers: `yahooBatchQuotes(symbols:)` (iOS `:298`), `simple/price` (iOS `:328`, Android `:386`) |
| Filtros de mercado e idioma viajan en cada pedido al feed, con o sin cuenta | iOS `APIClient.swift:76-82` · Android `ApiService.kt:42-49` |
| Los filtros se guardan en el servidor cuando hay sesión | iOS `UserService.swift:15-21` · Android `ApiService.kt:102-108`, `PersonalizationRepository.kt:42,160`, `Personalization.kt:47` |
| Sin sesión, los filtros se quedan en el teléfono | El POST está dentro de `auth.accessToken?.let { … }` (Android `PersonalizationRepository.kt:41`) y de una llamada que exige token (iOS `UserService.swift:20`) |
| Al asistente van noticia, pregunta, filtros, idioma, holdings con **cantidad**, watchlist y perfil | iOS `AgentModels.swift:5-35` (`HoldingDto.quantity`, `ExplainRequest`) |
| El identificador de instalación es un UUID local que se manda al servicio de IA | iOS `SessionManager.swift:86-91`, `AgentModels.swift:88`, `APIClient.swift:129` · Android `PreferencesRepository.kt:244-250`, `SummaryViewModel.kt:107` |
| Push: APNs en iPhone, FCM en Android | iOS `PushNotificationManager.swift:23-26`, `UserService.swift:43-45` (`platform: "ios"`) · Android `build.gradle.kts:185` |
| A Sentry va el id de cuenta, nunca dato financiero, correo ni nombre | iOS `Monitoreo.swift:12-17,44-47,53-75` · Android `Monitoreo.kt:23-29` (espejo 1:1) |
| RevenueCat recibe el identificador de la cuenta | iOS `SubscriptionManager.swift:268-269` (`Purchases.shared.logIn(uid)`) |
| El nombre llega al servidor en el login social | feed `src/auth.py:64-72` (`OAuthRequest.full_name`), `src/oauth_verify.py:197` (Google, del token), iOS `LoginView.swift:169-185` + `AuthService.swift:58-66` (Apple, del cliente) |
| Ni ubicación, ni contactos, ni identificador publicitario | Android `AndroidManifest.xml:4-6` (sólo INTERNET, ACCESS_NETWORK_STATE, POST_NOTIFICATIONS) · iOS `Config/Info.plist` sin ninguna *usage description* |
| El video embebido (YouTube, Vimeo, Dailymotion) se carga solo al abrir el artículo, sin tocar play | Android `EmbebidoDeVideo.kt:41-48` (lista `HOSTS`), `NewsDetailScreen.kt:492,520` (`loadUrl` en la composición) · iOS `NewsDetailView.swift:287-290` (`WebPlayerView` si hay `video_url`), `:351-353` (`load` en `updateUIView`) — main al 2026-10-02: iOS `c7c4d35`, Android `b244dae` |
| En iPhone puede ser otro servicio de video | iOS no valida el host: carga cualquier `URL(string: videoUrl)` (`NewsDetailView.swift:287`). Android sí (`EmbebidoDeVideo.kt:50-67`) |
| Ninguna app fuerza `youtube-nocookie.com` | Android lo acepta pero carga la URL tal cual (`EmbebidoDeVideo.kt:43`, `NewsDetailScreen.kt:520`); iOS ni lo menciona. Por eso el documento no promete el modo de privacidad reforzada |

## 2. Qué guarda el servidor

| Tabla / lugar | Qué guarda | Dónde se verifica |
|---|---|---|
| `portfolio_holdings`, `portfolio_holding_events` | símbolo, tipo, cantidad y su historial | iOS `PortfolioRepository.swift:59,124` |
| `user_watchlist` | símbolos seguidos | feed `src/user.py:138-196` |
| `user_preferences` | filtros de mercado | feed `src/user.py:105-137` |
| `user_notification_preferences` | resumen diario sí/no y horario | Android `Personalization.kt:62-64` · feed `src/user.py:230-260` |
| `user_push_tokens` | token + plataforma | feed `src/user.py:197-229` |
| `user_subscriptions` | estado de la suscripción | feed `src/subscription.py:4` |
| `usage_logs` | `user_id`, `kind='summary'`, día, contador | `migrations/consume_summary_credit.sql:19-23` |
| `summary_grants` | `user_id`, `news_id`, día — de qué artículos leíste el resumen | `migrations/consume_summary_credit_per_article.sql:20-21,50-56` |
| `user_auth_identities` | proveedor, subject del proveedor, `user_id`, correo | `migrations/user_auth_identities.sql:16-28` |
| `user_metadata` de GoTrue | `full_name`, proveedor, correo del proveedor | feed `src/auth.py:330-339,341-352` |
| `signal_events` | tipo de evento, `news_id`, `user_id`, `device_id`, idioma, símbolos de cartera (**sin cantidad**), y en `metadata` mercados, watchlist, perfil, horizonte y **el título de la noticia** | agent `migrations/001_signal_events.sql:22-53` · `src/api/routes.py:141-157,187-192,281-284` |
| caché de resúmenes | el resumen generado por artículo e idioma, sin id de cuenta | agent `src/api/routes.py:280` |

Lo que `signal_events` **no** guarda, y por eso el documento no lo dice:
el texto de la pregunta (sólo `has_custom_question`, `routes.py:154`), el
mensaje del chat (`:189-191` guarda sólo la profundidad del hilo) y la
respuesta del asistente.

## 3. Retención

| Afirmación | Dónde se verifica |
|---|---|
| La conversación vive en memoria del proceso y se cae a la hora de inactividad | agent `src/sessions.py:39-64` (dict en memoria, `_expired`, `sweep`) · `src/config.py:55` (`SESSION_TTL=3600`) |
| Los registros de interacción no tienen vencimiento | agent `migrations/001_signal_events.sql:3` ("append-only") y `docs/PROPRIETARY_SIGNAL_ROADMAP.md:112-115`, que deja la ventana como pregunta abierta desde julio |
| La captura está encendida por defecto | agent `src/config.py:73-76` (`SIGNAL_CAPTURE` por defecto `true`) |
| Hoy no se entrena ningún modelo con eso, pero para eso se junta | agent `src/signals.py:1` ("data accumulation only — no model") y `migrations/001_signal_events.sql:3-4` |
| A Anthropic no le llega la cantidad | agent `src/prompts.py:_holding_phrase` ("deliberately omit quantity") y `:103-114` |

## 4. Borrado de cuenta

| Afirmación | Dónde se verifica |
|---|---|
| Hay borrado in-app en las dos apps, y es inmediato | iOS `SettingsView.swift:120-143,174-197` → `AccountStore.swift:252-258` → `AuthService.swift:78-94` · Android `SettingsScreen.kt:226` → `AuthRepository.kt:208-220` → `ApiService.kt:96-97` · feed `src/auth.py:568-625` (borra en línea) |
| Se borran esas ocho tablas | feed `src/auth.py:556-565` |
| Y después el usuario de autenticación | feed `src/auth.py:611-613` |
| Eso arrastra el vínculo social y los créditos por artículo | `migrations/user_auth_identities.sql:19` y `consume_summary_credit_per_article.sql:21`, las dos con `on delete cascade` |
| Los registros de interacción **no** se borran | no están en `_USER_TABLES` (`src/auth.py:556-565`) y su `user_id` es `text` sin clave foránea (`001_signal_events.sql:45`), así que tampoco caen por cascada |
| Borrar la cuenta no cancela la suscripción | no hay ninguna llamada a Apple ni a Google en `delete_account` (`src/auth.py:568-625`) |

## 5. Seguridad

| Afirmación | Dónde se verifica |
|---|---|
| Las rutas de cuenta exigen token | feed `src/user.py:81-85` (`_bearer` tira 401 sin cabecera) |
| La cartera va a Supabase con la sesión del usuario y RLS por `auth.uid()` | iOS `SupabaseClient.swift:31-38`, `PortfolioRepository.swift:19,59` |
| El servicio de IA acepta pedidos sin token, a propósito | agent `src/auth.py:1-22` ("a missing Authorization header is ALLOWED (anonymous tier), never rejected") |
| Los dos backends corren en Render | `render.yaml` en `argentum-news-feed` y en `argentum-agent-service` |

---

## Lo que NO se pudo verificar, y por eso no se afirma

1. **`support@argentumhq.com` como buzón de privacidad.** Existe en el código
   (`argentum-android/app/src/main/res/values/strings.xml:166`) pero sólo se le
   muestra al usuario en el camino de "no se puede cobrar en este dispositivo"
   del paywall (`PaywallScreen.kt:302,316`). Es la única dirección que las apps
   muestran; `privacy@argentum.app`, que el documento publicaba antes, no
   aparece en ninguno de los cuatro repos. Que el buzón reciba y que alguien lo
   lea no se puede comprobar desde el código.
2. **`push_logs`.** El backend escribe ahí (`src/push.py:132`) pero no hay DDL
   en ningún repo, así que no se sabe si tiene cascada al borrar el usuario. El
   documento no dice que se borre ni que no se borre. Guarda `user_id`, `kind`
   y el día.
3. **Si el push de iOS se entrega.** El token de APNs se registra y se guarda,
   pero el envío filtra por plataforma: `src/push.py:146` sólo manda por FCM a
   `platform == "android"`, y el comentario de `:13-14` dice que APNs va "out of
   band … sent only when configured". El documento describe la recolección del
   token, que sí ocurre, y no promete una entrega que no pude comprobar.
4. **Lo jurídico.** Si el texto alcanza para GDPR, CCPA o la Ley 25.326, quién
   es el responsable del tratamiento, bajo qué jurisdicción, y cuál es la base
   legal de los registros de interacción. Nada de eso se decide leyendo código.
5. **Traducciones.** Las apps shippean EN/ES/PT; el documento está sólo en
   inglés. No es una afirmación falsa, así que no se tocó.
