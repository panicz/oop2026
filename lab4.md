# Laboratorium 4 — Obserwator, pakiety i modyfikatory dostępu

## Cel zajęć

Celem laboratorium jest:
- poznanie wzorca projektowego **Obserwator** (ang. *Observer*),
- zrozumienie, dlaczego warto, żeby obiekt sam informował innych
  o swoich zmianach, zamiast czekać, aż ktoś go o nie zapyta,
- podział programu na **pakiety** i świadome dobieranie
  **modyfikatorów dostępu**.

Punktem wyjścia jest aplikacja `CounterApp` w wersji z poprzedniego
laboratorium (z klasą `AbstractCounter` i dekoratorem `Beeping`).

---

## Wprowadzenie

W obecnej wersji aplikacji wartość wyświetlana na etykiecie zmienia
się tylko w jednym momencie: kiedy klikniemy przycisk. Metoda
`actionPerformed` zwiększa licznik, a potem *pyta* go o aktualną
wartość i wpisuje ją do etykiety.

Takie podejście działa, dopóki jedynym powodem zmiany licznika jest
kliknięcie w naszym oknie. Za chwilę przekonamy się, co się dzieje,
kiedy to założenie przestaje być prawdziwe.

## Zadanie 1 - licznik, który tyka sam

Na poprzednim laboratorium napisaliśmy dekorator `Beeping`, który
dodaje do dowolnego licznika wydawanie dźwięku. W podobny sposób
napiszemy dekorator `Ticking`, który sam zwiększa opakowany licznik
co sekundę.

Do odmierzania czasu użyjemy klasy `javax.swing.Timer`. Jej
konstruktor pobiera odstęp w milisekundach oraz obiekt implementujący
znany nam już interfejs `ActionListener`, którego metoda
`actionPerformed` będzie wywoływana co zadany odstęp:

```java
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;
import javax.swing.Timer;

...

new Timer(1000, new ActionListener() {
    public void actionPerformed(ActionEvent e) {
        increase();
    }
}).start();
```

**Uwaga**: w bibliotece standardowej Javy są dwie klasy o nazwie
`Timer`: `javax.swing.Timer` i `java.util.Timer`. Dlatego klasę
`Timer` importujemy pojedynczo. Jeśli w tym samym pliku pojawią się
`import javax.swing.*;` i `import java.util.*;`, kompilator zgłosi
błąd `reference to Timer is ambiguous`. Potrzebujemy właśnie
`javax.swing.Timer`, bo wywołuje on `actionPerformed` w tym samym
wątku, w którym Swing obsługuje okno, więc z tej metody można
bezpiecznie zmieniać zawartość okna.

Zdefiniuj klasę `Ticking`, która:
- implementuje interfejs `Counter`,
- jako argument konstruktora pobiera obiekt typu `Counter` i oddelegowuje
  do niego wszystkie metody interfejsu (tak jak `Beeping`),
- w konstruktorze uruchamia powyższy `Timer`,
- ma czytelną nazwę (np. `zwykły (tykający)`).

Dodaj do listy liczników w `CounterApp` element
`new Ticking(new SimpleCounter())`, wybierz go i odczekaj kilka sekund.
Następnie kliknij przycisk.

W sprawozdaniu odpowiedz na pytania:
- metoda `actionPerformed` klasy anonimowej wywołuje `increase()`.
  Metodę `increase()` którego obiektu?
- co wyświetla etykieta przed kliknięciem, a co po nim? Dlaczego?
- który obiekt w programie "wie", że wartość licznika się zmieniła,
  a który powinien się o tym dowiedzieć?

## Zadanie 2 - słuchacze licznika

Rozwiązaniem problemu z zadania 1 jest odwrócenie kierunku
komunikacji: to licznik powinien *powiadamiać* zainteresowanych o
każdej swojej zmianie. Z tym rozwiązaniem zetknęliśmy się już przy
okazji przycisków: `JButton` nie wie nic o naszej aplikacji, a mimo
to potrafi ją powiadomić o kliknięciu, wywołując metodę
`actionPerformed` każdego obiektu, który wcześniej zarejestrował
metodą `addActionListener`.

Ten sposób organizacji programu nazywamy wzorcem **Obserwator**.

Zdefiniuj interfejs:

```java
public interface CounterListener {
    void counterChanged(Counter source);
}
```

a do interfejsu `Counter` dodaj metodę:

```java
void addListener(CounterListener listener);
```

Klasa `AbstractCounter` powinna przechowywać listę zarejestrowanych
słuchaczy (np. w obiekcie `java.util.ArrayList<CounterListener>`) i po
każdej zmianie licznika wywoływać metodę `counterChanged` każdego z
nich, przekazując jako argument samą siebie (`this`).

Zastanów się, w którym miejscu wywoływać powiadamianie słuchaczy.
Pamiętaj, że licznik zmienia się nie tylko w metodzie `increase()`,
ale też w metodzie `reset()`.

Metodę `increase()` definiuje obecnie każda podklasa osobno. Jak
sprawić, żeby autor nowej podklasy nie mógł zapomnieć o powiadomieniu
słuchaczy?

(**Wskazówka**: klasa `AbstractCounter` może sama zdefiniować metodę
`increase()` jako `final`, a podklasom zostawić do zdefiniowania
jedynie samo "przesunięcie" licznika, np. w metodzie abstrakcyjnej
`step()`. Taki sposób organizacji kodu nazywa się wzorcem **metody
szablonowej**, ang. *Template Method*.)

Jeżeli w Twoim projekcie została klasa `BeepingCounter` z zadania 4
z poprzedniego laboratorium, spróbuj zbudować program. Co się stało i
dlaczego? Czy to dobrze? Usuń tę klasę z projektu (jej rolę i tak
pełni teraz dekorator `Beeping`).

Klasy `Beeping` i `Ticking` również muszą teraz implementować metodę
`addListener`. Na razie niech po prostu oddelegowują ją do
opakowanego licznika.

Kod klas `CounterListener`, `AbstractCounter` i jednej z podklas wklej
do sprawozdania.

## Zadanie 3 - aplikacja jako obserwator

Zmodyfikuj klasę `CounterApp` tak, żeby:
- implementowała interfejs `CounterListener` i rejestrowała się jako
  słuchacz każdego licznika z listy (elementy listy `JComboBox`
  można przejrzeć za pomocą metod `getItemCount()` i `getItemAt(i)`),
- w metodzie `counterChanged` aktualizowała etykietę, ale tylko
  wtedy, gdy zmienił się licznik aktualnie wybrany na liście (czyli
  gdy `source` jest *tym samym obiektem*, co
  `counters.getSelectedItem()`),
- metoda `actionPerformed` nie zajmowała się już wcale etykietą.

Uruchom aplikację. Wybierz zwykły licznik i kliknij kilka razy
przycisk, a potem przycisk "Reset". Czy etykieta się zmienia?

Co pokazuje etykieta zaraz po wybraniu na liście innego licznika?
Popraw to. (**Wskazówka**: lista `JComboBox`, podobnie jak przycisk,
pozwala zarejestrować `ActionListener`, który jest wywoływany po
zmianie wybranego elementu. Jeśli zarejestrujesz jako słuchacza listy
obiekt `this`, przypomnij sobie zadanie 6 z drugiego laboratorium:
skąd metoda `actionPerformed` ma wiedzieć, czy kliknięto przycisk,
czy zmieniono wybór na liście?)

Na koniec wybierz licznik tykający. Czy jego etykieta zmienia się
sama? Zanotuj, co widzisz; wrócimy do tego w następnym zadaniu.

W sprawozdaniu odpowiedz na pytania:
- co zostało w metodzie `actionPerformed`?
- czy klasy z pakietu liczników (`AbstractCounter`, `SimpleCounter`
  itd.) muszą cokolwiek wiedzieć o Swingu i o klasie `CounterApp`?
  Dlaczego to dobrze?

## Zadanie 4 - dekoratory i tożsamość obiektu

Etykieta licznika tykającego z poprzedniego zadania nie zmienia się
sama. Podobnie jest z licznikami z dźwiękiem: wybierz z listy
licznik `new Beeping(new SimpleCounter())` i kliknij przycisk kilka
razy.

Ustal, który obiekt jest przekazywany jako argument `source` metody
`counterChanged`, a który obiekt jest wybrany na liście (możesz w
tym celu wypisywać je na konsolę w metodzie `counterChanged`). Opisz
w sprawozdaniu, na czym polega problem.

(Jeśli u Ciebie etykieta licznika tykającego zmieniała się sama już
w zadaniu 3, to znaczy, że Twoje dekoratory nie oddelegowują
`addListener` tak, jak sugerowało zadanie 2. Opisz w sprawozdaniu, co
zrobił(a)ś inaczej i dlaczego. Następnie na chwilę zastąp swoją
implementację `addListener` w klasie `Ticking` prostym
oddelegowaniem, `counter.addListener(listener)`, i sprawdź, co się
stanie. Potem przywróć swoją wersję i przejdź od razu do pytań o
powtarzający się kod.)

Popraw dekoratory tak, żeby słuchacz dostawał jako `source` ten obiekt,
u którego się zarejestrował. (**Wskazówka**: dekorator może sam
zarejestrować się jako słuchacz opakowanego licznika i przekazywać
powiadomienia dalej, do własnych słuchaczy, podając siebie jako
źródło.)

Ile kodu musiał(a)byś powtórzyć w klasach `Beeping` i `Ticking`? Czy
da się ten kod umieścić w jednym miejscu? (Przypomnij sobie, jak na
poprzednim laboratorium wyodrębniliśmy klasę `AbstractCounter`.)

Sprawdź, czy po poprawkach działają również liczniki
`new Ticking(new Beeping(new OverflowCounter()))` oraz
`new Beeping(new Beeping(new DoubleCounter()))`.

Czy obsługa listy słuchaczy w dekoratorach przypomina obsługę listy
słuchaczy w klasie `AbstractCounter`? Czy ją także dałoby się
zdefiniować tylko raz?

## Zadanie 5 - pakiety i modyfikatory dostępu

Przenieś wszystkie klasy i interfejsy związane z licznikami do pakietu
`counters`, a klasę `CounterApp` do pakietu `app`. Pamiętaj, że:
- na początku każdego pliku należy umieścić deklarację pakietu, np.
  `package counters;`,
- pliki muszą trafić do katalogów o nazwach odpowiadających pakietom,
- klasa z innego pakietu jest dostępna po zaimportowaniu, np.
  `import counters.Counter;` albo `import counters.*;`.

Spróbuj zbudować program i przeczytaj komunikaty kompilatora.

Jeśli budujesz program z wiersza poleceń, a nie w środowisku
programistycznym, wykonaj w katalogu, w którym znajdują się katalogi
`counters` i `app`:

```
javac -d out counters/*.java app/*.java
java -cp out app.CounterApp
```

Następnie dobierz modyfikatory dostępu (`public`, `protected`, brak
modyfikatora, `private`) tak, żeby **każdy element był możliwie jak
najmniej widoczny**, a program nadal się budował i działał. W
szczególności zdecyduj o:
- polu `counter` w klasie `AbstractCounter`,
- liście słuchaczy w klasie `AbstractCounter`,
- metodzie powiadamiającej słuchaczy,
- metodzie `step()` (o ile ją wprowadzono w zadaniu 2),
- konstruktorach i klasach dekoratorów.

W sprawozdaniu umieść tabelę, w której dla każdego z tych elementów
podasz wybrany modyfikator i krótko uzasadnisz wybór.

(**Uwaga**: sprawdź w dokumentacji albo w podręczniku, który z
modyfikatorów `protected` i "brak modyfikatora" daje *szerszy*
dostęp. Odpowiedź bywa zaskakująca.)

Odpowiedz też na pytanie: czy po Twoich zmianach można by napisać
nowy rodzaj licznika (podklasę `AbstractCounter`) w pakiecie `app`?
Czy to dobrze, czy źle?

## Zadanie 6 - drugi widok

Dodaj do okna drugi sposób prezentacji wartości wybranego licznika,
np. pasek postępu (`JProgressBar`; jego zakres podaje się w
konstruktorze, np. `new JProgressBar(0, 20)`) albo zmianę tytułu okna.

Ile miejsc w kodzie trzeba było zmienić? Ile trzeba by ich zmienić,
gdyby aplikacja nadal *pytała* liczniki o wartość po kliknięciu, tak
jak przed zadaniem 3?

## Zadanie 7 - lista, która nadąża za licznikami

Zmień metodę `toString()` liczników tak, żeby oprócz nazwy zwracała
też aktualną wartość licznika, np. `zwykły: 7`. Uruchom aplikację i
pozmieniaj wartości kilku liczników. Czy napisy na liście
`JComboBox` się zmieniają?

Lista `JComboBox` nie przechowuje liczników bezpośrednio, tylko w
*modelu*, czyli obiekcie implementującym interfejs `ComboBoxModel`
(domyślnie jest to `DefaultComboBoxModel`). Sama lista jest
obserwatorem swojego modelu: przerysowuje się wtedy, gdy model
powiadomi ją o zmianie. O zmianach wewnątrz liczników model jednak
nic nie wie.

Zdefiniuj klasę `CounterListModel`, która:
- rozszerza klasę `DefaultComboBoxModel<Counter>`,
- w konstruktorze pobiera tablicę liczników, przekazuje ją do
  konstruktora klasy nadrzędnej i rejestruje się jako słuchacz
  każdego z liczników,
- w metodzie `counterChanged` odszukuje zmieniony licznik metodą
  `getIndexOf` i powiadamia listę o zmianie, wywołując odziedziczoną
  metodę `fireContentsChanged(this, i, i)`, gdzie `i` to indeks tego
  licznika.

Następnie utwórz listę w `CounterApp` za pomocą konstruktora
`JComboBox`, który przyjmuje model:

```java
new JComboBox<>(new CounterListModel(new Counter[]{ ... }))
```

Sprawdź, czy napisy na liście nadążają za zmianami, również w
przypadku liczników tykających i liczników z dźwiękiem.

W sprawozdaniu odpowiedz na pytania:
- dlaczego klasa `CounterListModel` musi *dziedziczyć* po
  `DefaultComboBoxModel`, a nie może go po prostu zawierać, tak jak
  dekoratory zawierają licznik? (Sprawdź w dokumentacji, jaki
  modyfikator dostępu ma metoda `fireContentsChanged`.)
- co by się stało, gdyby dekoratory nadal podawały jako źródło
  powiadomienia opakowany licznik, tak jak przed poprawkami z
  zadania 4?
- narysuj (np. w notacji Mermaid) łańcuch powiadomień: od licznika do
  przerysowania listy na ekranie. Ile w nim jest obserwatorów?
