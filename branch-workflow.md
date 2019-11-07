# Workflow til at samarbejde i branches

## Formål

Vi benytter branches til at udvikle features, rette bugs, osv. Denne beskrivelse viser, hvordan det gøres med `git` på kommandolinjen og **Github**.

## Flow

### Hent seneste data fra repo
- Kør `git pull` for at hente nyeste versioner fra **Github**.

### Opret ny branch til feature/issue og skift til den nye branch
- Kør `git branch feauture/feature-navn`
- Kør `git checkout feature/feature-navn`

### Skriv kode, test og commit
- Bla bla bla
- Test test test
- `git add .`
- `git commit`
- Skriv commit-besked

### Nu er der nok gået noget tid, så hent lige den nyeste kode fra repo igen...
- Kør `git pull` for at hente nyeste versioner fra **Github**.

### Merge-konflikter?
- Så må du lige fikse dem først, og adde+committe en gang til...

### Push upstream (til **Github**) med en ny branch
- `git push --set-upstream origin feature/feature-navn`
- Nu kan du se den nye branch på Github

### Lav pull-request
- Vælg den nye branch
- Klik på knappen *Compare & pull request*
- Skriv kommentarer til pull request
- Assign en person til review og merge.
