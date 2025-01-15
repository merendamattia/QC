```table-of-contents
title: 
style: nestedList # TOC style (nestedList|nestedOrderedList|inlineFirstLevel)
minLevel: 0 # Include headings from the specified level
maxLevel: 0 # Include headings up to the specified level
includeLinks: true # Make headings clickable
debugInConsole: false # Print debug info in Obsidian console
```

---
# Che cosa è un ansatz?
Un ansatz è un circuito quantistico parametrizzato che rappresenta una famiglia di stati quantistici. L'ansatz contiene parametri liberi (es. angoli di rotazione di porte quantistiche) che vengono ottimizzati da un algoritmo classico per raggiungere un obiettivo, come minimizzare l'energia o massimizzare una funzione obiettivo.

# Perchè usiamo $R_y$ per la rotazione?
La porta  $R_y$  crea sovrapposizioni bilanciate tra gli stati  $|0\rangle$  e  $|1\rangle$ , modificando le ampiezze in modo continuo e uniforme lungo l'asse  $y$  della sfera di Bloch.
- Ad esempio, con $\theta = \pi/2$ ,  $R_y$  trasforma  $|0\rangle$  in  $|+\rangle = \frac{1}{\sqrt{2}}(|0\rangle + |1\rangle)$ .
$R_y$  è una rotazione unitaria parametrizzabile con un solo parametro ($\theta$), il che la rende semplice da ottimizzare. Inoltre, ruotando lungo l'asse  $y$ , si generano stati che non sono troppo sbilanciati verso  $|0\rangle$  o  $|1\rangle$ , ma che includono una combinazione utile per le sovrapposizioni.

L'uso di  $R_y$  nel VQE è anche consolidato da una pratica empirica: molti esperimenti hanno dimostrato che  $R_y$  è sufficiente per preparare stati utili per un'ampia gamma di problemi.
## Perchè non usare $R_z$?
$R_z$  ruota intorno all'asse  $z$ , modificando solo la fase relativa degli stati  $|0\rangle$  e  $|1\rangle$ .
Non è sufficiente da sola per generare stati di sovrapposizione, rendendola meno adatta a scopi come la preparazione di ansatz nel VQE.
## Perchè non usare $R_x$?
$R_x$  ruota lungo l'asse  $x$  della sfera di Bloch e può anch'essa generare stati di sovrapposizione.
Tuttavia,  $R_y$  è spesso preferita perché:
- Si abbina meglio con molteplici blocchi di entanglement come  $CZ$  o  $CX$ , poiché questi ultimi sono più efficaci nel connettere stati preparati da  $R_y$ .
- In molti casi di studio (specialmente in chimica quantistica), l'asse  $y$  è sufficiente per esplorare gli stati richiesti.

# Cosa succede se aggiungo più ripetizioni all'ansatz in fase di inizializzazione?
Maggiore è il numero di ripetizioni, più espressivo diventa il circuito. Questo significa che può rappresentare una gamma più ampia di stati quantistici, aumentando la probabilità di trovare una soluzione ottimale. Un numero elevato di ripetizioni aumenta il potenziale del circuito, ma anche il numero di parametri, rendendo l'ottimizzazione più complessa. Può portare a un maggiore impatto del rumore sui dispositivi quantistici reali.

# Perchè usiamo $CZ$ per il blocco di entanglement?
La porta  $CZ$  introduce una rotazione condizionata che modifica la fase del secondo qubit solo se il primo è nello stato  $|1\rangle$. Questo effetto è cruciale per generare correlazioni quantistiche (entanglement) senza cambiare le ampiezze degli stati di base.
$$CZ |x, y\rangle = (-1)^{x \cdot y}|x, y\rangle$$
In altre parole,  $CZ$  agisce solo sulla fase relativa degli stati, mantenendo la struttura del circuito compatta e semplice.

 $CZ$  è comunemente usato perché offre un buon compromesso tra la generazione di entanglement e la semplicità del circuito. Inoltre, esperimenti empirici hanno dimostrato che  $CZ$ , in combinazione con ansatz come  $R_y$ , riesce a preparare stati di qualità elevata con meno errori.
## Perchè non usiamo $CX$?
La porta  $CX$  (controlled-X o CNOT) esegue un flipping del secondo qubit condizionato dallo stato del primo.
$$CX |x, y\rangle = |x, x \oplus y\rangle$$
Sebbene  $CX$  sia utile per creare entanglement, modifica anche le ampiezze degli stati, il che può rendere più complessa la parametrizzazione e ottimizzazione dello stato target.
- $CZ$ : Preserva le ampiezze e altera solo le fasi, rendendolo particolarmente adatto per problemi di ottimizzazione e simulazione quantistica (es. chimica quantistica).
- $CX$ : Modifica sia le ampiezze che le fasi, il che potrebbe introdurre complessità indesiderate nel modello parametrico del VQE.

Inoltre, $CZ$  si integra bene con rotazioni come  $R_y$ , che generano stati di sovrapposizione bilanciati. In questo contesto:
- Le rotazioni iniziali  $R_y$  creano sovrapposizioni di base.
- $CZ$  introduce entanglement “pulito”, influenzando solo la fase.
- La combinazione è efficiente per esplorare lo spazio degli stati e minimizzare l'Hamiltoniana obiettivo.
- $CX$ , al contrario, può introdurre entanglement troppo aggressivo, che richiede una maggiore regolazione (più parametri) per rappresentare correttamente lo stato target.

# Perchè usiamo l'entanglement "full"?

Nel contesto di un ansatz come il  `TwoLocal` , un entanglement full connette ogni qubit a tutti gli altri attraverso le porte di entanglement (ad esempio,  $CZ$  o  $CX$ ).
In altre parole, viene introdotto entanglement tra tutte le coppie di qubit. Questo garantisce che le correlazioni quantistiche non siano limitate solo a vicini immediati, ma siano distribuite globalmente.

L'entanglement completo permette di esplorare uno spazio degli stati quantistici più ampio rispetto a configurazioni più limitate come:
- Entanglement lineare (linear): Connette solo i qubit adiacenti.
- Entanglement circolare (circular): Collega i qubit in una struttura ad anello.

 Problemi come l'ottimizzazione del portafoglio (Portfolio Optimization) spesso coinvolgono correlazioni tra tutte le variabili (o asset). Un ansatz con entanglement completo consente al VQE di rappresentare meglio tali correlazioni, migliorando la convergenza verso la soluzione.

Sebbene il full entanglement sia vantaggioso, presenta alcuni compromessi:
1. Numero di porte aumentato:
	- Con  n  qubit, un'entanglement completo richiede  $\binom{n}{2}$  connessioni (ossia  $\frac{n(n-1)}{2}$ ).
	- Questo aumenta il numero totale di porte quantistiche nel circuito, rendendolo più profondo e quindi più suscettibile agli errori.
2. Rumore hardware:
	- Nei computer quantistici attuali, il rumore introdotto dall'esecuzione di porte quantistiche può ridurre la qualità dello stato preparato.
	- In scenari rumorosi, può essere preferibile un'entanglement più semplice (ad esempio, lineare) per ridurre la profondità del circuito.

# VQE
https://en.wikipedia.org/wiki/Variational_quantum_eigensolver

# QAOA
https://en.wikipedia.org/wiki/Quantum_optimization_algorithms#Quantum_approximate_optimization_algorithm