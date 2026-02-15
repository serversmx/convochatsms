# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Planned

- Laravel Notifications channel
- Template system
- Webhook handling
- Analytics and metrics
- Multi-tenant support

---

## [4.0.1] - 2026-02-15

### Fixed

- Fixed `BaseConvoChatService::makeRequest()` sending GET request data as JSON body instead of query parameters, causing "Invalid Parameters!" errors on endpoints like `/validate/whatsapp`, `/get/wa.accounts`, and all other GET endpoints

---

## [4.0.0] - 2026-01-31

### Security

- Fixed API secret override vulnerability in all 5 services — user-supplied params can no longer overwrite the `secret` key
- Removed infinite re-dispatch loop in `SendBulkSmsJob` that could spawn unlimited jobs on failure
- Resolved 3 CVEs in dev dependencies via `composer update --dev`

### Changed

- **BREAKING:** Removed `ConvoChatCache` and `RateLimiter` classes (unused dead code)
- **BREAKING:** All services now extend `BaseConvoChatService` with constructor signature `(?Client $client = null, ?array $config = null)` — existing Facade/DI usage is unaffected
- Replaced anonymous class in ServiceProvider with typed `ConvoChatManager`
- Replaced `empty()` with strict `=== ''` in `validateRequiredParams` to accept `0` and `"0"` as valid values
- Fixed `getGatewayRates()` call in Artisan command (now `getRates()`)
- Laravel version badge updated to reflect supported versions (10, 11, 12)

### Added

- `BaseConvoChatService` abstract class eliminating ~370 lines of duplication
- `ConvoChatManager` typed class for Facade resolution
- Unit tests for OTP, USSD, and Contacts services (+23 tests, total 64)
- Regression test for secret override vulnerability
- `.env.example` with all `CONVOCHAT_*` variables
- Code coverage CI job (Xdebug, clover + text)
- Dependabot for composer and GitHub Actions (weekly)
- Composer scripts: `test`, `analyse`, `format`, `format:check`, `quality`
- `docs/CONFIGURATION.md`, `docs/TESTING.md`, `docs/UPGRADE.md`
- Consumer smoke test (`ConsumerSmokeTest.php`)

### Fixed

- PHPStan suppressions reduced from 9 to 1 (`missingType.iterableValue`)
- Console command included in PHPStan analysis (8 errors fixed)
- Added return types to all Console command methods
- Added return type to Facade `getFacadeAccessor()`
- PHPStan CI step now uses `--memory-limit=512M`
- `actions/cache` upgraded from v3 to v4

---

## [3.3.0] - 2024-12-19

### Added

- **WhatsApp Account Management**
  - `linkWhatsAppAccount()` - Connect new WhatsApp accounts
  - `relinkWhatsAppAccount()` - Reconnect existing accounts
  - `deleteWhatsAppAccount()` - Remove WhatsApp accounts
  - `getWhatsAppSubscription()` - Get subscription details

- **Data Management Operations**
  - `deleteSmsReceived()` - Delete received SMS messages
  - `deleteSmsSent()` - Delete sent SMS messages
  - `deleteSmsCampaign()` - Delete SMS campaigns
  - `deleteWhatsAppReceived()` - Delete received WhatsApp messages
  - `deleteWhatsAppSent()` - Delete sent WhatsApp messages
  - `deleteWhatsAppCampaign()` - Delete WhatsApp campaigns
  - `deleteNotification()` - Delete Android notifications

- **USSD Service (NEW!)**
  - Complete `ConvoChatUssdService` class
  - `sendUssd()` - Execute USSD commands
  - `getUssdRequests()` - Get USSD request history
  - `deleteUssdRequest()` - Delete USSD requests

- **Financial Features**
  - `getEarnings()` - Get partner earnings information

- **Enhanced Documentation**
  - Complete API reference in ENDPOINTS.md
  - Usage examples for all new endpoints
  - Multi-language examples (cURL, Python, Node.js, PHP)
  - Updated README.md with USSD service documentation

### Changed

- Updated ServiceProvider to include USSD service
- Enhanced Facade with all new method signatures
- Updated test suite with new constants and methods
- Improved code structure and organization

### Fixed

- Fixed duplicate method declarations in SMS service
- Resolved linting issues (unused parameters, duplicate literals)
- Corrected code style violations

### Technical Details

- **New Service**: `ConvoChatUssdService` for USSD operations
- **New Constants**: Added constants for all new endpoints
- **New Methods**: 15+ new methods across services
- **Test Coverage**: Updated tests for all new functionality
- **Documentation**: Comprehensive updates to all documentation files

### Impact

This release provides **100% coverage** of all available ConvoChat API endpoints, transforming the SDK from a basic SMS/WhatsApp tool into a complete ConvoChat API integration solution.

---

## [3.2.1] - 2025-01-15

### 🚀 Added

- **WhatsApp Subscription Method**:
  - `getWhatsAppSubscription()` method in ConvoChatWhatsAppService
  - `SUBSCRIPTION_ENDPOINT` constant for WhatsApp subscription endpoint
  - Test coverage for new subscription method

### 🔧 Fixed

- **Documentation Corrections**:
  - Corrected base URL reference in ENDPOINTS.md documentation
  - Restored complete README.md header with all badges and package information

## [3.2.0] - 2025-01-15

### 📚 Added

- **Complete Documentation Package**:
  - `README.md`: Comprehensive SDK guide with installation, configuration, and usage
  - `ENDPOINTS.md`: Detailed endpoint reference with examples and parameters
  - `EXAMPLES.md`: Real-world service implementations and patterns
- **Developer Experience Improvements**:
  - Step-by-step installation guide
  - Complete API reference for all services
  - Detailed parameter descriptions
  - Real-world service implementations
  - Error handling patterns
  - Testing examples with mocks
  - Command line tools examples
  - Analytics and reporting examples

### 📊 Documentation Coverage

- **4 services documented**: SMS, WhatsApp, Contacts, OTP
- **30+ endpoints** with detailed examples
- **15+ service implementations** ready for production
- **Complete error handling patterns**
- **Testing strategies** with mocks
- **Real-world use cases** covered
- **Production deployment guides**

### 🎯 Target Audience

- Developers installing the SDK
- Teams implementing ConvoChat integration
- System administrators configuring the service
- QA teams writing tests

---

## [3.1.1] - 2025-01-15

### 🔧 Fixed

- **Code Style Violations**: Fixed PHP CS Fixer violations
  - `ordered_imports`: Reordered use statements alphabetically
  - `single_blank_line_at_eof`: Ensured single blank line at end of files
- **GitHub Actions**: All CI/CD checks now pass
- **Code Quality**: 100% compliance with coding standards

### 📋 Files Fixed

- `src/ConvoChatServiceProvider.php`
- `src/Facades/ConvoChat.php`
- `src/Services/ConvoChatContactsService.php`

### 🛡️ Quality Assurance

- **All tests pass**: 40/40 tests passing
- **Code style compliance**: Achieved
- **CI/CD pipeline**: Now green

---

## [3.1.0] - 2025-01-15

### 🔧 Fixed

- **Endpoint Corrections**: Fixed all endpoints to match real ConvoChat API
  - Updated ConvoChatSmsService with real API endpoints
  - Updated ConvoChatWhatsAppService with real API endpoints
  - Updated ConvoChatContactsService with real API endpoints
  - Fixed all endpoint constants to match real API patterns

### 🗑️ Removed

- **Non-existent Services**: Removed services that don't exist in the real API
  - ConvoChatAuthService
  - ConvoChatListsService
  - ConvoChatCampaignsService
  - ConvoChatWebhooksService
  - ConvoChatSettingsService
  - ConvoChatReportsService

### ✨ Added

- **ConvoChatOtpService**: New service for OTP functionality that exists in API
- **Real Endpoints Implementation**:
  - SMS: `/send/sms`, `/send/sms.bulk`, `/get/sms.*`, `/delete/sms.*`, `/remote/start.sms`, `/remote/stop.sms`
  - WhatsApp: `/send/whatsapp`, `/send/whatsapp.bulk`, `/get/wa.*`, `/validate/whatsapp`, `/remote/start.chats`, `/remote/stop.chats`
  - Contacts: `/get/contacts`, `/create/contact`, `/delete/contact`, `/get/groups`, `/create/group`, `/delete/group`
  - OTP: `/send/otp`, `/get/otp`
  - Account: `/get/credits`, `/get/subscription`, `/get/rates`, `/get/devices`

### 🔄 Changed

- **ServiceProvider**: Updated to register only existing services
- **Facade**: Updated to expose only existing services and methods
- **Tests**: Updated to reflect corrected endpoints
- **Method Signatures**: Updated to match real API parameters

### 🎯 Breaking Changes

- **Removed non-existent services and methods**
- **Updated method signatures to match real API parameters**
- **Changed endpoint patterns to match actual ConvoChat API**

### 📊 Statistics

- **4 services implemented**: SMS, WhatsApp, Contacts, OTP
- **30+ real endpoints** available
- **All tests pass**: 40/40
- **100% API compatibility** achieved

---

## [3.0.0] - 2025-01-15

### ✨ Added

- **Complete API Expansion**:
  - ConvoChatAuthService for authentication endpoints
  - ConvoChatContactsService for contact management (CRUD)
  - ConvoChatListsService for list/group management (CRUD)
  - ConvoChatCampaignsService for campaign management (CRUD)
  - ConvoChatWebhooksService for webhook management
  - ConvoChatSettingsService for settings and balance
  - ConvoChatReportsService for SMS, WhatsApp and campaign reports
- **Enhanced Services**:
  - ConvoChatSmsService: Added bulk SMS, history, details and delete
  - ConvoChatWhatsAppService: Added media, history, devices and connect/disconnect
- **New Endpoints**:
  - SMS: Bulk SMS, history, details, delete operations
  - WhatsApp: Media, history, devices, connect/disconnect
  - Authentication: Login, register, logout, user info
  - Contacts: Full CRUD operations
  - Lists/Groups: Full CRUD operations
  - Campaigns: Full CRUD operations
  - Webhooks: Create, list, delete
  - Settings: Get/update settings, balance
  - Reports: SMS, WhatsApp, Campaigns

### 🔧 Enhanced Features

- **HTTP Methods Support**: GET, POST, PUT, DELETE
- **Complete CRUD Operations**: For all entities
- **Robust Error Handling**: Enhanced error management
- **Integrated Logging**: Comprehensive logging system
- **Updated Facade**: All new services and methods
- **ServiceProvider**: Correctly configured for all services

### 🐛 Fixed

- **WhatsApp Endpoints**: Fixed to use dots instead of underscores
- **Endpoint Constants**: Corrected in tests
- **Service Registration**: All services properly registered

### 📊 Statistics

- **9 services implemented**
- **50+ endpoints available**
- **Full CRUD operations** for all entities
- **HTTP methods support** (GET, POST, PUT, DELETE)
- **Robust error handling** and logging
- **Facade updated** with all services
- **ServiceProvider configured** correctly
- **All tests pass** (41/41)

### 🎯 Breaking Changes

- **None**: Fully backward compatible

### 📚 Documentation

- **Updated facade** with all new services
- **Complete method documentation**
- **Usage examples** for all services

---

## [2.0.0] - 2025-10-09

### ✨ Added

- **Laravel 12 support**: Agregado soporte completo para Laravel 12.x
- **Laravel 11 support**: Agregado soporte completo para Laravel 11.x
- **PHP 8.4 support**: Compatibilidad completa con PHP 8.4
- **Claude Code integration**:
  - Agentes especializados: `laravel-package-expert`, `php-testing-expert`, `api-integration-specialist`
  - Comandos slash personalizados: `/test`, `/fix`, `/release`, `/docs`, `/new-feature`
  - CLAUDE.md: Guía completa para asistentes IA
- **VSCode configuration**:
  - Configuración compartida del workspace
  - Extensiones recomendadas para desarrollo PHP/Laravel
  - Tasks automatizadas para tests y análisis
  - Snippets personalizados del proyecto
- **Compatibility matrix**: Tabla de compatibilidad detallada en documentación

### 🔄 Changed

- **Minimum PHP version**: Actualizado de 8.0 a 8.1
- **Dropped Laravel 8.x support**: Laravel 8 ya no es soportado (EOL)
- **Updated dependencies**:
  - `illuminate/support`: ^9.0|^10.0|^11.0|^12.0 (anteriormente ^8.0|^9.0|^10.0)
  - `phpunit/phpunit`: ^10.0|^11.0 (anteriormente ^9.5|^10.0|^11.0)
  - `orchestra/testbench`: ^7.0|^8.0|^9.0|^10.0 (añadidos v9 y v10)
  - `mockery/mockery`: ^1.6 (actualizado desde ^1.4)
- **GitHub Actions matrix**: Actualizada para incluir Laravel 11 y 12 con todas las combinaciones de PHP

### 📚 Documentation

- **README.md**: Actualizado con versiones PHP 8.1-8.4 y Laravel 9-12
- **CLAUDE.md**: Documentación completa de agentes y comandos Claude Code
- **Compatibility matrix**: Tabla detallada de versiones soportadas
- **.vscode/README.md**: Documentación de configuración VSCode
- **.claude/README.md**: Documentación de agentes y comandos

### 🧪 Testing

- **CI/CD expansion**: Tests en 12 combinaciones de PHP/Laravel (anteriormente 9)
- **Laravel 11.x testing**: PHP 8.2, 8.3, 8.4 con Testbench ^9.0
- **Laravel 12.x testing**: PHP 8.2, 8.3, 8.4 con Testbench ^10.0

### 🗑️ Removed

- **Laravel 8.x support**: Eliminado soporte para Laravel 8 (EOL)
- **PHP 8.0 support**: Eliminado soporte para PHP 8.0

---

## [1.1.1] - 2025-09-18

### 🔧 Fixed

- **Logger null-safe calls**: Agregado operador null-safe (?->) a llamadas logger() en SendBulkSmsJob y ConvoChatCache
- **PHPStan Level 8 compliance**: Eliminados todos los errores de análisis estático relacionados con logger()
- **GitHub Actions stability**: Estabilizada matriz de CI con combinaciones confiables de PHP/Laravel

### 🔄 Changed

- **Laravel 11 support**: Temporalmente removido soporte para Laravel 11 debido a incompatibilidades de dependencias
- **Supported Laravel versions**: Ahora soporta Laravel 8.x, 9.x, 10.x (matriz estable y probada)
- **CI/CD matrix**: Reducida a 9 combinaciones bien probadas para mayor confiabilidad

### 📚 Documentation

- **README badges**: Agregados badges de versiones PHP y Laravel soportadas
- **Test status badge**: Agregado badge dinámico del estado de GitHub Actions
- **Version accuracy**: Badges ahora reflejan las versiones realmente soportadas

### 🛡️ Quality Assurance

- **100% test success**: 41/41 tests pasando en todas las combinaciones soportadas
- **PHPStan Level 8**: Análisis estático sin errores
- **Stable CI pipeline**: GitHub Actions ejecutándose consistentemente sin fallos

---

## [1.1.0] - 2024-09-17

### ✨ Added

#### 🔧 Análisis de código y calidad
- **PHPStan level 8**: Análisis estático completo con cero errores
- **PHP CS Fixer**: Formateo automático de código con estándares PSR-12
- **GitHub Actions CI/CD**: Tests automáticos en múltiples versiones PHP (8.0-8.4) y Laravel (8-11)
- **Codecov integration**: Reportes de cobertura de código

#### ⚡ Mejoras de rendimiento y arquitectura
- **Inyección de dependencias mejorada**: HTTP client configurable y testeable
- **Constantes organizadas**: Eliminación de strings mágicos en servicios
- **Validación robusta de configuración**: Validación automática de API keys, URLs y timeouts
- **Logging estructurado**: Logs con contexto completo y API keys redactados
- **Cache layer**: Sistema de caché inteligente para respuestas de API
- **Queue integration**: Jobs para envíos masivos asincrónicos
- **Rate limiting**: Protección contra spam y abuso

#### 🧪 Testing y debugging
- **41 tests completos**: Cobertura de funcionalidad, configuración y errores
- **Tests de timeout**: Verificación de configuraciones personalizadas de timeout
- **Tests de constantes**: Validación de todos los valores constantes
- **Tests de configuración**: Validación de parámetros requeridos

#### 📚 Documentación ampliada
- **README comprehensivo**: Ejemplos avanzados, troubleshooting y mejores prácticas
- **Guías de seguridad**: Patrones de protección de API keys y validación
- **Ejemplos de performance**: Optimizaciones para envíos masivos
- **Guías de debugging**: Herramientas y técnicas de diagnóstico

### 🔄 Changed

- **Arquitectura de servicios**: Constructores mejorados con inyección de dependencias
- **Manejo de errores**: Sistema más robusto con logging detallado
- **Tests**: Migración a PHPUnit 12 con método names en lugar de @test annotations
- **Timeout configuration**: Ahora configurable por servicio y globalmente

### 🐛 Fixed

- **Tests de error handling**: Corrección de tests que esperaban arrays en lugar de excepciones
- **Composer dependencies**: Actualización a versiones más recientes y compatibles
- **PHPStan issues**: Resolución de todos los problemas de análisis estático
- **Code formatting**: Aplicación consistente de estándares de código

### 🛡️ Security

- **API key protection**: Redacción automática en logs
- **Input validation**: Validación estricta de números de teléfono y URLs
- **Rate limiting**: Protección contra abuso de API

### 📈 Performance

- **Connection reuse**: Reutilización de conexiones HTTP
- **Smart caching**: Caché con TTL optimizado por tipo de datos
- **Batch processing**: Soporte para envíos masivos eficientes
- **Memory optimization**: Uso reducido de memoria en operaciones bulk

---

## [1.0.0] - 2024-09-13

### Added

- Initial release of ConvoChat Laravel SMS Gateway
- SMS sending functionality with device and credit modes
- WhatsApp messaging support (text, media, documents)
- Laravel Service Provider with auto-discovery
- Facade for easy usage (`ConvoChat::sms()`, `ConvoChat::whatsapp()`)
- Configuration file with environment variable support
- Comprehensive error handling and logging
- WhatsApp account linking and management
- Phone number validation for WhatsApp
- Support for Laravel 8.x, 9.x, 10.x, 11.x
- PHP 8.0+ compatibility

### Features

#### SMS
- Send SMS via Android devices or credits
- SIM slot selection (1 or 2)
- Priority settings (high/normal)
- Gateway rate checking
- Device management
- Credit balance checking
- Subscription information

#### WhatsApp

- Text message sending
- Media messages (image, audio, video)
- Document sending (PDF, Excel, Word)
- Account linking with QR code
- Account relinking
- Phone number validation
- Multiple account support

#### Laravel Integration

- Service Provider with singleton bindings
- Publishable configuration
- Facade support
- Environment-based configuration
- Request logging (configurable)
- Comprehensive error handling

## [Unreleased]

### Planned

- Laravel Notifications channel
- Queue job support for bulk messaging
- Template system
- Webhook handling
- Artisan commands
- Analytics and metrics
- Multi-tenant support
