# Regole del progetto AUREOLISSIMA

Sito statico pubblicato con GitHub Pages.
Repository pubblico: https://github.com/corsista06-dev/aureolissima

Queste regole valgono sempre, in ogni sessione di lavoro.

## 1. Tecnologia

- Solo HTML, CSS e JavaScript scritti a mano.
- Nessuna libreria esterna, nessun CDN, nessun framework.
- Nessun passaggio di build: i file del repository sono esattamente
  quelli che il browser riceve.

## 2. Struttura dei file

- `index.html` sta nella cartella principale del repository.
- Usare **solo percorsi relativi** (per esempio `immagini/foto.jpg`,
  mai `/immagini/foto.jpg` e mai percorsi del computer locale).
  GitHub Pages pubblica il sito in una sottocartella
  (`corsista06-dev.github.io/aureolissima/`): i percorsi che iniziano
  con `/` punterebbero fuori dal sito e non funzionerebbero.

## 3. Commit

- Commit piccoli e frequenti: una modifica compiuta per commit.
- Messaggi in italiano che spiegano **che cosa è cambiato**,
  non genericamente "aggiornamenti".
  Esempio buono: `Aggiunto il pulsante per ripetere l'esercizio`.
  Esempio da evitare: `fix`, `modifiche varie`.

## 4. Git e pubblicazione

- Usare **git** da riga di comando. Non usare mai `gh` (GitHub CLI):
  non è installato su questo computer.
- **Prima di ogni `git push` va chiesta conferma all'utente.**
  Nessun push automatico o silenzioso.
- Nota tecnica: in questo computer git è installato in
  `C:\Program Files\Git\cmd\git.exe`. Se il comando `git` non viene
  riconosciuto, richiamarlo con il percorso completo.

## 5. Come comunicare

- L'utente non è un programmatore.
- Spiegare ogni operazione con parole semplici, dicendo che cosa si sta
  facendo e perché, senza dare per scontato il gergo tecnico.
- Segnalare sempre in anticipo le operazioni difficili da annullare.

## 6. Privacy

- Il repository è **pubblico**: chiunque può leggere tutto,
  compresa la cronologia delle versioni passate.
- **Mai inserire dati personali reali di studenti**: niente nomi,
  cognomi, foto, voti, email, classi identificabili o altri dati
  riferibili a una persona reale.
- Per esempi e prove usare nomi di fantasia.
- Attenzione: un dato personale finito in un commit resta visibile nella
  cronologia anche dopo essere stato cancellato dal file. Meglio non
  inserirlo mai.
