# CLAUDE.md - ConvoChat Laravel SMS & WhatsApp Gateway

> Guía completa para asistentes de IA sobre el proyecto ConvoChat Laravel SMS & WhatsApp Gateway

## Información del Proyecto

**Nombre:** ConvoChat Laravel SMS & WhatsApp Gateway
**Package:** `convochatsms/laravel-sms-whatsapp-gateway`
**Versión actual:** 4.0.1
**Licencia:** MIT
**Repositorio:** https://github.com/RenatoAscencio/convochatsms
**Tipo:** Paquete Laravel para integración con ConvoChat API

### Descripción

Este es un paquete Laravel moderno y robusto que proporciona integración completa con la API de ConvoChat para envío de SMS y mensajes de WhatsApp. Incluye soporte para múltiples modos de envío, configuración avanzada, monitoreo, caché y sistema de cola para envíos masivos.

## Requisitos del Sistema

- **PHP:** 8.1 - 8.4
- **Laravel:** 10.x, 11.x, 12.x
- **Dependencias principales:**
  - `illuminate/support`: ^10.0|^11.0|^12.0
  - `guzzlehttp/guzzle`: ^7.0
- **API Key válida de ConvoChat:** Se obtiene desde https://sms.convo.chat/dashboard/tools/keys

### Matriz de Compatibilidad

| Laravel | PHP        | Orchestra Testbench | PHPUnit  | Support Status |
|---------|------------|---------------------|----------|----------------|
| 10.x    | 8.1 - 8.3  | ^8.0                | ^10.0    | ✅ Active      |
| 11.x    | 8.2 - 8.4  | ^9.0                | ^10.0-11.0 | ✅ Active      |
| 12.x    | 8.2 - 8.4  | ^10.0               | ^10.0-11.0 | ✅ Active      |

> **Nota**: Laravel 9.x ya no es soportado desde la versión 2.0. Para Laravel 9, utiliza la rama 1.x del paquete.

## Estructura del Proyecto

```
convochatsms/
├── .claude/
│   ├── agents/
│   │   ├── laravel-package-expert.md     # Agente experto en paquetes Laravel
│   │   ├── php-testing-expert.md         # Agente experto en testing PHP/PHPUnit
│   │   └── api-integration-specialist.md # Agente experto en integración de APIs
│   ├── commands/
│   │   ├── test.md             # Comando /test - Suite de tests
│   │   ├── fix.md              # Comando /fix - Corrección automática
│   │   ├── release.md          # Comando /release - Nueva versión
│   │   ├── docs.md             # Comando /docs - Actualizar docs
│   │   └── new-feature.md      # Comando /new-feature - Nueva feature
│   └── settings.local.json     # Configuración permisos Claude
├── .github/
│   └── workflows/
│       └── tests.yml           # CI/CD con GitHub Actions
├── .vscode/
│   ├── extensions.json         # Extensiones recomendadas VSCode
│   ├── settings.json           # Configuración workspace
│   ├── launch.json             # Configuraciones debug
│   ├── tasks.json              # Tareas automatizadas
│   └── snippets.code-snippets  # Snippets personalizados
├── config/
│   └── convochat.php           # Configuración principal
├── src/
│   ├── Cache/
│   │   └── ConvoChatCache.php  # Sistema de caché
│   ├── Console/
│   │   └── TestConvoChatCommand.php  # Comando Artisan de testing
│   ├── Facades/
│   │   └── ConvoChat.php       # Facade principal
│   ├── Jobs/
│   │   └── SendBulkSmsJob.php  # Job para envíos masivos
│   ├── Security/
│   │   └── RateLimiter.php     # Rate limiting
│   ├── Services/
│   │   ├── ConvoChatSmsService.php      # Servicio SMS
│   │   └── ConvoChatWhatsAppService.php # Servicio WhatsApp
│   └── ConvoChatServiceProvider.php     # Service Provider
├── tests/
│   ├── Feature/                # Tests de integración
│   ├── Unit/                   # Tests unitarios
│   └── TestCase.php
├── .php-cs-fixer.php           # Configuración PHP CS Fixer
├── phpstan.neon                # Configuración PHPStan nivel 8
├── phpunit.xml                 # Configuración PHPUnit
├── composer.json               # Dependencias y autoload
├── CLAUDE.md                   # Esta guía para asistentes IA
├── README.md                   # Documentación completa en español
├── CHANGELOG.md                # Registro de cambios
└── PUBLISHING.md               # Guía de publicación en Packagist
```

## Arquitectura y Componentes Principales

### 1. Service Provider (`ConvoChatServiceProvider`)

Ubicación: [src/ConvoChatServiceProvider.php](src/ConvoChatServiceProvider.php)

Responsabilidades:
- Registra los servicios como singletons en el contenedor de Laravel
- Publica configuración al directorio `config/` del proyecto
- Registra comandos Artisan
- Proporciona bindings para:
  - `convochat` (facade principal)
  - `convochat.sms` (servicio SMS)
  - `convochat.whatsapp` (servicio WhatsApp)

```php
// Servicios registrados como singletons
$this->app->singleton('convochat.sms', fn($app) => new ConvoChatSmsService());
$this->app->singleton('convochat.whatsapp', fn($app) => new ConvoChatWhatsAppService());
```

### 2. Servicio SMS (`ConvoChatSmsService`)

Ubicación: [src/Services/ConvoChatSmsService.php](src/Services/ConvoChatSmsService.php)

**Constantes importantes:**
```php
const DEFAULT_MODE = 'devices';        // Modo dispositivos Android
const CREDITS_MODE = 'credits';        // Modo créditos/gateway
const SMS_ENDPOINT = '/send/sms';
const DEVICES_ENDPOINT = '/get/devices';
const CREDITS_ENDPOINT = '/get/credits';
const GATEWAY_RATES_ENDPOINT = '/get/gateway/rates';
const SUBSCRIPTION_ENDPOINT = '/get/subscription/package';
const DEFAULT_BASE_URL = 'https://sms.convo.chat/api';
const DEFAULT_TIMEOUT = 30;
```

**Métodos principales:**
- `sendSms(array $params)`: Envío genérico con parámetros personalizados
- `sendSmsWithDevice(string $phone, string $message, string $deviceId, array $options)`: Envío con dispositivo Android
- `sendSmsWithCredits(string $phone, string $message, ?string $gatewayId, array $options)`: Envío con créditos
- `getDevices()`: Obtiene lista de dispositivos conectados
- `getCredits()`: Consulta créditos disponibles
- `getGatewayRates()`: Obtiene tarifas de gateways
- `getSubscription()`: Información de suscripción

**Validaciones:**
- Valida API key, base URL y timeout en el constructor
- Valida parámetros requeridos antes de cada request
- Sanitiza datos sensibles en logs (API key redactada)

### 3. Servicio WhatsApp (`ConvoChatWhatsAppService`)

Ubicación: [src/Services/ConvoChatWhatsAppService.php](src/Services/ConvoChatWhatsAppService.php)

**Constantes importantes:**
```php
const DEFAULT_TYPE = 'text';
const MEDIA_TYPE = 'media';
const DOCUMENT_TYPE = 'document';
const DEFAULT_PRIORITY = 2;              // 1=alta, 2=normal
const WHATSAPP_ENDPOINT = '/send/whatsapp';
const WA_SERVERS_ENDPOINT = '/get/wa_servers';
const WA_ACCOUNTS_ENDPOINT = '/get/wa_accounts';
const WA_LINK_ENDPOINT = '/create/wa/link';
const WA_RELINK_ENDPOINT = '/create/wa/relink';
const WA_VALIDATE_ENDPOINT = '/get/wa/validate_number';
```

**Métodos principales:**
- `sendMessage(array $params)`: Envío genérico de mensajes WhatsApp
- `sendText(string $account, string $recipient, string $message, int $priority)`: Envío de texto
- `sendMedia(string $account, string $recipient, string $message, string $mediaUrl, string $mediaType, int $priority)`: Envío de multimedia (imagen, video, audio)
- `sendDocument(string $account, string $recipient, string $message, string $documentUrl, string $documentName, string $documentType, int $priority)`: Envío de documentos
- `getWhatsAppServers()`: Lista servidores disponibles
- `linkWhatsAppAccount(int $serverId)`: Vincula nueva cuenta WhatsApp
- `relinkWhatsAppAccount(int $serverId, string $uniqueId)`: Re-vincula cuenta existente
- `validateWhatsAppNumber(string $accountId, string $phone)`: Valida número WhatsApp
- `getWhatsAppAccounts()`: Lista cuentas vinculadas

### 4. Facade (`ConvoChat`)

Ubicación: [src/Facades/ConvoChat.php](src/Facades/ConvoChat.php)

Proporciona acceso estático a los servicios:
```php
ConvoChat::sms()->sendSmsWithDevice($phone, $message, $deviceId);
ConvoChat::whatsapp()->sendText($account, $recipient, $message);
```

### 5. Componentes adicionales

**Cache** ([src/Cache/ConvoChatCache.php](src/Cache/ConvoChatCache.php)):
- Sistema de caché para respuestas de API
- TTL configurables por tipo de dato
- Warming estratégico de datos frecuentes

**Rate Limiter** ([src/Security/RateLimiter.php](src/Security/RateLimiter.php)):
- Limita requests por usuario/tiempo
- Previene abuso de API

**Bulk Job** ([src/Jobs/SendBulkSmsJob.php](src/Jobs/SendBulkSmsJob.php)):
- Job para envíos masivos en cola
- Soporte para retry con backoff progresivo

**Test Command** ([src/Console/TestConvoChatCommand.php](src/Console/TestConvoChatCommand.php)):
- Comando Artisan para probar la integración
- Uso: `php artisan convochat:test`

## Configuración

### Archivo de configuración (`config/convochat.php`)

```php
return [
    'api_key' => env('CONVOCHAT_API_KEY', ''),
    'base_url' => env('CONVOCHAT_BASE_URL', 'https://sms.convo.chat/api'),
    'timeout' => (int) env('CONVOCHAT_TIMEOUT', 30),
    'log_requests' => env('CONVOCHAT_LOG_REQUESTS', false),

    'sms' => [
        'default_mode' => env('CONVOCHAT_SMS_MODE', 'devices'),
        'default_priority' => env('CONVOCHAT_SMS_PRIORITY', 2),
        'default_device' => env('CONVOCHAT_SMS_DEVICE', null),
        'default_gateway' => env('CONVOCHAT_SMS_GATEWAY', null),
        'default_sim' => env('CONVOCHAT_SMS_SIM', 1),
    ],

    'whatsapp' => [
        'default_account' => env('CONVOCHAT_WA_ACCOUNT', ''),
        'default_priority' => env('CONVOCHAT_WA_PRIORITY', 2),
    ],
];
```

### Variables de entorno requeridas

```env
# Requerido
CONVOCHAT_API_KEY=tu_api_key_aqui

# Opcional
CONVOCHAT_BASE_URL=https://sms.convo.chat/api
CONVOCHAT_TIMEOUT=30
CONVOCHAT_LOG_REQUESTS=false

# SMS
CONVOCHAT_SMS_MODE=devices
CONVOCHAT_SMS_PRIORITY=2
CONVOCHAT_SMS_DEVICE=tu_device_id
CONVOCHAT_SMS_SIM=1

# WhatsApp
CONVOCHAT_WA_ACCOUNT=tu_account_id
CONVOCHAT_WA_PRIORITY=2
```

## Testing

### Configuración de tests

**PHPUnit** ([phpunit.xml](phpunit.xml)):
- Tests organizados en Feature (integración) y Unit (unitarios)
- Coverage configurado
- Uses TestCase base con Mockery

**PHPStan** ([phpstan.neon](phpstan.neon)):
- Nivel 8 (máximo análisis estático)
- Excluye vendor y Console
- Ignora algunos errores específicos de Laravel/Mockery

**PHP CS Fixer** ([.php-cs-fixer.php](.php-cs-fixer.php)):
- PSR-12 estándar
- Configuración personalizada para Laravel

### Ejecutar tests

```bash
# Tests completos
./vendor/bin/phpunit

# Tests con coverage
./vendor/bin/phpunit --coverage-html coverage

# PHPStan análisis estático
./vendor/bin/phpstan analyse

# PHP CS Fixer validación
./vendor/bin/php-cs-fixer fix --dry-run --diff

# PHP CS Fixer auto-corrección
./vendor/bin/php-cs-fixer fix
```

### Tests existentes

**Unit Tests:**
- [tests/Unit/ConvoChatConstantsTest.php](tests/Unit/ConvoChatConstantsTest.php): Validación de constantes
- [tests/Unit/ConvoChatSmsServiceTest.php](tests/Unit/ConvoChatSmsServiceTest.php): Servicio SMS
- [tests/Unit/ConvoChatWhatsAppServiceTest.php](tests/Unit/ConvoChatWhatsAppServiceTest.php): Servicio WhatsApp
- [tests/Unit/ConvoChatConfigurationTest.php](tests/Unit/ConvoChatConfigurationTest.php): Configuración

**Feature Tests:**
- [tests/Feature/ConvoChatIntegrationTest.php](tests/Feature/ConvoChatIntegrationTest.php): Integración completa
- [tests/Feature/FacadeTest.php](tests/Feature/FacadeTest.php): Testing facade
- [tests/Feature/ConvoChatTimeoutTest.php](tests/Feature/ConvoChatTimeoutTest.php): Timeouts

## Patrones de uso comunes

### 1. Envío básico SMS

```php
use ConvoChat\LaravelSmsGateway\Facades\ConvoChat;

// Con dispositivo Android
$result = ConvoChat::sms()->sendSmsWithDevice(
    phone: '+573001234567',
    message: 'Hola desde ConvoChat',
    deviceId: 'abc123'
);

// Con créditos
$result = ConvoChat::sms()->sendSmsWithCredits(
    phone: '+573001234567',
    message: 'Hola desde ConvoChat'
);
```

### 2. Envío básico WhatsApp

```php
// Texto
$result = ConvoChat::whatsapp()->sendText(
    account: 'wa_account_id',
    recipient: '+573001234567',
    message: 'Hola por WhatsApp'
);

// Multimedia
$result = ConvoChat::whatsapp()->sendMedia(
    account: 'wa_account_id',
    recipient: '+573001234567',
    message: 'Mira esta imagen',
    mediaUrl: 'https://ejemplo.com/imagen.jpg',
    mediaType: 'image'
);
```

### 3. Inyección de dependencias

```php
use ConvoChat\LaravelSmsGateway\Services\ConvoChatSmsService;

class NotificationController extends Controller
{
    public function __construct(
        private ConvoChatSmsService $smsService
    ) {}

    public function send(Request $request)
    {
        $result = $this->smsService->sendSmsWithCredits(
            $request->phone,
            $request->message
        );

        return response()->json($result);
    }
}
```

### 4. Envío masivo con Jobs

```php
use ConvoChat\LaravelSmsGateway\Jobs\SendBulkSmsJob;

$recipients = ['+573001234567', '+573007654321'];
$message = 'Mensaje masivo';

dispatch(new SendBulkSmsJob($recipients, $message));
```

## Manejo de errores

### Tipos de errores

1. **Errores de configuración:**
   - API key no configurada
   - URL inválida
   - Timeout inválido

2. **Errores de validación:**
   - Parámetros requeridos faltantes
   - Formato inválido (teléfono, URL, etc.)

3. **Errores de API:**
   - Device offline
   - Créditos insuficientes
   - Rate limit excedido
   - WhatsApp account no vinculado

4. **Errores de red:**
   - Timeout de conexión
   - Error HTTP (500, 502, etc.)

### Manejo recomendado

```php
try {
    $result = ConvoChat::sms()->sendSmsWithDevice($phone, $message, $deviceId);

    if (isset($result['status']) && $result['status'] === 'success') {
        // Éxito
        Log::info('SMS enviado', $result);
    } else {
        // Error en respuesta de API
        Log::warning('Error API', $result);
    }
} catch (\InvalidArgumentException $e) {
    // Error de validación/configuración
    Log::error('Validation error: ' . $e->getMessage());
} catch (\Exception $e) {
    // Error de red o API
    Log::error('ConvoChat error: ' . $e->getMessage());
}
```

## Logging

El paquete incluye logging automático cuando `CONVOCHAT_LOG_REQUESTS=true`:

**Logs exitosos:**
```php
Log::info('ConvoChat SMS API Request Success', [
    'endpoint' => '/send/sms',
    'response_status' => 'success',
    'request_time' => '2025-10-09 10:30:00',
    'base_url' => 'https://sms.convo.chat/api',
]);
```

**Logs de error:**
```php
Log::error('ConvoChat SMS API Error', [
    'endpoint' => '/send/sms',
    'error_message' => 'Connection timeout',
    'error_code' => 28,
    'request_data' => ['phone' => '+57...', 'secret' => '[REDACTED]'],
    'base_url' => 'https://sms.convo.chat/api',
    'timeout' => 30,
]);
```

**Importante:** Los API keys siempre se redactan en logs por seguridad.

## CI/CD y Automatización

### GitHub Actions ([.github/workflows/tests.yml](.github/workflows/tests.yml))

- **Tests automáticos** en PHP 8.1, 8.2, 8.3, 8.4
- **Laravel versions:** 10.x, 11.x, 12.x
- **PHPStan** nivel 8 en pipeline
- **PHP CS Fixer** validación
- **Coverage reports** (si está configurado)

### Badges en README

```markdown
[![Tests](https://github.com/RenatoAscencio/convochatsms/actions/workflows/tests.yml/badge.svg)]
[![PHPStan Level 8](https://img.shields.io/badge/PHPStan-level%208-brightgreen.svg)]
[![PHP Versions](https://img.shields.io/badge/PHP-8.1%20%7C%208.2%20%7C%208.3%20%7C%208.4-blue.svg)]
[![Laravel Versions](https://img.shields.io/badge/Laravel-10%20%7C%2011%20%7C%2012-red.svg)]
```

## Guías para modificación

### Agregar nuevo método al servicio SMS

1. Agregar método en `ConvoChatSmsService`:
```php
public function sendScheduledSms(string $phone, string $message, string $scheduleTime): array
{
    return $this->sendSms([
        'phone' => $phone,
        'message' => $message,
        'schedule_time' => $scheduleTime,
    ]);
}
```

2. Agregar test en `tests/Unit/ConvoChatSmsServiceTest.php`
3. Documentar en README.md
4. Ejecutar tests y PHPStan
5. Actualizar CHANGELOG.md

### Agregar nueva configuración

1. Agregar en `config/convochat.php`:
```php
'retry_attempts' => env('CONVOCHAT_RETRY_ATTEMPTS', 3),
```

2. Usar en servicio:
```php
$this->retryAttempts = $config['retry_attempts'] ?? config('convochat.retry_attempts', 3);
```

3. Documentar variable en README
4. Agregar test para validación

### Crear nueva feature

1. Crear branch: `git checkout -b feature/nueva-funcionalidad`
2. Implementar código con tests
3. Ejecutar suite completa: `composer test`
4. Actualizar documentación
5. Commit y push
6. Crear Pull Request

## Comandos útiles

### Comandos de terminal

```bash
# Instalación en proyecto Laravel
composer require convochatsms/laravel-sms-whatsapp-gateway

# Publicar configuración
php artisan vendor:publish --tag=convochat-config

# Test de conexión
php artisan convochat:test

# Tests completos
composer test

# PHPStan
composer analyse

# Fix code style
composer format

# Limpiar cache Laravel
php artisan config:clear
php artisan cache:clear
```

### Comandos Claude (Slash Commands)

El proyecto incluye comandos Claude personalizados para automatizar tareas comunes:

#### `/test` - Suite Completa de Tests
Ejecuta automáticamente:
- PHPUnit con todos los tests
- PHPStan análisis estático (nivel 8)
- PHP CS Fixer verificación PSR-12
- Muestra resumen y sugiere correcciones si hay errores

#### `/fix` - Corrección Automática de Código
- Ejecuta PHP CS Fixer para arreglar formato
- Analiza con PHPStan y sugiere soluciones
- Verifica que tests sigan pasando
- Muestra archivos modificados

#### `/release` - Preparar Nueva Versión
- Verifica que todo esté limpio (git status)
- Ejecuta suite completa de tests
- Actualiza CHANGELOG.md
- Crea git tag con nueva versión
- Muestra instrucciones para Packagist

#### `/docs` - Actualizar Documentación
- Revisa cambios recientes en código
- Actualiza README.md con nuevas features
- Mantiene CHANGELOG.md al día
- Verifica consistencia en CLAUDE.md
- Actualiza ejemplos de código

#### `/new-feature` - Workflow para Nueva Feature
Workflow completo que incluye:
1. Planificación y diseño
2. Implementación siguiendo estándares
3. Creación de tests (unitarios + integración)
4. Documentación completa
5. Verificación final y commit

**Uso:** En Claude Code, simplemente escribe `/test`, `/fix`, etc. y el agente ejecutará el workflow completo.

### Agentes Especializados de Claude Code

El proyecto incluye agentes especializados en `/.claude/agents/` que proporcionan experiencia profunda en áreas específicas del desarrollo. Estos agentes son invocados automáticamente por Claude Code cuando detecta tareas que coinciden con su especialización.

#### `laravel-package-expert` - Experto en Paquetes Laravel

**Cuándo se usa:**
- Desarrollo de nuevas features para el paquete
- Modificación de service providers, facades o configuración
- Publicación y versionado del paquete en Packagist
- Integración con Laravel (dependency injection, configuration, etc.)
- Arquitectura de paquetes y mejores prácticas

**Capacidades:**
- Diseño de arquitectura de paquetes Laravel
- Service providers, facades y bindings
- Testing con Orchestra Testbench
- Composer y Packagist (publicación, versionado, semantic versioning)
- Compatibilidad multi-versión (Laravel 10.x-12.x, PHP 8.1-8.4)
- PHPStan nivel 8, PHP CS Fixer (PSR-12)
- CI/CD con GitHub Actions

**Ejemplo:**
```
user: "Necesito agregar un servicio de notificaciones al paquete ConvoChat"
assistant: Usaré el laravel-package-expert para diseñar e implementar el servicio siguiendo las mejores prácticas de Laravel.
```

#### `php-testing-expert` - Experto en Testing PHP/PHPUnit

**Cuándo se usa:**
- Creación de tests unitarios o de integración
- Debugging de tests fallidos
- Mejora de code coverage
- Configuración de PHPUnit
- Mocking de dependencias (HTTP clients, servicios, etc.)
- Testing de APIs y servicios externos

**Capacidades:**
- PHPUnit (assertions, data providers, mocking)
- Mockery y test doubles avanzados
- Orchestra Testbench para testing de paquetes
- Code coverage y análisis de calidad
- Test-Driven Development (TDD)
- HTTP client mocking (Guzzle)
- Testing de excepciones, timeouts y edge cases

**Ejemplo:**
```
user: "Necesito tests para el nuevo método sendScheduledSms"
assistant: Usaré el php-testing-expert para crear tests completos con mocking de HTTP y validación de parámetros.
```

#### `api-integration-specialist` - Especialista en Integración de APIs

**Cuándo se usa:**
- Integración de nuevos endpoints de la API ConvoChat
- Problemas de timeout o errores de red
- Implementación de retry logic y error handling
- Validación de requests/responses
- Autenticación y seguridad de APIs
- Optimización de performance (caching, async requests)

**Capacidades:**
- Diseño de clientes HTTP con Guzzle
- Error handling completo (status codes, timeouts, retries)
- Request/response validation
- Authentication (API keys, OAuth, JWT)
- Retry logic con exponential backoff
- Rate limiting y circuit breaker patterns
- Caching estratégico de respuestas
- Security best practices (credential management, HTTPS)

**Ejemplo:**
```
user: "Las llamadas a la API están fallando por timeout, ¿cómo lo resuelvo?"
assistant: Usaré el api-integration-specialist para implementar retry logic con exponential backoff y mejorar el manejo de timeouts.
```

### Cómo Funcionan los Agentes

Los agentes son invocados automáticamente por Claude Code cuando:
1. Detecta que tu tarea coincide con la especialización del agente
2. Necesita experiencia profunda en un dominio específico
3. La tarea es compleja y requiere conocimiento especializado

**Ventajas:**
- **Expertise profunda**: Cada agente es un experto en su dominio
- **Consistencia**: Siguen los estándares y patrones del proyecto
- **Productividad**: Resuelven problemas complejos más rápidamente
- **Calidad**: Aplican mejores prácticas automáticamente

**Ubicación:** `/.claude/agents/`
**Formato:** Markdown con frontmatter (name, description, model, color)
**Documentación interna:** Cada agente incluye documentación completa de sus capacidades

## Referencias y recursos

### Documentación oficial
- **API ConvoChat:** https://docs.convo.chat
- **Dashboard:** https://sms.convo.chat
- **API Keys:** https://sms.convo.chat/dashboard/tools/keys
- **Packagist:** https://packagist.org/packages/convochatsms/laravel-sms-whatsapp-gateway

### Archivos importantes
- [README.md](README.md): Documentación completa con ejemplos
- [CHANGELOG.md](CHANGELOG.md): Historial de versiones
- [PUBLISHING.md](PUBLISHING.md): Guía para publicar en Packagist
- [composer.json](composer.json): Dependencias y metadatos

### Soporte
- **Issues:** https://github.com/RenatoAscencio/convochatsms/issues
- **Source:** https://github.com/RenatoAscencio/convochatsms

## Commits recientes

```
dfdd50f - Actualizar nombre del paquete para incluir WhatsApp
f71a3e8 - Cambiar nombre del paquete para instalación en vendor
bf4a4f6 - Update CHANGELOG for v1.1.1 release
c847515 - Agregar badge de status de tests en README
1378732 - Corregir errores de PHPStan con logger() null
```

## Estado actual del proyecto

- **Branch actual:** main
- **PHPStan:** Nivel 8 ✅
- **Tests:** Pasando ✅
- **PHP CS Fixer:** Configurado ✅
- **CI/CD:** GitHub Actions activo ✅
- **Coverage:** Configurado (badges disponibles)
- **Versión estable:** 4.0.1

## Notas para asistentes de IA

### Al modificar código:
1. Siempre mantener PHPStan nivel 8 sin errores
2. Seguir PSR-12 y estándares de Laravel
3. Agregar tests para nuevas funcionalidades
4. Actualizar documentación en README.md
5. Redactar API keys en logs
6. Usar typed properties (PHP 7.4+)
7. Documentar con PHPDoc cuando sea necesario

### Al sugerir cambios:
1. Verificar compatibilidad con PHP 8.1+
2. Mantener compatibilidad con Laravel 10.x, 11.x, 12.x
3. Seguir convenciones del proyecto existente
4. Proporcionar ejemplos de uso
5. Considerar backward compatibility

### Testing:
1. Mockear HTTP calls con Guzzle/Http Facade
2. Usar Orchestra Testbench para tests de paquetes
3. Separar unit tests (aislados) y feature tests (integración)
4. Validar edge cases y errores

### Seguridad:
1. Nunca loggear API keys completas
2. Validar y sanitizar inputs
3. Usar HTTPS exclusivamente
4. Implementar rate limiting cuando sea apropiado
5. Documentar riesgos de seguridad

## Entorno de Desarrollo VSCode

### Extensiones Instaladas

El proyecto incluye configuración completa de VSCode en `.vscode/`:

**Extensiones PHP/Laravel:**
- Intelephense - Autocompletado PHP inteligente
- Laravel Extra Intellisense - Autocompletado routes, views, configs
- Laravel Blade Snippets - Syntax highlighting Blade
- Laravel Goto View - Navegación rápida a vistas
- Laravel Intellisense - Facades y helpers

**Testing y Calidad:**
- PHPUnit - Ejecutar tests desde VSCode
- PHPStan - Análisis estático nivel 8 en tiempo real
- PHP CS Fixer - Formateo automático PSR-12
- PHP CodeSniffer - Validación estándares

**Productividad:**
- GitLens - Git superpoderes
- Git Graph - Visualización árbol commits
- TODO Tree - Organización TODOs
- Error Lens - Errores inline
- Better Comments - Comentarios con colores

**Otros:**
- EditorConfig - Consistencia formato
- Composer - Gestión dependencias
- Markdown All in One - Edición Markdown

### Configuración Aplicada

- ✅ Formateo automático al guardar (PSR-12)
- ✅ PHPStan nivel 8 en tiempo real
- ✅ PHPUnit integrado con botones "Run"
- ✅ Autocompletado Laravel completo
- ✅ Git Lens para historial
- ✅ Error Lens inline
- ✅ Rulers a 120 caracteres

### Snippets Disponibles

- `test` → Método test PHPUnit completo
- `convosms` → Envío SMS ConvoChat
- `convowa` → Envío WhatsApp ConvoChat
- `tryconvo` → Try-catch para ConvoChat
- `lcontroller` → Método controlador Laravel
- `doc` → PHPDoc block

### Tareas Disponibles (Cmd+Shift+B)

- Run PHPUnit Tests (default)
- Run PHPUnit with Coverage
- Run PHPStan Analysis
- Run PHP CS Fixer (Check/Fix)
- Composer Install/Update
- Full Test Suite (ejecuta todo)

---

**Última actualización:** 2025-10-09
**Mantenido por:** ConvoChat Team
**Contacto:** support@convo.chat
