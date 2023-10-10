# Usar gmail para enviar correos a través de Wazuh
Documentacion oficial de wazuh para servidores de correo con autenticacion https://documentation.wazuh.com/current/user-manual/manager/manual-email-report/smtp-authentication.html

## Paso a paso con GMAIL

### 1.- Crear cuenta de Gmail

https://accounts.google.com/signup 

### 2.- Cree una 'contraseña de aplicación' de Google para postfix.

Ir a https://accounts.google.com and log-in con tu usario y contraseña de gmail.
Ir a Security >> 2-Steps verification >> App Passwords (at the bottom)
Cree una nueva aplicación llamada 'postfix' y copie la contraseña que se genera automáticamente.
Obtendrás un pass de 16 caracteres con espacios, como este: 'xxxx xxxx xxxx xxxx'

### 3.- instalar postfix (CentOS - RedHat-based OS):
yum update && yum install postfix mailx cyrus-sasl cyrus-sasl-plain

### 4.- Configurar /etc/postfix/main.cf, agregar al final:
relayhost = [smtp.gmail.com]:587
smtp_sasl_auth_enable = yes
smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd
smtp_sasl_security_options = noanonymous
smtp_tls_CAfile = /etc/ssl/certs/ca-bundle.crt
smtp_use_tls = yes
inet_protocols = ipv4 #This avoids IPv6 connection with a 'Network unreachable' message.

### 5.- Configurar /etc/postfix/sasl_passwd
echo '[smtp.gmail.com]:587 <USER>@wazuh.com:xxxx xxxx xxxx xxxx' > /etc/postfix/sasl_passwd #PASSWORD YOU'VE GOT FROM GOOGLE 'APP PASSWORDS'

### 6.- plicar el cambio, proteger los archivos y reiniciar el servicio:
postmap /etc/postfix/sasl_passwd
chmod 400 /etc/postfix/sasl_passwd
chown root:root /etc/postfix/sasl_passwd /etc/postfix/sasl_passwd.db
chmod 0600 /etc/postfix/sasl_passwd /etc/postfix/sasl_passwd.db
systemctl restart postfix

### 7.- ¡Prueba el servicio postfix!:
echo "Test mail from postfix" | mail -s "Test Postfix" -r "<USER>@gmail.com" <USER>@gmail.com
Puedes verificar los registros en  /var/log/maillog
Sep 18 18:27:55 wazuh-server postfix/smtp[9832]: 01F6221724B0: to=<XXXXX@gmail.com>, relay=smtp.gmail.com[XX.XX.XX.XX]:587, delay=443, delays=439/0.02/2.7/1.1, dsn=2.0.0, status=sent (250 2.0.0 OK  1695061674 e11-20020aca130b000000b003a3bsm4284674oii.40 - gsmtp)
Sep 18 18:27:55 wazuh-server postfix/qmgr[9564]: 01F6221724B0: removed
