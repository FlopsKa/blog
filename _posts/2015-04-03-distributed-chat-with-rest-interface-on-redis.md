---
layout: post
title: Redis Cluster als Basis für einen verteilten Chat
---

- [Anforderungen](#anforderungen)
- [Software Stack](#stack)
- [Software Architektur](#softwarearchitektur)
- [Hardware Architektur](#hardwareachitektur)
- [Ausblick](#ausblick)

### Anforderungen <a id="anforderungen" />###

- mehrere Clients pro Benutzer
- Integrationstest des Servers als Slim Client
- Verlauf abrufbar
- verschiedene Server mit Authorisierung (falls einzelne Komponenten ausfallen immer noch lauffähig)
- Authorisierung an jedem Server möglich
- Persistierung der Nachrichten auf allen Servern/Nodes
- kein Datenschutz
- Clients pullen, Server pushen
- Nachrichten so schnell wie möglich auf alle anderen Server
- alle schreiben an alle (ein "chatroom", broadcast)
- Login mit E-Mail und Password

### Software Stack <a id="stack" />###

- [redis](http://redis.io/) wird als zentrale Datenbank für die Benutzer und die gesendeten Nachrichten verwendet. Mit redis 3.0.0 ist clustering und somit der verteilte Einsatz in den _stable_-Branch gekommen.
- [jedis](https://github.com/xetorthio/jedis) ist das empfohlene Java Interface für die Kommunikation mit redis.
- [restlet](http://restlet.com/) stellt ein Framework für RESTful Anwendungen zur Verfügung. Neben dem Server können auch Clients realisiert werden. Diese Funktionalität wird in den Integrationstests verwendet.
- [junit](http://junit.org) 
- [hamcrest](http://hamcrest.org/JavaHamcrest/)
- [lombok](http://projectlombok.org/)

### Software Architektur <a id="softwarearchitektur" />###

Mit restlet werden zwei Endpunkte für Anfragen zur Verfügung gestellt:

~~~bash
http://localhost:8182/user/
http://localhost:8182/message/
~~~

#### /user/ ####

~~~java
public interface RESTUser {

	@Put("json")
	Representation login(ChatUser chatUser);

	@Post("json")
	ChatUser createUser(ChatUser chatUser);

}
~~~

Mit der _createUser(ChatUser chatUser)_ Methode kann ein neuer Benutzer am System registriert werden. Die E-Mail Adresse ist dabei eindeutig, d.h. es kann pro E-Mail Adresse nur einen Benutzer im System geben. Wird der Nutzer angelegt enthält der Rückgabewert die generierte ID und den Authentifizierungsschlüssel des neuen Benutzers.

Die _login(ChatUser chatUser)_ Methode wird zum Einloggen eines Benutzers verwendet. Der Benutzername und das Passwort werden überprüft und bei einer Übereinstimmung wird der entsprechende Authentifizierungsschlüssel als Cookie zurückgegeben. Bei einem Fehler sendet der Server ein HTTP Status _401_.

#### /message/ ####

~~~java
public interface RESTMessage {

	@Post("json")
	Message sendMessage(Message message);

	@Get("json")
	List<Message> readMessages();

}
~~~

Das Senden einer Nachricht setzt voraus, dass der Benutzer sich vorher authorisiert hat. Das bedeutet, es muss neben der Nachricht auch ein Cookie mit dem Authentifizierungsschlüssel gesendet werden. War das Senden der Nachricht mit _sendMessage(Message message)_ erfolgreich enthält die zurückgegebene Nachricht den _timestamp_, bei dem die Nachricht am Server eingegangen ist.

Zum Lesen von Nachrichten ist keine Authentifizierung erforderlich. _readMessages_ liefert alle auf dem Server gespeicherten Nachrichten zurück.

### Hardware Architektur <a id="hardwarearchitektur" />###
<img src="{{ site.url }}/assets/chat_architecture.png" alt="Die Hardware Architektur" style="width: 300px;"/>

Der Chat verwendet die klassische Architektur für Webapplikationen: Die Anfragen der Nutzer werden von einem Load Balancer entgegen genommen und auf die Web Server verteilt. Soll eine größere Last verarbeitet werden, können weitere Web Server hinzugefügt werden. Die Authentifizierungsinformationen für Benutzer sind clientseitig in Cookies und serverseitig in der Datenbank gespeichert. Somit hält die Webapplikation keinen State: Fällt ein Server aus routet der Load Balancer die Anfragen an einen anderen Server und der Benutzer bemerkt nichts vom Ausfall eines Servers.

redis cluster hat einen Failover Mechanismus eingebaut: Beim Ausfall eines Masters wird automatisch ein bestehender Slave zum Master. In der verwendeten Konfiguration können also im Besten Fall bis zu drei Datenbankserver ausfallen.

### Ausblick <a id="ausblick" />###

In der oben beschriebenen Konfiguration ist der Load Balancer ein Single Point of Failure. Eine Möglichkeit auf den Ausfall des Load Balancers zu reagieren ist der Einsatz von sogenannten _Elastic IP Addresses_ (siehe [Elastic IP Addresses bei AWS](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/elastic-ip-addresses-eip.html)). Fällt der Load Balancer aus kann seine IP zeitnah einem parallel laufenden Load Balancer zugewiesen werden.
