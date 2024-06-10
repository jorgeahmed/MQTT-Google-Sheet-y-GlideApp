# Proyecto: Automatización de Datos en Tiempo Real con Glide y MQTT

Este proyecto demuestra cómo integrar Glide y MQTT para actualizar datos en tiempo real en Google Sheets. A continuación se presenta una guía paso a paso para implementar este proyecto.

Descripción del Proyecto
Requisitos Previos
Instalación
Configuración
Ejecutar el Proyecto
Estructura del Proyecto
Contribución
Licencia

#Descripción del Proyecto
Este proyecto integra Glide y MQTT para automatizar la actualización de Google Sheets en tiempo real. Utiliza Node.js para suscribirse a tópicos de MQTT, procesar los mensajes y actualizar los datos en Google Sheets. Glide se utiliza para crear una interfaz de usuario accesible que permita monitorear los datos en tiempo real.

#Requisitos Previos
Antes de comenzar, asegúrate de tener instalados los siguientes requisitos:

Node.js (versión 12 o superior)
npm (Node Package Manager)
Una cuenta de Google con acceso a Google Sheets
Archivo de credenciales de la API de Google (google.json)
Acceso a un servidor MQTT

#Instalación
Clona el repositorio:

bash

git clone https://github.com/tu-usuario/nombre-del-proyecto.git
cd nombre-del-proyecto
Instala las dependencias:

bash
npm install

#Configuración
Configura las credenciales de Google API:

Asegúrate de tener el archivo google.json en el directorio raíz del proyecto. Este archivo debe contener las credenciales de la API de Google para acceder a Google Sheets.

Configura las variables del entorno:

Crea un archivo .env en el directorio raíz del proyecto y añade las siguientes variables:

env
Copy code
GOOGLE_SPREADSHEET_ID=your_spreadsheet_id
MQTT_BROKER_URL=mqtts://broker-url:port
MQTT_USERNAME=your_mqtt_username
MQTT_PASSWORD=your_mqtt_password
CA_CERT_PATH=path/to/your/ca-cert.pem

#Estructura de las Hojas de Cálculo de Google:

Asegúrate de que tu Google Sheet tenga las siguientes hojas y columnas:

Mqtt Status: Columnas A, B, C, D para Serial, Conectado, Fecha, Hora.
Instalaciones: Columnas AJ para Series de la PCB.
Ejecutar el Proyecto
Para ejecutar el proyecto, usa el siguiente comando:

bash
npm start

#Estructura del Proyecto
plaintext
├── google.json          # Archivo de credenciales de Google API
├── .env                 # Archivo de configuración del entorno
├── package.json         # Archivo de configuración de npm
├── server.js            # Archivo principal del servidor Node.js
└── README.md            # Documentación del proyecto
#Contribución
¡Las contribuciones son bienvenidas! Si deseas contribuir a este proyecto, sigue los siguientes pasos:

Haz un fork del repositorio.
Crea una nueva rama (git checkout -b feature/nueva-funcionalidad).
Realiza tus cambios y haz commit (git commit -am 'Agrega nueva funcionalidad').
Haz push a la rama (git push origin feature/nueva-funcionalidad).
Abre un Pull Request.
#Licencia
Este proyecto está bajo la licencia MIT. Consulta el archivo LICENSE para obtener más detalles.

