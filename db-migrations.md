# Fælles håndtering af databasestruktur med migrationer :rocket:

Her er et par tips & tricks til at arbejde med databasen mens vi udvikler på projektet.
Skriv gerne til når/hvis du har flere tips & tricks.

# Indhold
- [Hvad er migrations?](#Hvad-er-migrations)
- [Django-kommandoer til migrations](#Django-kommandoer-til-migrations)
- [Migrations opstår når...](#Migrations-opstår-når)
- [Principper for versionsstyring af migrations](#Principper-for-versionsstyring-af-migrations)
- [Eksempel: Migrations og versionsstyring på udviklingsmaskine](#Eksempel-Migrations-og-versionsstyring-på-udviklingsmaskine)
- [Eksempel fortsat: Migrations og versionsstyring på produktionsserver](#Eksempel-fortsat-Migrations-og-versionsstyring-på-produktionsserver)
- [Kendte fejl og mulige fixes :ambulance:](#Kendte-fejl-og-mulige-fixes)
  * [Tøm database :toilet:](#Tøm-database)
  * [DuplicateTable: relation ... already exists](#DuplicateTable-relation-already-exists)
  * [Permission denied, eller lignende](#Permission-denied-eller-lignende)
  * [Permanent uløselige fejl](#Permanent-uløselige-fejl)
  * [Nuke it! :warning:](#I-tilfælde-af-at-intet-virker-for-databasen-ved-migrations-eller-server-setup-Nuke-it)
- [Load startdata (fixtures)](#Load-startdata-fixtures)
- [Kig i databasen](#Kig-i-databasen)
- [Udfordringer når vi nærmer os en endelig version](#Udfordringer-når-vi-nærmer-os-en-endelig-version)


## Hvad er migrations?
I Django er migrations de _.py_-filer, som  fortæller Django, hvordan den skal implementere ændringer i en SQL-database. Det er en slags opskrift.
Migrations laves automatisk _per app_. Så _demo-modul_ har sine egne migrations, der ligger i `demo_module/migrations`. Der kan dog være afhængigheder til andre apps (fx via ForeignKeys).
Man kan som udvikler så vælge at implementere ændringerne (udføre opskriften), dvs. `migrate`. Det kan være både frem til nye versioner eller tilbage til ældre versioner.

Migrations er nummererede og lineære/sekventielle. Fx er migration _0007...py_ afhængig af _0006...py_, osv.
Kæden af migrations skal være lineær og ubrudt, og den skal være konsistent på alle maskiner, for:
- Hvis man sletter en migrations-fil, så kan man ikke "rulle tilbage".
- Hvis man er ude af sync og mangler en ældre fil, så kan det være svært at komme up-to-date.
- Hvis Django kan se, at stien af migrations divergerer og ikke er lineæer, fx hvis _0007...py_ forgrenes i to forskellige retninger, så kan den ikke migrere databasen
  * Django vil forsøge at foreslå et fix. Det er muligvis nødvendigt at lave et manuelt fix.

Django's dokumentation kan evt. læses her: [Django Migrations (2.2)][4].


## Django-kommandoer til migrations
- `python manage.py makemigrations` -> Laver opskrifterne i _.py_-filer baseret på _models.py_ fra alle apps.
- `python manage.py migrate` -> Implementerer alle ændringer ned i databasen.
- `python manage.py migrate <app-navn> <migrations-navn>` -> Ruller app frem/tilbage til et bestemt punkt.
  * `python manage.py migrate demo_module zero` -> Ruller tilbage til nul.
    - Men det kan man ikke, hvis man mangler migrations-filerne (hvis de fx er slettet).
- `python manage.py showmigrations` -> vis alle migrations, både implementerede og ikke-implementerede.

Nb. hvis du arbejder uden for Django-containeren, så brug `docker-compose exec webinterface python manage.py <kommando>`.

I databasetabellen `django_migrations` kan man også se migrations, der er implementeret. :construction_worker:

## Migrations opstår når...
Når der ændres i _models.py_ (det kan være en ny tabel, ændring af felttype, osv), så har du en ikke-implementeret ændring til databasen:
 - Når du derefter kører `makemigrations`, så sammenligner Django _models.py_ med gamle migrations-filer (den sammenligner _ikke_ med databasens nuværende struktur). Den opretter så en ny _.py_-fil til at implementere ændringerne.
- Nogle migrationer kan implementeres uden at give konflikt med databasen (fx en ny tabel).
- Andre migrationer vil give konflikter (fx ændring af datatype for en søjle, fx fra boolean til tekst).
- Migrationer kan give datatab; hvis du sletter et felt/en tabel i _models.py_, og migrerer, så bliver tilhørende data slettet.
  * Det samme sker, hvis du ruller tilbage til en version af databasen, hvor felt/tabel ikke fandtes.
  
- Når man ændre i databasen, prøver django/psql selv på at mindske fejlene der vil opstå f.eks. hvis man ændre hvilke data der er    primary key, vil der (når man kører migrations) blive foreslået om den selv skal assigne værdier til det foregående data der ellers vil bliver "tabt", men det er dog ikke en permanent løsning, og der vil anbefales at man retter tabel indholdet selv!


## Principper for versionsstyring af migrations
Migrationer skal versionsstyres ligesom kode, fordi:
- Migrations gør det muligt at rulle databasen frem og tilbage ensartet på alle udviklingsmaskiner.
- Det er muligt at rulle databasen tilbage til en tidligere version, hvis noget går galt.
- Migrations viser udvikling i databasens struktur mellem forskellige versioner. Det giver traceability.
- Migrations hænger sammen med versionering af Webinterface (fx ved release af features ver. 2.00 + database ver. 2.00)


## Eksempel: Migrations og versionsstyring på udviklingsmaskine
0. Antaget at databasen på Marcs PC stemmer overens med Master fra Timebox 7.
   * Hvis der er noget rod dér, så må han nok tilbage til Timebox 6.
1. I Timebox 8 ændrer Jan i _models.py_, kører `makemigrations` og `migrate`.
2. Jan tester og alt er OK.
3. Den nye migrations-fil comittes sammen med resten af koden.
4. Jan pusher ændringerne op på Github, og får dem merget ind i Master.
5. Marc trækker nyeste Master ned fra Github. Marcs database er 1 Timebox bagud i forhold til Jans.
6. For at komme up-to-date kører Marc `migrate`. Dvs. han benytter den migrations-fil, som Jan har lavet.
   * NB.: Marc kører _ikke_ `makemigrations`.


## Eksempel fortsat: Migrations og versionsstyring på produktionsserver
7. Daniel skal opdatere AU-serveren, og gør det samme som Marc gjorde.
8. Hvis der er irreversible ændringer i databasen, fx ændring af datatayper, og data ikke må gå tabt:
   * Så skal Daniel og Jan aftale en måde at kopiere data fra gamle tabeller over i midlertidige tabeller.
   * Daniel kører `migrate` og accepterer de irreversible ændringer.
   * Endelig kopierer og konverterer de data fra de midlertidige tabeller over i de nye tabeller.
   * Hvis dette sker tit, så overvej at bruge _fixtures_, se længere nede (load startdata).


## Kendte fejl og mulige fixes

### Tøm database
:toilet: Hvis du vil tømme databasen (dvs. nulstille alle tabeller, men ikke slette tabellerne):
- Flush, dvs. tøm, hele databasen med kommandoen `docker-compose exec webinterface python manage.py flush`.
  * Hvis du kan nøjes med at tømme en enkelt database, så brug option `--database <database-navn>`.
  * Pt. har vi dog kun 1 database ved navn 'default' (svarer til webinterface_dev, se _settings.py_).


### DuplicateTable: relation ... already exists
Hvis du får denne fejl, er der nok blevet slettet en migrations-fil, så Djangos liste over migrations i databasen ikke stemmer overens med filerne fra Github.
Det er et permanent synkroniseringsproblem, der nok kun kan løses manuelt (Note 1).
Ændringer er ikke længere styret vha. Django, og `flush` vil ikke kunne fjerne det problem, som giver fejl.
- Rul databasen tilbage til nul: `docker-compose exec webinterface python manage.py migrate demo_module zero`
- Kig ind i databasen (se hvordan længere nede):
  * I tabellen `django_migrations` finder du den migration, der ikke kunne rulles tilbage for demo-module.
    - Fjern dens række! (i pgAdmin: Marker række, tryk på slet (skraldespand), tryk på Save data changes). 
  * De tilbageværende tabeller med navne som demo_module_tablename er de, som ikke kunne rulles tilbage, og derfor giver problemer.
  * Fjern manuelt problemet (fx med SQL `drop table ...`, eller markér tabel og vælg Drop cascade).
- Kør migrations igen (`migrate`).
- Bekræft at databasen nu er opdateret til og med nyeste migrations-fil.

Note 1) Hvis du er 100% sikker på, at databasen stemmer overens med migrations, kan du prøve at redde situationen med `docker-compose exec webinterface python manage.py migrate --fake-initial`


### Permission denied, eller lignende
Vi har oplevet, at læse/skrive rettigheder til database og migrations ikke stemmer overens, og at Django ikke kan/vil migrere. Man bør nok undersøge hvilke filrettigheder, den er gal med, men man _kan_ gennemtvinge migrations som `root` (`-u 0`) ved fx:
- `docker-compose exec -u 0 webinterface python manage.py migrate`.


### Permanent uløselige fejl
Hvis intet virker, så:
- Slet alle filer i `migrations/`-mappen, pånær __init__.py filen
- Drop databasen / alle tabeller i databasen
- Kør så `makemigrations`
- Kør `migrate`.
- Men PAS PÅ med at committe sletning af filerne! Diskutér med teamet først!


### I tilfælde af at intet virker for databasen, ved migrations eller server setup: Nuke it! 
:warning: Følg disse trin, hvor " " er terminal kommandoer
1. Gå ind i projekt mappen.
2. "docker-compose down"
3. "docker system prune"
4. "docker volume rm data-volume"
5. "docker rmi (postgres image id)"
          Image id'et kan findes med "docker images"
6. "docker volume create data-volume"
7. gå ind i migration mappen i appen som giver problemer.
          f.eks. cd webinterface/demo_module/migrations
8. slet alle migrations filer udover __init__.py
9. gå tilbage til projekt mappen ("cd ../../..")
10. i Dockerfile under webinterface mappen, udkommenter feltet der vil sætte bruger som non-su
          dette felt skulle gerne stå tæt på bunden af Dockerfilen!
11. "docker-compose up --build"

Denne sekvens vil fjerne ALT indhold i databasen! så brug med omtanke, og husk at ændre Dockerfilen tilbage!



## Load startdata (fixtures)
- Det er muligt at loade startdata til tabeller. Det er en måde at sikre, at evt. påkrævet stamdata kan genskabes hurtigt.
- Se [Initial data][3].


## Kig i databasen
- Benyt `python manage.py dbshell` som dok. i [dbshell (Django 2.2)][1].
  * Kræver psql (postgresql-client) installeret i webinterface, som den er fra 3. april 2020.
- Alternativt, forbind til databasen med GUI vha. [pgAdmin4][2] på port 5432 (localhost:5432 eller server:5432).


## Udfordringer når vi nærmer os en endelig version
- Vi har forskellige versioner af "databasen" på vores forskellige udviklingsmaskiner.
  * Vi har kørt forskellige versioner af migrations, har forskellige versioner af _models.py_.
- På udviklingsserveren (mooo) kan vi godt leve med, at databasen bliver slettet / flushet en gang i mellem.
- På produktionsserveren (AU-server) kører master-version altid _inklusive_ data i databasen, så der kan vi ikke bare slette / flushe.
- Master skal holdes konsistent for alle maskiner ud fra vores fælles kodebase.


[1]: https://docs.djangoproject.com/en/2.2/ref/django-admin/#dbshell
[2]: https://www.pgadmin.org/
[3]: https://docs.djangoproject.com/en/2.2/howto/initial-data/
[4]: https://docs.djangoproject.com/en/2.2/topics/migrations/
