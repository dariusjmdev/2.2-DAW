# Configuración de un Host Virtual en Apache: taskflow.local

**Autor:** Darius Joanito Marinescu

---

## 1. Objetivo

El objetivo de esta práctica consiste en crear un host virtual independiente en Apache denominado `taskflow.local`, configurando un directorio propio para servir contenido web, habilitando registros independientes y garantizando el aislamiento respecto al sitio web por defecto.

---

## 2. ¿Qué es un VirtualHost?

Un VirtualHost es una configuración de Apache que permite alojar varios sitios web en un mismo servidor utilizando diferentes nombres de dominio.

Apache decide qué sitio servir utilizando principalmente el valor especificado en la directiva:

```apache
ServerName
```

Cuando un navegador solicita una URL, Apache compara el nombre recibido con los VirtualHost configurados y selecciona el sitio correspondiente.

### Casos de uso habituales

- Hospedar múltiples sitios web en un único servidor.
- Separar entornos de desarrollo y producción.
- Gestionar dominios independientes.
- Organizar aplicaciones diferentes bajo una misma infraestructura.

---

## 3. Preparación del entorno

Se partió de un servidor Ubuntu Server 22.04 LTS con Apache instalado y configurado.

Los módulos necesarios ya se encontraban habilitados desde la práctica anterior:

```text
proxy
proxy_http
ssl
rewrite
headers
```

---

## 4. Creación del contenido web

Se creó un directorio específico para el sitio web:

```bash
sudo mkdir -p /var/www/taskflow
```

Posteriormente se añadió una página HTML utilizada como contenido inicial del sitio.

Ubicación:

```text
/var/www/taskflow/index.html
```

---

## 5. Configuración del VirtualHost

Se creó el fichero:

```text
/etc/apache2/sites-available/taskflow.conf
```

Configuración utilizada:

```apache
<VirtualHost *:80>

    ServerName taskflow.local

    DocumentRoot /var/www/taskflow

    ErrorLog ${APACHE_LOG_DIR}/taskflow-error.log
    CustomLog ${APACHE_LOG_DIR}/taskflow-access.log combined

    <Directory /var/www/taskflow>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>

</VirtualHost>
```

<img width="940" height="508" alt="Captura de pantalla 2026-11-11 183706" src="https://github.com/user-attachments/assets/5644b2c5-8fa2-4aea-bc35-838332230758" />


---

## 6. Activación del sitio

Se activó el nuevo VirtualHost:

```bash
sudo a2ensite taskflow.conf
```

Para evitar interferencias se deshabilitó el sitio por defecto:

```bash
sudo a2dissite 000-default.conf
```

Esta acción garantiza el aislamiento completo entre sitios.

---

## 7. Asociación del nombre de dominio

Para resolver el nombre localmente se añadió una entrada al archivo hosts del sistema Windows:

```text
192.168.1.152 taskflow.local
```

Archivo modificado:

```text
C:\Windows\System32\drivers\etc\hosts
```

Con esta configuración las peticiones realizadas a:

```text
http://taskflow.local
```

se envían al servidor Ubuntu.

---

## 8. Validación de la configuración

Antes de reiniciar Apache se verificó la sintaxis:

```bash
sudo apache2ctl configtest
```

Resultado:

```text
Syntax OK
```

<img width="917" height="112" alt="Captura de pantalla 2026-11-11 171720" src="https://github.com/user-attachments/assets/b3c89e5f-9af7-4f76-a3bb-8a69d56175a0" />


---

## 9. Reinicio y comprobación del servicio

Después de validar la configuración se reinició Apache:

```bash
sudo systemctl restart apache2
```

Verificación:

```bash
sudo systemctl status apache2
```

Resultado esperado:

```text
active (running)
```

<img width="910" height="87" alt="Captura de pantalla 2026-11-11 171643" src="https://github.com/user-attachments/assets/221f29d4-6845-413f-9d51-93a97881d220" />


---

## 10. Verificación de VirtualHost

Para comprobar la configuración de hosts virtuales se ejecutó:

```bash
apache2ctl -S
```

Apache identificó correctamente el VirtualHost:

```text
taskflow.local
```

<img width="946" height="501" alt="Captura de pantalla 2026-11-11 183508" src="https://github.com/user-attachments/assets/a3cca677-2ad3-41d2-bdf2-e5b54319705d" />


---

## 11. Comprobación funcional

Se accedió desde el navegador a:

```text
http://taskflow.local
```

El sitio cargó correctamente mostrando el contenido definido en el archivo `index.html`.

Esto confirma:

- ServerName correcto.
- DocumentRoot correcto.
- Resolución DNS local correcta.
- Funcionamiento del VirtualHost.

<img width="955" height="967" alt="Captura de pantalla 2026-11-11 183752" src="https://github.com/user-attachments/assets/5421c02b-d5d9-4876-88bc-cbb4545aa44a" />


---

## 12. Configuración de logs independientes

Se configuraron archivos de log específicos para el sitio:

```apache
ErrorLog ${APACHE_LOG_DIR}/taskflow-error.log
CustomLog ${APACHE_LOG_DIR}/taskflow-access.log combined
```

Verificación:

```bash
ls -l /var/log/apache2/
```

Resultado esperado:

```text
taskflow-access.log
taskflow-error.log
```

<img width="936" height="225" alt="Captura de pantalla 2026-11-11 183546" src="https://github.com/user-attachments/assets/58b87ea7-59c1-411c-8da2-7c234d2a4d57" />

---

## 13. Generación de registros

Tras acceder al sitio desde el navegador se verificó que el VirtualHost genera sus propios registros:

```bash
sudo tail /var/log/apache2/taskflow-access.log
```

Resultado esperado:

```text
GET / HTTP/1.1
```

Esto demuestra que el tráfico del sitio queda separado del resto.

<img width="922" height="640" alt="Captura de pantalla 2026-11-11 183653" src="https://github.com/user-attachments/assets/e2032263-c723-44d7-b59d-6a17d2a696e6" />

---

## 14. Aislamiento de sitios

El sitio por defecto fue desactivado mediante:

```bash
sudo a2dissite 000-default.conf
```

Como resultado:

- `taskflow.local` sirve exclusivamente su propio contenido.
- El contenido de otros sitios no interfiere.
- Los registros permanecen separados.
- Apache selecciona correctamente el VirtualHost correspondiente.

---

## 15. Añadir nuevos sitios en el futuro

Para crear otro sitio bastaría con:

1. Crear un nuevo directorio web.
2. Crear un nuevo fichero en:

```text
/etc/apache2/sites-available/
```

3. Configurar un nuevo `ServerName`.
4. Activar el sitio con:

```bash
sudo a2ensite nombre-sitio.conf
```

5. Reiniciar Apache.

---
