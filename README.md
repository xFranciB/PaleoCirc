# paleocirc
API web scraping per ottenere circolari dal sito dell'I.T.I.S. Paleocapa di Bergamo

## Installazione
Per installare questa libreria è necessario eseguire ```pip install paleocirc``` in un terminale.

## Utilizzo

### Ricavare informazioni di una pagina
Puoi ottenere le informazioni di una pagina utilizzando<br>
```python
from paleocirc.circolari import Circolari

circolari = Circolari()
circolare = circolari.getPages(numero_pagine)
```
Nota: getPages(n) ritorna le prime <i>n</i> pagine di circolari (ciò significa che passando 2, ritornerà le prime due pagine). Per ottenere __solo__ la seconda pagina, bisogna passare "_range=False" nel metodo: `getPages(2, _range=False)`

### Ricavare informazioni di una circolare
Puoi ottenere le informazioni di una circolare utlizzando<br>
```python
from paleocirc.circolari import Circolari

circolari = Circolari()
circolare = circolari.get(numero_circolare)

circolare.number      #stringa: numero circolare (ad esempio 21, 250 bis)
circolare.name        #stringa: nome della circolare
circolare.date        #stringa: data in cui la circolare è uscita
circolare.url         #stringa: URL che porta alla pagina della circolare (non agli allegati)
circolare.restricted  #bool: True se la circolare è solo per i membri dello staff, altrimenti False

```

### Ottenere tutte le circolari dalla *n* fino all'ultima
Se vogliamo ottenere tutte le circolari dopo la *n*, possiamo farlo utilizzando la funzione getFrom().
Questo può essere molto pratico se utilizzato per ottenere le circolari appena escono, come in questo esempio:
```python
from paleocirc.circolari import Circolari
import time

circolari = Circolari()

while True:
  with open('latest.txt', 'w') as file:               #apri il file latest.txt, dove viene
                                                      #salvato il numero dell'ultima circolare
                                                      
    circolareList = circolari.getFrom(file.read())    #ottieni ogni circolare uscita dopo n
    
    for circolare in circolareList:                   #esegui operazione su
      # esegui operazioni                             #ogni circolare
    
    file.write(circolareList[0].number)               #salva l'ultima circolare in latest.txt
    time.sleep(1800)                                  #ripeti l'operazione dopo 30 minuti 
```

### Scaricare una circolare
È anche possibile scaricare le circolari utilizzando:<br>
```python
from paleocirc.circolari import Circolari

circolari = Circolari()
circolare = circolari.get(numero_circolare)

downloads = circolare.download(
  path=percorso_file,     #stringa: obbligatorio, la cartella in cui verrà
                          #scaricata la circolare (esempio: 'circolari')
  
  pngConvert=False,       #bool: opzionale, se impostata su True converte i PDF in PNG
  
  docConvert=False,       #bool: opzionale, se impostata su True converte
                          #i DOC e i DOCX in PNG (SOLO SU WINDOWS)
  
  keepDoc=False,          #bool: opzionale, se impostata su True mantiene i file
                          #DOC e DOCX dopo la conversione in PDF
  
  poppler=None            #stringa: opzionale, percorso di poppler per la
                          #conversione in PNG, se poppler non è presente in PATH
)

>>> downloads
{
  '1': {
    'name': 'Nome primo allegato',
    'filename': 'path/to/attachment1',
    files: [
      'path/to/png1',
      'path/to/png2'
    ]
  },
  
  '2': {
    'name': 'Nome secondo allegato',
    'filename': 'path/to/attachment2',
    files: [
      'path/to/png1',
      'path/to/png2'
    ]
  }
}
```
Nota: per convertire gli allegati in PDF è necessario avere poppler installato (https://github.com/oschwartz10612/poppler-windows/releases/) che deve essere presente in PATH. Se per qualche motivo Python non dovesse rilevare poppler in PATH, allora si può specificare il suo percorso quando si scarica una circolare, passando il percorso del file usando "poppler=path/to/poppler/bin".<br>
Inoltre, per convertire gli allegati in DOC e DOCX è **necessario** avere Microsoft Word installato su un sistema operativo Windows.

### Archivio di circolari
Per evitare di scaricare una circolare ogni volta, si può anche utilizzare un archivio.
```python
from paleocirc.circolari import Circolari

circolari = Circolari(archiveDir=archivePath) #passando "archiveDir" quando chiamiamo
                                              #la libreria, creiamo un archivio.
                                              
circolare = circolare.get(numero_circolare)   #quando l'archivio è stato caricato, la libreria cercherà 
                                              #la circolare al suo interno prima di fare una richiesta al sito.
                                               
circolare.download() #La circolare viene automaticamente scaricata nell'archivio,
                     #se "path" viene specificato verrà ignorato.
```
Nota: è altamente consigliato utilizzare un archivio di circolari quando possibile.

### Eliminare una circolare
È possibile eliminare tutti i file di una circolare e, se un archivio è stato caricato, anche le informazioni da esso.
```python
from paleocirc.circolari import Circolari

circolari = Circolari(archiveDir=archivePath)
circolare = circolare.get(numero_circolare)
circolare.download(pngConvert=True)

circolare.delete(
  archive=True,  #bool: opzionale, se impostato su True (default)
                 #elimina la circolare dall'archivio (se caricato e se la circolare è presente)
  
  files=True     #bool: opzionale, se impostato su True (default)
                 #elimina tutti i file relativi alla circolare (se esistono)
)
```

## Async
Oltre alla versione sync, questa libreria presenta anche una scritta in async. È consigliato utilizzare questa quando possibile, perché non ferma tutto il codice quando viene fatta una richiesta al sito. I metodi sono gli stessi, ma funzionano in modo un po' diverso.
Per importare la versione asincrona è necessario usare
```python
from paleocirc.circolariasync import Circolari
```

### Ottenere circolari in modo asincrono
Per utilizzare la versione asincrona bisogna chiamare i metodo usando "await" all'interno di una funzione asincrona.
I metodi che richiedono l'await sono quelli che fanno richieste al sito, cioè:
```python
await circolari.getPages()
await circolari.get()
await circolari.getFrom()
await circolare.download()
```

Per chiamare `Circolari()` è necessario utilizzare `async with`:
```python
from paleocirc.circolariasync import Circolari
import asyncio

async def main():
  async with Circolari(archiveDir=archivePath) as circolari:
    circolare = await circolari.get(numero_circolare)
    await circolare.download()
    ...
    
  circolare.delete()
    
    
# Codice necessario per eseguire la funzione in modo asincrono
loop = asyncio.get_event_loop()
loop.run_until_complete(main())
loop.close()
```

