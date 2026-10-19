Tutorial: compilare e installare AltTab 10.12.0 su macOS con Xcode

Questa guida riassume i passaggi che abbiamo seguito per compilare AltTab 10.12.0 su un Mac Apple Silicon, installarlo nella cartella Applicazioni e avviarlo automaticamente all’accesso.

La procedura utilizza Xcode 27, CocoaPods e un certificato Apple Development personale.

1. Preparare l’ambiente

Occorrono:

* Mac con processore Apple Silicon
* Xcode installato
* Homebrew
* CocoaPods
* Codice sorgente di AltTab 10.12.0

Installa CocoaPods, se non è già presente:

brew install cocoapods

Scarica il codice sorgente di AltTab 10.12.0 da  ⁠GitHub, progetto AltTab.

Estrai l’archivio e apri il Terminale nella cartella del progetto:

cd ~/Downloads/alt-tab-macos-10.12.0

2. Installare le dipendenze

Esegui:

pod install

Attendi che CocoaPods completi l’installazione.

Apri quindi il workspace:

open alt-tab-macos.xcworkspace

Importante: apri sempre il file .xcworkspace, non il .xcodeproj, perché il progetto utilizza CocoaPods.

3. Correggere il deployment target

Durante i nostri tentativi, Xcode 27 segnalava errori perché alcune configurazioni richiedevano macOS 10.12.

Abbiamo risolto impostando macOS 12.0 come deployment target.

Dalla cartella del progetto esegui:

sed -i '' \
  's/MACOSX_DEPLOYMENT_TARGET = 10\.12;/MACOSX_DEPLOYMENT_TARGET = 12.0;/g' \
  alt-tab-macos.xcodeproj/project.pbxproj

Ripeti per CocoaPods:

sed -i '' \
  's/MACOSX_DEPLOYMENT_TARGET = 10\.12;/MACOSX_DEPLOYMENT_TARGET = 12.0;/g' \
  Pods/Pods.xcodeproj/project.pbxproj

Modifica anche il file di configurazione principale:

sed -i '' \
  's/MACOSX_DEPLOYMENT_TARGET = 10\.12/MACOSX_DEPLOYMENT_TARGET = 12.0/' \
  config/base.xcconfig

Verifica:

xcodebuild \
  -workspace alt-tab-macos.xcworkspace \
  -scheme Debug \
  -configuration Debug \
  -showBuildSettings 2>/dev/null \
  | grep 'MACOSX_DEPLOYMENT_TARGET ='

Il risultato atteso è:

MACOSX_DEPLOYMENT_TARGET = 12.0

4. Evitare che gli avvisi blocchino la compilazione

Con Xcode 27 abbiamo incontrato errori relativi ad API Swift deprecate, tra cui ultraDark.

Per evitare che i warning vengano trattati come errori, aggiungi queste righe a config/base.xcconfig:

cat >> config/base.xcconfig <<'EOF'
SWIFT_TREAT_WARNINGS_AS_ERRORS = NO
GCC_TREAT_WARNINGS_AS_ERRORS = NO
EOF

Questa modifica non elimina gli avvisi, ma permette alla compilazione di proseguire quando si tratta effettivamente di warning.

5. Configurare la firma in Xcode

Apri il workspace:

open alt-tab-macos.xcworkspace

In Xcode:

1. Seleziona il progetto alt-tab-macos.
2. Seleziona il target alt-tab-macos.
3. Apri Signing & Capabilities.
4. Attiva Automatically manage signing.
5. Nel campo Team, seleziona il tuo Personal Team.
6. Controlla le configurazioni Debug e Release.

Nel nostro caso è stato necessario configurare il certificato di firma affinché la compilazione Release andasse a buon fine.

Se Xcode richiede l’accesso al certificato Apple Development nel portachiavi, autorizzalo.

6. Compilare AltTab

In Xcode, seleziona lo schema Debug per verificare che il progetto funzioni.

Premi:

⌘ + B

Se compare Build Succeeded, prova ad avviare l’applicazione con il pulsante Run.

Successivamente passa alla configurazione Release:

1. Apri Product → Scheme → Edit Scheme.
2. Seleziona Run.
3. Imposta Build Configuration su Release.
4. Chiudi la finestra.
5. Premi nuovamente ⌘ + B.

Attendi il messaggio:

Build Succeeded

Nota: nei nostri tentativi iniziali, le compilazioni da Terminale fallivano senza un errore esplicito, mentre la compilazione tramite interfaccia Xcode è riuscita. Per questo consiglio di utilizzare Xcode.

7. Installare AltTab in Applicazioni

Una volta completata la compilazione Release:

1. In Xcode apri il Project Navigator con ⌘ + 1.
2. Espandi Products.
3. Clicca con il tasto destro su AltTab.app.
4. Seleziona Show in Finder.
5. Verifica di aver individuato la versione Release.

Se esiste già una versione di AltTab, chiudila e conservala come backup.

Copia quindi AltTab.app nella cartella:

/Applications/

Avvia l’applicazione:

open /Applications/AltTab.app

Se funziona, AltTab può essere utilizzata indipendentemente da Xcode.

8. Correggere il numero di versione

Nel nostro caso, il comando:

defaults read /Applications/AltTab.app/Contents/Info CFBundleShortVersionString

restituiva:

#VERSION#

Questo accade perché il progetto contiene un segnaposto per la versione, normalmente sostituito durante la preparazione della distribuzione.

La soluzione migliore è correggere il valore nel progetto prima della compilazione, evitando di modificare un’applicazione già firmata.

Apri il file Info.plist utilizzato dal target e imposta:

CFBundleShortVersionString = 10.12.0

Poi ricompila la Release.

Modificare direttamente Info.plist dentro /Applications/AltTab.app invalida invece la firma digitale e può impedire l’avvio dell’applicazione.

9. Attivare l’avvio automatico

Per avviare AltTab automaticamente quando accedi al Mac:

1. Apri Impostazioni di Sistema.
2. Vai su Generali → Elementi login ed estensioni.
3. Nella sezione Apri al login, premi +.
4. Seleziona /Applications/AltTab.app.
5. Premi Aggiungi.

In alternativa, puoi attivare Launch at login direttamente nelle preferenze di AltTab.

L’applicazione verrà avviata automaticamente all’accesso al tuo account, senza dover aprire Xcode.

⸻

Problemi incontrati e relative soluzioni

Problema	Soluzione
CocoaPods non installato	brew install cocoapods
Dipendenze mancanti	pod install
Errori relativi a macOS 10.12	Impostare deployment target a 12.0
API Swift deprecate trattate come errori	Disattivare SWIFT_TREAT_WARNINGS_AS_ERRORS
Signing requires a development team	Selezionare il Personal Team in Xcode
Richieste ripetute del portachiavi	Autorizzare il certificato Apple Development
Build Failed senza errori espliciti da Terminale	Provare la compilazione tramite Xcode
App danneggiata o bloccata da macOS	Verificare la firma ed evitare build ad hoc incomplete
Versione visualizzata come #VERSION#	Correggere il valore nel progetto e ricompilare
AltTab si chiude quando interrompi Xcode	Installare e avviare la Release da /Applications
AltTab non parte all’accensione	Aggiungerla agli elementi di login

Risultato finale

Con questa procedura ottieni una copia di AltTab 10.12.0 compilata per Apple Silicon, installabile in /Applications e configurabile per l’avvio automatico.

La compilazione Release è riuscita nel nostro caso. Rimane importante verificare che l’app installata sia firmata correttamente e che funzioni anche dopo un riavvio, senza Xcode aperto.
