# Laboratorium 6 — Testy i wyjątki

## Cel zajęć

Celem laboratorium jest:
- nauczenie się pisania i uruchamiania **testów jednostkowych** w
  bibliotece JUnit 5,
- zastanowienie się, *ile* testów wystarczy i *co* właściwie
  sprawdzają,
- poznanie **wyjątków** jako sposobu zgłaszania sytuacji, w których
  metoda nie może wykonać swojego zadania,
- zrozumienie różnicy między wyjątkami **kontrolowanymi**
  i **niekontrolowanymi** oraz decydowanie, *gdzie* wyjątek należy
  obsłużyć.

Pracujemy na aplikacji z licznikami z poprzednich laboratoriów.

---

## Wprowadzenie

Do tej pory sprawdzaliśmy, czy program działa, uruchamiając go i
klikając. Na laboratorium 4 taki sposób sprawdzania pozwolił nam w
końcu znaleźć błąd z dekoratorami, ale kosztowało to sporo czasu, a
po każdej kolejnej zmianie w kodzie trzeba by klikać od nowa.

**Test jednostkowy** to krótka metoda, która tworzy kilka obiektów,
wywołuje ich metody i sprawdza, czy wyniki są takie, jakich się
spodziewamy. Takich testów mogą być setki, a uruchomienie wszystkich
trwa ułamek sekundy, więc można to robić po każdej zmianie.

## Zadanie 1 - pierwszy test

Testy umieszczamy w osobnym katalogu `test`, ale w **tym samym
pakiecie**, co testowane klasy. Utwórz plik
`test/counters/TestSimpleCounter.java`:

```java
package counters;

import static org.junit.jupiter.api.Assertions.*;
import org.junit.jupiter.api.Test;

class TestSimpleCounter {
    @Test
    void startsAtZero() {
        assertEquals(0, new SimpleCounter().currentValue());
    }
}
```

Adnotacja `@Test` oznacza metodę jako test. Metoda
`assertEquals(oczekiwana, rzeczywista)` kończy test niepowodzeniem,
jeśli obie wartości nie są równe.

- **W Eclipse**: kliknij projekt prawym przyciskiem i wybierz
  *New > Source Folder*, nazwij go `test`. Następnie wybierz
  *New > JUnit Test Case*, zaznacz *New JUnit Jupiter test*, jako
  *Source folder* podaj `test`, a jako pakiet `counters`. Eclipse
  zaproponuje dodanie biblioteki JUnit 5 do projektu; zgódź się. Testy
  uruchamiamy, klikając klasę testową (albo cały katalog `test`) prawym
  przyciskiem i wybierając *Run As > JUnit Test*.
  
- **Z wiersza poleceń** (dla osób, które nie korzystają z Eclipse):
  pobierz plik
	  [junit-platform-console-standalone-1.14.4.jar](https://repo1.maven.org/maven2/org/junit/platform/junit-platform-console-standalone/1.14.4/junit-platform-console-standalone-1.14.4.jar)
  i umieść go w katalogu, w którym znajdują się katalogi `counters`,
  `app` i `test`. Następnie wykonaj:

	```
	javac -d out counters/*.java app/*.java
	javac -d out-test -cp out:junit-platform-console-standalone-1.14.4.jar test/counters/*.java
	java -jar junit-platform-console-standalone-1.14.4.jar execute --class-path out:out-test --scan-class-path
	```

	Kiedy w zadaniu 5 pojawią się testy w katalogu `test/app`, dopisz
	`test/app/*.java` na końcu drugiego polecenia.

	(W systemie Windows elementy ścieżki `-cp` i `--class-path` oddziela
	się średnikiem zamiast dwukropka, a w PowerShellu całą ścieżkę trzeba
	ująć w cudzysłów, np. `"out;out-test"`.)

Dopisz do klasy `TestSimpleCounter` testy, które sprawdzają, że:
- po trzech wywołaniach `increase()` licznik wskazuje 3,
- po `reset()` licznik znowu wskazuje 0.

Uruchom testy. Następnie *celowo zepsuj* metodę `step()` w klasie
`SimpleCounter` (np. niech zwiększa licznik o 2) i uruchom testy
ponownie. Wklej komunikat o błędzie do sprawozdania.

Zamień teraz w jednym z testów kolejność argumentów `assertEquals` i
uruchom testy jeszcze raz. Co mówi komunikat? Dlaczego kolejność
argumentów ma znaczenie, skoro równość jest symetryczna?

Napraw `SimpleCounter`.

W sprawozdaniu odpowiedz na pytania:
- testy są w pakiecie `counters`, więc mają dostęp do wszystkiego, co
  na laboratorium 4 zostało pozostawione bez modyfikatora dostępu. Czy
  Twoje testy z tego korzystają? Czy dałyby się skompilować, gdyby
  były w innym pakiecie?
- czy test powinien sprawdzać to, co widać przez interfejs `Counter`,
  czy wewnętrzne szczegóły klasy (np. pole `counter`)? Co się stanie z
  takimi testami, kiedy zmienimy implementację klasy?

## Zadanie 2 - ile testów wystarczy?

Napisz klasę `TestOverflowCounter` z testami licznika
`OverflowCounter`, które Twoim zdaniem wystarczą, żeby uznać, że
licznik działa poprawnie. Zanim zaczniesz, zapisz w sprawozdaniu,
jakie przypadki zamierzasz sprawdzić i dlaczego właśnie te.

Kiedy testy przechodzą, wyobraź sobie, że kolega z zespołu
„uprościł” metodę `step()` w klasie `OverflowCounter`. Zaproponował
trzy wersje:

```java
// wersja A
void step() { counter = (counter + 1) % 9; }

// wersja B
void step() {
    if (counter <= 9) {
        ++counter;
    } else {
        counter = 0;
    }
}

// wersja C
void step() { counter = (counter + 1) % 10; }
```

Wstaw po kolei każdą z wersji w miejsce oryginalnej metody i uruchom
swoje testy. Zapisz w tabeli, które testy nie przeszły przy każdej z
wersji. Na koniec przywróć oryginalną metodę.

W sprawozdaniu odpowiedz na pytania:
- czy Twoje testy wykryły każdą błędną wersję? Jeśli nie, to jakiego
  przypadku zabrakło? Dopisz brakujące testy.
- czy któraś z wersji jest poprawna? Skąd to wiesz, i czy to dobrze,
  że testy ją przepuszczają?

## Zadanie 3 - testowanie powiadomień

Na laboratorium 4 szukaliśmy błędu, przez który etykieta nie nadążała
za licznikami z dźwiękiem: dekorator powiadamiał słuchacza, podając
jako źródło opakowany licznik zamiast siebie.

Napisz testy, które sprawdzają, że:
- licznik powiadamia zarejestrowanego słuchacza po `increase()` i po
  `reset()`,
- dekorator (np. `new Beeping(new SimpleCounter())`) podaje jako
  źródło powiadomienia *siebie*,
- licznik `new Beeping(new Beeping(new DoubleCounter()))` powiadamia
  słuchacza po jednym `increase()` *dokładnie raz*.

Testy nie mogą korzystać z okna aplikacji. Zastanów się, jak w takim
razie sprawdzić, czy słuchacz został powiadomiony, i przez kogo.
(**Wskazówka**: nic nie stoi na przeszkodzie, żeby w klasie testowej
zdefiniować własną klasę implementującą `CounterListener`. Do
sprawdzenia, czy dwie referencje wskazują *ten sam* obiekt, służy
metoda `assertSame`. Tego rodzaju klasy nazywa się niekiedy *dublerami
testowymi*)

Następnie na chwilę przywróć w dekoratorach zachowanie z początku
laboratorium 4: niech metoda `addListener` dekoratora po prostu
przekazuje słuchacza opakowanemu licznikowi,
`counter.addListener(listener)`, a dekorator niech nie rejestruje się
jako słuchacz. Uruchom testy i przeczytaj komunikaty. Potem przywróć
poprawną wersję.

W sprawozdaniu odpowiedz na pytania:
- jakiej klasy użył(a)ś zamiast okna aplikacji? Czym różni się od
  `CounterApp` i dlaczego do testu wystarczy?
- czy taki test oszczędziłby czas na laboratorium 4?
- skąd biorą się napisy w komunikatach o niepowodzeniu `assertSame`?

## Zadanie 4 - najpierw testy, potem kod

Chcemy mieć nowy rodzaj licznika: **licznik z limitem**, który nie
może przekroczyć podanej w konstruktorze wartości. Tym razem zaczniemy
od testów. Utwórz plik `test/counters/TestLimitedCounter.java`:

```java
package counters;

import static org.junit.jupiter.api.Assertions.*;
import org.junit.jupiter.api.Test;

class TestLimitedCounter {
    @Test
    void countsUpToLimit() {
        Counter counter = new LimitedCounter(2);
        counter.increase();
        counter.increase();
        assertEquals(2, counter.currentValue());
    }

    @Test
    void refusesToExceedLimit() {
        Counter counter = new LimitedCounter(1);
        counter.increase();
        assertThrows(IllegalStateException.class, 
			() -> counter.increase());
        assertEquals(1, counter.currentValue());
    }

    @Test
    void zeroLimitRefusesAtOnce() {
        Counter counter = new LimitedCounter(0);
        assertThrows(IllegalStateException.class, 
			() -> counter.increase());
    }

    @Test
    void rejectsNegativeLimit() {
        assertThrows(IllegalArgumentException.class, 
			() -> new LimitedCounter(-1));
    }

    @Test
    void resetAllowsCountingAgain() {
        Counter counter = new LimitedCounter(1);
        counter.increase();
        counter.reset();
        counter.increase();
        assertEquals(1, counter.currentValue());
    }
}
```

Przeczytaj uważnie powyższy kod. Stanowi on specyfikację
dla klasy `LimitedCounter`, którą mamy stworzyć. W szczególności
zwróć uwagę na użycie metody `assertThrows`.

Metoda ta wykonuje podaną funkcję i sprawdza, czy zakończyła się ona
wyjątkiem podanej klasy. (Konstruktorowi rekordu `Range` z
poprzedniego laboratorium kazaliśmy w podobnej sytuacji zgłosić
wyjątek instrukcją `throw`. W tym przypadku również będziemy chcieli
rzucać określone typy wyjątków.)

Testy na razie się nie kompilują, bo klasy `LimitedCounter` jeszcze
nie ma.

Zaimplementuj ją (jako podklasę `AbstractCounter`) tak, żeby wszystkie
testy przeszły.

Dopisz jeszcze test, który sprawdza, czy słuchacz licznika z limitem
jest powiadamiany, kiedy licznik *odmówił* zwiększenia. Czy powinien
być? Czy Twoja implementacja zachowuje się zgodnie z Twoją odpowiedzią?
Dlaczego?

Dodaj licznik `new LimitedCounter(5)` do listy w aplikacji. Zwiększ
go do 5 i kliknij przycisk jeszcze raz. Co się stało z aplikacją? Gdzie
trafił komunikat o wyjątku? Dodaj następnie do listy licznik
`new Ticking(new LimitedCounter(3))` i obserwuj go przez kilkanaście
sekund (oraz konsolę).

W sprawozdaniu odpowiedz na pytania:
- czy kompilator ostrzegł Cię w jakikolwiek sposób, że wywołanie
  `increase()` w `CounterApp` lub w `Ticking` może się teraz skończyć
  wyjątkiem?
- interfejs `Counter` nic nie mówi o tym, że `increase()` może się
  nie udać. Czy klasy, które korzystają z liczników *tylko przez
  interfejs* `Counter` (np. `CounterApp`, `Ticking`), mogły się tego
  spodziewać? Czy `LimitedCounter` jest w takim razie dobrym
  licznikiem?
- zaproponuj co najmniej dwa sposoby rozwiązania tego problemu (np.
  zmiana zachowania licznika z limitem, zmiana interfejsu `Counter`,
  obsługa wyjątku w aplikacji). Każdy z nich oceń: ile klas trzeba
  zmienić i co by się stało z istniejącymi testami?

Wybierz jeden ze sposobów i zrealizuj go. Jeżeli wybierzesz obsługę
wyjątku w aplikacji, to użyj konstrukcji
`try { ... } catch (IllegalStateException e) { ... }` i pokaż
użytkownikowi komunikat, np. za pomocą
`JOptionPane.showMessageDialog(frame, e.getMessage())`.

## Zadanie 5 - zapis historii do pliku

Dodaj do aplikacji przycisk "Zapisz historię", który zapisuje
historię odczytów z poprzedniego laboratorium do pliku tekstowego,
po jednym odczycie w wierszu. Do zapisu użyj metody
`java.nio.file.Files.write(path, lines)`, gdzie `lines` to lista
napisów (`List<String>`), a ścieżkę `path` utwórz np. jako
`Path.of("historia.txt")`. (Elementy modelu
`DefaultListModel` można skopiować do listy za pomocą
`java.util.Collections.list(history.elements())`.)

Spróbuj skompilować program i przeczytaj komunikat kompilatora.
Porównaj go z sytuacją z zadania 4: czym różni się wyjątek
`IOException` od `IllegalStateException`? (Sprawdź w dokumentacji, po
jakich klasach dziedziczy każdy z nich.)

Kompilator daje dwie możliwości: obsłużyć wyjątek w miejscu wywołania
(`try`/`catch`) albo zadeklarować, że metoda go przekazuje dalej
(`throws IOException`). Zdecyduj, gdzie powinien zostać obsłużony.
Kto wie, jak powiadomić użytkownika o błędzie?

Napisz test sprawdzający, czy zapis działa: czy po zapisaniu dwóch
odczytów plik zawiera dwa wiersze o oczekiwanej treści. Czy da się
przetestować kod umieszczony bezpośrednio w metodzie
`actionPerformed`? Jeśli nie, przenieś zapis do osobnej klasy (np.
`HistoryFile` w pakiecie `app`) tak, żeby dało się go przetestować
bez okna.

Testy nie powinny zostawiać plików w katalogu projektu. JUnit może
przed każdym testem utworzyć pusty katalog tymczasowy i usunąć go po
teście:

```java
@TempDir
Path directory;   // import org.junit.jupiter.api.io.TempDir;
```

Plik w tym katalogu tworzymy jako `directory.resolve("historia.txt")`,
a jego zawartość możemy odczytać metodą `Files.readAllLines`.

Dopisz też test, który sprawdza, że zapis do pliku w nieistniejącym
katalogu kończy się wyjątkiem `IOException`. Uruchom aplikację i
sprawdź, co zobaczy użytkownik w takiej sytuacji (na chwilę zmień w
tym celu ścieżkę, np. na `nie-ma-takiego-katalogu/historia.txt`).

W sprawozdaniu odpowiedz na pytania:
- dlaczego twórcy Javy zdecydowali, że `IOException` *trzeba*
  obsłużyć, a `IllegalStateException` czy `IllegalArgumentException`
  nie? (Pomyśl, czy programista może *zapobiec* każdemu z tych
  błędów, pisząc poprawny kod.)
- dlaczego klasa zapisująca historię nie powinna sama łapać wyjątku
  `IOException`? Co by się stało, gdyby złapała go i nic z nim nie
  zrobiła (`catch (IOException e) {}`)?
- czy przeniesienie zapisu do osobnej klasy, wymuszone przez test,
  poprawiło też projekt aplikacji? Uzasadnij.

## Zadanie 6 - czego nie przetestowaliśmy?

Spróbuj napisać test licznika tykającego, który sprawdza, że po
upływie trzech sekund licznik `new Ticking(new SimpleCounter())`
wskazuje 3.

W sprawozdaniu odpowiedz na pytania:
- jakie trudności napotkał(a)ś? Ile trwałby taki test i czy zawsze
  dawałby ten sam wynik?
- co należałoby zmienić w *projekcie* klasy `Ticking` (nie w teście),
  żeby dało się ją przetestować szybko i niezawodnie? (Przypomnij
  sobie, jak w zadaniu 3 zastąpił(a)ś okno aplikacji czymś prostszym.)
- czy podczas uruchamiania testów z zadania 3 było słychać dźwięk?
  Czy test powinien wydawać dźwięki? Co by było, gdyby zamiast
  dźwięku dekorator wysyłał SMS-a?
- które fragmenty klasy `CounterApp` nadal nie są przetestowane?
  Co z nich dałoby się wydzielić do klas, które można przetestować bez
  okna?
