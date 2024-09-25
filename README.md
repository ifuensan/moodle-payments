# moodle-payments

## Introducción
El objetivo de este diseño es definir la guía para la creación del plugin de Moodle que permita a los usuarios pagar los cursos mediante Lightning Network utilizando LNbits como proveedor de pagos. Incluye la estructura de archivos, el flujo de proceso de pago y la integración con la API de LNbits. Así como los principales componentes de código que ayudarán posteriormente a desarrollar e implementar la solución en el entorno.

## Desarrollo del plugin de Moodle

Primero, necesitamos un plugin para realizar pagos en Moodle que interactúe con Lightning Network. Moodle ya tiene una estructura para manejar pagos a través de plugins como PayPal o Stripe, y se puede usar como referencia. Aquí dejo lo pasos que pienso podrían hacer falta para conseguir el objetivo:

1. Configura un plugin de tipo pago:
    - En Moodle, los plugins de pago están dentro de la categoría de “enrolment” (matrícula). El plugin que desarrolles se debe basar en este tipo.
    - Crea la estructura básica del plugin en moodle/enrol/lightning:
        - Carpeta del plugin: enrol/lightning
        - Archivos principales:
            - version.php: Define la versión y las dependencias del plugin.
            - enrol.php: Contiene la lógica de matrícula para el curso.
            - settings.php: Permite configurar opciones del plugin (por ejemplo, la integración con LNbits).
            - lib.php: Funciones de integración.
            - pluginname.php: Lenguajes y nombres del plugin.

2. Configura el manejo de pagos:
- Usa las funciones de LNbits API para crear facturas Lightning (invoices). Puedes generar una nueva invoice al momento de que el usuario quiera matricularse en el curso.
- Al recibir el pago, la API de LNbits te enviará una confirmación que debes usar para matricular al usuario.

3. Enlace con el sistema de matriculación de Moodle:
- Modifica la función enrol_user() para matricular automáticamente al usuario cuando se confirme el pago por la factura Lightning.
- Al recibir la confirmación de pago desde LNbits, puedes hacer que el usuario sea registrado en el curso específico.

3. Configura la interfaz de usuario:
- Añade una opción en la página del curso para pagar usando Lightning Network. Esto podría ser un botón que interactúe con LNbits para generar el invoice.
- Usa tecnologías como Ajax para actualizar el estado del pago en tiempo real.

4. Prueba el plugin:
- Instala tu plugin en una instancia de Moodle de prueba. Verifica que se realice la transacción y que los usuarios se matriculen correctamente después de pagar.

## Desarrollo del plugin de LNbits

LNbits ya tiene una API para crear y manejar invoices. Tu plugin debería ser capaz de conectarse a esta API y manejar la confirmación de los pagos.
Pasos:

1. Crea un plugin en LNbits:
- Desarrolla un plugin de LNbits que pueda generar facturas para cursos en Moodle.
- Este plugin necesitará una integración sencilla con la API de Moodle, o bien, podría enviar las confirmaciones de pago de vuelta a tu servidor Moodle para completar el proceso de matrícula.

2. Integra Moodle y LNbits:
- Al generar una factura en LNbits, el plugin debe enviar el estado del pago a Moodle.
- Usa las API de LNbits para crear y verificar facturas. La API puede notificar a Moodle cuando un pago sea completado (por ejemplo, usando Webhooks).

3. Autenticación y seguridad:
- Asegúrate de que la integración sea segura. LNbits y Moodle deben intercambiar tokens de seguridad o claves API para validar las solicitudes.

3. Pasos Adicionales
- Configura LNbits: Debes tener una instancia de LNbits corriendo, o puedes usar un servidor público. Configura una billetera para recibir los pagos de los cursos.
- Pruebas con regtest o testnet: Puedes probar toda la infraestructura en un entorno de pruebas utilizando regtest o testnet para Lightning Network antes de pasarlo a producción.

Documentación y recursos útiles:
- [Desarrollo de plugins en Moodle]()
- [LNbits API]()

Este enfoque te permitirá crear un plugin de Moodle que use Lightning Network para los pagos y un plugin de LNbits que facilite la integración entre ambas plataformas.

## Estructura del Plugin de Moodle (Lightning Payment)
### Directorio del plugin: moodle/enrol/lightning/
```bash
moodle/
└── enrol/
    └── lightning/
        ├── version.php         # Versión del plugin y dependencias.
        ├── enrol.php           # Lógica de matrícula.
        ├── settings.php        # Configuraciones del plugin (API de LNbits).
        ├── lib.php             # Funciones auxiliares.
        ├── pluginname.php      # Archivos de lenguaje.
        ├── classes/
        │   └── invoice_manager.php  # Clase para manejar invoices (con LNbits API).
        └── db/
            └── install.xml     # Estructura de la base de datos (opcional).
```
### Archivo version.php

Define la versión del plugin y las dependencias, por ejemplo, el número de la versión de Moodle requerida.
```php
defined('MOODLE_INTERNAL') || die();
$plugin->version = 2024092500;  // Versión del plugin.
$plugin->requires = 2020110900; // Requiere Moodle 3.10 o superior.
$plugin->component = 'enrol_lightning'; // Nombre del plugin.
```

### Archivo enrol.php
```php
Gestiona el proceso de inscripción de los usuarios tras recibir confirmación de pago.
function enrol_lightning_plugin_enrol_user($userid, $courseid, $payment_status) {
    if ($payment_status == 'paid') {
        // Enroll user in course if payment confirmed.
        enrol_user($userid, $courseid);
    }
}
```
### Archivo lib.php

Aquí estarán las funciones para interactuar con la API de LNbits y generar facturas.

```
function generate_lightning_invoice($courseid, $amount) {
    // Llama a la API de LNbits para generar una factura.
    $lnbits_api_key = get_config('enrol_lightning', 'lnbits_api_key');
    $url = "https://lnbits.example.com/api/v1/payments";

    $params = [
        'amount' => $amount * 1000,  // Satoshis
        'memo' => 'Payment for course ' . $courseid
    ];

    $ch = curl_init();
    curl_setopt($ch, CURLOPT_URL, $url);
    curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
    curl_setopt($ch, CURLOPT_POSTFIELDS, json_encode($params));
    curl_setopt($ch, CURLOPT_HTTPHEADER, [
        'Content-Type: application/json',
        'X-Api-Key: ' . $lnbits_api_key
    ]);

    $response = curl_exec($ch);
    curl_close($ch);

    return json_decode($response, true);
}
```
## Integración con LNbits
Componentes principales:

    LNbits API: El plugin de Moodle se conectará a la API de LNbits para generar facturas y obtener el estado de los pagos.
    Webhooks de LNbits: LNbits puede enviar confirmaciones de pago a Moodle, utilizando una URL que reciba los webhooks. Esto se manejará dentro del plugin de Moodle.

Proceso de Pago:

    Generar factura Lightning:
        Al seleccionar un curso y optar por el pago con Lightning, Moodle envía una solicitud a la API de LNbits para generar una factura.
        LNbits responde con una factura (un código QR) que el usuario puede escanear.

    Verificación del pago:
        Moodle usa la API de LNbits para verificar el estado de la factura.
        Una vez que se detecta el pago, Moodle matricula automáticamente al usuario en el curso.
## Diagrama de flujo del proceso de pago

```mermaid
graph TD;
    A[Usuario selecciona un curso] --> B[Opción de pagar con LN];
    B --> C[Generar invoice con LNbits API];
    C --> D[Factura generada: QR o Lightning URL];
    D --> E[Usuario paga usando su billetera LN];
    E --> F[LNbits confirma pago a Moodle];
    F --> G[Moodle matricula al usuario en el curso];
```

## Webhook para recibir confirmaciones de pago

En el archivo enrol.php, puedes configurar un endpoint en Moodle que reciba la confirmación de pago desde LNbits.
```php
// Endpoint para recibir Webhook de LNbits
function handle_lnbits_webhook() {
    $input = file_get_contents('php://input');
    $data = json_decode($input, true);

    if ($data['status'] == 'paid') {
        $userid = $data['metadata']['userid'];
        $courseid = $data['metadata']['courseid'];
        
        enrol_lightning_plugin_enrol_user($userid, $courseid, 'paid');
    }
}
```

## Configuración del plugin en Moodle
Archivo settings.php

Este archivo proporciona la interfaz para configurar el plugin dentro de Moodle, como agregar la clave API de LNbits.
```php
if ($ADMIN->fulltree) {
    $settings->add(new admin_setting_configtext(
        'enrol_lightning/lnbits_api_key',
        get_string('lnbitsapikey', 'enrol_lightning'),
        get_string('lnbitsapikey_desc', 'enrol_lightning'),
        ''
    ));
}
```

## Ejemplo de conexión entre Moodle y LNbits
```php
$invoice = generate_lightning_invoice($courseid, $courseprice);

if ($invoice) {
    // Muestra el código QR o Lightning URL al usuario
    echo 'Por favor, paga escaneando este código QR:';
    echo '<img src="https://chart.googleapis.com/chart?chs=300x300&cht=qr&chl=' . urlencode($invoice['payment_request']) . '" />';
}
```
