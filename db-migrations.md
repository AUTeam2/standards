# Databasemigrationer
I Django er en databasemigration en _.py_-fil, som implementerer ændringer i en database.
Migrationer laves _per app_, dvs. _demo-modul_ har sine egne migrationer, der ligger i `demo_module/migrations`. Der kan være afhængigheder til andre apps (fx via ForeignKeys).

Migrationer er nummererede og lineære/sekventielle. Fx er migration _0007...py_ afhængig af _0006...py_, osv.
Der skal være en lineær kæde af migrationer. 
Hvis Django kan se, at stien af migrations divergerer og ikke er lineæer, fx hvis _0007...py_ forgrenes i to forskellige retninger, vil den foreslå et fix. Det er muligvis nødvendigt at lave et manuelt fix.


## Django-kommandoer til migrationer
- `python manage.py makemigrations` -> laver _.py_-filer baseret på _models.py_ fra alle apps.
- `python manage.py migrate` -> implementerer alle ændringer ned i databasen.
- `python manage.py migrate <app-navn> <migrationsnavn>` -> migrerer app frem/tilbage til et bestemt punkt.
  * `python manage.py migrate demo_module zero` -> migrerer tilbage til nul.
    - Men det kan man ikke, hvis man mangler migrationsfilerne (hvis man har slettet dem).
- `python manage.py showmigrations` -> vis alle migrationer, implementerede og ikke-implementerede.

Nb. hvis du arbejder uden for Django-containeren, så brug `docker-compose exec webinterface python manage.py <kommando>`.

I databasetabellen `django_migrations` kan man også se migrations, der er implementeret, og hvornår.

## Migrationer opstår, fordi...
Ændringer til databasen opstår, når der ændres i _models.py_. Det kan være en ny tabel, ændring af felttype, osv.
- Nogle migrationer kan implementeres uden at give i konflikt med databasen (fx en ny tabel).
- Andre migrationer vil give konflikter (fx ændring af datatype for en søjle, fx fra boolean til tekst).
- Migrationer kan give datatab; hvis du sletter et felt/en tabel i _models.py_, og migrerer, så bliver tilhørende data slettet.
  * Det samme sker, hvis du ruller tilbage til en version af databasen, hvor felt/tabel ikke fandtes.


## Udfordring
- Vi har forskellige versioner af "databasen" på vores forskellige udviklingsmaskiner.
  * Vi har kørt forskellige versioner af migrationer, har forskellige versioner af _models.py_.
- På produktionsserveren findes en master-version _inklusive_ data. Denne må **per princip** ikke ødelægges / flushes.
- Master skal være konsistent for alle maskiner.
- Det hele skal håndteres i én kodebase.


## Principper
Migrationer _skal_ versionsstyres ligesom kode, fordi:
- Migrationer viser udvikling i databasens struktur og giver traceability.
- Migrationer hænger sammen med versionering af Webinterface (features ver. 2.00 + database ver. 2.00)
- Migrationer gør det muligt at "rulle tilbage" til en tidligere version, hvis noget går galt.


## Migrationer og versionsstyring
1. Når en ændring i 


## Udrulning af ændringer på udviklingsmaskiner


## Udrulning af ændringer på produktionsserver (AU-server)


## Sidste udvej: Ultra-destruktiv måde at starte fra nul, hvis alt andet er gået i lort...
- Flush, dvs. slet, hele databasen: `python manage.py flush`.
  * Hvis du kan nøjes med at slette en enkelt database, så brug option `--database <database-navn>`.
  * Pt. har vi dog kun 1 database ved navn 'default' (svarer til webinterface_dev, se _settings.py_).
- Kør alle migrationer (`migrate`).
- Hvis dette ikke virker, så `flush`, slet alle filer i `migrations/`-mappen og kør så `makemigrations` og endelig `migrate`.


## Load startdata (fixtures)
- Se [Initial data][3].


## Kig i databasen
- Benyt `python manage.py dbshell` som dok. i [dbshell (Django 2.2)][1].
  * Kræver psql (postgresql-client) installeret i webinterface, som den er fra 3. april 2020.
- Alternativt, forbind til databasen med GUI vha. [pgAdmin4][2] på port 5432 (localhost:5432 eller server:5432).


[1]: https://docs.djangoproject.com/en/2.2/ref/django-admin/#dbshell
[2]: https://www.pgadmin.org/
[3]: https://docs.djangoproject.com/en/2.2/howto/initial-data/
