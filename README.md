# Mancate Vendite — deploy gratuito (GitHub + Vercel + Firebase)

App per registrare le mancate vendite, sincronizzata in tempo reale su tutti i dispositivi tramite Firebase Firestore. Hosting statico su Vercel, codice su GitHub.

**Costo: 0€**, entro i limiti dei piani gratuiti:
- Firebase Spark (gratuito): 50.000 letture/giorno, 20.000 scritture/giorno, 1 GB storage — ampiamente sufficiente per questo utilizzo.
- Vercel Hobby (gratuito): hosting statico illimitato per progetti non commerciali.
- GitHub: repository gratuito.

---

## 1. Crea il progetto Firebase

1. Vai su [console.firebase.google.com](https://console.firebase.google.com) e crea un nuovo progetto (nome libero, es. `mancate-vendite`).
2. Nel menu a sinistra: **Build > Firestore Database** → **Crea database** → scegli una regione vicina (es. `eur3 (europe-west)`) → modalità **produzione** (le regole personalizzate del progetto sostituiranno quelle di default).
3. Nel menu a sinistra: **Build > Authentication** → **Get started** → scheda **Sign-in method** → abilita il provider **Anonimo**. Serve solo a distinguere "chi ha aperto il sito" da un bot esterno; non chiede nulla all'utente, è invisibile.
4. Nella pagina principale del progetto, clicca l'icona **`</>`** (Aggiungi app web), dai un nome all'app, **non serve** spuntare Firebase Hosting. Ti mostrerà un oggetto `firebaseConfig`.

## 2. Configura il codice

Apri `index.html` e sostituisci i valori segnaposto con quelli copiati al punto 4:

```js
const firebaseConfig = {
  apiKey: "...",
  authDomain: "...",
  projectId: "...",
  storageBucket: "...",
  messagingSenderId: "...",
  appId: "..."
};
```

> Questi valori **non sono segreti**: in un'app web Firebase sono per forza visibili a chi apre la pagina (chiunque può vederli nel codice sorgente). La sicurezza reale è nel file `firestore.rules`.

## 3. Pubblica le regole di sicurezza (`firestore.rules`)

Due modi, scegli quello più comodo:

**A — dalla console (più veloce, nessuna installazione):**
Console Firebase → Firestore Database → scheda **Regole** → incolla il contenuto di `firestore.rules` → **Pubblica**.

**B — da riga di comando (Firebase CLI):**
```bash
npm install -g firebase-tools
firebase login
firebase use --add        # seleziona il progetto creato al passo 1
firebase deploy --only firestore:rules
```

## 4. Carica il codice su GitHub

```bash
cd mancate-vendite-web
git init
git add .
git commit -m "Prima versione"
git branch -M main
git remote add origin https://github.com/TUO-UTENTE/mancate-vendite.git
git push -u origin main
```
(Crea prima il repository vuoto su github.com, senza README, poi usa l'URL che ti fornisce al posto di quello sopra.)

## 5. Pubblica su Vercel

1. Vai su [vercel.com](https://vercel.com) → accedi con l'account GitHub.
2. **Add New... > Project** → seleziona il repository appena creato.
3. Framework Preset: **Other**. Build Command: vuoto. Output Directory: vuoto (radice del progetto). Non serve alcuna build: è un sito statico.
4. **Deploy**.
5. Dopo pochi secondi Vercel ti dà un URL pubblico tipo `https://mancate-vendite.vercel.app` — è il link da condividere con i colleghi, raggiungibile da qualsiasi PC o cellulare con internet.

Da questo momento, ogni volta che fai `git push` su `main`, Vercel ripubblica automaticamente la nuova versione (CI/CD già incluso, nessuna configurazione aggiuntiva).

## Note tecniche

- **Autenticazione**: all'apertura del sito, ogni visitatore riceve automaticamente una sessione anonima Firebase (`signInAnonymously`). Non c'è login/password: chiunque abbia il link può leggere e scrivere, esattamente come nella versione precedente su claude.ai. Se in futuro vuoi restringere l'accesso (es. sapere chi ha inserito cosa con un vero account, o impedire l'accesso a chi non ha il link), si passa ad Authentication con email/password o Google Sign-In: modifica localizzata solo in `firestore.rules` e nella parte di login di `index.html`.
- **Struttura dati**: identica alla versione precedente — collezione `voci`, campi `precodice`, `codice`, `descrizione`, `data`, `operatore`, `stato` (`attiva`/`archiviata`), `createdAt`, `archivedAt`.
- **Realtime**: `onSnapshot()` di Firestore sostituisce 1:1 il meccanismo usato nell'artifact — stesso comportamento, propagazione automatica delle modifiche a tutti i client connessi.
- **Limiti del piano gratuito Firebase**: se in futuro l'uso crescesse molto (centinaia di operatori, migliaia di scritture al giorno), Firebase passa automaticamente a fatturazione a consumo (piano Blaze) solo oltre le soglie gratuite — per un singolo punto vendita/ufficio è estremamente improbabile superarle.
