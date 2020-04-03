# Opdatering af databasestruktur med migrationer

Her er et par tips & tricks til at arbejde med databasen mens vi udvikler på projektet.
Skriv gerne til når/hvis du har flere tips & tricks.


## Databasemigrationer
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

I databasetabellen `django_migrations` kan man også se migrations, der er implementeret.

## Migrations opstår...
Når der ændres i _models.py_ (det kan være en ny tabel, ændring af felttype, osv), så har du en ikke-implementeret ændring til databasen:
 - Når du derefter kører `makemigrations`, så sammenligner Django _models.py_ med gamle migrations-filer (den sammenligner _ikke_ med databasens nuværende struktur). Den opretter så en ny _.py_-fil til at implementere ændringerne.
- Nogle migrationer kan implementeres uden at give i konflikt med databasen (fx en ny tabel).
- Andre migrationer vil give konflikter (fx ændring af datatype for en søjle, fx fra boolean til tekst).
- Migrationer kan give datatab; hvis du sletter et felt/en tabel i _models.py_, og migrerer, så bliver tilhørende data slettet.
  * Det samme sker, hvis du ruller tilbage til en version af databasen, hvor felt/tabel ikke fandtes.


## Udfordringer når vi nærmer os en endelig version
- Vi har forskellige versioner af "databasen" på vores forskellige udviklingsmaskiner.
  * Vi har kørt forskellige versioner af migrations, har forskellige versioner af _models.py_.
- På udviklingsserveren (mooo) kan vi godt leve med, at databasen bliver slettet / flushet en gang i mellem.
- På produktionsserveren (AU-server) kører master-version altid _inklusive_ data i databasen, så der kan vi ikke bare slette / flushe.
- Master skal holdes konsistent for alle maskiner ud fra vores fælles kodebase.


## Principper
Migrationer skal versionsstyres ligesom kode, fordi:
- Migrationer viser udvikling i databasens struktur mellem forskellige versioner. Det giver traceability.
- Migrationer hænger sammen med versionering af Webinterface (release features ver. 2.00 + database ver. 2.00)
- Migrationer gør det muligt at "rulle tilbage" til en tidligere version, hvis noget går galt.


## Migrations og versionsstyring på udviklingsmaskine - eksempel
0. Antaget at databasen på Marcs PC stemmer overens med Master fra Timebox 7.
   * Hvis der er noget rod dér, så må han `flush` den (se længere nede).
1. I Timebox 8 ændrer Jan i _models.py_, kører `makemigrations` og `migrate`.
2. Jan tester og alt er OK.
3. Den nye migrations-fil comittes sammen med resten af koden
4. Jan pusher ændringerne op på Github, og får dem merget ind i Master.
5. Marc trækker nyeste Master ned fra Github. Marcs database er bagud i forhold til Jans.
6. For at komme up-to-date kører Marc `migrate`. Dvs. han benytter den migrations-fil, som Jan har lavet.
   * NB.: Marc kører _ikke_ `makemigrations`.


## Migrations og versionsstyring på produktionsserver - eksempel fortsat
7. Daniel skal opdatere AU-serveren, og gør det samme som Marc gjorde.
8. Hvis der er irreversible ændringer i databasen, fx ændring af datatayper, og data ikke må gå tabt:
   * Så skal Daniel og Jan aftale en måde at kopiere data fra gamle tabeller over i midlertidige tabeller
   * Daniel kører `migrate` og accepterer de irreversible ændringer.
   * Endelig kopierer og konverterer de data fra de midlertidige tabeller over i de nye tabeller.


## Permission denied, eller lignende
Vi har oplevet, at læse/skrive rettigheder til database og migrations ikke stemmer overens, og at Django ikke vil migrere. Man bør nok undersøge hvilke filrettigheder, den er gal med, men man kan gennemføre migrations som `root` (`-u 0`) ved fx:
- `docker-compose -u 0 webinterface python manage.py migrate`.


## Sidste udvej: Ultra-destruktiv måde at starte fra nul, hvis alt andet er gået i lort...
Hvis fx migrations-filer er gået tabt, eller en anden håbløs situation:
- Flush, dvs. slet, hele databasen med kommandoen `python manage.py flush`.
  * Hvis du kan nøjes med at slette en enkelt database, så brug option `--database <database-navn>`.
  * Pt. har vi dog kun 1 database ved navn 'default' (svarer til webinterface_dev, se _settings.py_).
- Kør alle migrations (`migrate`).
- Hvis dette ikke virker, så `flush`, slet alle filer i `migrations/`-mappen og kør så `makemigrations` og endelig `migrate`.
  * Men du må _ikke_ committe, at du har slettet filerne!


## Load startdata (fixtures)
- Det er muligt at loade startdata til tabeller. Det er en måde at sikre, at evt. påkrævet stamdata kan genskabes hurtigt.
- Se [Initial data][3].


## Kig i databasen
- Benyt `python manage.py dbshell` som dok. i [dbshell (Django 2.2)][1].
  * Kræver psql (postgresql-client) installeret i webinterface, som den er fra 3. april 2020.
- Alternativt, forbind til databasen med GUI vha. [pgAdmin4][2] på port 5432 (localhost:5432 eller server:5432).


[1]: https://docs.djangoproject.com/en/2.2/ref/django-admin/#dbshell
[2]: https://www.pgadmin.org/
[3]: https://docs.djangoproject.com/en/2.2/howto/initial-data/
[4]: https://docs.djangoproject.com/en/2.2/topics/migrations/
