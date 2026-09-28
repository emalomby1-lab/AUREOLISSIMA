# Prompt per ricostruire "Aureolissima"

> Copia tutto quello che segue la riga tratteggiata e incollalo in Gemini.

---

Sei uno sviluppatore web. Devi costruire un'applicazione web completa e funzionante
chiamata **Aureolissima**, seguendo fedelmente la specifica qui sotto.

## 1. Vincoli tecnici (non negoziabili)

- **Un solo file**: `index.html`, che contiene HTML, CSS e JavaScript.
- **Nessuna libreria esterna**, nessun CDN, nessun framework, nessun font scaricato
  da internet, nessuna immagine esterna. Tutto deve funzionare anche offline.
- **Nessun passaggio di compilazione**: il file si apre con un doppio clic e funziona.
- **Nessun server, nessun account, nessuna chiamata di rete.**
- Interfaccia **interamente in italiano**.
- Deve funzionare su GitHub Pages: solo percorsi relativi.
- JavaScript in `"use strict"`, senza moduli.
- Deve funzionare bene sia su un telefono sia proiettata su una LIM.

## 2. Che cos'è

Aureolissima è un gioco di classe per l'ora di religione (o di educazione civica)
alla scuola secondaria. Gli studenti accumulano **Punti Paradiso (PP)** studiando,
partecipando e facendo del bene agli altri. Il traguardo simbolico è **100 PP**,
rappresentati da un'**aureola** che si riempie man mano.

Principio fondamentale, da rispettare ovunque: **nessuno si assegna punti da solo.**
Ogni richiesta è una proposta che finisce in una coda; solo il docente approva.

Tono dei testi: caldo e letterario, mai infantile e mai bigotto. Serio ma con
ironia leggera — il sottotitolo dell'app è "Il gioco dei Punti Paradiso".

## 3. Privacy: è un requisito di progetto, non un dettaglio

L'app è pensata per **minorenni** e il codice è pubblico. Quindi:

- Gli studenti si identificano con un **nickname di fantasia** e un **PIN numerico**,
  mai con nome e cognome reali.
- L'app deve **proporre nickname automatici**, combinando due liste di parole
  (per esempio "Pellicano Sereno", "Cometa Ostinata"), proprio per scoraggiare
  l'uso del nome vero.
- Deve esistere una **Modalità Privacy LIM**: quando è attiva, ogni nome mostrato
  viene ridotto a prima parola più iniziale della seconda ("Marco R.").
- Serve una schermata di **informativa sulla privacy** che spieghi dove finiscono
  i dati, che i PIN non sono una misura di sicurezza informatica ma solo un modo
  per evitare scherzi tra compagni, e che non vanno mai inseriti dati delicati
  (salute, situazione familiare, provvedimenti disciplinari).

## 4. Dove vivono i dati

Tre livelli, tutti locali:

1. **localStorage**, chiave `aureolissima.v1`. È il salvataggio automatico di base.
2. **Archivio su file** (opzionale): con la File System Access API
   (`showSaveFilePicker` / `showOpenFilePicker`) il docente collega un file `.json`.
   Da quel momento l'app ci riscrive dentro a ogni modifica, con un ritardo di
   circa 700 ms per non scrivere troppo spesso. Il riferimento al file va
   conservato in **IndexedDB** (database `aureolissima`, store `handle`), così il
   collegamento sopravvive alla chiusura del browser; alla riapertura va richiesto
   di nuovo il permesso di scrittura. Se il browser non supporta l'API, dirlo con
   garbo. Nell'intestazione mostra una **spia** con lo stato del file: assente,
   attivo, da riautorizzare, errore.
3. **Backup manuale**: esportazione di un `.json` scaricabile e reimportazione,
   con conferma prima di sovrascrivere i dati attuali.

## 5. Modello dati

```js
D = {
  v: 1,
  pinDocente: "1234",          // PIN iniziale
  privacyLIM: false,
  classi: [
    {
      id, nome,
      aperta: true,            // iscrizione libera degli studenti
      studenti: [
        { id, nick, pin, santo, pp: 0,
          storico: [ { ts, delta, causale } ] }   // dal più recente
      ],
      archivi: [               // quadrimestri chiusi
        { ts, etichetta, classifica: [ { nick, pp } ] }
      ]
    }
  ],
  richieste: [
    { id, classeId, studId, segnalatoreId,
      tipo,      // "merito" | "partecipazione" | "carita" | "quiz" | "nomina"
      testo, pp,
      stato,     // "attesa" | "approvata" | "respinta"
      ts }
  ]
}
```

Lo stato dell'interfaccia (vista corrente, classe aperta, studente collegato,
carta pescata, selezione multipla, modalità proiezione) va tenuto in un oggetto
separato e **non salvato**.

Architettura consigliata: un router semplicissimo, cioè una funzione `disegna()`
che ricostruisce l'HTML della vista corrente dentro un contenitore, e **un solo
gestore di click delegato** sul `document`, che smista le azioni in base a un
attributo `data-act` sui pulsanti.

## 6. L'aureola

È il cuore visivo dell'app. Va disegnata in **SVG inline**, non come immagine:

- un **arco** aperto verso l'alto, raggio 46, centro in (60, 58), da −172° a −8°,
  con `viewBox="0 0 120 62"`;
- un secondo arco dorato sovrapposto, lungo in proporzione ai punti su 100;
- **otto pallini** lungo l'arco, alle soglie **10, 20, 30, 45, 55, 65, 80, 100**,
  che si accendono quando il punteggio le supera;
- tre di queste soglie — **30, 65, 100** — sono **tappe**: pallino più grande e,
  quando acceso, un alone luminoso;
- al centro il numero dei punti; nelle versioni grandi anche la scritta
  "PUNTI PARADISO".

## 7. Le schermate

### Ingresso

Titolo: *"Un'aula, cento punti, qualche aureola."* Sotto, una frase che spiega che
i punti si guadagnano studiando, partecipando e facendo del bene, e che ogni
richiesta passa dalle mani del docente. Poi due grandi porte affiancate:
**"Sono il docente"** e **"Sono uno studente"**.

### Area docente (protetta da PIN, quello iniziale è `1234`)

**Le tue classi** — una scheda per classe con numero di slot, PP totali e un
pallino dorato con il numero di richieste da valutare. Pulsante per creare una
nuova classe.

**Dentro una classe** — griglia di schede studente, ognuna con nickname, santo
accompagnatore, aureola e punteggio, più i pulsanti `+1`, `Punti` e `Gestisci`.
Barra degli strumenti con: **Mazzo di San Pietro**, **Richieste** (con contatore),
**Registro**, **Riepilogo**, **Aggiungi slot**, **Aggiungi in massa**,
**Iscrizione libera: attiva/chiusa**, **Modalità proiezione**, **Selezione multipla**.

**Mazzo di San Pietro** — schermata a fondo scuro pensata per la proiezione. Si
pesca una carta a caso da un mazzo che mescola due generi:

- **Domanda spot** (carta chiara): ha una risposta corretta, tenuta coperta finché
  il docente non preme "Mostra la risposta". Vale da 3 a 6 PP.
- **Spunto di riflessione** (carta verde): domanda aperta senza risposta unica; la
  nota dice esplicitamente di valutare chi si è espresso meglio. Vale da 4 a 6 PP.

Non deve mai uscire due volte di fila la stessa carta. Un pulsante assegna i punti
della carta a uno studente scelto da un elenco.

**Richieste** — la coda delle proposte in attesa, con testo, autore e punti
proposti. Per ciascuna: **Approva**, **Approva con altri punti**, **Respingi**.
In più un pulsante per **approvare in blocco tutti i quiz** superati della classe.

**Registro** — tutti i movimenti della classe dal più recente, con segno, causale
e data.

**Riepilogo** — classifica, media della classe, crescita di ciascuno negli ultimi
30 giorni e indicazione di chi è cresciuto di più. Due azioni:

- **Esporta CSV** (separatore `;`, con BOM iniziale, così Excel italiano lo apre
  correttamente);
- **Chiudi il quadrimestre**: archivia la classifica attuale, aggiunge a ciascuno
  un movimento negativo di azzeramento e riporta tutti a zero. Lo storico resta
  consultabile. Va chiesta conferma esplicita.

**Aggiungi in massa** — il docente incolla un elenco di nickname, uno per riga.
L'app crea gli slot generando per ognuno un **PIN casuale di 4 cifre** e un santo
a caso, saltando i nickname già presenti. Subito dopo mostra una pagina di
**bigliettini ritagliabili**: una griglia con bordi tratteggiati, un riquadro per
studente con classe, nickname e PIN, pronta per la stampa. In stampa i pulsanti
spariscono.

**Impostazioni** — cambio del PIN docente, gestione dell'archivio su file, backup
ed esportazione, collegamento all'informativa privacy, azzeramento totale con
conferma.

### Area studente

**Accesso** — sceglie la classe, il nickname da un elenco e digita il PIN. Se
nella classe l'iscrizione libera è attiva, può **crearsi lo slot da solo**:
sceglie il nickname (con un pulsante "Proponi" che ne suggerisce uno di fantasia),
il PIN ripetuto due volte e un santo da un elenco, con una scheda che ne racconta
la storia.

**La sua schermata** — saluto, aureola grande, quanti punti mancano alla prossima
tappa, e quattro azioni: **Chiedi dei punti**, **Gioca a un quiz**, **Segnala un
compagno**, **I tuoi movimenti**. Sotto: una scheda sul proprio santo, una scheda
**"Lo sapevi?"** con una curiosità storica pescata a caso e un pulsante per
cambiarla, e l'elenco delle ultime quattro richieste con il loro esito
(in attesa, approvata, respinta).

Su schermo stretto (sotto i 680 px) le quattro azioni diventano una **barra fissa
in basso**, scorrevole in orizzontale, con pulsanti alti almeno 48 px, così stanno
a portata di pollice.

**Chiedi dei punti** — sceglie il motivo (merito scolastico, partecipazione alla
lezione, gesto di carità), scrive che cosa ha fatto (minimo 10 caratteri, massimo
400) e propone da 1 a 10 punti. La richiesta finisce nella coda del docente.

**Segnala un compagno** — sceglie un compagno, racconta un gesto che ha notato e
propone dei punti. I punti vanno al compagno; **se il docente approva, anche chi
ha segnalato riceve 1 punto**, per essere stato attento agli altri. In cima alla
schermata, una citazione di San Paolo (Romani 12,9-10) sull'amarsi a vicenda.

**Quiz** — domanda a scelta multipla oppure a risposta scritta. Il confronto della
risposta scritta ignora maiuscole, accenti e punteggiatura. Dopo la risposta
compare una spiegazione che aggiunge sempre qualcosa in più, anche quando si
sbaglia. **Se la risposta è giusta parte automaticamente una richiesta di punti al
docente**, non un accredito diretto.

**I tuoi movimenti** — lo storico personale dei punti.

## 8. Regole dei punti

- I punti non scendono mai sotto zero.
- Ogni assegnazione registra un movimento con data, variazione e causale leggibile.
- All'approvazione, la causale unisce il motivo e il testo della richiesta.
- Il pulsante `+1` assegna un punto con causale "Partecipazione".
- La **selezione multipla** permette di assegnare gli stessi punti a più studenti
  insieme, con una barra che resta visibile in alto mentre si scorre la pagina.

## 9. Momenti speciali

**La scena della soglia** — quando uno studente supera 30, 65 o 100 PP compare a
schermo intero una scena celebrativa su fondo notturno: aureola grande animata,
nickname, la tappa raggiunta e il nome del santo che lo accompagna. Dura circa tre
secondi e si chiude da sola. Se più studenti superano una tappa nello stesso
momento, le scene si mettono in coda e si susseguono.

**Modalità proiezione** — pensata per la LIM: fondo scuro ad alto contrasto,
nickname a 34 px, punteggi a 46 px, pulsanti più grandi, sei schede per riga
(tre sotto i 1400 px, due sotto gli 820 px).

## 10. I contenuti da scrivere

Servono cinque raccolte di contenuti originali, in italiano, storicamente corrette
e adatte a ragazzi di 11-16 anni. Livello culturale alto, niente banalità:

- **26 santi**, ciascuno con nome, ambito di patronato e un ritratto di due righe
  che racconti un dettaglio concreto e memorabile. Mescola epoche e tipi: dai
  classici (Francesco, Chiara, Tommaso d'Aquino, Filippo Neri, Giovanni Bosco) ai
  contemporanei (Teresa di Calcutta, Massimiliano Kolbe, Carlo Acutis).
- **30 domande** per il Mazzo, ognuna con la sua risposta, da 3 a 6 PP: Vangeli,
  storia della Chiesa, grandi religioni, diritti umani, questioni etiche.
- **12 spunti di riflessione** senza risposta giusta, da 4 a 6 PP: domande vere,
  di quelle che aprono una discussione in classe.
- **32 curiosità** storiche brevi e sorprendenti, una frase ciascuna.
- **20 quiz** a scelta multipla, ognuno con quattro opzioni, la risposta corretta,
  una spiegazione che aggiunge un dettaglio, e 3 o 4 PP.

## 11. Stile visivo

Palette da manoscritto miniato, chiara e ariosa: azzurro cielo `#e9f1fb` per lo
sfondo, blu `#2c6bad` per le azioni principali, oro `#c4941f` per l'aureola e i
momenti di festa, blu notte `#111c33` per le schermate da proiettare, inchiostro
`#16233c` per il testo.

Titoli in **serif**, usando solo caratteri di sistema (Iowan Old Style, Palatino,
Georgia); testo corrente in sans di sistema. Pulsanti a pillola, pannelli bianchi
con angoli di 14 px e ombre morbide. Nessun font scaricato da internet.

## 12. I dettagli che fanno la differenza

- Messaggi di avviso come piccola pillola scura in basso al centro, che compare e
  svanisce da sola dopo circa 2,6 secondi.
- Ogni azione distruttiva (eliminare uno slot o una classe, chiudere il
  quadrimestre, azzerare tutto) passa da una finestra di conferma che dice
  chiaramente che cosa si perde.
- `Esc` chiude le finestre; `Invio` conferma i campi PIN e la risposta al quiz.
- Ogni testo scritto dagli utenti va **sempre** passato in una funzione di escape
  prima di finire nell'HTML.
- Il focus da tastiera deve restare sempre visibile (contorno dorato) e le
  etichette dei campi vanno collegate ai rispettivi input.

## 13. Che cosa consegnare

Un unico file `index.html` completo e funzionante, con tutti i contenuti già
scritti dentro — non segnaposto — pronto da aprire in un browser.
