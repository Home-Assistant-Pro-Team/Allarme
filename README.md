https://user-images.githubusercontent.com/62516592/217784134-2fa7c4e3-53d5-4c80-a358-ac2b9ae5cb4b.mp4


# Allarme

Il pkg è diviso in più cartelle.

I file principali si trovano nella cartella *alarm_control_panel*

## alarm_control_panel:

#### GERNERAL_ALARM:

- viene creata l'entità alarm_control_panel
- viene creato *input_boolean.alarm_triggered_state*:	
	-  Utilizzato per capire se l'allarme è scattato da quando è stato inserito. 
	-  Lo stato di questo di bool viene utilizzato in altri file del pkg
- vengono creati due gruppi: **exclude_alarm_entities include_alarm_entities** che rendono possibile includere ed escludere entità non necessarie al pkg o non viste in automatico. Le entità funzionano per i seguenti file:
	- SENSORI BATTERY  "file: person": 
		- se inseriti in include_alarm_entities il controllo stato batteria verrà effettuato **solo** su questi dispositivi
   		- se inseriri in exclude_alarm_entities il controllo stato batteria avviene su **tutti** i device con app installata esclusi quelli inseriti
		- se non inseriti nei gruppi vengono presi tutti i sensori batteria dei device con **app companion** installata
	- PULSANTI FISICI "file: real_input_alarm":
		- E' possibile **solo** escludere sensori 
		- NB: entita che utilizzano il servizio *event: shelly.click* (shelly utilizzati con pulsanti 'Momentary') non possono essere ne esclusi ne inseriti
	- SENSORI PER ALLARME file:"armed_away & armed_night":
		- E' possibile mettere binary_sensor in entrambi i gruppi, quindi è possibile sia aggiungere che toglie entità dalla lista contemporaneamente
		- NB: Usare solo binary_sensor
	- ALLARME SCATTATO: file: action_alarm:
		- Se viene disattivato durante il pending(luci accese) vengono spente le luci con la possibilità di lasciare accese quelle inserite nel group.exclude_alarm_entities, solo se **dopo tramonto**
	
#### ARMED_NIGHT E ARMED_AWAY:

- viene creato un template select che permette di avere la lista di tutti i sensori utiliti per il funzionamento dell'allarme filtrati per dominio e device_class 
- quando scegliamo un sensore dal select in automatico viene aggiunto o se presente rimosso dal group.list_sensor_alarm_xxxx (sensori usati per allarme)
- quando lo stato dell'allarme passa a ARMED_NIGHT o ARMED_AWAY. vengono esclusi (non tolti dal gruppo) dal trigger i sensori con *device_class:window* che si trovano nello stato on
- SOLO PER **armed_night** quando una finestra che è stato esclusa dal funzionamento viene chiusa dopo 10 minuti viene reinserita tra i sensori per controllo allarme
- l'automazione gestisce tutto. quindi avviene il trigger quando un sensore passa allo stato on
- e possibile escludere/includere i singoli sensori anche con allarme in corso da ui
- nelle notifiche si sa sempre qual'è l'ultimo sensore che realmente ha cambiato di stato con orario
- E' possibile aggiungere o rimuovere i sensori dalla lista del select mettendo le entità nei gruppi *exclude_alarm_entities,include_alarm_entities* (file general_alarm)
- NB assicurarsi di avere corretto **device_class: window** sui device utilizzati per le finestre. Se non correttamente impostato, non viene tenuto conto dello stato della finestra, quindi non viene esclusa dal funzionamento se lasciata aperta ma viene trattata come un sensore generico dell'allarme.

## skill_device

Nella cartella **skill_device è possibile** usare i singoli file in base all'uso personale. *Nessuno è indispensabile per il funzionamento dell'allarme*.

#### CAMPANELLO:

Nel mio caso ho un esp al campanello di casa per notificare quando suonano alla porta. L'ho utilizzato come 'piccola sirena' da interno(passatemi in termine)

- Se scatta allarme fai suonare il campanello ad intermittenza (1sec X 30 tot 60sec)
- L'intermittenza del campanello si può in inttermepere anche disattivando l'allarme
#### CCTV:

- Invio snapshot (android) con action per vedere telecamere live o registrazione di 30sec.

PS:non sono riuscito su android ad avere l'anteprima del video registrato perchè il servizio camera.record genera un file tmp che alla fine della registrazione viene rinominato nel formato da noi scelto.

#### CONTROLCODE:

- ATTENZIONE: file funzionante solo con la card
- ATTENZIONE per utilizzare questo file il codice allarme, il codice emergenza e il codice setting *non può iniziare con zero*
- Il file esegue un controllo per tentativi inserimento codice disarmo. 
	- E' possibile scegliere i tentativi codice disarmo oltre il quale scatta l'allarme (input_number.alarm_code_attempts).
	- Se tentativi codice impostato su 0 il controllo codice errati viene disattivato
	- E' possibile impostare un codice di 'emergenza' che permette di disattivare l'allarme inviando una notifica di allert
	- Quando vengono superati i tentativi impostati l'allarme passa nello stato scattato
	- Quando inseriamo un codice sul "tastierino" e non si effettua la conferma o non viene cancellato, dopo un minuto torna allo stato inserire codice.

- Con il codice 'input_number.setting_code' utilizzato dal tastierino è possibile cambiare le impostazioni dalla card con allarme inserito (toglie lock)

#### DETECT_JAMMER:
Il controllo jummer avviene in due modi: tramite i device che passano allo stato offline, tramite la differenza della media db misura
- viene creato un template select che permette di avere la lista di tutti i sensori filtrati per dominio e device_class utiliti per il funzionamento
- E' possibile personalizzare di UI: inserire nel gruppo i dispositivi
- E' possibile personalizzare di UI: quanti dispositivi passano allo stato offline (voti) del group.jammer_detect_device (scelti dal select)
- E' possibile personalizzare da UI: se eseguire il controllo db solo per Shelly, solo per Esphome o per tutti.
- E' possibile personalizzare da UI: quanta differenza espressa in db deve essere rilevata per far scattare l'allarme
#### LED ALARM: 
Ho un led vicino e fuori la porta che mi indica lo stato dell'allarme
- spento se disarmed
- acceso se non disarmed
- lampegiante se scattato. Lo stato lampeggiante termina quando l'allarme passa in disarmed e sono trascorsi 10 min 
#### NFC:
- attivo SOLO allarme away 
- disattivo away e night 
- viene eseguito anche il controllo id del cellulare
#### PERSON
Ha un doppio funzionamento:
  - Attivo  e disattiva l'allarme *away* in base alla presenza in casa
  - Notifica batteria scarica cellulare se allarme attivo in *away*. Di default controlla tutti i dispositivi con app installa ma è possibile scegliere i singoli device inserendolo in group.include_alarm_entities oppure i singoli dispositivi da escludere con group.exclude_alarm_entities (file GERNERAL_ALARM) 
#### REAL_IMPUT_ALARM: *DEPRECATO FILE SOSTITUITO DA CARTELLA DETACHED* 
Utile a chi non vuole usare detached o non ha dispositivi compatibili

#### DETACHED

  - **detached_shelly.yaml**: recurera tramite api l'impostazione dei tasti dei dispositivi e crea url da inviare alli stessi. Altre note nel file
  - **general_detached_alarm.yaml**: è composto da due automazioni
  	- Notifica e trigger alarme alla pressione tasti 
  	- Detached alarm: abilita detached quando allarme scattato. Rispristina impostazione tasti precedente quando l'allarme torna disarmed ed è scattato. Questa funzione è per dispositivi Shelly e Esphome (con apposito file)

##### STATUS_ROUTER_UPS:

notifica se va via o torna la corrente 

notifica se va via o torna internet (utilizzo un fritz6890 con fallback lte)

## SCENE

##### ACTION_ALARM:

Vengono decise le azioni della casa (non notifiche) nel momento in cui scatta un sensore con allarme inserito.

E' possibile inserire le automazione che vengano disattivate inserendole nel group (es. accensione luce con pir)
- Quando rilevata (solo) per la prima volta l'infrazione (pending) :
	- si accendono tutte le luci 
	- viene salvato lo stato delle automazioni da mettere in off con una scena
	- vengono messe le automazioni inserite nel gruppo in off
- Quando scatta l'allarme
	- vengono spente tutte le luci
	- vengono chiuse tutte le tapparelle
- Quando viene tolto l'allarme
	- Se viene disattivato durante il pending(luci accese)
		- vengono spente le luci con la possibilità di lasciare accese quelle inserite nel group.exclude_alarm_entities (file: general_alarm) solo se dopo tramonto
		- se prima del tramonto vengono spente tutte le luci
- Quando viene tolto allarme dopo che è scattato
	- vengono riattivate le automazioni con la scena creata prima
