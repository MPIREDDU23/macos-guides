# Compilare e installare AltTab 10.12.0 su macOS

> Guida pratica per compilare AltTab da sorgente su Apple Silicon, installare una build Release e avviarla automaticamente al login.
>
> **Ambiente usato:** Mac Apple Silicon, Xcode 27, Homebrew, CocoaPods. Le modifiche sotto descrivono una soluzione sperimentata su questa configurazione, non una procedura ufficiale del progetto.

## Indice

1. [Prerequisiti](#1-prerequisiti)
2. [Dipendenze](#2-dipendenze)
3. [Deployment target](#3-deployment-target)
4. [Warning trattati come errori](#4-warning-trattati-come-errori)
5. [Firma del codice](#5-firma-del-codice)
6. [Build Debug e Release](#6-build-debug-e-release)
7. [Installazione](#7-installazione)
8. [Versione mostrata come `#VERSION#`](#8-versione-mostrata-come-version)
9. [Avvio automatico](#9-avvio-automatico)
10. [Problemi comuni](#10-problemi-comuni)

## 1. Prerequisiti

- Un Mac con processore Apple Silicon.
- Xcode installato e configurato.
- Homebrew e Git.
- I sorgenti di [AltTab](https://github.com/lwouis/alt-tab-macos), versione **10.12.0**.

Se hai scaricato l'archivio ZIP ed estratto la cartella in Download:

```bash
cd ~/Downloads/alt-tab-macos-10.12.0
```

> **Nota:** se la cartella si trova altrove, modifica il percorso nel comando.

## 2. Dipendenze

Installa CocoaPods, se necessario:

```bash
brew install cocoapods
```

Dalla radice dei sorgenti:

```bash
pod install
open alt-tab-macos.xcworkspace
```

Apri il **workspace `.xcworkspace`** e non soltanto il progetto `.xcodeproj`, perché AltTab utilizza CocoaPods.

## 3. Deployment target

In questa configurazione alcune impostazioni puntavano a macOS 10.12, causando problemi con la toolchain recente. Abbiamo portato il deployment target a **macOS 12.0**.

Esegui i seguenti comandi **dalla radice dei sorgenti**:

```bash
sed -i '' \
  's/MACOSX_DEPLOYMENT_TARGET = 10\.12;/MACOSX_DEPLOYMENT_TARGET = 12.0;/g' \
  alt-tab-macos.xcodeproj/project.pbxproj

sed -i '' \
  's/MACOSX_DEPLOYMENT_TARGET = 10\.12;/MACOSX_DEPLOYMENT_TARGET = 12.0;/g' \
  Pods/Pods.xcodeproj/project.pbxproj

sed -i '' \
  's/MACOSX_DEPLOYMENT_TARGET = 10\.12/MACOSX_DEPLOYMENT_TARGET = 12.0/g' \
  config/base.xcconfig
```

Controlla il valore effettivo:

```bash
xcodebuild \
  -workspace alt-tab-macos.xcworkspace \
  -scheme Debug \
  -configuration Debug \
  -showBuildSettings 2>/dev/null \
  | grep 'MACOSX_DEPLOYMENT_TARGET ='
```

Il valore atteso per il target dell'app è `12.0`. L'output può includere più righe relative a target differenti.

> **Attenzione:** `pod install` può rigenerare alcune impostazioni del progetto Pods. Se lo riesegui, ricontrolla il deployment target.

## 4. Warning trattati come errori

Con Xcode 27, un avviso relativo a un'API deprecata, tra cui `ultraDark`, bloccava la compilazione. Per impedire che i warning siano considerati errori, aggiungi queste impostazioni **una sola volta** a `config/base.xcconfig`:

```xcconfig
SWIFT_TREAT_WARNINGS_AS_ERRORS = NO
GCC_TREAT_WARNINGS_AS_ERRORS = NO
```

Puoi aprire il file con:

```bash
open -a TextEdit config/base.xcconfig
```

Questa modifica non risolve eventuali errori reali di compilazione e non elimina i warning.

## 5. Firma del codice

In Xcode:

1. Seleziona il progetto **alt-tab-macos** nel navigatore.
2. Seleziona il target **alt-tab-macos**.
3. Apri **Signing & Capabilities**.
4. Abilita **Automatically manage signing**.
5. Imposta **Team** sul tuo account, ad esempio **Personal Team**.
6. Controlla sia **Debug** sia **Release**.

Se Xcode richiede l'autorizzazione per il certificato **Apple Development** nel Portachiavi, concedila.

> **Importante:** non correggere problemi di firma applicando `codesign --sign -` all'app installata. La firma ad hoc ha causato blocchi all'avvio nei nostri tentativi. Usa la firma gestita da Xcode e verifica che l'app risultante si apra correttamente.

## 6. Build Debug e Release

### Verifica Debug

1. Seleziona lo schema **Debug** in Xcode.
2. Premi **⌘B** per compilare.
3. Se la compilazione riesce, prova **Run** per verificare il funzionamento.

### Compilazione Release

1. Apri **Product → Scheme → Edit Scheme**.
2. Seleziona **Run**.
3. Imposta **Build Configuration** su **Release**.
4. Chiudi la finestra e premi **⌘B**.
5. Verifica la comparsa di **Build Succeeded**.

Nel caso documentato, la compilazione da interfaccia Xcode è riuscita, mentre alcuni tentativi iniziali con `xcodebuild` da Terminale fallivano senza diagnostica chiara.

## 7. Installazione

Dopo aver compilato la Release:

1. Apri **Project Navigator** con **⌘1**.
2. Espandi **Products**.
3. Fai clic destro su **AltTab.app → Show in Finder**.
4. Verifica che l'app provenga dalla build **Release**.
5. Chiudi eventuali istanze di AltTab e conserva un backup della versione già installata.
6. Copia **AltTab.app** in `/Applications`.

Avvia la copia installata:

```bash
open /Applications/AltTab.app
```

Per verificare la firma senza modificarla:

```bash
codesign --verify --deep --strict --verbose=2 /Applications/AltTab.app
```

Se l'app non si apre, controlla gli errori prima di intervenire sulla firma o sulle impostazioni di sicurezza.

## 8. Versione mostrata come `#VERSION#`

Nel caso documentato, questo comando:

```bash
defaults read /Applications/AltTab.app/Contents/Info CFBundleShortVersionString
```

restituiva:

```text
#VERSION#
```

Il progetto contiene quindi un segnaposto non sostituito durante questa compilazione. Se l'app funziona, il problema può essere soltanto di visualizzazione della versione.

Per correggerlo, individua il file `Info.plist` effettivamente usato dal target o l'impostazione di build che genera `CFBundleShortVersionString`, imposta il valore `10.12.0` **nei sorgenti** e ricompila la Release.

**Non modificare direttamente** `Contents/Info.plist` nell'app già installata: invalideresti la firma esistente.

## 9. Avvio automatico

Su macOS:

1. Apri **Impostazioni di Sistema**.
2. Vai su **Generali → Elementi login ed estensioni**.
3. Nella sezione **Apri al login**, premi **+**.
4. Seleziona `/Applications/AltTab.app`.
5. Conferma con **Aggiungi**.

In alternativa, verifica se nelle preferenze di AltTab è disponibile l'opzione **Launch at login**.

L'app verrà avviata quando accedi al tuo account, senza bisogno di Xcode.

## 10. Problemi comuni

| Problema | Soluzione |
| --- | --- |
| `pod` non trovato | Installa CocoaPods con `brew install cocoapods`. |
| Dipendenze mancanti | Esegui `pod install` e apri il `.xcworkspace`. |
| Deployment target macOS 10.12 non supportato | Aggiorna i file di progetto e `config/base.xcconfig` a macOS 12.0. |
| Warning Swift trattati come errori | Imposta `SWIFT_TREAT_WARNINGS_AS_ERRORS = NO`. |
| `Signing requires a development team` | Seleziona il Team in **Signing & Capabilities**. |
| Build da Terminale fallita senza errori espliciti | Prova la compilazione da Xcode e consulta il Report Navigator. |
| App segnalata come danneggiata | Verifica la firma; non applicare firme ad hoc o disattivare Gatekeeper. |
| Versione `#VERSION#` | Correggi la configurazione nei sorgenti e ricompila. |
| App chiusa quando interrompi Xcode | Avvia la copia Release installata in `/Applications`. |
| App non avviata al login | Aggiungila agli elementi login di macOS. |

---

## Riferimenti

- [AltTab, repository ufficiale](https://github.com/lwouis/alt-tab-macos)
- [CocoaPods](https://cocoapods.org/)

*Guida basata su una compilazione locale di AltTab 10.12.0. Le opzioni e le schermate di Xcode possono variare tra versioni.*
