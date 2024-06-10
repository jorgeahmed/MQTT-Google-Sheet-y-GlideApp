# MQTT Glide Automation

## Introduction
This project automates the process of updating Google Sheets in real-time based on data received from an MQTT server. It uses Node.js to subscribe to MQTT topics, parse the data, and update the Google Sheets. Additionally, it integrates with Glide to provide a user-friendly interface for monitoring the data.

## Features
- Real-time data updates from MQTT to Google Sheets
- Integration with Glide for periodic monitoring
- Automatic handling of new and existing entries in Google Sheets
- Easy configuration and deployment

## Prerequisites
- Node.js installed on your machine
- A Google Cloud project with Google Sheets API enabled
- MQTT broker credentials

## Installation

1. Clone the repository:
    ```bash
    git clone https://github.com/jorgeahmed/mqtt-glide-automation.git
    cd mqtt-glide-automation
    ```

2. Install the dependencies:
    ```bash
    npm install
    ```

3. Create a `.env` file in the root of the project with the following content:
    ```
    GOOGLE_PROJECT_ID=your-google-project-id
    GOOGLE_CLIENT_EMAIL=your-google-client-email
    GOOGLE_PRIVATE_KEY="your-google-private-key"
    SPREADSHEET_ID=your-google-spreadsheet-id
    MQTT_BROKER_URL=your-mqtt-broker-url
    MQTT_USERNAME=your-mqtt-username
    MQTT_PASSWORD=your-mqtt-password
    ```

4. Place your `google.json` service account key file in the root of the project.

## Usage

1. Start the server:
    ```bash
    node server.js
    ```

## Code Explanation

### `server.js`

```javascript
// Importamos las librerías necesarias
const { google } = require('googleapis');
const express = require('express');
const mqtt = require('mqtt');
const fs = require('fs');
require('dotenv').config();

// Inicializamos la aplicación Express
const app = express();
const port = 3000;

// Configuramos la autenticación con Google
const auth = new google.auth.GoogleAuth({
    keyFile: './google.json', // Ruta del archivo de credenciales de Google
    scopes: ['https://www.googleapis.com/auth/spreadsheets'] // Alcance para acceder a Google Sheets
});

// ID de la hoja de cálculo de Google Sheets
const spreadsheetId = process.env.GOOGLE_SPREADSHEET_ID;
// Rango de celdas en la hoja de cálculo donde se escribirá la información
const statusRange = 'Mqtt Status!A:D'; // A:D cubre las columnas Serial, Conectado, Fecha y Hora

// Función para escribir datos en la hoja de cálculo de Google Sheets
async function writeToSheet(values, range) {
    const sheets = google.sheets({ version: 'v4', auth });

    try {
        // Obtener los datos actuales de la hoja de cálculo
        const currentData = await sheets.spreadsheets.values.get({
            spreadsheetId,
            range
        });

        const existingData = currentData.data.values || [];
        const existingRows = existingData.slice(1);
        const existingSerials = existingRows.map(row => row[0]);
        const newSerial = values[0][0];

        // Si el número de serie ya existe, actualizamos la fila correspondiente
        if (existingSerials.includes(newSerial)) {
            const rowIndex = existingSerials.indexOf(newSerial) + 2; // +2 para ajustar la fila según la hoja
            const updateRange = `${range.split('!')[0]}!A${rowIndex}:${range.split('!')[1].slice(-1)}${rowIndex}`;
            const resource = { values };

            const res = await sheets.spreadsheets.values.update({
                spreadsheetId,
                range: updateRange,
                valueInputOption: 'USER_ENTERED',
                resource
            });
            await updateXlookupFormula(rowIndex);
            return res.data;
        } else {
            // Si el número de serie no existe, añadimos una nueva fila
            const resource = { values };

            const res = await sheets.spreadsheets.values.append({
                spreadsheetId,
                range,
                valueInputOption: 'USER_ENTERED',
                resource
            });
            const rowIndex = existingData.length + 2; // Nueva fila añadida
            await updateXlookupFormula(rowIndex);
            return res.data;
        }
    } catch (error) {
        console.error(`Error al escribir en la hoja de cálculo para el rango ${range}:`, error);
    }
}

// Función para añadir fórmulas BUSCARX después de actualizar o añadir una fila
async function updateXlookupFormula(rowIndex) {
    const sheets = google.sheets({ version: 'v4', auth });

    const xlookupFormulaE = `=BUSCARX(A${rowIndex}, Instalaciones!AJ:AJ, Instalaciones!A:A)`;
    const xlookupFormulaF = `=BUSCARX(A${rowIndex}, Instalaciones!AJ:AJ, Instalaciones!B:B)`;

    const resource = {
        values: [
            [xlookupFormulaE, xlookupFormulaF]
        ]
    };

    const range = `Mqtt Status!E${rowIndex}:F${rowIndex}`;

    try {
        const res = await sheets.spreadsheets.values.update({
            spreadsheetId,
            range,
            valueInputOption: 'USER_ENTERED',
            resource
        });
        console.log(`Fórmulas BUSCARX añadidas para la fila ${rowIndex}:`, res.data);
    } catch (error) {
        console.error(`Error al añadir las fórmulas BUSCARX para la fila ${rowIndex}:`, error);
    }
}

// Configuración de la conexión MQTT
const client = mqtt.connect(process.env.MQTT_BROKER_URL, {
    username: process.env.MQTT_USERNAME,
    password: process.env.MQTT_PASSWORD,
    ca: fs.readFileSync(process.env.CA_CERT_PATH)
});

// Manejo de la conexión al servidor MQTT
client.on('connect', () => {
    console.log('Conectado al servidor MQTT');
    // Suscribirse al tópico sismico/status/#
    client.subscribe('sismico/status/#', { qos: 1 }, (err) => {
        if (!err) {
            console.log('Suscrito al tópico sismico/status/#');
        } else {
            console.error('Fallo al suscribirse al tópico', err);
        }
    });
});

// Manejo de los mensajes recibidos
client.on('message', async (topic, message) => {
    console.log(`Mensaje recibido en ${topic}: ${message.toString()}`);
    let data;
    try {
        data = JSON.parse(message.toString());
    } catch (e) {
        console.error('Error parseando el mensaje MQTT:', e);
        return;
    }

    const now = new Date();
    const fecha = now.toISOString().split('T')[0]; // Fecha en formato YYYY-MM-DD
    const hora = now.toTimeString().split(' ')[0]; // Hora en formato HH:MM:SS

    // Si el mensaje es del tópico sismico/status/#
    if (topic.startsWith('sismico/status/')) {
        const values = [
            [
                data.serial,
                data.connected ? 'TRUE' : 'FALSE',
                fecha,
                hora
            ]
        ];
        console.log('Actualizando estado en Google Sheets...');
        await writeToSheet(values, statusRange);
    }
});

// Ruta raíz de la aplicación Express
app.get('/', (req, res) => {
    res.send('¡Hola Mundo!');
});

// Iniciando el servidor Express
app.listen(port, () => {
    console.log(`Servidor ejecutándose en http://localhost:${port}`);
});
