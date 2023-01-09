# SMTPSendOnly

**Vorraussetzungen:**
- 1. Ein kostenloser Account in https://app.mailjet.com/ - Hier gibt es die Möglichekeit 200 Mails am Tag kostenlos. Alle anderen Mails nach der 200. Mail kommen in die Warteschlange und werden dann am nächsten Tag weiter verarbeitet und verschickt. Oder 600 Mails als Limit innerhalb 3 Tagen und 6000 innerhlab einem Monat. Dii Ablaufzeit in der Warteschlange ist nur für 3 Tage gültig.
- 

- 2. Ein minimales Linuxsystem(in dieser Anleitung Ubuntu Server 22.04.1), bei dem Postfix installiert wird. Dieser Server muss eine Portfreigabe für 25 oder 587 haben.

- 3. Eine Domain oder Subdomain wie z. B. sub.main.example.com

- 4. Ein Webserver, der unter der Domain aufrufbar ist. Das ist wichtig, um die Maildomain in **Mailjet** validieren zu können.

#### Installation von Postfix:
``Linuxserver:$ sudo install postfix libsasl2-modules``
- * Bei der Paket-Konfiguration Internet-Site auswählen.
- * Als Nächstes Mailsystemname eingeben: sub.main.example.com

Nach der Postfix-Installation bearbeite die folgende Datei:

``Linuxserver:$ sudo nano /etc/postfix/main.cf``

Finde den folgenden Eintrag:

``relayhost = ``
Dort muss die Adresse der Mailjet-SMTP eingetragen werden.

Um diese Daten sehen zu können, gehe zu Kontoeinstellung in Mailjet:

- ![image](https://user-images.githubusercontent.com/99675262/211280984-a6189402-e275-4248-a528-39230d6b6fa1.png)


Und dann in **Sender & Domains** auf **SMTP und SEND API EInstellungen**:

- ![image](https://user-images.githubusercontent.com/99675262/211281862-d342cc85-0163-413c-9cd6-7166c6c16dda.png)

Und so sehen dann z. B. die Adresse der Mailjet-SMTP aus:

- ![image](https://user-images.githubusercontent.com/99675262/211279293-ecea705a-9ec0-4bfd-bc39-db0eac864766.png)

Und dann anschließend sollen die API-SCHLÜSSEL und GEHEIMER SCHLÜSSEL in Anmeldedaten(API und SMTP) erstellt werden!

- ![image](https://user-images.githubusercontent.com/99675262/211283057-f226ad8e-f30b-41a4-9bd3-baad93b30865.png)

Und weiter in der Postfix Konfigurationsdatei **main.cf** setze den Eintrag für relayhost "in-v3.mailjet.com:587 oder 25"

``relayhost = in-v3.mailjet.com:587``

Anschließend finde den Eintrag:

``mynetworks = ...`` Dort trägt ihr nun die IP-Adresse oder euer Subnet eures internen SMTP-Server.

Und dann füge die folgenden Zeilen am Ende der Postfix Konfigurationsdatei **main.cf** :

```
smtp_sasl_auth_enable = yes
smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd
smtp_sasl_security_options = noanonymous
smtp_tls_security_level = may
header_size_limit = 4096000
```

Danach erstelle eine Datei in /etc/postfix sasl_passwd und trage dort die Daten von eurem Mailjet-Konto ein!

```
Linuxserver:$ sudo touch /etc/postfix/sasl_passwd
Linuxserver:$ sudo echo in-v3.mailjet.com:587 api-key:secret-key
```

Nach der Konfiguration die Passwortdatei mappen und den Postfix Dienst neustarten!

```
Linuxserver:$ sudo postmap /etc/postfix/sasl_passwd
Linuxserver:$ sudo systemctl restart postfix
```

Der SMTP-Relay ist damit fertig konfiguriert. Damit aber von einer Absenderdomain @sub.main.example.com Mails gesendet werden können,
muss der Absenderdomain validiert werden.

Bei **Sender & Domains** in "Eine Absenderdomain oder –adresse hinzufügen!" die eigne Maildomain hinzufügen

![image](https://user-images.githubusercontent.com/99675262/211311779-c3665575-3a35-448c-b112-4d27c36dfadd.png)

![image](https://user-images.githubusercontent.com/99675262/211312267-7ee622aa-c43b-4aa0-8a38-abe87390e283.png)

![image](https://user-images.githubusercontent.com/99675262/211312728-0758ac20-155d-4af4-9f55-62815945a956.png)

Und danach muss die Domain durch einen Webserver validiert werden, sowie sie in den folgenden Bildern beschrieben. Klicke hierfür das Zahnrad an und wähle validieren.

![image](https://user-images.githubusercontent.com/99675262/211313022-eb6414b4-9f7f-434f-8594-b14c1d8d5a2f.png)

Die Validierung sollte über einen Webserver gemacht werden, wenn DSN-Einträge nicht möglich ist.

![image](https://user-images.githubusercontent.com/99675262/211313123-e9ab9341-e4e0-412f-b176-abe09548673b.png)

Sobald die Maildomain validiert ist, kann nun der Linuxserver als Relay für andere Mailserver innerhalb des Subnets verwendet werden.


### Testen von SMTP-Relay
**Installieren von mailutils**

```Linuxserver:$ sudo apt install mailutils``

**Einen Sendetest durchführen:**

``Linuxserver:$ echo "The body part of the email!" | mail -s "Subject of the email" -a "From: sender@sub.main.example.com" receiver@example.com``








