# Videoguides til at komme i gang med Webinterface :rocket:

Indhold
- [1. Kom i gang med docker-compose og PyCharm](#1-Docker-compose-og-PyCharm)
- [2. Kom i gang med Django](#2-Django)
- [3. Testcases med Selenium IDE](#3-Testcases-med-Selenium-IDE)

## 1. Docker-compose og PyCharm
### Få Webinterface op at køre
I denne video henter vi koden, gør Docker og docker-compose klar, og starter Webinterface op.

[Kom i gang 1 - Få startet Webinterface op (7:53)](https://youtu.be/o60uKeKV2kc).

Bonus: [docker-compose pull i stedet for build](https://youtu.be/WdQuxeNPE-k).

### Få en bedre oplevelse med at kode
I denne video starter vi PyCharm op, og installerer noget ekstra, så oplevelsen med at skrive kode bliver bedre.

[Kom i gang 2 - Installér Pip3 og pipenv (7:21)](https://youtu.be/nAIftKo4zL4)

### En diskussion om hvorfor vi installerer pipenv
Efter denne video ved du, hvordan vi kan anvende alle de fede Python libraries, selvom de skal afvikles inde i Docker-containers.

[Kom i gang 3 - lidt info om pipenv (3:39)](https://youtu.be/oOBcc2cMtIM).

### Så er PyCharm klar
Lidt demo med code completion og en superhurtig snak om kodestandard.

[Kom i gang 4 - PyCharm nu med code completion (2:56)](https://youtu.be/zQ1FhAhG5r4).

God fornøjelse! :beers:


## 2. Django 

### Request-Response
Se hvordan Django håndterer Request-Response mellem browser og Webinterface

[Django flow 1 (11:26)](https://youtu.be/CsIFsD1pd9w)

### Ruter, View-funktioner og Templates i Django
Se hvordan du opretter en ny feature, den får en URL-rute, en View-funktion til at håndtere logik, og en Template til at præsentere svar i browseren.

[Django flow 2 (13:02)](https://youtu.be/1IgfWK_YF04)

### Formularer, GET og POST
Se hvordan du laver en ny formular og får den præsenteret til brugeren i browseren.

[Django flow 3 (18:04)](https://youtu.be/BECwC34fio4)

### Træk data ud af en formular
Se hvordan du får data ud af en formular, så du kan bruge den videre. Fx til at sende over MQTT eller bare vise i Template.

[Django flow 4 (9:19)](https://youtu.be/f2WxUdw3pgo)

### Benyttelse af data og videresendelse af flere variabler til en Template
Se hvordan al data egentlig er tekst, og at du skal caste den til andre typer for bruge den. Se også hvordan flere variable kan gemmes i en dict, inden Template-rendering.

[Django flow 5 (4:07)](https://youtu.be/zbdoZOneGik)


## 3. Testcases med Selenium IDE
Se her hvordan du afvikler tests af brugerinterface ved hjælp af Selenium IDE i browseren.

[Demo af Selenium til at afvikle tests](https://youtu.be/VvI-IKYkFOc).
