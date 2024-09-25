# moodle-payments

1. Desarrollo del plugin de Moodle

Primero, necesitas un plugin de pago en Moodle que interactúe con Lightning Network. Moodle ya tiene una estructura para manejar pagos a través de plugins como PayPal o Stripe, y puedes usarla como referencia.
Pasos:

    Configura un plugin de tipo pago:
        En Moodle, los plugins de pago están dentro de la categoría de “enrolment” (matrícula). El plugin que desarrolles se debe basar en este tipo.
        Crea la estructura básica del plugin en moodle/enrol/lightning:
            Carpeta del plugin: enrol/lightning
            Archivos principales:
                version.php: Define la versión y las dependencias del plugin.
                enrol.php: Contiene la lógica de matrícula para el curso.
                settings.php: Permite configurar opciones del plugin (por ejemplo, la integración con LNbits).
                lib.php: Funciones de integración.
                pluginname.php: Lenguajes y nombres del plugin.

    Configura el manejo de pagos:
        Usa las funciones de LNbits API para crear facturas Lightning (invoices). Puedes generar una nueva invoice al momento de que el usuario quiera matricularse en el curso.
        Al recibir el pago, la API de LNbits te enviará una confirmación que debes usar para matricular al usuario.

    Enlace con el sistema de matriculación de Moodle:
        Modifica la función enrol_user() para matricular automáticamente al usuario cuando se confirme el pago por la factura Lightning.
        Al recibir la confirmación de pago desde LNbits, puedes hacer que el usuario sea registrado en el curso específico.

    Configura la interfaz de usuario:
        Añade una opción en la página del curso para pagar usando Lightning Network. Esto podría ser un botón que interactúe con LNbits para generar el invoice.
        Usa tecnologías como Ajax para actualizar el estado del pago en tiempo real.

    Prueba el plugin:
        Instala tu plugin en una instancia de Moodle de prueba. Verifica que se realice la transacción y que los usuarios se matriculen correctamente después de pagar.

2. Desarrollo del plugin de LNbits

LNbits ya tiene una API para crear y manejar invoices. Tu plugin debería ser capaz de conectarse a esta API y manejar la confirmación de los pagos.
Pasos:

    Crea un plugin en LNbits:
        Desarrolla un plugin de LNbits que pueda generar facturas para cursos en Moodle.
        Este plugin necesitará una integración sencilla con la API de Moodle, o bien, podría enviar las confirmaciones de pago de vuelta a tu servidor Moodle para completar el proceso de matrícula.

    Integra Moodle y LNbits:
        Al generar una factura en LNbits, el plugin debe enviar el estado del pago a Moodle.
        Usa las API de LNbits para crear y verificar facturas. La API puede notificar a Moodle cuando un pago sea completado (por ejemplo, usando Webhooks).

    Autenticación y seguridad:
        Asegúrate de que la integración sea segura. LNbits y Moodle deben intercambiar tokens de seguridad o claves API para validar las solicitudes.

3. Pasos Adicionales

    Configura LNbits: Debes tener una instancia de LNbits corriendo, o puedes usar un servidor público. Configura una billetera para recibir los pagos de los cursos.
    Pruebas con regtest o testnet: Puedes probar toda la infraestructura en un entorno de pruebas utilizando regtest o testnet para Lightning Network antes de pasarlo a producción.

Documentación y recursos útiles:

    Desarrollo de plugins en Moodle
    LNbits API

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

