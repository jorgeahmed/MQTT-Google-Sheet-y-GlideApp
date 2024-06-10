# Proyecto: Automatización de Datos en Tiempo Real con Glide y MQTT

Este proyecto demuestra cómo integrar Glide y MQTT para actualizar datos en tiempo real en Google Sheets. A continuación se presenta una guía paso a paso para implementar este proyecto.

## Tabla de Contenidos

- [Descripción del Proyecto](#descripción-del-proyecto)
- [Requisitos Previos](#requisitos-previos)
- [Instalación](#instalación)
- [Configuración](#configuración)
- [Ejecutar el Proyecto](#ejecutar-el-proyecto)
- [Estructura del Proyecto](#estructura-del-proyecto)
- [Contribución](#contribución)
- [Licencia](#licencia)

## Descripción del Proyecto

Este proyecto integra Glide y MQTT para automatizar la actualización de Google Sheets en tiempo real. Utiliza Node.js para suscribirse a tópicos de MQTT, procesar los mensajes y actualizar los datos en Google Sheets. Glide se utiliza para crear una interfaz de usuario accesible que permita monitorear los datos en tiempo real.

## Requisitos Previos

Antes de comenzar, asegúrate de tener instalados los siguientes requisitos:

- Node.js (versión 12 o superior)
- npm (Node Package Manager)
- Una cuenta de Google con acceso a Google Sheets
- Archivo de credenciales de la API de Google (google.json)
- Acceso a un servidor MQTT

## Instalación

1. **Clona el repositorio:**

   ```bash
   git clone https://github.com/tu-usuario/nombre-del-proyecto.git
   cd nombre-del-proyecto
