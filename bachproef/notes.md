# BachProef AZL Notes

## Optie 1: DigitalOcean Spaces met AWS-SDK

### Backend Implementatie

Voor het ophalen van bestanden kan de `aws-sdk` package in Node.js worden gebruikt. Dit gebeurt met de `listObjectsV2` methode, die een lijst van bestanden in een DigitalOcean Space ophaalt.

#### Codevoorbeeld (Node.js API):

```javascript
const AWS = require('aws-sdk');

const spacesEndpoint = new AWS.Endpoint('ams3.digitaloceanspaces.com');
const s3 = new AWS.S3({
    endpoint: spacesEndpoint,
    accessKeyId: process.env.SPACES_KEY,
    secretAccessKey: process.env.SPACES_SECRET
});

app.get('/api/files', async (req, res) => {
    const params = { Bucket: 'your-bucket-name' };
    try {
        const data = await s3.listObjectsV2(params).promise();
        res.json(data.Contents);
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});
```

### Frontend Implementatie

Op de Angular-kant kan deze API worden aangeroepen om bestanden op te halen en weer te geven in een bestandsexplorer-component.

#### Mogelijke UI-component:

- **ngx-explorer**: Een bestaande Angular package voor het tonen en beheren van bestanden in een file explorer.
  - NPM package: [ngx-explorer](https://www.npmjs.com/package/ngx-explorer)
  - Integratie in een Angular-component:

```typescript
import { HttpClient } from '@angular/common/http';
import { Component, OnInit } from '@angular/core';

interface FileEntry {
  Key: string;
  LastModified: string;
  Size: number;
}

@Component({
  selector: 'app-file-explorer',
  template: `
    <ngx-explorer [files]="files"></ngx-explorer>
  `
})
export class FileExplorerComponent implements OnInit {
  files: FileEntry[];

  constructor(private http: HttpClient) {}

  ngOnInit() {
    this.http.get<FileEntry[]>('/api/files').subscribe({
      next: data => this.files = data,
      error: err => console.error('Error loading files', err)
    });
  }
}
```

### Voordelen:

- Naadloze integratie met DigitalOcean Spaces waar ander services al draaien
- Bestaande Angular file explorer component (ngx-explorer)
- Mogelijkheid om rollen en rechten te implementeren via de backend, bijvoorbeeld door mappenstructuur te maken per rol. Andere manieren ook mogelijk

---

## Optie 2: Google Drive API

### Backend Implementatie

Google Drive biedt een API waarmee bestanden kunnen worden beheerd en opgehaald. Dit vereist authenticatie via OAuth en het gebruik van de Google Drive Node.js client.

#### Codevoorbeeld (Node.js API):

```javascript
const { google } = require('googleapis');
const drive = google.drive({ version: 'v3', auth: oauth2Client });

app.get('/api/files', async (req, res) => {
    try {
        const response = await drive.files.list({
            fields: 'files(id, name, mimeType)'
        });
        res.json(response.data.files);
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});
```

### Frontend Implementatie

- **ngx-filemanager**: Een package dat ondersteuning biedt voor Google Drive.
  - NPM package: [ngx-filemanager](https://www.npmjs.com/package/ngx-filemanager)

### Voordelen:

- Bekende en betrouwbare cloudopslag
- Web interface beschikbaar buiten de eigen site (kan een voordeel zijn, makkelijker beheer voor de webmaster)
- Sterke beveiliging met Google authenticatie (niet persé een voordeel aangezien oauth toch niet gebruikt wordt is het moeilijker integreerbaar)

### Nadelen:

- OAuth-authenticatie vereist voor gebruikers wat niet handig is aangezien de authenticatie nu zelf op de backend gebeurt zonder provider
- Beperkte API mogelijkheden

---

## Optie 3: Nextcloud

### Backend Implementatie

Nextcloud biedt een WebDAV API waarmee bestanden kunnen worden opgehaald en beheerd.

#### Codevoorbeeld (Node.js API):

```javascript
const axios = require('axios');
const nextcloudUrl = 'https://nextcloud.example.com/remote.php/dav/files/username/';
const auth = { username: 'your-username', password: 'your-password' };

app.get('/api/files', async (req, res) => {
    try {
        const response = await axios.get(nextcloudUrl, { auth });
        res.json(response.data);
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});
```

### Frontend Implementatie

- **ngx-webdav-client**: Ondersteunt WebDAV en kan gebruikt worden voor Nextcloud.
  - NPM package: [ngx-webdav-client](https://www.npmjs.com/package/webdav-client)

### Voordelen:

- Volledig self-hosted oplossing, geen externe cloud nodig
- Kan geïntegreerd worden in de bestaande DigitalOcean infrastructuur
- Ondersteuning voor rollen en gebruikersbeheer.

### Nadelen:

- Vereist eigen serverbeheer
- Extra configuratie nodig om WebDAV correct in te stellen
- Veel meer werk om te onderhouden

---

## Conclusie

Hieronder een samenvatting van de mogelijke opties:

| Optie                   | Cloudprovider | Integratiemethode                  | Voordelen                            | Nadelen                                            |
| ----------------------- | ------------- | ---------------------------------- | ------------------------------------ | -------------------------------------------------- |
| **DigitalOcean Spaces** | DigitalOcean  | AWS-SDK + ngx-explorer             | Naadloze integratie, goedkope opslag | Minder ingebouwde functionaliteit dan Google Drive |
| **Google Drive**        | Google        | Google Drive API + ngx-filemanager | Bekend platform, sterke beveiliging  | OAuth vereist, API-limieten                        |
| **Nextcloud**           | Self-hosted   | WebDAV + ngx-webdav-client         | Volledige controle, self-hosted      | Extra serverbeheer nodig                           |

Voor AZL Lebbeke lijkt DigitalOcean Spaces de meest logische keuze aangezien de bestaande services, buiten de emailservice, in digital ocean draaien. Indien self hosted de voorkeur krijgt kan ook een nextcloud omgeving worden opgezet die runt in digital ocean, zorgt wel voor extra werk en onderhoud. 
