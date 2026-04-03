# Serverkonfiguration

Mit dem Standard-Docker-Image kann die gesamte erforderliche Konfiguration über den Browser vorgenommen werden. Je nach Bereitstellung kann es jedoch notwendig sein, die Serverkonfiguration anzupassen.

Diese Seite listet alle Methoden auf, um die Konfiguration zu ändern, sowie alle vorhandenen Konfigurationsoptionen.


## Konfigurationsdatei vs. Umgebungsvariablen

Für die Einstellungen können Sie entweder eine Konfigurationsdatei oder Umgebungsvariablen verwenden.

Wenn Sie das [Docker Compose-basierte Setup](deployment.md) verwenden, können Sie eine Konfigurationsdatei einfügen, indem Sie den folgenden Listeneintrag unter dem Schlüssel `volumes:` im Block `grampsweb:` hinzufügen:

```yaml
      - /path/to/config.cfg:/app/config/config.cfg
```
wobei `/path/to/config.cfg` der Pfad zur Konfigurationsdatei im Dateisystem Ihres Servers ist (die rechte Seite bezieht sich auf den Pfad im Container und darf nicht geändert werden).

Bei der Verwendung von Umgebungsvariablen,

- prefixen Sie jeden Einstellungsnamen mit `GRAMPSWEB_`, um den Namen der Umgebungsvariable zu erhalten
- Verwenden Sie doppelte Unterstriche für verschachtelte Dictionary-Einstellungen, z. B. wird `GRAMPSWEB_THUMBNAIL_CACHE_CONFIG__CACHE_DEFAULT_TIMEOUT` den Wert der Konfigurationsoption `THUMBNAIL_CACHE_CONFIG['CACHE_DEFAULT_TIMEOUT']` setzen

Beachten Sie, dass Konfigurationsoptionen, die über die Umgebung festgelegt werden, Vorrang vor denjenigen in der Konfigurationsdatei haben. Wenn beide vorhanden sind, "gewinnt" die Umgebungsvariable.

## Vorhandene Konfigurationseinstellungen
Die folgenden Konfigurationsoptionen sind vorhanden.

### Erforderliche Einstellungen

Schlüssel | Beschreibung
----|-------------
`TREE` | Der Name der zu verwendenden Familienstammbaum-Datenbank. Zeigen Sie verfügbare Bäume mit `gramps -l` an. Wenn ein Baum mit diesem Namen nicht existiert, wird ein neuer leerer Baum erstellt.
`SECRET_KEY` | Der geheime Schlüssel für Flask. Der Schlüssel darf nicht öffentlich geteilt werden. Eine Änderung macht alle Zugriffstoken ungültig.
`USER_DB_URI` | Die Datenbank-URL der Benutzerdatenbank. Jede URL, die mit SQLAlchemy kompatibel ist, ist erlaubt.

!!! info
    Sie können einen sicheren geheimen Schlüssel z. B. mit dem Befehl generieren

    ```
    python3 -c "import secrets;print(secrets.token_urlsafe(32))"
    ```

### Optionale Einstellungen

Schlüssel | Beschreibung
----|-------------
`MEDIA_BASE_DIR` | Pfad, der als Basisverzeichnis für Mediendateien verwendet werden soll, und das Basisverzeichnis für Medien in Gramps überschreibt. Bei Verwendung von [S3](s3.md) muss es die Form `s3://<bucket_name>` haben.
`TREE_ID` | Der Verzeichnisname der zu verwendenden Familienstammbaum-Datenbank im Einzelbaum-Modus (wenn `TREE` nicht auf `MULTI` gesetzt ist). Wenn gesetzt, identifiziert der Server den Baum anhand seines Verzeichnisnamens anstelle seines Anzeigenamens, was robuster gegenüber Umbenennungen ist. Erforderlich, wenn Sie den Baum über die API umbenennen möchten. Der Verzeichnisname kann über `GET /api/trees/-` (das Feld `id`) gefunden werden.
`SEARCH_INDEX_DB_URI` | Datenbank-URL für den Suchindex. Nur `sqlite` oder `postgresql` sind als Backends erlaubt. Standardmäßig auf `sqlite:///indexdir/search_index.db` gesetzt, wodurch eine SQLite-Datei im Ordner `indexdir` relativ zu dem Pfad erstellt wird, von dem das Skript ausgeführt wird.
`STATIC_PATH` | Pfad, um statische Dateien bereitzustellen (z. B. ein statisches Web-Frontend).
`BASE_URL` | Basis-URL, unter der die API erreichbar ist (z. B. `https://mygramps.mydomain.com/`). Dies ist notwendig, um z. B. korrekte Links zum Zurücksetzen von Passwörtern zu erstellen.
`CORS_ORIGINS` | Ursprünge, von denen CORS-Anfragen erlaubt sind. Standardmäßig sind alle nicht erlaubt. Verwenden Sie `"*"`, um Anfragen von jeder Domain zuzulassen.
`EMAIL_HOST` | SMTP-Server-Host (z. B. zum Senden von E-Mails zum Zurücksetzen von Passwörtern).
`EMAIL_PORT` | SMTP-Server-Port. Standardmäßig auf 465.
`EMAIL_HOST_USER` | SMTP-Server-Benutzername.
`EMAIL_HOST_PASSWORD` | SMTP-Server-Passwort.
`EMAIL_USE_TLS` | **Veraltet** (verwenden Sie stattdessen `EMAIL_USE_SSL` oder `EMAIL_USE_STARTTLS`). Boolean, ob TLS zum Senden von E-Mails verwendet werden soll. Standardmäßig auf `True`. Bei Verwendung von STARTTLS setzen Sie dies auf `False` und verwenden einen anderen Port als 25.
`EMAIL_USE_SSL` | Boolean, ob implizites SSL/TLS für SMTP verwendet werden soll (v3.6.0+). Standardmäßig auf `True`, wenn `EMAIL_USE_TLS` nicht explizit gesetzt ist. Typischerweise mit Port 465 verwendet.
`EMAIL_USE_STARTTLS` | Boolean, ob explizites STARTTLS für SMTP verwendet werden soll (v3.6.0+). Standardmäßig auf `False`. Typischerweise mit Port 587 oder 25 verwendet.
`DEFAULT_FROM_EMAIL` | "Von"-Adresse für automatisierte E-Mails.
`THUMBNAIL_CACHE_CONFIG` | Dictionary mit Einstellungen für den Thumbnail-Cache. Siehe [Flask-Caching](https://flask-caching.readthedocs.io/en/latest/) für mögliche Einstellungen.
`REQUEST_CACHE_CONFIG` | Dictionary mit Einstellungen für den Anfrage-Cache. Siehe [Flask-Caching](https://flask-caching.readthedocs.io/en/latest/) für mögliche Einstellungen.
`PERSISTENT_CACHE_CONFIG` | Dictionary mit Einstellungen für den persistenten Cache, der z. B. für Telemetrie verwendet wird. Siehe [Flask-Caching](https://flask-caching.readthedocs.io/en/latest/) für mögliche Einstellungen.
`CELERY_CONFIG` | Einstellungen für die Celery-Hintergrundaufgabenwarteschlange. Siehe [Celery](https://docs.celeryq.dev/en/stable/userguide/configuration.html) für mögliche Einstellungen.
`REPORT_DIR` | Temporäres Verzeichnis, in dem die Ausgaben von ausgeführten Gramps-Berichten gespeichert werden.
`EXPORT_DIR` | Temporäres Verzeichnis, in dem die Ausgaben des Exports der Gramps-Datenbank gespeichert werden.
`REGISTRATION_DISABLED` | Wenn `True`, neue Benutzerregistrierungen nicht zulassen (Standard `False`).
`DISABLE_TELEMETRY` | Wenn `True`, Statistiken-Telemetrie deaktivieren (Standard `False`). Siehe [Telemetrie](telemetry.md) für Details.
`PILLOW_MAX_IMAGE_PIXELS` | Setzt den Parameter PIL.Image.MAX_IMAGE_PIXELS, der angibt, wie viele Pixel das verarbeitete Bild enthalten kann. Siehe [Docs](https://pillow.readthedocs.io/en/stable/reference/Image.html#PIL.Image.MAX_IMAGE_PIXELS) für Details.


!!! info
    Bei der Verwendung von Umgebungsvariablen für die Konfiguration müssen boolesche Optionen wie `EMAIL_USE_TLS` entweder der String `true` oder `false` (groß-/kleinschreibungssensitiv!) sein.


### Einstellungen nur für PostgreSQL-Backend-Datenbank

Dies ist erforderlich, wenn Sie Ihre Gramps-Datenbank so konfiguriert haben, dass sie mit dem [PostgreSQL-Addon](https://gramps-project.org/wiki/index.php/Addon:PostgreSQL) funktioniert.

Schlüssel | Beschreibung
----|-------------
`POSTGRES_USER` | Der Benutzername für die Datenbankverbindung.
`POSTGRES_PASSWORD` | Das Passwort für den Datenbankbenutzer.


### Einstellungen, die für das Hosting mehrerer Bäume relevant sind

Die folgenden Einstellungen sind relevant, wenn Sie [mehrere Bäume hosten](multi-tree.md).


Schlüssel | Beschreibung
----|-------------
`MEDIA_PREFIX_TREE` | Boolean, ob ein separates Unterverzeichnis für die Mediendateien jedes Baumes verwendet werden soll oder nicht. Standardmäßig auf `False`, aber es wird dringend empfohlen, `True` in einem Multi-Baum-Setup zu verwenden.
`NEW_DB_BACKEND` | Das Datenbank-Backend, das für neu erstellte Familienstammbäume verwendet werden soll. Muss eines von `sqlite`, `postgresql` oder `sharedpostgresql` sein. Standardmäßig auf `sqlite`.
`POSTGRES_HOST` | Der Hostname des PostgreSQL-Servers, der zum Erstellen neuer Bäume verwendet wird, wenn ein Multi-Baum-Setup mit dem SharedPostgreSQL-Backend verwendet wird.
`POSTGRES_PORT` | Der Port des PostgreSQL-Servers, der zum Erstellen neuer Bäume verwendet wird, wenn ein Multi-Baum-Setup mit dem SharedPostgreSQL-Backend verwendet wird.


### Einstellungen für OIDC-Authentifizierung

Diese Einstellungen sind erforderlich, wenn Sie die OpenID Connect (OIDC) Authentifizierung mit externen Anbietern verwenden möchten. Für detaillierte Installationsanleitungen und Beispiele siehe [OIDC-Authentifizierung](oidc.md).

Schlüssel | Beschreibung
----|-------------
`OIDC_ENABLED` | Boolean, ob die OIDC-Authentifizierung aktiviert werden soll. Standardmäßig auf `False`.
`OIDC_ISSUER` | OIDC-Anbieter-Aussteller-URL (für benutzerdefinierte OIDC-Anbieter).
`OIDC_CLIENT_ID` | OAuth-Client-ID (für benutzerdefinierte OIDC-Anbieter).
`OIDC_CLIENT_SECRET` | OAuth-Client-Geheimnis (für benutzerdefinierte OIDC-Anbieter).
`OIDC_NAME` | Benutzerdefinierter Anzeigename für den Anbieter. Standardmäßig auf "OIDC".
`OIDC_SCOPES` | OAuth-Scopes. Standardmäßig auf "openid email profile".
`OIDC_USERNAME_CLAIM` | Der Anspruch, der für den Benutzernamen verwendet werden soll. Standardmäßig auf "preferred_username".
`OIDC_OPENID_CONFIG_URL` | Optional: URL zum OpenID Connect-Konfigurationsendpunkt (wenn nicht der Standard `/.well-known/openid-configuration` verwendet wird).
`OIDC_DISABLE_LOCAL_AUTH` | Boolean, ob die lokale Benutzername/Passwort-Authentifizierung deaktiviert werden soll. Standardmäßig auf `False`.
`OIDC_AUTO_REDIRECT` | Boolean, ob automatisch zu OIDC umgeleitet werden soll, wenn nur ein Anbieter konfiguriert ist. Standardmäßig auf `False`.

#### Eingebaute OIDC-Anbieter

Für integrierte Anbieter (Google, Microsoft, GitHub) verwenden Sie diese Einstellungen:

Schlüssel | Beschreibung
----|-------------
`OIDC_GOOGLE_CLIENT_ID` | Client-ID für Google OAuth.
`OIDC_GOOGLE_CLIENT_SECRET` | Client-Geheimnis für Google OAuth.
`OIDC_MICROSOFT_CLIENT_ID` | Client-ID für Microsoft OAuth.
`OIDC_MICROSOFT_CLIENT_SECRET` | Client-Geheimnis für Microsoft OAuth.
`OIDC_GITHUB_CLIENT_ID` | Client-ID für GitHub OAuth.
`OIDC_GITHUB_CLIENT_SECRET` | Client-Geheimnis für GitHub OAuth.

#### OIDC-Rollen-Zuordnung

Diese Einstellungen ermöglichen es Ihnen, OIDC-Gruppen/Rollen von Ihrem Identitätsanbieter auf Gramps Web-Benutzerrollen abzubilden:

Schlüssel | Beschreibung
----|-------------
`OIDC_ROLE_CLAIM` | Der Anspruchsname im OIDC-Token, der die Gruppen/Rollen des Benutzers enthält. Standardmäßig auf "groups".
`OIDC_GROUP_ADMIN` | Der Gruppen-/Rollename von Ihrem OIDC-Anbieter, der der Gramps "Admin"-Rolle zugeordnet ist.
`OIDC_GROUP_OWNER` | Der Gruppen-/Rollename von Ihrem OIDC-Anbieter, der der Gramps "Owner"-Rolle zugeordnet ist.
`OIDC_GROUP_EDITOR` | Der Gruppen-/Rollename von Ihrem OIDC-Anbieter, der der Gramps "Editor"-Rolle zugeordnet ist.
`OIDC_GROUP_CONTRIBUTOR` | Der Gruppen-/Rollename von Ihrem OIDC-Anbieter, der der Gramps "Contributor"-Rolle zugeordnet ist.
`OIDC_GROUP_MEMBER` | Der Gruppen-/Rollename von Ihrem OIDC-Anbieter, der der Gramps "Member"-Rolle zugeordnet ist.
`OIDC_GROUP_GUEST` | Der Gruppen-/Rollename von Ihrem OIDC-Anbieter, der der Gramps "Guest"-Rolle zugeordnet ist.

### Einstellungen nur für KI-Funktionen

Diese Einstellungen sind erforderlich, wenn Sie KI-gestützte Funktionen wie Chat oder semantische Suche verwenden möchten.

Schlüssel | Beschreibung
----|-------------
`LLM_BASE_URL` | Basis-URL für die OpenAI-kompatible Chat-API. Standardmäßig auf `None`, was die OpenAI-API verwendet.
`LLM_MODEL` | Das Modell, das für die OpenAI-kompatible Chat-API verwendet werden soll. Wenn nicht gesetzt (Standard), ist der Chat deaktiviert. Ab v3.6.0 verwendet der KI-Assistent Pydantic AI mit Tool-Calling-Funktionen.
`VECTOR_EMBEDDING_MODEL` | Das [Sentence Transformers](https://sbert.net/) Modell, das für semantische Suchvektor-Einbettungen verwendet werden soll. Wenn nicht gesetzt (Standard), sind semantische Suche und Chat deaktiviert.
`LLM_MAX_CONTEXT_LENGTH` | Zeichenlimit für den Familienstammbaum-Kontext, der dem LLM bereitgestellt wird. Standardmäßig auf 50000.
`LLM_SYSTEM_PROMPT` | Benutzerdefinierter System-Prompt für den LLM-Chat-Assistenten (v3.6.0+). Wenn nicht gesetzt, wird der standardmäßige, genealogisch optimierte Prompt verwendet.


## Beispiel-Konfigurationsdatei

Eine minimale Konfigurationsdatei für die Produktion könnte so aussehen:
```python
TREE="Mein Familienstammbaum"
BASE_URL="https://meinstammbaum.beispiel.com"
SECRET_KEY="..."  # Ihr geheimer Schlüssel
USER_DB_URI="sqlite:////path/to/users.sqlite"
EMAIL_HOST="mail.beispiel.com"
EMAIL_PORT=465
EMAIL_USE_SSL=True  # Verwenden Sie implizites SSL für Port 465
EMAIL_HOST_USER="gramps@beispiel.com"
EMAIL_HOST_PASSWORD="..." # Ihr SMTP-Passwort
DEFAULT_FROM_EMAIL="gramps@beispiel.com"
