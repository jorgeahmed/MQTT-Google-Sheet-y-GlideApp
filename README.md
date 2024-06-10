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
const { google } = require('googleapis');
const express = require('express');
const mqtt = require('mqtt');
const fs = require('fs');

const app = express();
const port = 3000;

// Initialize Google Auth
const auth = new google.auth.GoogleAuth({
    keyFile: './google.json',
    scopes: ['https://www.googleapis.com/auth/spreadsheets']
});

const spreadsheetId = process.env.SPREADSHEET_ID;
const statusRange = 'Mqtt Status!A:D'; // Define the range for status data

// Function to write data to Google Sheets
async function writeToSheet(values, range) {
    const sheets = google.sheets({ version: 'v4', auth });
    try {
        const currentData = await sheets.spreadsheets.values.get({
            spreadsheetId,
            range
        });

        const existingData = currentData.data.values || [];
        const existingRows = existingData.slice(1);
        const existingSerials = existingRows.map(row => row[0]);
        const newSerial = values[0][0];

        if (existingSerials.includes(newSerial)) {
            const rowIndex = existingSerials.indexOf(newSerial) + 2;
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
            const resource = { values };

            const res = await sheets.spreadsheets.values.append({
                spreadsheetId,
                range,
                valueInputOption: 'USER_ENTERED',
                resource
            });
            const rowIndex = existingData.length + 2;
            await updateXlookupFormula(rowIndex);
            return res.data;
        }
    } catch (error) {
        console.error(`Error writing to Google Sheets:`, error);
    }
}

// Function to update XLOOKUP formulas in Google Sheets
async function updateXlookupFormula(rowIndex) {
    const sheets = google.sheets({ version: 'v4', auth });

    const xlookupFormulaE = `=XLOOKUP(A${rowIndex}, Instalaciones!AJ:AJ, Instalaciones!A:A)`;
    const xlookupFormulaF = `=XLOOKUP(A${rowIndex}, Instalaciones!AJ:AJ, Instalaciones!B:B)`;

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
        console.log(`XLOOKUP formulas added for row ${rowIndex}:`, res.data);
    } catch (error) {
        console.error(`Error adding XLOOKUP formulas:`, error);
    }
}

// Connect to the MQTT broker
const client = mqtt.connect(process.env.MQTT_BROKER_URL, {
    username: process.env.MQTT_USERNAME,
    password: process.env.MQTT_PASSWORD,
    ca: fs.readFileSync('isrgrootx1 (1).pem')
});

client.on('connect', () => {
    console.log('Connected to MQTT broker');
    client.subscribe('sismico/status/#', { qos: 1 }, (err) => {
        if (!err) {
            console.log('Subscribed to sismico/status/#');
        } else {
            console.error('Failed to subscribe to topic', err);
        }
    });
});

client.on('message', async (topic, message) => {
    console.log(`Message received on ${topic}: ${message.toString()}`);
    let data;
    try {
        data = JSON.parse(message.toString());
    } catch (e) {
        console.error('Error parsing MQTT message:', e);
        return;
    }

    const now = new Date();
    const fecha = now.toISOString().split('T')[0]; // Format date as YYYY-MM-DD
    const hora = now.toTimeString().split(' ')[0]; // Format time as HH:MM:SS

    if (topic.startsWith('sismico/status/')) {
        const values = [
            [
                data.serial,
                data.connected ? 'TRUE' : 'FALSE',
                fecha,
                hora
            ]
        ];
        await writeToSheet(values, statusRange);
    }
});

app.get('/', (req, res) => {
    res.send('Hello World!');
});

app.listen(port, () => {
    console.log(`Server running at http://localhost:${port}`);
});
