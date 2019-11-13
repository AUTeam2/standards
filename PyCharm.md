# PyCharm :rocket:

Her er et par tips and tricks til PyCharm fra JetBrains.

## Download og installation
- Lav en konto med din e-mail fra AU og få en fuld studielicens: [JetBrains](https://www.jetbrains.com/).
- Download og installér **Pro**-udgaven.
	- _Lad være_ med at installere IdeaVim plugin, medmindre du :heart: ViM.
- De andre IDE'er fra JetBrains er også fede, fx CLion til C/C++.

![Downloads](/img/downloads.png)

## Kodestandard
- Vi benytter [PEP8](https://www.python.org/dev/peps/pep-0008/) i vores [Kodestandard til Python](https://github.com/AUTeam2/standards/blob/master/kodestandard-team-web.md#python).
- PyCharm er sat op til at benytte PEP8 som kodestandard. 
- Det behøver du ikke ændre (men du kan da lige tjekke det... :wink:)
- Standarden styres i
	- *File > Settings > Editor > Code Style > Python* (på PC)
	- *Preferences > Editor > Code Style > Python* (på Mac)

![Code Style](/img/codestyle.png)

- PyCharm viser dig, om du overholder kodestandarden, når *Inspections > Python > PEP 8 coding style violation* og *Python > PEP 8 naming convention violation* er slået til.
- Du kan selv styre hvilken slags *fejl* det skal give.

![Python Inspections](/img/python-inspections.png)

- Tjek også at alle *Inspections > Django* er slået til.

![Django Inspections](/img/django-inspections.png)

- Tjek også at *docker-compose*, *HTML*, *CSS* og *JavaScript* er rimeligt indstillet.

### Formattering til kodestandard
- PyCharm formatterer din kode til standarden med *Code > Reformat Code*.
- Der er en keyboard-genvej, som du nok lige så godt kan lære.

![Reformat Code](/img/reformat.png)


## Tips til Windows
- Start PyCharm som *administrator*.
- Tjek at dine filer gemmes i **UTF-8** med **LF** linjeskift (ellers skift det ved at trykke på statuslinjen):

![Line Endings](/img/line-endings.png) 

## Flere tips og tricks
- [42 PyCharm Tips & Tricks](https://www.jetbrains.com/pycharm/guide/playlists/42/)

:snake:
