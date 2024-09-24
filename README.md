# PROGETTO D'ESAME- RETI DI CALCOLATORI 2024/2025
## Studenti: Lanzillo Giovanna [0124/2682] e Inguti Mara [0124/2637]
## INTRODUZIONE  
*Il progetto prevede lo sviluppo di un'applicazione client/server parallelo per la gestione degli esami universitari. L'applicazione consentirà di gestire simultaneamente le richieste di più studenti e segreteria universitaria.*
## TRACCIA:Università
*Scrivere un’applicazione client/server parallelo per gestire gli esami universitari*

### Gruppo 2 studenti
Il server universitario ad ogni richiesta di prenotazione invia alla segreteria il numero di prenotazione progressivo assegnato allo studente e la segreteria a sua volta lo inoltra allo studente 

### Segreteria:
*Il server universitario ad ogni richiesta di prenotazione invia alla segreteria il numero di prenotazione progressivo assegnato allo studente e la segreteria a sua volta lo inoltra allo studente*

### Studente:
*Chiede alla segreteria se ci siano esami disponibili,invia una richiesta di prenotazione di un esame alla segreteria Server universitario, puo vedere l'aggiunta di nuovi esami,riceve la conferma di una  prenotazione di un esame.*
*Ad ogni richiesta di prenotazion,il server universitario invia alla segreteria il numero di prenotazione progressivo assegnato allo studente e la segreteria a sua volta lo inoltra allo studente.*

# SPECIFICHE DEL PROGETTO
#### Network.c
```
#include "../Source/lib.h"

// Funzione per creare un socket TCP IPv4
int Socket() {

    // Crea un socket TCP
    const int socketfd = socket(AF_INET, SOCK_STREAM, 0);
    if (socketfd < 0) {

        // Gestisce l'errore se la creazione del socket fallisce
        Error("Errore nella creazione del socket");
    }
    // Restituisce il descrittore del socket creato
    return socketfd;
}

// Funzione per impostare le opzioni del socket (riutilizzo dell'indirizzo)
void Sock_Options(const int socketfd) {
    const int option = 1;
    if (setsockopt(socketfd, SOL_SOCKET, SO_REUSEADDR, &option, sizeof(option)) == -1) {

        // Gestisce l'errore se l'impostazione delle opzioni fallisce
        Error("Errore durante il riutilizzo dell'indirizzo");
    }
}

// Funzione per collegare il socket a un indirizzo specificato
void Bind(const int socketfd, struct sockaddr_in address) {
    if (bind(socketfd, (struct sockaddr *) &address, sizeof(address)) < 0) {

        // Gestisce l'errore se il bind del socket fallisce
        Error("Errore nel bind del socket");
    }
}

// Funzione per mettere il socket in ascolto alle connessioni in entrata
int Listen(const int socketfd) {
    const int max_connections = 10;
    if (listen(socketfd, max_connections) < 0) {

        // Gestisce l'errore se la listen del socket fallisce
        Error("Errore nella listen del socket");
    }
    printf("LISTEN: Il canale è in ascolto.\n");
    return 0;
}

// Funzione per configurare un indirizzo IPv4 con il port specificato
struct sockaddr_in Configura_Indirizzo(const int port) {
    struct sockaddr_in address;
    address.sin_family = AF_INET;                   // Utilizza IPv4
    address.sin_addr.s_addr = htonl(INADDR_ANY);    // Accetta connessioni da qualsiasi interfaccia di rete
    address.sin_port = htons(port);                 // Imposta il numero di porta

    printf("\nCONFIG: Eseguita configurazione indirizzo sul port: %d.\n", port);

    // Restituisce l'indirizzo configurato
    return address;
}

// Funzione per accettare una connessione in entrata
int Accept(const int socketfd, struct sockaddr *address, socklen_t *adress_length) {

    // Accetta la connessione in entrata
    const int connfd = accept(socketfd, address, adress_length);
    if (connfd < 0) {

        // Gestisce l'errore se l'accettazione della connessione fallisce
        perror("Errore durante l'accettazione della connessione");
        return -1;
    }

    // Restituisce il descrittore del socket della nuova connessione
    return connfd;
}

// Funzione per chiudere il socket
void Close(const int socketfd) {
    if (close(socketfd) < 0) {

        // Gestisce l'errore se la chiusura del socket fallisce
        Error("Errore nella chiusura del socket");
    }
}

// Funzione per inviare un messaggio a un indirizzo specificato
void Sendto(const int socketfd, const char *messaggio, struct sockaddr_in address) {

    // Buffer per contenere il messaggio
    char buffer[MALLOC];

    // Formatta il messaggio nel buffer
    sprintf(buffer, "%s", messaggio);

    // Invia il messaggio
    const ssize_t n = sendto(socketfd, buffer, strlen(buffer), 0, (struct sockaddr *) &address, sizeof(address));
    if (n < 0) {

        // Gestisce l'errore se l'invio del messaggio fallisce
        Error("Errore durante la spedizione");
    }
}

// Funzione per ricevere un messaggio da un socket
char *Receive(const int socketfd, struct sockaddr_in *address) {

    // Buffer per contenere il messaggio ricevuto
    char buffer[MALLOC];

    // Alloca memoria per il messaggio
    char *messaggio = malloc(MALLOC + 1);
    if (messaggio == NULL) {

        // Gestisce l'errore se l'allocazione di memoria fallisce
        Error("Errore nella allocazione di memoria");
    }
    socklen_t len = sizeof(*address);
    memset(buffer, 0, sizeof(buffer));

    // Riceve il messaggio
    const ssize_t n = recvfrom(socketfd, buffer, MALLOC, 0, (struct sockaddr *) address, &len);
    if (n < 0) {

        // Gestisce l'errore se la ricezione del messaggio fallisce
        Error("Errore durante la ricezione");
    }

    // Termina il buffer con null character
    buffer[n] = '\0';

    // Copia il messaggio nel buffer allocato
    strncpy(messaggio, buffer, MALLOC);

    // Restituisce il messaggio ricevuto
    return messaggio;
}

// Funzione per connettersi a un server remoto
void Connessione_Client(int *socketfd, struct sockaddr_in *server_address, const int port) {

    // Crea un socket
    *socketfd = Socket();

    // Configura l'indirizzo del server
    *server_address = Configura_Indirizzo(port);
    int tentativi = 5;

    if (inet_pton(AF_INET, MAIN_IP, &server_address->sin_addr) <= 0) {

        // Gestisce l'errore se la conversione dell'indirizzo IP fallisce
        Error("Errore nella conversione dell'indirizzo IP");
    }

    // Messaggio di inizio connessione
    printf("CONNECT: Inizio tentativi (%d) di connessione al server.\n", tentativi);
    while (connect(*socketfd, (struct sockaddr *) server_address, sizeof(*server_address)) < 0) {
        tentativi--;

        // Stampa il numero di tentativi rimanenti
        printf("Connessione fallita. Numero tentativi: %d\n", tentativi);
        if (tentativi == 0) {

            // Gestisce l'errore se i tentativi di connessione sono esauriti
            Error("Errore nella connessione al server");
        }

        // Attende prima di un nuovo tentativo
        sleep(3);
    }

    // Messaggio di connessione riuscita
    printf("CONNECT: Canale connesso con successo al server con %d tentativi rimanenti.\n", *socketfd);
}
```

Network.c offre una serie di funzioni per la gestione dei socket TCP IPv4. Le operazioni supportate comprendono la creazione e configurazione dei socket, il binding a un indirizzo, l'ascolto e l'accettazione di connessioni in entrata, l'invio e la ricezione di dati, la chiusura dei socket, nonché la connessione a server remoti.

### Dettagli principali
#### Creazione e Gestione dei Socket:
La funzione Socket() si occupa di creare un socket TCP IPv4, gestendo eventuali errori durante il processo. La funzione Close() chiude un socket e gestisce possibili errori associati.
#### Configurazione dei Socket: 
La funzione Sock_Options() abilita l'opzione SO_REUSEADDR per permettere il riutilizzo dell'indirizzo IP. La funzione Configura_Indirizzo() imposta una struttura sockaddr_in con il numero di porta specificato.
#### Binding e Ascolto: 
La funzione Bind() associa il socket a un indirizzo locale. La funzione Listen() abilita il socket per ricevere connessioni in arrivo.
#### Gestione delle Connessioni:
La funzione Accept() accetta connessioni in ingresso e restituisce il descrittore del socket associato alla nuova connessione.
#### Trasferimento dei Messaggi:
La funzione Sendto() invia un messaggio a un indirizzo specifico, mentre la funzione Receive() riceve un messaggio da un socket e lo restituisce come stringa.
#### Connessione a Server Remoti:
La funzione Connessione_Client() crea un socket, configura l'indirizzo del server e stabilisce una connessione con il server specificato.

#### Framework.c

```
#include "../Source/lib.h"

// Funzione per pulire il buffer di input (rimuovere caratteri rimanenti dopo l'input)
void Clear_Input_Buffer() {
    int c;
    while ((c = getchar()) != '\n' && c != EOF) {
        // Continua a leggere caratteri fino a quando non si raggiunge una nuova linea o EOF
    }
}

// Funzione per gestire gli errori, stampando il messaggio di errore fornito
void Error(const char *msg) {

    // Stampa il messaggio di errore con perror
    perror(msg);

    // Esce dal programma con stato di errore
    exit(1);
}

// Funzione per estrarre un token dal puntatore ai dati, separati dal delimitatore ';'
char *Estrai_Token(char **dati) {

    // Estrae il token fino al prossimo ';'
    const char *token = strsep(dati, ";");
    if (token == NULL) {

        // Stampa un errore se il token è NULL
        fprintf(stderr, "ERROR: Token NULL\n");
        return NULL;
    }

    // Restituisce una copia allocata dinamicamente del token estratto
    return strdup(token);
}

// Funzione per connettersi al database SQLite
sqlite3 *ConnessioneDB() {
    sqlite3 *db;

    // Apre la connessione al database SQLite
    const int rc = sqlite3_open("identifier.sqlite", &db);
    if (rc) {

        // Stampa un errore se non riesce ad aprire il database
        fprintf(stderr, "Errore apertura database: %s\n", sqlite3_errmsg(db));

        // Esce dal programma con stato di errore
        exit(1);
    }
    printf("SQLITE: Connessione al database SQLite eseguita correttamente.\n");

    // Restituisce il puntatore alla connessione al database
    return db;
}

// Funzione per eseguire una query SQL sul database SQLite
int Esegui_Query(sqlite3 *db, const char *sql, sqlite3_stmt **sql_query) {

    // Prepara la query SQL
    const int sql_result = sqlite3_prepare_v2(db, sql, -1, sql_query, NULL);
    if (sql_result != SQLITE_OK) {

        // Stampa un errore se la preparazione della query fallisce
        fprintf(stderr, "Errore SQLite: %s\n", sqlite3_errmsg(db));

        // Restituisce il risultato dell'operazione SQLite
        return sql_result;
    }
    // Restituisce OK se la preparazione della query è andata a buon fine
    return SQLITE_OK;
}

```
### Dettagli specifici

Framework.c fornisce una serie di funzioni essenziali per la gestione degli input, la gestione degli errori, l'estrazione di token e l'interazione con un database SQLite in linguaggio C. Le funzionalità offerte includono la pulizia del buffer degli input, la visualizzazione di messaggi di errore, l'estrazione di token da stringhe delimitate e la connessione a un database SQLite per l'esecuzione di query.

### Dettagli principali
#### Gestione degli Input: 
La funzione Clear_Input_Buffer() si occupa di svuotare il buffer di input dopo la lettura di un valore.
#### Gestione degli Errori: 
La funzione Error() visualizza un messaggio di errore generico e termina il programma con un codice di errore.
#### Estrazione di Token:
La funzione Estrai_Token() estrae un token da una stringa delimitata dal carattere ; e restituisce una copia del token allocata dinamicamente.
#### Interazione con Database SQLite: 
La funzione ConnessioneDB() apre una connessione al database SQLite chiamato "identifier.sqlite" e restituisce un puntatore alla connessione aperta. La funzione Esegui_Query() prepara ed esegue una query SQL sul database SQLite indicato e restituisce il risultato dell'operazione.

#### Segreteria_Helper.c
```
#include "../Source/lib.h"

// Funzione thread per la gestione delle operazioni dello studente
void *Thread_Gestione_Studente(void *arg) {

    // Cast dell'argomento al tipo corretto
    const Thread_Segreteria *args = arg;

    // Estrazione dei parametri necessari dalla struttura args
    const int studentefd = args->studentefd;
    struct sockaddr_in segreteria_address = args->segreteria_address;
    const int serverfd = args->serverfd;
    struct sockaddr_in server_address = args->server_address;
    const int counter = args->counter;

    // Loop infinito per gestire le comunicazioni con lo studente
    while (1) {
        printf("\nIn attesa di richieste dallo studente %d.\n", counter);

        // Ricezione dei dati inviati dallo studente
        char *dati_studente = Receive(studentefd, &segreteria_address);
        if (strlen(dati_studente) == 0) {
            printf("Lo studente %d ha chiuso la connessione.\n", counter);

            // Chiude il socket dello studente
            Close(studentefd);

            // Termina il thread
            return NULL;
        }
        printf("\nRicezione dei dati dello studente %d.    | %s\n", counter, dati_studente);

        // Invio dei dati ricevuti al server
        Sendto(serverfd, dati_studente, server_address);
        printf("Invio dei dati al server.               | %s\n", dati_studente);

        // Ricezione della risposta dal server
        char *risposta_server = Receive(serverfd, &server_address);
        if (strlen(risposta_server) == 0) {
            printf("Il server ha chiuso la connessione.\n");

            // Chiude il socket del server
            Close(serverfd);

            // Termina il thread
            return NULL;
        }
        printf("Ricezione dei dati dal server.          | %s\n", risposta_server);

        // Invio della risposta ricevuta allo studente
        Sendto(studentefd, risposta_server, segreteria_address);
        printf("Invio dei dati allo studente %d.         | %s\n", counter, risposta_server);

        // Liberazione della memoria allocata per i dati
        free(dati_studente);
        free(risposta_server);
    }
}

// Funzione thread per gestire l'input dalla segreteria
void *Thread_Input_Segreteria(void *arg) {
    while (1) {

        // Ottiene l'opzione selezionata dalla segreteria
        const int input = Selezione_Richiesta_Segreteria();
        if (input == 3) {

            // Gestisce la richiesta di aggiunta di un esame
            Richiesta_Aggiunta_Esame(arg);
        } else if (input == 4) {

            // Gestisce la richiesta di aggiunta di un appello
            Richiesta_Aggiunta_Appello(arg);
        }
    }
}

// Funzione per permettere alla segreteria di selezionare un'operazione
int Selezione_Richiesta_Segreteria() {
    int option;

    // Mostra le opzioni disponibili per la segreteria
    printf("\nLISTA OPERAZIONI SEGRETERIA:\n");
    printf("3) Aggiungi Esame;\n");
    printf("4) Aggiungi Appello;\n");
    printf("Scegli un'opzione.\n\n");

    // Legge l'opzione selezionata dall'utente
    scanf("%d", &option);

    // Pulisce il buffer di input
    Clear_Input_Buffer();

    // Restituisce l'opzione selezionata
    return option;
}

// Funzione per gestire la richiesta di aggiunta di un esame da parte della segreteria
void Richiesta_Aggiunta_Esame(const void *arg) {

    // Cast dell'argomento al tipo corretto
    const Thread_Segreteria *args = arg;

    // Estrazione dei parametri necessari dalla struttura args
    const int serverfd = args->serverfd;
    struct sockaddr_in server_address = args->server_address;
    char buffer[MALLOC];

    // Legge il Nome dell'esame
    char nome_esame[50];
    printf("\nInserisci il nome dell'esame da aggiungere: ");
    if (fgets(nome_esame, sizeof(nome_esame), stdin) != NULL) {

        // Rimuove il newline dal nome dell'esame
        nome_esame[strcspn(nome_esame, "\n")] = 0;
    }

    // Legge i CFU dell'esame
    int cfu;
    printf("Inserisci i CFU dell'esame: ");
    scanf("%d", &cfu);
    Clear_Input_Buffer();

    // Legge il Corso di Studi dell'esame dell'esame
    char corso_di_studi[50];
    printf("Inserisci il corso di studi dell'esame: ");
    if (fgets(corso_di_studi, sizeof(corso_di_studi), stdin) != NULL) {

        // Rimuove il newline dal corso di studi
        corso_di_studi[strcspn(corso_di_studi, "\n")] = 0;
    }

    // Formatta i dati dell'esame da inviare al server
    snprintf(buffer, sizeof(buffer), "3;%s;%d;%s;", nome_esame, cfu, corso_di_studi);

    // Invia i dati al server
    Sendto(serverfd, buffer, server_address);

    // Riceve la conferma dal server sull'operazione
    *buffer = *Receive(serverfd, &server_address);
    if (strcmp(buffer, "SUCCESS") == 0) {
        printf("L'esame è stato aggiunto con successo.\n");
    } else if (strcmp(buffer, "FAILURE") == 0) {
        printf("Si è verificato un errore durante l'aggiunta dell'esame.\n");
    }
}

// Funzione per gestire la richiesta di aggiunta di un appello da parte della segreteria
void Richiesta_Aggiunta_Appello(const void *arg) {

    // Cast dell'argomento al tipo corretto
    const Thread_Segreteria *args = arg;

    // Estrazione dei parametri necessari dalla struttura args
    const int serverfd = args->serverfd;
    struct sockaddr_in server_address = args->server_address;
    char buffer[MALLOC];

    // Legge l'ID dell'esame
    int id_esame;
    printf("\nInserisci l'ID dell'esame per l'appello: ");
    scanf("%d", &id_esame);
    Clear_Input_Buffer();

    // Legge il Nome dell'appello
    char nome_appello[50];
    printf("Inserisci il nome dell'appello: ");
    if (fgets(nome_appello, sizeof(nome_appello), stdin) != NULL) {

        // Rimuove il newline dal nome dell'appello
        nome_appello[strcspn(nome_appello, "\n")] = 0;
    }

    // Legge la Data dell'appello
    char data_appello[11];
    printf("Inserisci la data dell'appello (formato YYYY-MM-DD): ");
    if (fgets(data_appello, sizeof(data_appello), stdin) != NULL) {
        data_appello[strcspn(data_appello, "\n")] = 0; // Rimuove il newline dalla data dell'appello
    }

    // Formatta i dati dell'appello da inviare al server
    snprintf(buffer, sizeof(buffer), "4;%d;%s;%s;", id_esame, nome_appello, data_appello);

    // Invia i dati al server
    Sendto(serverfd, buffer, server_address);

    // Riceve la conferma dal server sull'operazione
    *buffer = *Receive(serverfd, &server_address);
    if (strcmp(buffer, "SUCCESS") == 0) {
        printf("L'appello è stato aggiunto con successo.\n\n");
    } else if (strcmp(buffer, "FAILURE") == 0) {
        printf("Si è verificato un errore durante l'aggiunta dell'appello.\n\n");
    }
}
```

Segreteria_Helper.c fornisce una serie di funzioni per gestire le comunicazioni tra la segreteria e un server, l’elaborazione delle richieste degli studenti e l’esecuzione di operazioni su un database. Le funzionalità comprendono la creazione di thread per la gestione delle interazioni con studenti e per l’acquisizione di input da parte della segreteria, l’invio e la ricezione di dati tramite socket, la formattazione dei messaggi e l’esecuzione di operazioni specifiche, come l’aggiunta di esami e appelli.

#### Dettagli principal

#### Gestione dei Thread:
La funzione Thread_Gestione_Studente() avvia un thread dedicato alle comunicazioni con ciascuno studente. Thread_Input_Segreteria() crea un thread specifico per acquisire input dalla segreteria e avviare le operazioni corrispondenti.
#### Comunicazione con il Server:
Le funzioni Sendto() e Receive() sono utilizzate per inviare e ricevere dati tramite socket UDP. I dati sono inviati come stringhe formattate, utilizzando un separatore (;) per distinguere i tipi di operazione e i relativi parametri. Le risposte del server vengono elaborate per determinare il risultato e comunicarlo alla segreteria o allo studente.
#### Gestione delle Richieste della Segreteria:
La funzione Selezione_Richiesta_Segreteria() presenta un menu alla segreteria per scegliere l'operazione desiderata. Funzioni come Richiesta_Aggiunta_Esame() e Richiesta_Aggiunta_Appello() gestiscono le richieste per l'inserimento di esami e appelli, raccogliendo i dati necessari dall'utente, formattandoli e inviandoli al server.
### Interazione con il Database (Non implementata): 
Anche se l'interazione con il database non è implementata nel codice attuale, l'interazione con il database dovrebbe essere integrata per garantire la persistenza dei dati.

### Server_Helper.c
```
#include "../Source/lib.h"

// Funzione per Verificare le Credenziali nel database
void Verifica_Credenziali(sqlite3 *db, const int segreteriafd, const struct sockaddr_in segreteria_address, char *dati) {

    // Estrai Matricola e Password dai dati ricevuti
    strsep(&dati, ";");
    char *matricola = Estrai_Token(&dati);
    char *password = Estrai_Token(&dati);

    // Stampare le credenziali ricevute (debug)
    printf("Credenziali ricevute:\nMatricola: %s | Password %s\n", matricola, password);

    // Costruzione della query SQL per verificare le credenziali
    char sql_query_text[MALLOC];
    snprintf(sql_query_text, sizeof(sql_query_text),"SELECT * FROM STUDENTE WHERE MATRICOLA = '%s' AND PASSWORD = '%s';", matricola, password);

    // Preparazione dello statement SQL
    sqlite3_stmt *prepared_statement;
    int sqlite_result_code = sqlite3_prepare_v2(db, sql_query_text, -1, &prepared_statement, NULL);
    if (sqlite_result_code != SQLITE_OK) {

        // Invia un messaggio di errore alla segreteria
        Sendto(segreteriafd, "Errore query credenziali\n", segreteria_address);
        fprintf(stderr, "Errore SQLite: %s\n", sqlite3_errmsg(db));
        sqlite3_finalize(prepared_statement);
        return;
    }

    // Esecuzione della query SQL
    sqlite_result_code = sqlite3_step(prepared_statement);

    // Feedback dell'esito della query
    printf("Query SQL per verificare le credenziali eseguita correttamente.\n");

    // Gestione dei risultati della query
    if (sqlite_result_code == SQLITE_ROW) {

        // Se le credenziali sono corrette, invia SUCCESS e il piano di studi
        char *piano_di_studi = strdup((char *) sqlite3_column_text(prepared_statement, 3));
        char buffer[MALLOC];
        snprintf(buffer, sizeof(buffer), "SUCCESS;%s", piano_di_studi);
        Sendto(segreteriafd, buffer, segreteria_address);
        printf("Il login dello studente %s è andato a buon fine ed ha ora accesso al server.\n", matricola);
    } else if (sqlite_result_code == SQLITE_DONE) {

        // Se le credenziali non sono corrette, invia FAILURE
        Sendto(segreteriafd, "FAILURE", segreteria_address);
        printf("Il login dello studente %s è fallito in quanto le credenziali non coincidono.\n", matricola);
    } else {

        // Gestione degli altri errori SQLite
        fprintf(stderr, "Errore SQLite: %s\n", sqlite3_errmsg(db));
    }

    // Finalizzazione dello statement SQL
    sqlite3_finalize(prepared_statement);
}

// Funzione per visualizzare gli appelli nel database
void Visualizza_Appelli(sqlite3 *db, const int segreteriafd, const struct sockaddr_in segreteria_address, char *dati) {

    // Estrai il piano di studi dai dati ricevuti
    strsep(&dati, ";");
    char *piano_di_studi = Estrai_Token(&dati);

    // Costruzione della query SQL per visualizzare gli appelli
    char sql_query_text[MALLOC];
    sprintf(sql_query_text,"SELECT ESAME.NOME_ESAME, APPELLO.NOME_APPELLO, APPELLO.DATA_APPELLO, ESAME.ID_ESAME, APPELLO.ID_APPELLO FROM APPELLO JOIN ESAME ON APPELLO.ID_ESAME = ESAME.ID_ESAME WHERE ESAME.CORSO_DI_STUDI = '%s';", piano_di_studi);
    sqlite3_stmt *prepared_statement;

    // Preparazione e esecuzione della query SQL
    if (sqlite3_prepare_v2(db, sql_query_text, -1, &prepared_statement, NULL) != SQLITE_OK) {

        // In caso di errore, invia un messaggio di errore alla segreteria
        Sendto(segreteriafd, "Errore query appelli", segreteria_address);
        fprintf(stderr, "Errore SQLite: %s\n", sqlite3_errmsg(db));
        return;
    }

    // Feedback sull'esecuzione della query
    printf("Query SQL per reperire gli appelli eseguita correttamente.\n");

    // Costruzione del buffer per i risultati della query
    char buffer[MALLOC] = "";
    while (sqlite3_step(prepared_statement) == SQLITE_ROW) {

        // Recupera i dati degli appelli
        const char *nome_esame = (const char *) sqlite3_column_text(prepared_statement, 0);
        const char *nome_appello = (const char *) sqlite3_column_text(prepared_statement, 1);
        const char *data_appello = (const char *) sqlite3_column_text(prepared_statement, 2);
        const char *id_esame = (const char *) sqlite3_column_text(prepared_statement, 3);
        const char *id_appello = (const char *) sqlite3_column_text(prepared_statement, 4);

        // Costruzione del buffer di risposta
        strcat(buffer, nome_esame);
        strcat(buffer, ";");
        strcat(buffer, id_esame);
        strcat(buffer, ";");
        strcat(buffer, nome_appello);
        strcat(buffer, ";");
        strcat(buffer, data_appello);
        strcat(buffer, ";");
        strcat(buffer, id_appello);
        strcat(buffer, ";");
    }

    // Stampa del contenuto del buffer (debug)
    printf("Contenuto del buffer: %s\n", buffer);

    // Invio dei dati alla segreteria
    Sendto(segreteriafd, buffer, segreteria_address);

    // Finalizzazione dello statement SQL
    sqlite3_finalize(prepared_statement);

    // Feedback dell'operazione completata
    printf("Appelli esami reperiti e spediti correttamente.\n\n");

    // Liberazione della memoria allocata per il piano di studi
    free(piano_di_studi);
}

// Funzione per aggiungere una Prenotazione nel database
void Aggiungi_Prenotazione(sqlite3 *db, const int segreteriafd, const struct sockaddr_in segreteria_address, char *dati) {

    // Estrai la Matricola, l'ID dell'esame e l'ID dell'appello dai dati ricevuti
    strsep(&dati, ";");
    char *matricola = Estrai_Token(&dati);
    char *id_esame = Estrai_Token(&dati);
    char *id_appello = Estrai_Token(&dati);

    // Stampa dei dati di prenotazione ricevuti (debug)
    printf("Dati di prenotazione ricevuti:\nMatricola: %s | ID Esame: %s | ID Appello: %s\n", matricola, id_esame, id_appello);

    // Verifica se l'esame è associato al corso di studi dello studente
    char sql_check_course[MALLOC];
    sprintf(sql_check_course,"SELECT COUNT(*) FROM STUDENTE AS S JOIN ESAME AS E ON S.PIANO_DI_STUDI = E.CORSO_DI_STUDI WHERE S.MATRICOLA = %s AND E.ID_ESAME = %s;", matricola, id_esame);

    int count_course = 0;
    sqlite3_stmt *stmt_course;
    if (sqlite3_prepare_v2(db, sql_check_course, -1, &stmt_course, NULL) == SQLITE_OK) {
        while (sqlite3_step(stmt_course) == SQLITE_ROW) {
            count_course = sqlite3_column_int(stmt_course, 0);
        }
    }
    sqlite3_finalize(stmt_course);

    // Se l'esame non è associato al corso di studi, invia un messaggio di errore
    if (count_course == 0) {
        Sendto(segreteriafd, "INVALID COURSE", segreteria_address);
        printf("La prenotazione è fallita perché l'esame non è associato al corso di studi dello studente.\n");
    }

    // Verifica se l'appello è valido per l'esame specificato
    char sql_check_appello[MALLOC];
    sprintf(sql_check_appello, "SELECT COUNT(*) FROM APPELLO WHERE ID_APPELLO = %s AND ID_ESAME = %s;", id_appello, id_esame);
    int count_appello = 0;
    sqlite3_stmt *stmt_appello;
    if (sqlite3_prepare_v2(db, sql_check_appello, -1, &stmt_appello, NULL) == SQLITE_OK) {
        while (sqlite3_step(stmt_appello) == SQLITE_ROW) {
            count_appello = sqlite3_column_int(stmt_appello, 0);
        }
    }
    sqlite3_finalize(stmt_appello);

    // Se l'appello non è valido, invia un messaggio di errore
    if (count_appello == 0) {
        Sendto(segreteriafd, "APPELLO INVALIDO", segreteria_address);
        printf("La prenotazione è fallita perché l'ID dell'appello non corrisponde all'ID dell'esame inserito.\n");
    }

    // Verifica se lo studente ha già prenotato per quell'appello
    char sql_check[MALLOC];
    sprintf(sql_check, "SELECT COUNT(*) FROM PRENOTAZIONE WHERE MATRICOLA = %s AND ID_APPELLO = %s;", matricola, id_appello);
    int count = 0;
    sqlite3_stmt *stmt_check;
    if (sqlite3_prepare_v2(db, sql_check, -1, &stmt_check, NULL) == SQLITE_OK) {
        while (sqlite3_step(stmt_check) == SQLITE_ROW) {
            count = sqlite3_column_int(stmt_check, 0);
        }
    }
    sqlite3_finalize(stmt_check);

    // Se lo studente ha già prenotato per quell'appello, invia un messaggio di errore
    if (count > 0) {
        Sendto(segreteriafd, "ALREADY THERE", segreteria_address);
        printf("La prenotazione è fallita in quanto lo studente ha già prenotato per quell'appello.\n");
    } else {

        // Se tutte le verifiche passano, aggiungi la prenotazione
        char sql_insert[MALLOC];
        sprintf(sql_insert,"INSERT INTO PRENOTAZIONE (MATRICOLA, ID_APPELLO, DATA_PRENOTAZIONE) VALUES (%s, %s, DATE('now'));", matricola, id_appello);

        char *errore;
        const int rc = sqlite3_exec(db, sql_insert, 0, 0, &errore);
        if (rc != SQLITE_OK) {

            // In caso di errore durante l'inserimento, invia un messaggio di errore
            fprintf(stderr, "Errore SQLite: %s\n", errore);
            sqlite3_free(errore);
        } else {

            // Se l'inserimento va a buon fine, invia SUCCESS e l'ID di prenotazione generato
            const sqlite3_int64 id_prenotazione = sqlite3_last_insert_rowid(db);
            char msg[MALLOC];
            sprintf(msg, "SUCCESS;%lld", id_prenotazione);
            Sendto(segreteriafd, msg, segreteria_address);
            printf("La prenotazione dell'appello %s da parte dello studente %s è stata eseguita con successo.\n", id_appello, matricola);
            printf("Generato ID di prenotazione: %lld\n\n", id_prenotazione);
        }
    }

    // Liberazione della memoria allocata
    free(matricola);
    free(id_esame);
    free(id_appello);
}

// Funzione per aggiungere un Esame nel database
void Aggiungi_Esame(sqlite3 *db, const int segreteriafd, const struct sockaddr_in segreteria_address, char *dati) {

    // Estrai il Nome dell'esame dai dati ricevuti
    strsep(&dati, ";");
    char *nome_esame = Estrai_Token(&dati);
    if (nome_esame == NULL) {
        fprintf(stderr, "Errore: nome_esame è NULL\n");
        Sendto(segreteriafd, "FAILURE", segreteria_address);
        return;
    }

    // Estrai il CFU dell'esame dai dati ricevuti
    const int cfu = atoi(Estrai_Token(&dati));
    if (cfu <= 0) {
        fprintf(stderr, "Errore: CFU non valido\n");
        free(nome_esame);
        Sendto(segreteriafd, "FAILURE", segreteria_address);
        return;
    }

    // Estrai il Corso di Studi dell'esame dai dati ricevuti
    char *corso_di_studi = Estrai_Token(&dati);
    if (corso_di_studi == NULL) {
        fprintf(stderr, "Errore: corso_di_studi è NULL\n");
        free(nome_esame);
        Sendto(segreteriafd, "FAILURE", segreteria_address);
        return;
    }

    // Stampa dei dati dell'esame ricevuti (debug)
    printf("Dati dell'esame ricevuti:\nNome Esame: %s | CFU: %d | Corso di Studi: %s\n", nome_esame, cfu, corso_di_studi);

    // Costruzione della query SQL per aggiungere l'esame
    char sql_insert[MALLOC];
    snprintf(sql_insert, sizeof(sql_insert),"INSERT INTO ESAME (NOME_ESAME, CFU, CORSO_DI_STUDI) VALUES ('%s', %d, '%s');", nome_esame, cfu, corso_di_studi);

    // Esecuzione della query SQL
    char *errore;
    const int rc = sqlite3_exec(db, sql_insert, 0, 0, &errore);
    if (rc != SQLITE_OK) {

        // In caso di errore, stampa un messaggio di errore e invia FAILURE alla segreteria
        fprintf(stderr, "Errore SQLite: %s\n", errore);
        sqlite3_free(errore);
        printf("DEBUG: Invio risposta di errore alla segreteria...\n");
        Sendto(segreteriafd, "FAILURE", segreteria_address);
    } else {

        // Se l'inserimento va a buon fine, stampa un messaggio di successo e invia SUCCESS alla segreteria
        printf("L'esame è stato aggiunto con successo.\n");
        printf("DEBUG: Invio risposta di successo alla segreteria...\n");
        Sendto(segreteriafd, "SUCCESS", segreteria_address);
    }

    // Liberazione della memoria allocata
    free(nome_esame);
}

// Funzione per aggiungere un Appello nel database
void Aggiungi_Appello(sqlite3 *db, const int segreteriafd, const struct sockaddr_in segreteria_address, char *dati) {

    // Estrai l'ID dell'esame dai dati ricevuti
    strsep(&dati, ";");
    const int id_esame = atoi(Estrai_Token(&dati));
    if (id_esame <= 0) {
        fprintf(stderr, "Errore: ID esame non valido\n");
        Sendto(segreteriafd, "FAILURE", segreteria_address);
        return;
    }

    // Estrai il Nome dai dati ricevuti
    char *nome_appello = Estrai_Token(&dati);
    if (nome_appello == NULL) {
        fprintf(stderr, "Errore: nome_appello è NULL\n");
        Sendto(segreteriafd, "FAILURE", segreteria_address);
        return;
    }

    // Estrai la Data dai dati ricevuti
    char *data_appello = Estrai_Token(&dati);
    if (data_appello == NULL) {
        fprintf(stderr, "Errore: data_appello è NULL\n");
        free(nome_appello);
        Sendto(segreteriafd, "FAILURE", segreteria_address);
        return;
    }

    // Stampa dei dati dell'appello ricevuti (debug)
    printf("Dati dell'appello ricevuti:\nID Esame: %d | Nome Appello: %s | Data Appello: %s\n", id_esame, nome_appello, data_appello);

    // Query per ottenere l'ID successivo per l'appello
    char *errore;
    char sql_query[MALLOC];
    snprintf(sql_query, sizeof(sql_query), "SELECT MAX(ID_APPELLO) FROM APPELLO;");
    sqlite3_stmt *stmt;
    if (sqlite3_prepare_v2(db, sql_query, -1, &stmt, 0) != SQLITE_OK) {

        // In caso di errore, invia un messaggio di errore alla segreteria
        fprintf(stderr, "Errore SQLite: %s\n", sqlite3_errmsg(db));
        Sendto(segreteriafd, "FAILURE", segreteria_address);
        return;
    }

    // Ottieni il prossimo ID appello
    int id_appello = 0;
    if (sqlite3_step(stmt) == SQLITE_ROW) {
        id_appello = sqlite3_column_int(stmt, 0) + 1;
    }
    sqlite3_finalize(stmt);

    // Costruzione della query SQL per aggiungere l'appello
    char sql_insert[MALLOC];
    snprintf(sql_insert, sizeof(sql_insert),"INSERT INTO APPELLO (ID_APPELLO, NOME_APPELLO, DATA_APPELLO, ID_ESAME) VALUES (%d, '%s', '%s', %d);", id_appello, nome_appello, data_appello, id_esame);

    // Esecuzione della query SQL
    const int rc = sqlite3_exec(db, sql_insert, 0, 0, &errore);
    if (rc != SQLITE_OK) {

        // In caso di errore, stampa un messaggio di errore e invia FAILURE alla segreteria
        fprintf(stderr, "Errore SQLite: %s\n", errore);
        sqlite3_free(errore);
        printf("DEBUG: Invio risposta di errore alla segreteria...\n");
        Sendto(segreteriafd, "FAILURE", segreteria_address);
    } else {

        // Se l'inserimento va a buon fine, stampa un messaggio di successo e invia SUCCESS alla segreteria
        printf("L'appello è stato aggiunto con successo.\n");
        printf("DEBUG: Invio risposta di successo alla segreteria...\n");
        Sendto(segreteriafd, "SUCCESS", segreteria_address);
    }
}
```
Server_Helper.c implementa diverse funzionalità per gestire le comunicazioni tra il server e un'applicazione client. Le funzioni presenti si occupano della ricezione dei dati dal client, della verifica delle credenziali utente, dell'esecuzione di query SQL sul database, e della formattazione e invio delle risposte al client.
#### Dettagli principali
#### Gestione delle Comunicazioni:
La funzione Receivefrom() riceve i dati dal client tramite socket UDP, mentre Sendto() invia le risposte al client sempre tramite socket UDP. I dati sono formattati utilizzando un separatore (;) per distinguere i comandi e i parametri necessari.
#### Verifica delle Credenziali: 
La funzione Verifica_Credenziali() controlla nel database le credenziali di accesso di uno studente (matricola e password), e invia un esito positivo o negativo al client in base al risultato.
#### Visualizzazione degli Appelli:
La funzione Visualizza_Appelli() recupera le informazioni relative agli appelli disponibili per un certo corso di studi dal database e le trasmette al client.
#### Gestione delle Prenotazioni: 
La funzione Aggiungi_Prenotazione() permette di inserire una nuova prenotazione per un appello nel database, verificando la correttezza dei dati forniti e inviando al client un esito che indica successo o fallimento.
#### Inserimento Esami:
La funzione Aggiungi_Esame() consente l'inserimento di un nuovo esame nel database, dopo aver verificato i dati e informando il client dell'esito dell'operazione.
#### Inserimento Appelli: 
La funzione Aggiungi_Appello() permette di aggiungere un nuovo appello per un esame già presente nel database, verificando la validità delle informazioni fornite e inviando un messaggio al client per confermare o meno l'operazione.


### Studente_Helper.c
```
#include "../Source/lib.h"

// Funzione per effettuare il login dello studente
Studente *Login(int *studentefd, struct sockaddr_in *segreteria_address) {

    // Inizializza la connessione del client con la segreteria
    Connessione_Client(studentefd, segreteria_address, SEGRETERIA_PORT);

    // Alloca memoria per la struttura dati dello studente
    Studente *studente = malloc(sizeof(Studente));

    // Legge la Matricola dello studente
    printf("\nInserisci la tua matricola: ");
    fgets(studente->matricola, sizeof(studente->matricola), stdin);
    studente->matricola[strcspn(studente->matricola, "\n")] = '\0';

    // Legge la Password dello studente
    char password[50];
    printf("Inserisci la tua password: ");
    fgets(password, sizeof(password), stdin);
    password[strcspn(password, "\n")] = '\0';

    // Costruisce il messaggio da inviare al server della segreteria per il login
    char buffer[MALLOC];
    snprintf(buffer, sizeof(buffer), "0;%s;%s;", studente->matricola, password);

    // Invia il messaggio di login al server della segreteria
    Sendto(*studentefd, buffer, *segreteria_address);

    // Riceve la risposta dal server della segreteria
    char *risposta = Receive(*studentefd, segreteria_address);

    // Analizza la risposta per determinare l'esito del login
    const char *token = strtok(risposta, ";");
    if (strcmp(token, "SUCCESS") == 0) {
        token = strtok(NULL, ";");
        strncpy(studente->piano_di_studi, token, sizeof(studente->piano_di_studi) - 1);
        studente->piano_di_studi[sizeof(studente->piano_di_studi) - 1] = '\0';
        free(risposta);

        // Restituisce la struttura dati dello studente dopo il login
        return studente;
    }

    // Dealloca memoria e restituisce NULL in caso di fallimento del login
    free(risposta);
    free(studente);
    return NULL;
}

// Funzione per la selezione dell'operazione da parte dello studente
int Selezione_Richiesta_Studente() {
    int option;
    printf("\nLISTA OPERAZIONI STUDENTE:\n");
    printf("1) Visualizza appelli;\n");
    printf("2) Prenotazione appello;\n");
    printf("0) Esci;\n");
    printf("Scegli un'opzione: ");

    // Legge l'opzione selezionata dall'utente
    scanf("%d", &option);

    // Pulisce il buffer di input
    Clear_Input_Buffer();

    // Restituisce l'opzione scelta dall'utente
    return option;
}

// Funzione per la richiesta di visualizzazione degli appelli disponibili
void Richiesta_Visualizzazione_Appelli(const int studentefd, struct sockaddr_in segreteria_address, char *piano_di_studi) {
    char buffer[MALLOC];

    // Costruisce il messaggio da inviare al server
    snprintf(buffer, sizeof(buffer), "1;%s;", piano_di_studi);

    // Invia la richiesta di visualizzazione degli appelli al server della segreteria
    Sendto(studentefd, buffer, segreteria_address);

    // Riceve la risposta dal server della segreteria
    char *risultato = Receive(studentefd, &segreteria_address);
    char *token = strtok(risultato, ";");

    // Itera attraverso i token ricevuti e stampa i dettagli degli appelli
    while (token != NULL) {
        char *nome_esame = token;
        token = strtok(NULL, ";");
        if (token == NULL) break;

        char *id_esame = token;
        token = strtok(NULL, ";");
        if (token == NULL) break;

        char *nome_appello = token;
        token = strtok(NULL, ";");
        if (token == NULL) break;

        char *data_appello = token;
        token = strtok(NULL, ";");
        if (token == NULL) break;

        char *id_appello = token;
        token = strtok(NULL, ";");

        // Stampa i dettagli dell'esame e dell'appello
        printf("\nEsame: %s | ID Esame: %s\n", nome_esame, id_esame);
        printf("Appello: %s | Data: %s | ID Appello: %s\n", nome_appello, data_appello, id_appello);
    }

    // Libera la memoria allocata per la risposta
    free(risultato);
}

// Funzione per la richiesta di prenotazione di un appello
void Richiesta_Prenotazione_Appello(const int studentefd, struct sockaddr_in segreteria_address, char *matricola) {
    char id_esame[10];
    char id_appello[10];

    // Legge l'ID dell'esame desiderato
    printf("\nInserisci l'ID dell'esame a cui vuoi prenotarti: ");
    fgets(id_esame, 10, stdin);
    id_esame[strcspn(id_esame, "\n")] = 0;

    // Legge l'ID dell'appello desiderato
    printf("\nInserisci l'ID dell'appello a cui vuoi prenotarti: ");
    fgets(id_appello, 10, stdin);
    id_appello[strcspn(id_appello, "\n")] = 0;

    char buffer[MALLOC];
    snprintf(buffer, sizeof(buffer), "2;%s;%s;%s;", matricola, id_esame, id_appello); // Costruisce il messaggio da inviare al server

    // Invia la richiesta di prenotazione dell'appello al server della segreteria
    Sendto(studentefd, buffer, segreteria_address);

    // Riceve la risposta dal server della segreteria
    char *result = Receive(studentefd, &segreteria_address);

    // Gestisce la risposta ricevuta dal server della segreteria
    if (strncmp(result, "SUCCESS", 7) == 0) {
        char *id_prenotazione = result + 8;
        printf("\nLa prenotazione è andata a buon fine: ID Prenoggggtazione = %s.\n", id_prenotazione);
    } else if (strcmp(result, "ALREADY THERE") == 0) {
        printf("\nHai già effettuato una prenotazione su questo appello.\n");
    } else if (strcmp(result, "INVALID COURSE") == 0) {
        printf("\nNon puoi prenotarti a questo esame/appello perché non è associato al tuo corso di studi.\n");
    } else if (strcmp(result, "APPELLO INVALIDO") == 0) {
        printf("\nL'ID dell'appello non corrisponde all'ID dell'esame inserito.\n");
    }

    // Libera la memoria allocata per la risposta
    free(result);
}
```
Studente_Helper.c fornisce una serie di funzioni per gestire l’interazione tra uno studente e il server della segreteria universitaria. Le funzioni consentono agli studenti di autenticarsi, selezionare un’operazione da eseguire, visualizzare gli appelli disponibili per il proprio corso di studi e prenotarsi per un esame.

#### Dettagli principali
#### Autenticazione Studente:
La funzione Login() consente allo studente di inserire la propria matricola e password per accedere. Se l'autenticazione va a buon fine, la funzione restituisce una struttura dati "Studente" con informazioni come il piano di studi.
#### Selezione dell'Operazione: 
La funzione Selezione_Richiesta_Studente() mostra all’utente un menu con diverse opzioni (visualizzare gli appelli, prenotarsi a un appello, uscire). La funzione restituisce la scelta effettuata dallo studente.
#### Visualizzazione degli Appelli:
La funzione Richiesta_Visualizzazione_Appelli() invia una richiesta al server per ottenere l’elenco degli appelli disponibili per il corso di studi dello studente. Successivamente, riceve la risposta dal server e visualizza i dettagli di ogni appello (nome dell’esame, ID dell’esame, nome dell’appello, data e ID dell’appello).
#### Prenotazione di un Appello:
La funzione Richiesta_Prenotazione_Appello() permette allo studente di inserire l’ID di un esame e di un appello per effettuare la prenotazione. La richiesta viene inviata al server, che risponde indicando l’esito della prenotazione (successo, prenotazione già esistente, corso di studi errato, ID appello non valido).

### Server.c
```
#include "../Source/lib.h"

int main() {

    // Crea il socket del server
    const int serverfd = Socket();

    // Configura l'indirizzo del server
    const struct sockaddr_in server_address = Configura_Indirizzo(SERVER_PORT);

    // Riutilizzo indirizzo socket
    Sock_Options(serverfd);

    // Associa il socket all'indirizzo
    Bind(serverfd, server_address);

    // Struttura per l'indirizzo della segreteria
    struct sockaddr_in segreteria_address;

    // Lunghezza dell'indirizzo della segreteria
    socklen_t segreteria_length = sizeof(segreteria_address);

    // Connessione al database SQLite
    sqlite3 *db = ConnessioneDB();

    // Mette il server in ascolto di connessioni in entrata
    Listen(serverfd);

    // Ciclo principale per gestire le connessioni in entrata
    while (1) {

        // Effettua collegamento con la Segreteria.c
        const int segreteriafd = Accept(serverfd, (struct sockaddr *) &segreteria_address, &segreteria_length);
        printf("ACCEPT: Connessione stabilita con la segreteria.\n");

        // Ciclo interno per gestire le richieste della segreteria
        while (1) {

            // Messaggio di attesa di richieste
            printf("\nIn attesa di richieste dalla segreteria.\n");

            // Riceve un messaggio dalla segreteria
            char *buffer = Receive(segreteriafd, &segreteria_address);
            if (strlen(buffer) == 0) {

                // Messaggio se la segreteria chiude la connessione
                printf("La segreteria ha chiuso la connessione.\n");

                // Chiude la connessione con la segreteria
                Close(segreteriafd);

                // Esce dal ciclo interno
                break;
            }

            // Copia il messaggio ricevuto in una variabile
            char *richiesta = strdup(buffer);

            // Estrae il token dalla richiesta
            const char *token = Estrai_Token(&buffer);

            if (token != NULL) {

                // Converte il token in intero
                const int option = atoi(token);

                // Switch per gestire le diverse opzioni ricevute dalla segreteria
                switch (option) {
                    case 0:

                        // Funzione per verificare le credenziali
                        Verifica_Credenziali(db, segreteriafd, segreteria_address, richiesta);
                        break;
                    case 1:

                        // Funzione per visualizzare gli appelli
                        Visualizza_Appelli(db, segreteriafd, segreteria_address, richiesta);
                        break;
                    case 2:

                        // Funzione per aggiungere una prenotazione
                        Aggiungi_Prenotazione(db, segreteriafd, segreteria_address, richiesta);
                        break;
                    case 3:

                        // Funzione per aggiungere un esame
                        Aggiungi_Esame(db, segreteriafd, segreteria_address, richiesta);
                        break;
                    case 4:

                        // Funzione per aggiungere un appello
                        Aggiungi_Appello(db, segreteriafd, segreteria_address, richiesta);
                        break;
                    default:

                        // Messaggio se l'opzione non è valida
                        fprintf(stderr, "Opzione non valida: %d\n", option);
                        break;
                }
            } else {
                // Messaggio in caso di errore nel parsing del messaggio
                fprintf(stderr, "Errore nel parsing del messaggio\n");
            }
        }
    }
}
```
Server.c gestisce il lato server di un sistema di segreteria universitaria. Il server si mette in ascolto su una porta specifica, accetta connessioni da studenti e segreteria, e gestisce le loro richieste. Le funzioni permettono di verificare credenziali, visualizzare appelli, aggiungere prenotazioni, esami e appelli.

#### Dettagli principali
#### Gestione Connessioni:
Il server crea un socket, si associa a un indirizzo, e rimane in ascolto di richieste dai client. Gestisce una connessione alla volta, riceve messaggi dai client e invia risposte appropriate.
#### Verifica Credenziali:
La funzione Verifica_Credenziali() controlla matricola e password nel database, rispondendo con esito positivo o negativo al client.
#### Visualizzazione Appelli:
La funzione Visualizza_Appelli() recupera gli appelli disponibili dal database e li invia al client.
#### Aggiunta Prenotazione:
La funzione Aggiungi_Prenotazione() registra una prenotazione per un appello nel database e risponde al client con il risultato.
#### Aggiunta Esame:
La funzione Aggiungi_Esame() inserisce un nuovo esame nel database e comunica al client l'esito.
#### Aggiunta Appello:
La funzione Aggiungi_Appello() aggiunge un nuovo appello a un esame esistente e invia l'esito al client.

### Segreteria.c
```
#include "../Source/lib.h"

int main() {

    // Alloca memoria per la struttura Thread_Segreteria, che contiene gli argomenti passati ai thread
    Thread_Segreteria *args = malloc(sizeof(Thread_Segreteria));

    // Dichiarazione delle variabili per i thread e il contatore
    pthread_t main_thread;      // Thread principale per gestire la connessione studente
    pthread_t input_thread;     // Thread per gestire l'input dalla segreteria
    int counter = 0;            // Contatore per gli ID assegnati agli studenti
    args->counter = counter;    // Inizializzazione dell'argomento counter nella struttura args

    // Crea un nuovo socket
    const int segreteriafd = Socket();

    // Configurazione del socket della segreteria
    const struct sockaddr_in segreteria_address = Configura_Indirizzo(SEGRETERIA_PORT);

    // Riutilizzo indirizzo socket
    Sock_Options(segreteriafd);

    // Associa il socket all'indirizzo
    Bind(segreteriafd, segreteria_address);

    // Inizializzazione dei parametri della struttura args relativi al socket della segreteria
    args->segreteriafd = segreteriafd;
    args->segreteria_address = segreteria_address;

    // Effettua collegamento con il Server.c
    int serverfd;
    struct sockaddr_in server_address;
    Connessione_Client(&serverfd, &server_address, SERVER_PORT);

    // Inizializzazione dei parametri della struttura args relativi al socket del server
    args->serverfd = serverfd;
    args->server_address = server_address;

    // Variabili per gestire la connessione con lo studente
    struct sockaddr_in studente_address;
    socklen_t studente_length = sizeof(studente_address);

    // Avvio del thread per gestire l'input dalla segreteria
    pthread_create(&input_thread, NULL, Thread_Input_Segreteria, args);
    pthread_detach(input_thread);

    // In attesa di connessioni da parte degli studenti
    Listen(segreteriafd);
    while (1) {

        // Effettua collegamento con lo Studente.c
        const int studentefd = Accept(segreteriafd, (struct sockaddr *) &studente_address, &studente_length);

        // Incrementa il contatore degli studenti connessi
        counter++;

        // Aggiorna il contatore nella struttura e memorizza il socket dello studente
        args->counter = counter;
        args->studentefd = studentefd;
        printf("\nConnessione stabilita con un nuovo studente. | ID assegnato: %d.\n", counter);

        // Avvia un thread per gestire la connessione dello studente
        pthread_create(&main_thread, NULL, Thread_Gestione_Studente, args);
        pthread_detach(main_thread);
    }
}
```
Segreteria.c gestisce il lato segreteria di un sistema universitario, creando una socket per accettare connessioni dagli studenti e gestirle tramite thread dedicati. Il programma supporta la gestione di connessioni multiple, assegna un ID univoco a ciascun studente e utilizza thread per la gestione delle interazioni.

#### Dettagli principali
#### Gestione Connessioni:
Il server si associa a un indirizzo e rimane in ascolto per ricevere connessioni dagli studenti. Ogni connessione viene gestita in un ciclo dedicato.
#### Assegnazione ID Studente: 
Ogni studente connesso riceve un ID univoco tramite un contatore.
#### Gestione Thread:
Thread separati gestiscono sia le interazioni con gli studenti che l'input dalla segreteria, garantendo la sincronizzazione delle risorse condivise.


### Studente.c
```
#include "../Source/lib.h"

int main() {
    int studentefd;                         // Descrittore del socket dello studente
    struct sockaddr_in segreteria_address;  // Indirizzo della segreteria

    // Loop principale del programma studente
    while (1) {

        // Effettua il login dello studente
        Studente *studente = Login(&studentefd, &segreteria_address);

        // Verifica se il login è stato eseguito con successo
        if (studente != NULL) {
            printf("SUCCESS: Il login ha avuto successo.\n");

            // Loop per gestire le operazioni dello studente
            while (1) {

                // Ottiene l'opzione selezionata dallo studente
                const int scelta = Selezione_Richiesta_Studente();

                // Gestisce la scelta effettuata dallo studente
                if (scelta == 1) {
                    Richiesta_Visualizzazione_Appelli(studentefd, segreteria_address, studente->piano_di_studi);
                } else if (scelta == 2) {
                    Richiesta_Prenotazione_Appello(studentefd, segreteria_address, studente->matricola);
                } else if (scelta == 0) {

                    // Chiude il socket dello studente
                    Close(studentefd);

                    // Libera la memoria allocata per lo studente
                    free(studente);

                    // Esce dal loop di gestione delle operazioni
                    break;
                } else {

                    // Gestisce opzioni non valide
                    printf("Opzione non valida.\n");
                }
            }

            // Esce dal loop principale del programma studente
            break;
        }

        // Gestisce il fallimento del login
        printf("FAILURE: Credenziali scorrette. Riprova\n");
    }
}

```
Studente.c gestisce l’interfaccia studente per interagire con il sistema di segreteria universitaria. Permette agli studenti di fare il login, visualizzare gli appelli disponibili e prenotare un esame. Le funzioni principali gestiscono la comunicazione con il server e la selezione delle operazioni.

#### Dettagli principali
#### Login Studente: 
Richiede matricola e password, invia i dati al server e restituisce un risultato basato sulla verifica.
#### Visualizzazione Appelli:
Invia una richiesta al server per visualizzare gli appelli disponibili e mostra i dettagli ricevuti.
#### Prenotazione Appello: 
Permette di inserire l’ID dell’esame e dell’appello, invia la prenotazione al server e mostra l’esito.



 ### Universita_lib.h
 ```
#ifndef UNIVERSITA_LIB_H
#define UNIVERSITA_LIB_H

#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <arpa/inet.h>
#include <sys/socket.h>
#include <pthread.h>
#include <signal.h>
#include "../SQL/sqlite3.h"

// Definizioni
#define MAX(x, y) ((x) > (y) ? (x) : (y))
#define MAIN_IP "127.0.0.1"
#define SERVER_PORT 1024
#define SEGRETERIA_PORT 1025
#define MALLOC 512

// Struttura da utilizzare per la creazione di thread della Segreteria
typedef struct {
    int counter;
    int serverfd;
    int segreteriafd;
    int studentefd;
    struct sockaddr_in server_address;
    struct sockaddr_in segreteria_address;
} Thread_Segreteria;

// Struttura per immagazzinare i dati dello studente
typedef struct {
    char matricola[10];
    char piano_di_studi[50];
} Studente;

// Funzioni di Framework.c
void Clear_Input_Buffer();
void Error(const char *msg);
char *Estrai_Token(char **dati);
sqlite3 *ConnessioneDB();
int Esegui_Query(sqlite3 *db, const char *sql, sqlite3_stmt **sql_query);

// Funzioni di Network.c
int Socket();
void Sock_Options(int socketfd);
void Bind(int socketfd, struct sockaddr_in address);
int Listen(int socketfd);
struct sockaddr_in Configura_Indirizzo(int port);
int Accept(int socketfd, struct sockaddr *address, socklen_t *adress_length);
void Close(int socketfd);
void Sendto(int socketfd, const char *messaggio, struct sockaddr_in address);
char *Receive(int socketfd, struct sockaddr_in *address);
void Connessione_Client(int *socketfd, struct sockaddr_in *server_address, int port);

// Funzioni di Server_Helper.c
void Verifica_Credenziali(sqlite3 *db, int segreteriafd, struct sockaddr_in segreteria_address, char *dati);
void Visualizza_Appelli(sqlite3 *db, int segreteriafd, struct sockaddr_in segreteria_address, char *dati);
void Aggiungi_Prenotazione(sqlite3 *db, int segreteriafd, struct sockaddr_in segreteria_address, char *dati);
void Aggiungi_Esame(sqlite3 *db, int segreteriafd, struct sockaddr_in segreteria_address, char *dati);
void Aggiungi_Appello(sqlite3 *db, int segreteriafd, struct sockaddr_in segreteria_address, char *dati);

// Funzioni di Segreteria_Helper.c
void *Thread_Gestione_Studente(void *arg);
void *Thread_Input_Segreteria(void *arg);
int Selezione_Richiesta_Segreteria();
void Richiesta_Aggiunta_Esame(const void *arg);
void Richiesta_Aggiunta_Appello(const void *arg);

// Funzioni di Studente_Helper.c
Studente* Login(int *studentefd, struct sockaddr_in *segreteria_address);
int Selezione_Richiesta_Studente();
void Richiesta_Visualizzazione_Appelli(int studentefd, struct sockaddr_in segreteria_address, char *piano_di_studi);
void Richiesta_Prenotazione_Appello(int studentefd, struct sockaddr_in segreteria_address, char *matricola);

#endif //UNIVERSITA_LIB_H
```
Il file universita_lib.h funge da header principale per il progetto universitario, fornendo le interfacce per i diversi moduli del sistema. Contiene dichiarazioni di funzioni, strutture dati, costanti e macro utilizzate globalmente, favorendo la modularità, la riusabilità e la manutenibilità del codice.

### FINE
Progetto svolto da Ignuti Mara [0124/2637] e Lanzilo Giovanna [0124/2682] per l'esame di "Reti di calcolatori" dell'anno accadeico 2024/2025.
