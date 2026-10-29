# Laboratorium 5 — Obiekty i wartości

## Cel zajęć

Celem laboratorium jest:
- zrozumienie różnicy między **tożsamością** obiektu a **równością**
  wartości,
- poznanie kontraktu metod `equals` i `hashCode` i konsekwencji jego
  złamania,
- zrozumienie, dlaczego wartości powinny być **niezmienne**,
- poznanie **rekordów** (`record`), czyli składni Javy do definiowania
  złożonych wartości.

Do wykonania zadań potrzebna jest Java w wersji co najmniej 16.

---

## Wprowadzenie

Na poprzednim laboratorium porównywaliśmy liczniki operatorem `==`:
etykieta miała się zmieniać tylko wtedy, gdy źródło powiadomienia było
*tym samym obiektem*, co licznik wybrany na liście. I słusznie: dwa
zwykłe liczniki, które akurat oba wskazują 3, to wciąż dwa różne
liczniki. Każdy z nich ma swoją **tożsamość**, swój **stan** (który
się zmienia) i swoje **zachowanie**.

Nie wszystko jednak jest takie jak licznik. Liczba 3 to po prostu
liczba 3: nie ma sensu pytać, *która* trójka jest która. Podobnie jest
z punktem na płaszczyźnie, datą czy kwotą pieniędzy. Takie byty
nazywamy **wartościami**: o ich równości decyduje to, z czego się
składają, a nie to, w którym miejscu pamięci się znajdują.

Język Java przez długi czas nie dawał wygodnych środków do
definiowania złożonych wartości. Dzisiaj zobaczymy, dlaczego zrobienie
tego ręcznie jest trudniejsze, niż się wydaje.

## Zadanie 1 - punkt jako obiekt

Zdefiniuj klasę:

```java
public class Point {
    int x;
    int y;

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }
}
```

W metodzie `main` (np. w klasie `Points`) sprawdź i wypisz na konsolę
wartości poniższych wyrażeń:

```java
new Point(1, 2) == new Point(1, 2)
new Point(1, 2).equals(new Point(1, 2))
```

Następnie utwórz listę `java.util.ArrayList<Point>` i zbiór
`java.util.HashSet<Point>`, dodaj do każdego z nich dwa razy
`new Point(1, 2)` i wypisz:
- rozmiar listy i rozmiar zbioru,
- wynik wywołania `contains(new Point(1, 2))` na liście i na zbiorze.

W sprawozdaniu odpowiedz na pytania:
- skąd klasa `Point` ma metodę `equals`, skoro jej nie zdefiniowaliśmy?
  Co ta metoda porównuje? (Zajrzyj do dokumentacji klasy
  `java.lang.Object`.)
- czy zachowanie kodu było zgodne z Twoimi oczekiwaniem?

## Zadanie 2 - równość wartości

Przesłoń w klasie `Point` metodę `equals` tak, żeby dwa punkty były
równe wtedy, gdy mają równe współrzędne:

```java
@Override
public boolean equals(Object o) {
    return o instanceof Point p && p.x == x && p.y == y;
}
```

(Zapis `o instanceof Point p` sprawdza, czy `o` jest punktem, i jeśli
tak, to pozwala odwoływać się do niego jako do zmiennej `p` typu
`Point`.)

Powtórz eksperymenty z zadania 1. Które wyniki się zmieniły, a które
nie?

Przeczytaj w dokumentacji klasy `java.lang.Object` opis metody
`hashCode` (tzw. *kontrakt* metod `equals` i `hashCode`). Który punkt
tego kontraktu łamie teraz klasa `Point`? Dlaczego ma to znaczenie dla
`HashSet`, a nie ma dla `ArrayList`?

Przesłoń w klasie `Point` metodę `hashCode`, np. za pomocą metody
pomocniczej `java.util.Objects.hash(x, y)`, i powtórz eksperymenty
jeszcze raz.

W sprawozdaniu wyjaśnij własnymi słowami, w jaki sposób `HashSet`
używa metod `hashCode` i `equals`, kiedy sprawdza, czy zawiera dany
element.

## Zadanie 3 - wartość, która się zmienia

Dodaj do klasy `Point` metodę:

```java
public void move(int dx, int dy) {
    x += dx;
    y += dy;
}
```

Następnie:
1. utwórz punkt `p = new Point(1, 2)` i dodaj go do zbioru `HashSet`,
2. wywołaj `p.move(4, 0)`,
3. sprawdź, czy zbiór zawiera `p`, czy zawiera `new Point(5, 2)`, a
   czy `new Point(1, 2)`,
4. spróbuj usunąć `p` ze zbioru metodą `remove` i wypisz rozmiar
   zbioru,
5. wypisz wszystkie elementy zbioru w pętli `for`.

Opisz w sprawozdaniu, co się stało i dlaczego. (**Wskazówka**: w
którym miejscu zbioru punkt `p` został zapisany w kroku 1, a w którym
miejscu zbiór szuka go w kroku 3?)

Popraw klasę `Point` tak, żeby taka sytuacja nie mogła się zdarzyć:
- oznacz pola słowem kluczowym `final`,
- zamiast metody `move`, która zmienia punkt, zdefiniuj metodę
  `Point moved(int dx, int dy)`, która zwraca *nowy* punkt.

Odpowiedz w sprawozdaniu na pytanie: w jaki sposób podobny problem
rozwiązano w klasie `String`? (Co zwraca metoda `toUpperCase`?)

## Zadanie 4 - rekordy

Klasy takie jak nasza poprawiona klasa `Point` są w Javie tak częste,
że od wersji 16 język ma dla nich osobną składnię. Kiedy napiszemy

```java
record Point(int x, int y) {}
```

kompilator potraktuje to mniej więcej tak, jakbyśmy napisali:

```java
public final class Point extends java.lang.Record {
    private final int x;
    private final int y;

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    public int x() { return x; }
    public int y() { return y; }

    @Override
    public boolean equals(Object o) {
        return o instanceof Point p && p.x == x && p.y == y;
    }

    @Override
    public int hashCode() {
        return java.util.Objects.hash(x, y);
    }

    @Override
    public String toString() {
        return "Point[x=" + x + ", y=" + y + "]";
    }
}
```

Słowo `final` występuje tu w dwóch znaczeniach: `final class` oznacza,
że po klasie nie można dziedziczyć, a `final` przy polu oznacza, że
po inicjalizacji nie można zmienić jego wartości.

1. Zastąp klasę `Point` rekordem i dopisz do niego metodę
   `Point moved(int dx, int dy)` z zadania 3, która zwraca *nowy*
   punkt (rekordy mogą zawierać własne metody). Uruchom ponownie
   eksperymenty z zadań 1 i 2. Co się stanie, jeśli spróbujesz
   dopisać do rekordu metodę `move` z zadania 3, która zmienia pola
   punktu?

2. Napisz w sprawozdaniu, jak kompilator potraktuje rekord
   `record Point3D(int x, int y, int z) {}`, i wskaż, czym ta definicja
   różni się od definicji `Point`.

3. Rekord może zawierać *konstruktor kompaktowy*, który sprawdza
   poprawność argumentów, zanim zostaną przypisane polom:

   ```java
   record Range(int min, int max) {
       Range {
           if (min > max) {
               throw new IllegalArgumentException("min > max");
           }
       }
   }
   ```

   Sprawdź, co się stanie po wywołaniu `new Range(3, 1)`. (Wyjątkom
   przyjrzymy się dokładniej na kolejnym laboratorium.) Dlaczego
   sprawdzanie poprawności w konstruktorze wystarczy, żeby *każdy*
   obiekt `Range` w programie był poprawny? Czy to samo dałoby się
   powiedzieć o klasie `Point` z zadania 1?

## Zadanie 5 - historia odczytów

Wróćmy do aplikacji z licznikami z poprzedniego laboratorium. Chcemy
dodać do niej przycisk "Zapamiętaj", który dopisuje bieżący stan
wybranego licznika do historii wyświetlanej w oknie.

Okno aplikacji powinno wyglądać mniej więcej tak:

```
+--------------------------------------------------------------+
| [zwykły: 3 v] [Kliknij!] [Reset] [Zapamiętaj]  3             |
+--------------------------------------------------------------+
| zwykły: 3                                                    |
| zwykły: 3                                                    |
| do 10: 7                                                     |
|                                                              |
+--------------------------------------------------------------+
```

Na górze są dotychczasowe elementy okna i nowy przycisk, a poniżej
lista z historią. Dodaj do klasy `CounterApp` pola:

```java
private final JButton rememberButton = new JButton("Zapamiętaj");
private final DefaultListModel<Counter> history = new DefaultListModel<>();
private final JList<Counter> historyList = new JList<>(history);
```

Lista `JList` wyświetla elementy przechowywane w modelu
`DefaultListModel`. Nowy element dodajemy do listy, wywołując
`history.addElement(...)`.

W metodzie `run` zastąp instrukcje dodające elementy do okna (od
`frame.setLayout(...)` do ostatniego `frame.add(...)`) następującymi:

```java
JPanel controls = new JPanel();   // panel układający elementy w rzędzie
controls.add(counters);
controls.add(button);
controls.add(resetButton);
controls.add(rememberButton);
controls.add(label);
// ... i pozostałe elementy dodane na poprzednim laboratorium

frame.setLayout(new BorderLayout());
frame.add(controls, BorderLayout.NORTH);                       // u góry okna
frame.add(new JScrollPane(historyList), BorderLayout.CENTER);  // reszta okna, z przewijaniem
```

Pamiętaj, żeby podłączyć akcję do przycisku `rememberButton`. W jego
akcji dopisuj do historii wybrany licznik.

Uruchom aplikację, zapamiętaj kilka razy ten sam licznik, zmieniając
pomiędzy zapamiętaniami jego wartość. Jeśli w Twojej wersji metoda
`toString()` liczników nie pokazuje wartości (zadanie 7 z poprzedniego
laboratorium), wyświetlaj w historii wartość w inny sposób, np.
wypisując elementy historii na konsolę za pomocą `currentValue()`.

Co pokazuje historia? Dlaczego?

Zdefiniuj rekord

```java
record Reading(Counter counter, int value) {}
```

i przechowuj w historii obiekty `Reading` zamiast liczników. Czy
historia działa teraz poprawnie?

W sprawozdaniu odpowiedz na pytania:
- czym różniło się przechowywanie w historii licznika od
  przechowywania odczytu?
- przyjrzyj się napisom, którymi `JList` wyświetla odczyty (wygeneruje
  je metoda `toString()` rekordu). Która ich część się zmienia, a
  która nie? Czy rekord `Reading` jest rzeczywiście niezmienny?
  (Rekord gwarantuje, że jego pola się nie zmienią. Co jest
  przechowywane w polu `counter`?)

## Zadanie 6 - czy licznik jest wartością?

Skoro przesłonięcie `equals` i `hashCode` tak dobrze sprawdziło się w
przypadku punktów, spróbujmy zrobić to samo z licznikami.

Przesłoń w klasie `AbstractCounter` metody `equals` i `hashCode` tak,
żeby dwa liczniki tej samej klasy były równe wtedy, gdy mają tę samą
bieżącą wartość. (Klasę obiektu można sprawdzić metodą `getClass()`.)

Upewnij się, że na liście w aplikacji są co najmniej dwa liczniki tej
samej klasy (np. dwa zwykłe), oba z wartością 0. Uruchom aplikację i
spróbuj:
- przełączyć się z jednego z nich na drugi,
- zwiększyć wartość drugiego z nich (jeśli wykonano zadanie 7 z
  poprzedniego laboratorium, zwróć uwagę na napisy na liście).

Następnie w osobnym programie dodaj zwykły licznik do zbioru
`HashSet<Counter>`, zwiększ go i sprawdź, czy zbiór nadal go zawiera.

Opisz w sprawozdaniu, co się stało i dlaczego. (**Wskazówka**: model
`DefaultComboBoxModel` używa metody `equals` m.in. w metodach
`getIndexOf` i `setSelectedItem`.)

Usuń z klasy `AbstractCounter` dodane metody `equals` i `hashCode`.

## Zadanie 7 - obiekty i wartości

Na podstawie dzisiejszych zadań zdecyduj, które z poniższych bytów
powinny być w programie obiektami (z tożsamością), a które wartościami
(porównywanymi przez `equals`), i uzasadnij krótko każdą decyzję:

- licznik (`Counter`),
- punkt (`Point`),
- odczyt licznika (`Reading`),
- napis (`String`),
- przycisk w oknie (`JButton`),
- data (`java.time.LocalDate`),
- konto bankowe,
- kwota pieniędzy.

Sformułuj na tej podstawie regułę: jaki związek zachodzi między tym,
czy coś jest wartością, a tym, czy może się zmieniać?
