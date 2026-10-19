# ce-cpu

Una CPU giocattolo a 16 bit ("TC16") con un piccolo assemblatore, scritta in
**[Ce++](https://github.com/Duffionis/Ce-)** (un linguaggio di sistema piccolo,
sicuro e tutto italiano che compila in C).

E' il primo programma vero scritto in Ce++ *fuori* dal repository del
compilatore: mostra cosa sa fare il linguaggio e cosa gli manca ancora.

    ./ce esegui src/principale.ce

Alla prima esecuzione scarica il compilatore Ce++ (fissato a una versione
provata, vedi la riga `PIN` in `./ce`; se cambia, si aggiorna da solo).
Servono `git`, `python3` e `gcc` o `clang`. Il programma esegue i propri test e
stampa `TC16: tutti i test ok` (codice di uscita 0) se tutto funziona.

## La CPU

8 registri (`r0`..`r7`, 16 bit), 512 parole di memoria, istruzioni a 16 bit:

    tipo R:  oooo ddd sss 000000     codice, registro destinazione, registro sorgente
    tipo I:  oooo ddd iiiiiiiii      codice, registro destinazione, costante a 9 bit

| cod | nome | effetto |
|----|------|--------|
| 0 | NOP | niente |
| 1 | LDI rd, imm | rd = imm |
| 2 | ADD rd, rs | rd = rd + rs (aggiorna i segnali) |
| 3 | SUB rd, rs | rd = rd - rs (aggiorna i segnali) |
| 4-6 | AND / OR / XOR rd, rs | operazioni sui bit |
| 7 | LD rd, rs | rd = memoria[rs] |
| 8 | ST rd, rs | memoria[rs] = rd |
| 9 | CMP rd, rs | segnali da rd - rs, risultato scartato |
| 10 | JMP imm | pc = imm |
| 11 / 12 | JZ / JNZ imm | salta se il segnale zero e' attivo / spento |
| 13 | OUT rd | aggiunge rd all'uscita |
| 14 | HALT | ferma la CPU |
| 15 | ADDI rd, imm | rd = rd + imm a 9 bit con segno |

I segnali (zero, negativo, riporto) sono una struttura impacchettata di tre
campi da 1 bit. Leggere o scrivere fuori memoria, o girare troppo a lungo,
restituisce un valore di errore invece di andare in crash. I mnemonici
dell'assembly (LDI, ADD, JMP...) restano quelli internazionali.

## File

    src/cpu.ce         registri, segnali, memoria, esecuzione di un passo, ciclo
    src/programmi.ce   assemblatore (a.ldi(0, 5) ...), programmi di esempio, disassemblatore
    src/principale.ce  test: Fibonacci, memoria, aritmetica e bit, casi di errore
    ce                 script che scarica ed esegue il compilatore Ce++ fissato

## Cosa di Ce++ usa questo programma

- campi a bit e struttura impacchettata (`Segnali`), aritmetica `naturale16`
  con overflow definito
- memoria dell'heap (`riserva` / `libera`) e fette con controllo dei limiti
- tipi generici della libreria: `Vettore<naturale16>` (registro dell'uscita),
  `Mappa<intero, testo>` (nomi delle istruzioni), `Risultato<T, E>` (errori
  senza crash), `Opzione<T>`
- parametri per riferimento e sintassi a metodo: `c.passo()`, `a.emetti(w)`
- moduli (`usa`)

## Scrivere un programma tuo

Aggiungi in `src/programmi.ce` una funzione che prende `a: &Assemblatore` e
chiama `a.ldi(...)`, `a.add(...)`, `a.out(...)`, `a.halt()`; le etichette sono
semplicemente `a.n come naturale16`. Poi caricala in un test come
`prova_fibonacci` in `src/principale.ce`.

## Cose notate scrivendolo (dove Ce++ puo' migliorare)

- Non c'e' `switch`: il decodificatore delle istruzioni e' una catena di `se`.
- `Segnali` non si costruisce da una parola grezza (ancora nessun cast sicuro
  tra un intero e una struttura impacchettata).
- Il disassemblatore stampa gli operandi come `r0 / 0` anche per le istruzioni
  che non li usano.
