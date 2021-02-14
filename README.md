<p align="center">
  <img src="https://engeto.cz/wp-content/uploads/2019/01/engeto-square.png" width="200" height="200">
</p>

# Java 2 - 5. lekce

## Co nás čeká.. VLÁKNA

 - [Úvod](#úvod)
 
 - [Životní cyklus](#životní-cyklus)
 
 - [Vytvoření](#vytvoření)
 
 - [Hlavní vlákno](#hlavní-vlákno)
 
 - [Práce s vícero vlákny](#práce-s-vícero-vlákny)
 
 - [Sdílení paměti](#sdílení-paměti)
 
 - [Deadlock](#deadlock)
 
## Úvod

### Proces

Proces je koncept, kterým simulujeme vykonání určitého programu. Existuje vícero definicí pojmu proces:

 - spuštění programu, běžící instance programu
 - entita, které je právě přiřazený procesor

### Vlákna (threads)
Vlákna můžeme vnímat jako zjednodušené procesy (thread = lightweight process). Na rozdíl od procesu, který má přidělené samostatné místo v paměti, vlákna jednoho procesu sdílí jedno paměťové místo v rámci procesu. Proces tedy může mít několik vláken, které pod ním běží – multithreading. Příkladem nám může být databázový nebo webový server – když půjdeme na stránku engeto.com, budeme obslouženi naším webovým serverem (zpravidla nginx nebo Apache) a určité vlákno tohoto procesu nám poskytne obsah stránky.

 - umožňují programům být efektivnější tím, že řeší vícero věci najednou
 
 - umožňují také provádění náročnějších výpočtů na pozadí

## Životní cyklus

 <p align="left">
  <img src="https://github.com/ENGETO-Java-Akademie/java2_lekce_05/blob/main/zivotni_cyklus_vlaken.png" width="600">
</p>

Vlákna se během svého života mohou nacházet v těchto pěti stavech:

 - [Nové](#nové)

 - [Připravené](#připravené)

 - [Běžící](#běžící)

 - [Čekající](#čekající)

 - [Ukončené](#ukončené)

### Nové

Toto je stav, do kterého se vlákno dostane po vytvoření.

### Připravené

Vlákno připřavené na spuštení, které čeká na přidělení systémových prostředků.

### Běžící

Vlákno právě běží.

### Čekající

Vlákno čeká na nějakou událost, popř. bylo uspané.

### Ukončené

Toto je konečný stav. Do stavu ukončené se vlákno dostane tehdy, když buď dokončí vše, co mělo provést, nebo nastala nějaká nečekaná situace.

## Vytvoření

Vlákno můžeme vytvořit čtyřmi způsoby:

 - pomocí třídy implementující rozhraní <b>Runnable</b>

```java
public class DemoThreading implements Runnable {
  public void run() {
    // sem přijde kód, který chceme spouštět ve vlákně
  }
}
```

 - pomocí třídy rozšiřující třídu <b>Thread</b>

```java
public class DemoThreading extends Thread {
  public void run() {
    // sem přijde kód, který chceme spouštět ve vlákně
  }
}
```

Takhle to zatím vypadá velmi podobně. Rozdíl je v tom, jak jednotlivá vlákna spustíme:

 - implements Runnable - vytvoříme si instaci té naší třídy implementující rozhraní Runnable a tuto instanci předáme jako parametr konstruktoru třídy Thread - na takto vytvořené instanci třídy thread pak zavoláme metodu start()
 
 ```java
 DemoThread demoThread = new DemoThread();
 Thread thread = new Thread(demoThread);
 thread.start();
 ```
 
 - extends Thread - tady stačí vytvořit instaci té naší třídy rozšiřující třídu Thread a zavolat na ní metodu start()

```java
DemoThread demoThread = new DemoThread();
demoThread.start();
```

- dále pak pomocí anonymní implementace rozhraní <b>Runnable</b>
```java
Runnable demoRunnable = new Runnable() {
  public void run() {
    // sem přijde kód, který chceme spouštět ve vlákně
  }
}

Thread thread = new Thread(demoRunnable);
 thread.start();
```

 - a nakonec pomocí implementace rozhraní <b>Runnable</b> použitím Lambda výrazu
```java
Runnable demoRunnable = () -> {
  // sem přijde kód, který chceme spouštět ve vlákně
};

Thread thread = new Thread(demoRunnable);
 thread.start();
```

Pozor na to, že je potřeba zavolat metodu start() a ne metodu run(). Výsledek se sice může zdát stejný, ale pokud zavoláte metodu run(), tak nedojde ke spuštění v novém vlákně.

## Hlavní vlákno

Po spuštění programu Java se okamžitě spustí jedno vlákno. Obvykle se tomu říká hlavní vlákno programu, protože je to vlákno, které se spustí při spuštění programu. Hlavní vlákno je důležité hned ze dvou důvodů:

 -  je to vlákno, ze kterého budou vytvořena další „podřízená“ vlákna
 -  musí to být poslední vlákno, které dokončí provádění. Když se hlavní vlákno zastaví, váš program se ukončí

Ačkoli je hlavní vlákno vytvořeno automaticky při spuštění programu, lze jej ovládat pomocí objektu vlákna.

```java
Thread mainThread = Thread.getCurrentThread();
```

## Práce s vícero vlákny

Vláken můžeme mít i větší počet, ale k tomu, aby aby nám to bylo ku prospěchu, je potřeba vědět, jak spolu navzájem interagují a co se nám může pokazit.

Vlákna jedné aplikace pracují nezávisle a nad stejnými daty, takže je třeba si dát pozor na to, že pokud nijak neošetříme vzájemnou komunikaci, tak se rozhodně nemůžeme spolehnout na to, že dříve spuštěné vlákno bude i dříve provedené.

Také je třeba si dát pozor na to, že k uspání vlákna může dojít i pro nás v nevhodnou chvíli (třeba uprostřed aktualizace nějakého zdroje).

Kupříkladu tento nevinně vypadající kód ve skutečnosti skýtá několik nástrah..

```java
public class RunnableIncrementator implements Runnable {

    private Integer incrementee;
    private int steps;

    public RunnableIncrementator(Integer incrementee, int steps) {
        this.incrementee = incrementee;
        this.steps = steps;
    }

    @Override
    public void run() {
        for (int i = 0; i < steps; i++) {
            incrementee++;
        }
        System.out.println(incrementee);
    }

}

public class Main {

    public static void main(String[] args) {

        Integer myIncrementee = 0;

        RunnableIncrementator ri = new RunnableIncrementator(myIncrementee, 1000);
        Thread t1 = new Thread(ri);
        Thread t2 = new Thread(ri);

        t1.start();
        t2.start();

    }
}
```

Vytvoříme si třídu implemntující rozhraní Runnable, která dostane dva parametry - Integer, který budeme postupně inkrementovat a int specifikující, kolikrát ho budeme inkrementovat. V metodě run() pak v cyklu postupně ten specifikovaný Integer inkrementujeme a nakonec jeho hodnotu vypíšeme.

Pokud začneme na 0 a budeme inkrementovat ve dvou vláknech o tisícovku, tak by se dalo čekat, že se nám nejprve vypíše číslo v rozmezí od 1000 do 1999 (jedno z vláken doběhlo, takže hodnota bude minimálně 1000, ale druhé už také mohlo stihnout několik cyklů, takže hodnota může být o něco větší), jenže když si tento kód zkusíte spustit, tak zjistíte že budeme dostávat různé "nečekané" kombinace hodnot..
První vypsané číslo může být vyšší, než druhé, první může být menší, než 1000 a druhé může být menší, než 2000. Proč? Protože vlákna žijí vlastním životem a čas na procesoru jim může být přidělován a odebírán v ty nejnevhodnější momenty a proto, že ač inkrementace vypadá jako jednokroková operace, tak ve skutečnosti jde o tři kroky - načtení hodnoty, zvýšení o jedna a uložení hodnoty a nám se může stát to, že jedno vlákno si hodnotu načte, zvýší i jedna a pak je mu odebrán čas na procesoru a je přidělen druhému vláknu. To si opět načte inkrementovanou hodnotu, zvýší o jedna a uloží.. toto může udělat i několikrát za sebou, ale když se ke slovu dostane opět první proces, tak hodnotu přepíše tou hodnotou, kterou naposledy nestihl zapsat. Nám by se tedy hodilo, aby některé části kódu byly prováděny atomicky - provedou se vždy jako celý blok, čímž předejdeme těmto problémům a k tomu máme v Javě synchronized.

### Synchronized

Synchronized můžeme použít buď v kombinaci s nějakou metodou nebo s nějakým blokem kódu. Co toto klíčové slovo zajistí? To, že k metodě, či k bloku kôdu s tímto klíčovým slovem bude mít v jednu chvíli přístup vždy jen jedno vlákno. Jak to vypadá?

V případě metod takto:

```java
public synchronized void doStuff() {
  // kód
}
```

případně:
```java
public static synchronized void doStuff() {
  // kód
}
```

Už víme, že static metoda je metodou třídy a metoda bez klíčového slova static pak patří konkrétní instanci třídy, což tady hraje další roli. V případě, že jde o static metodu, tak ji vždy může v jeden moment provádět jen jedno vlákno, ale v případě metody instance se toto omezení vstahuje na vždy na konkrétní instanci. Pokud si tedy vytvoříme dvě instance třídy, která má metodu s klíčovým slovem synchronized, tak u každé instance může tu metodu v jeden moment provádět jen jedno vlákno, ale nemusí jít o stejné vlákno. Pokud máme ve třídě víc metod s tímto klíčovým slovem, tak se toto omezení vstahuje vždy na všechny metody jako množinu - pokud jedno vlákno provádí nějakou z těchto metod, tak žádné jiné vlákno nemůže provádět žádnou z metod z této množiny.

Podobně to funguje i při synchronizaci bloků kódu, jen tam vždy ještě specifikujeme objekt, vůči kterému se ta synchronizace bude provádět:

```java
public void doStuff() {
  // kód
  synchronized(this) {
    // synchronizovaná část kódu
  }
  // kód
}
```

### Volatile

Pokud máme proměnnou, do které zapisuje nějaké vlákno, ale čte z ní víc vláken, tak pomocí klíčového slova volatile zajistíme to, že kdykoli načítáme tuto hodnotu, tak ji načítáme přímo z hlavní paměti a kdykoli ji zapisujeme, tak ji opět zapisujeme přímo do hlavní paměti. Tímto předejdeme situacím, kdy je aktuální hodnota pouze v cache zapisujícího vlákna a ostatní vlákna na ni nevidí.

Použití je naprosto jednoduché:

```java
private volatile String volatileText;
```

### Zámky

Zámky, podobně jako klíčové slovo synchronized, slouží k synchronizaci mezi jednotlivými vlákny a krom jiného mohou poskytovat i férovost přidělování zdrojů.

Když jsme měli klíčové slovo synchronized, tak jsme jim chránili metodu, či blok kódu před tím, aby ho najednou používalo víc vlákem:

```java
// nechráněný kód
synchronized(this) {
 //chránený blok kódu
}
//nechráněný kód
```
V případě záknu by kód vypadal nějak takto:
```java
// nechráněný kód
// zamknutí zámku
{
  // chránený kód
}
// odemknutí zámku
// nechráněný kód
```

Aby nám to fungovalo, tak potřebujeme zámek - nástroj, který funguje následovně:
 - pokud je zámek odemcěný, může ho jedno vlákno zamčít
 - pokud je zámek zamčený a nějaké vlákno ho potřebuje, tak to vlákno musí čekat až bude odemčený
 - zamčený zámek po dokončení své činnosti odemyká zase to vlákno, které ho zamčelo

Vzorová implementace základního zámku by pak mohla vypadat takto:

```java
public class SimpleLock {

    private boolean isLocked = false;

    public synchronized void lock() throws InterruptedException {
        while (isLocked) {
            wait();
        }
        isLocked = true;
    }

    public synchronized void unlock() {
        isLocked = false;
        notify();
    }

}
```

Existuje také v javě již přítomná implementace ReentrantLock, která umožňuje oproti základnímu zámku vícenásobné zamykání jedním stejným vláknem, což se může hodit, pokud máme vícero metod, kdy každá má ošetřeno to, že potřebuje nějaký stejný zdroj. V případě vícnásobného zamčení musí to vlákno, které ho zamklo, zase stejný počet krát odemčít.

Při použití zámků v kombinaci s kódem, který může vyhodit výjimku, je dobrou praxí umístit odemykání zámku do finally bloku, aby došlo k jeho uvolnění i tehdy, když nastane nějaká výjimka.

```java
lock.lock()
try {
  // sem přijde kód, který může vyhodit výjimku
} catch () {
  // ošedtčení výjimky
} finally {
  lock.unlock();
}
```

### Komunikace mezi vlákny

K přímé komunikaci mezi vlákny jde použít také metod wait() a notify() volaných na nějakém monitorovacím objektu - ty nám poslouží k tomu, aby vlákno zbytečně aktivně nečekalo na nejakou událost - pomocí metody .wait() čekající vlákno uspíme a až situace nastane, tak je probereme pomocí .notify().

## Sdílení paměti

- RAW - Read And Write - ještě nemusí být k dispozici 

- WAW - Write And Write - která hodnota "vyhraje"

- WAR - Write And Read -  může dojít k přečtení staré hodbnoty

## Deadlock

Deadlock, neboli uváznutí, je nežádoucí situace, která může nastat, pokud jsou splněny 4 nutné podmínky a pokud nastane, tak se náš program "zasekne", protože jedno vlákno potřebuje k dokončení zdroj držený druhým vláknem a druhé vlákno zase čeká na zdroj držený tím prvním vláknem. Této nežádoucí situaci se předchází tím, že se zneplatní aspoň jedna ze čtyř nutných podmínek pro deadlock.

### Nutné podmínky

- Vzájemné vyloučení - ke konkrétnímu prostředku může v konkrétní chvíli přistupovat jen jedno vlákno

- Hold and Wait - vlákna se mohou snažit získat prostředky, i když už nějaké mají

- Neodnímaetelnost - vláknům nelze jednou přidělené prostředky odejmout; aby byly opět k dispozici, tak je musí samo vlákno uvolnit

- Cyklické čekání - z procesů a prostředků, na které tyto procesy čekají, jde uzavřít cyklus (Proces 1 má prostředek 1 a čeká na prostředek 2, proces 2 má prostředek 2 a čeká na prostředek 1)

### Předcházení deadlocku

Deadlocku jsme schopni předejít tak, že zajistíme, že minimálně jedna z nutných podmínek nenastane.

- Vzájemné vyloučení - umožnení přistupovat k jenomu prostředku vícero vláknům najednou 

- Hold and Wait - vlákna si musí o všechny prostředky, které budou potřebovat, zažádat najednou a buď obdrží všechny, nebo žádné

- Neodnímaetelnost - přidání možnosti odejmout prostředky

- Cyklické čekání - vzniku cyklu můžeme předejít tím, že bude existovat předem definované pořadí, v jakém se prostředky získávají
