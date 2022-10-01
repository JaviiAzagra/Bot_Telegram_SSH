# Enviar mensaje a Telegram en cualquier inicio de sesión SSH
Creación de un script de shell Bash para enviar mensajes a telegram. Despues utilizaremos este script para enviar una notificación en cada inicio de sesion ssh en el servidor.

# Crear bot de telegram
Empezamos abriendo una conversación con BotFather para la creación de nuestro bot. Con el comando /newbot le diremos al bot que queremos crear un nuevo bot. A continuación nos dira que nombre le queremos poner y que username. Una vez terminemos de decirle todo lo anterior nos mostrara un mensaje con el Token del bot que hemos creado.

![image](https://user-images.githubusercontent.com/74573174/193409759-78c6b0c3-e285-442a-b942-085299730bf3.png)

# Crear canal
Creamos un canal y agregamos al bot como miembro. Por lo cual tu bot podra enviar mensajes al canal.

Para poder obtener la ID del canal solo tendemos que meternos a telegram desde el navegador y en la url del canal podremos ver el ID de dicho canal.

![image](https://user-images.githubusercontent.com/74573174/193409778-a375eddd-450d-4b75-816b-891a59875133.png)

# Script para enviar mensajes
Creamos un archivo llamado msg-telegram.sh (o el nombre que mas te guste).

`nano msg-telegram.sh`

Luego agregue el siguiente script a este archivo con el id del canal y el token del bot correspondientes.

```
#!/bin/bash

USERID="YOUR_ID" #Chat al que queremos enviar el mensaje.

KEY="YOUR_TOKEN" #API Key generada por BotFather.

TIMEOUT="10" #Timeout de la petición a la API.

URL="https://api.telegram.org/bot$KEY/sendMessage" #URL de la API para enviar mensajes.

LOG="envio_telegram_`date "+%d%m%Y"`.log" #Log de envío de mensajes.

SONIDO=0 #Cuando SONIDO es 0, suena notificación, si es 1, es silenciosa.

FECHA_EJEC="$(date "+%d %b %H:%M:%S")" #Generamos fecha y hora de ejecución.

#Condicional para enviar sonido o no con la notificación. Segundo parámetro.

if [[ $2 -eq 1 ]] ; then

        SONIDO=1

fi

TEXTO="<b>$FECHA_EJEC:</b>\n<pre>$1</pre>" #Texto a enviar. Fecha de ejecución y primer parámetro del script.

curl -s --max-time $TIMEOUT -d "parse_mode=HTML&disable_notification=$SONIDO&chat_id=$USERID&disable_web_page_preview=1&text=`echo -e "$TEXTO"`" $URL >> $LOG 2>&1 #Realizamos la peticion

"echo" >> $LOG #Introducimos línea nueva en el log.
```

Para ejecutar este script debemos agregarle permisos.

`chmod +x msg-telegram.sh`

Ahora puedes probarlo.

`./msg-telegram.sh "Hola Mundo"`

Para poder usar este script desde todas las ubicaiones debemos moverlo a /usr/bin/.

`sudo mv msg-telegram.sh /usr/bin/msg-telegram **`

El propietario de todos los archivos en /usr/bin es el usuario root.

`sudo chown root:root /usr/bin/msg-telegram`

Ahora puedes probarlo.

`msg-telegram "Hola mundo"`

# Enviar notificaciones al iniciar sesión SSH

Agregamos un nuevo scrip.

`nano login_ssh.sh`



Agregue el siguiente código al script.

```
#!/bin/bash

LOGIN_IP="$(echo $SSH_CONNECTION | cut -d " " -f 1)"

LOGIN_DATE="$(date +"%e %b %Y, %a %r")"

LOGIN_NAME="$(whoami)"

LOGIN_PUBLIC="$(wget -qO- ifconfig.co/ip)"

MESSAGE="NEW LOGIN TO SERVER"$'\n'"$LOGIN_NAME"$'\n'"$LOGIN_IP"$'\n'"$LOGIN_DATE"$'\n'"$LOGIN_PUBLIC"

msg-telegram "$MESSAGE"
```

Si lo ha creado en otra carpeta muevalo a /etc/profile.d/.

`mv login_ssh.sh /etc/profile.d/login_ssh.sh`


**AHORA VUELVA A INICIAR SESIÓN EN SU SERVIDOR Y COMPRUEBE QUE FUNCIONE!!!**
