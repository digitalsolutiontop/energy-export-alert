# Energy Export Alert

Uno script per monitorare l'energia esportata da un impianto fotovoltaico (balcone o tradizionale) e inviare notifiche di allerta quando si supera una soglia preimpostata. Creato per sistemi EcoFlow e compatibile con dispositivi Shelly che supportano lo scripting.

---

## Funzionalità
- Monitora l'energia esportata utilizzando dispositivi Shelly.
- Invia notifiche tramite Shelly Cloud quando la soglia di esportazione viene superata.
- Configurabile per qualsiasi impianto grazie a parametri personalizzabili.

## Prerequisiti
- Un dispositivo **Shelly** compatibile con il supporto per scripting, come Shelly Pro 3EM.
- Accesso alla rete e configurazione del dispositivo tramite Shelly Cloud.

## Configurazione
1. Sostituisci il valore di `DEVICE_ID` con l'ID del tuo dispositivo Shelly.
2. Modifica `EXPORT_THRESHOLD` per impostare la soglia di energia esportata (in Watt, valore negativo).
3. Regola `CHECK_INTERVAL` per definire l'intervallo di controllo (in millisecondi).

## Script
Inserisci il seguente script nel pannello di scripting del tuo dispositivo Shelly:

```javascript
let EXPORT_THRESHOLD = -500; // Soglia di energia esportata (in Watt, valore negativo)
let CHECK_INTERVAL = 10000; // Controllo ogni 10 secondi
let DEVICE_ID = "<INSERISCI IL TUO DEVICE ID QUI>"; // ID del dispositivo Shelly

// Funzione per inviare una notifica tramite Shelly Cloud
function sendCloudNotification(exportPower) {
    Shelly.call(
        "Shelly.Cloud.SendNotification", {
            title: "Energia Restituita Alta",
            description: `L'energia restituita ha superato la soglia di ${Math.abs(EXPORT_THRESHOLD)} W.\nEnergia attuale: ${Math.abs(exportPower)} W`
        },
        function (res, err_code, err_msg) {
            if (err_code !== 0) {
                print("Errore nell'invio della notifica push:", err_msg);
            } else {
                print("Notifica push inviata con successo tramite Shelly Cloud.");
            }
        }
    );
}

// Funzione per controllare l'energia esportata
function checkExportedEnergy() {
    Shelly.call(
        "Shelly.GetStatus", { id: DEVICE_ID },
        function (res, err_code, err_msg) {
            if (err_code === 0) {
                if (res.total_act_power !== undefined) {
                    let totalExportedPower = res.total_act_power;

                    print("Energia esportata attuale:", totalExportedPower);

                    if (totalExportedPower < EXPORT_THRESHOLD) { // Deve essere inferiore alla soglia
                        print("Soglia di energia restituita superata, invio notifica.");
                        sendCloudNotification(totalExportedPower);
                    }
                } else {
                    print("Dati di energia esportata non disponibili nella risposta:", JSON.stringify(res));
                }
            } else {
                print("Errore nel recupero dello stato:", err_msg);
            }
        }
    );
}

// Avvia il controllo periodico
Timer.set(CHECK_INTERVAL, true, checkExportedEnergy);
