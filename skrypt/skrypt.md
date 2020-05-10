#Duża prezentacja na PDI2

## Wstęp
    Dzień dobry, nazywam się Marcin Skrzypkowski, jestem studentem Automatyki i Robotyki, 
    moim promotorem jest dr hab. inż. Wojciech Szynkiewicz z Instytutu Automatyki i Informatyki Stosowanej.
    Przez następne kilkadziesiąt minut przedstawię Państwu bliżej tematykę mojej pracy, 
    która opisuje autonomiczną nawigację robota mobilnego. 
    
    Celem pracy jest dalszy rozwój już dosyć zaawansowanej platformy badawczej 
    jaką jest robot manipulacyjny Velma na mobilnej bazie wielokierunkowej.
    Moim zadaniem jest rozszerzenie jego funkcjonalności o tworzenie mapy zamkniętych przestrzeni,
    ustalenie pozycji robota w zbudowanej mapie lub generowanej na bieżąco oraz
    generowanie i pokonanie ścieżki między dwoma punktami.
    Każdy z tych trzech problemów posiada wiele możliwych rozwiązań gdzie każde ma swoje wady i zalety,
    co krótko wytłumaczę.
    Przybliżę też kilka mniejszych problemów związanych ściśle z systemem nawigacji.
    
    Przejdźmy do dokładniejszego planu mojej prezentacji.
    
## Spis treści
      Na początek przedstawię robota Velma, jego budowę od strony mechanicznej jak i oprogramowania,
      żeby przybliżyć narzędzia potrzebne do osiągnięcia postawionych przed chwilą celów.
      Po tym wprowadzeniu przejdę do omawiania mapy, możliwych ich implementacji i reprezentacji,
      to samo zrobię z lokalizacją.
      Następnie omówię kilka zagadnień nawigacji, które pozwolą zrozumieć działalność poszczególnych jej elementów
      tak jak drzewo transformacji czy SLAM.
      Następnie opowiem krótko o systemie Navigation Stack i na koniec przedstawię kwestię planowania ścieżki.
      Potem opiszę zadania do wykonania w mojej pracy, co udało mi się osiągnąć i co zostało do skończenia.

## Nawigacja      
### Robot Velma
   1. Są to połączone w celu zobrazowania człowieka dwa manipulatory LWR Kuki o siedmiu stopniach swobody każdy, wyposażone w dodatkowe czujniki. 
    W celach prac laboratoryjnych został stworzony symulator, gdyż testowanie nowych rozwiązań od razu na robocie jest zbyt niebezpieczne zarówno dla prowadzących badania jak i samego sprzętu.
    Niedawno wersja symulacyjna została połączona z symulatorem drugiego robota, tym razem mobilnego, który dzięki wyposażeniu w koła szwedzkie może poruszać się w dowolnym kierunku bez zmiany orientacji.
    
   3. Oprogramowanie części mobilnej robota składa się z dwóch modułów. 
    Pierwszym modułem jest model symulacyjny, 
    w którym zawarte zostały część wizualna, inercja oraz kolizje robota.
        Częstą praktyką jest oddzielenie wizualnej od kolizji, gdzyż skomplikowane kształty 
    jak przykładowo koła szwedzkie doczepione do bazy zwiększyłyby w dużym stopniu nakład obliczeń podczas sprawdzania czy obiekt zderzył się z innym, nie wnosząc zbyt wiele dodatkowych informacji.
    Z tego powodu wspomniane koła szwedzkie mają symulowany system kolizji jako sfera.
        Drugim modułem jest moduł sterownika, który odpowiada za wykorzystanie informacji z pierwszego modułu oraz podane na wejście pożądane przez nas prędkości i obliczenie odpowiednich sił działających w symulacji na bazę, by zachowywała się w przybliżeniu zgodnie z rzeczywistością.
        
   4. Opisana część mobilna robota została zaimplementowana Z wykorzystaniem systemu ROS (Robot Operating System) oraz symulatora Gazebo. 
    ROS jest to system umożliwiający tworzenie niezależnych procesów komunikujących się między innymi dzięki jednokierunkowym tzw. tematom oraz serwisom, które zapewniają odpowiedź zwrotną.
     Dzięki temu można łatwo implementować i dołączać nowe funkcjonalności, czy wymieniać moduły które mają tę samą funkcjonalność ale inną wewnętrzną implementację.
    ROS wspiera pisanie w wielu językach, w tym Pythonie (niestety obecnie tylko 2, ROS Noetic ponoć ma zapewnić wsparcie dla pythona 3) czy C++. 
    Gazebo jest rozbudowanym symulatorem oferującym symulację obiektów w trójwymiarze, dodatkowo umożliwia proste dołączanie pluginów jako biblioteki dynamiczne C++.
    W ten właśnie sposób został zaimplementowany sterownik bazy mobilnej.
    
### Mapa
   1. Mapa jest modelem świata stworzonym na podstawie danych z czujników i przechowywanym w pamięci robota. 
    Można przyjąć, że początkowa pozycja robota jest w (0, 0) kartezjańskiego układu współrzędnych i chcąc dostać się do  dowolnego innego punktu będziemy korzystać z odometrii,
    o której powiem więcej w następnym punkcie, jednak to podejście powoduje akumulację błędu lokalizacji.
    Dlatego budowa mapy jest potrzebna, aby korzystając z punktów odniesienia,
    których położenie względem początku układu współrzędnych jest znane, minimalizować ten błąd.
            (W bibliografii RoboticsVisionAndControl) 
    \ dodaj wzó© z ^ str 161 - localizing with a Mapy
   
2. Przy budowie mapy potrzebne jest początkowe założenie, że znana jest dokładna pozycja robota. 
    Przykładowo przyjmuje się, że jest to pozycja (x:0, y:0, theta:0) w układzie odniesienia świata.
    Dzięki temu możliwe jest uzyskanie położenia punktów charakterystycznych otoczenia, które służą do stworzenia mapy. Ich położenie jednak jest już obarczone błędem pomiarowym.
    Istnieje wiele możliwych rozwiązań problemu budowania mapy, jednym przykładem jest zbiór punktów zebranych przez skaner laserowy, lub chmura w trzech wymiarach zebrana przez kamerę Kinect.
    Możliwe jest też wykorzystanie specjalnie oznaczonych punktów, które są identyfikowane przez robota i mają przypisaną zmierzoną pozycję, aby później ułatwić zadanie lokalizacji. 
3. dodaj zdjęcia mapy lidara i octomapy

4. Mapa zbudowana przy pomocy czujnika laserowego pozwala na względnie szybkie tworzenie obrazów pomieszczeń o płaskim podłożu, lecz nie zawiera informacji co się znajduje powyżej lub poniżej.
   Innym sposobem jest wykorzystanie chmury punktów generowanej przez kamerę Kinect. Pozwala na budowę trójwymiarowych obrazów pomieszczeń, pochłania to jednak więcej mocy obliczeniowej. 
    Jeżeli mapa ma zostać wykorzystana tylko do celów lokalizacji, to budowa mapy przy wykorzystaniu skanera często w zupełności wystarczy, jednak w przypadku mojej pracy jest to zbyt mało.

### Mapa kosztów
1. Zwykła mapa zawiera jedynie informacje o tym, czy w danym miejscu znajduje się jakaś przeszkoda.
Mapa kosztów każdemu punktowi na mapie przypisuje pewną wartość, co jest potrzebne w algorytmach nawigacji do obrania optymalnej ścieżki. 
Przedstawię tu dwuwymiarowe mapy kosztów, gdyż takie właśnie wykorzystam do zaimplementowania modułu nawigacji. 
Spośród wielu rodzajów można wymienić mapy binarne, gdzie wartość 1 to przeszkoda, a 0 to przestrzeń po której robot może się swobodnie poruszać, istnieją też mapy o większym stopniowaniu przestrzeni.   
Powszechnie niska wartość oznacza, że pozycja na mapie jest dostępna, wysoka oznacza przeszkodę, przykładowo ścianę. 
Dodatkowo dzięki wartościom pomiędzy najniższą a najwyższą można dodawać miejsca, które chcemy aby robot unikał, chociażby pobliże ścian może powodować problemy z uderzaniem manipulatorów o przeszkody lub blokowanie systemów nawigacji, które mają źle zaimplementowany algorytm odblokowania robota z zakleszczeń.
                
2. (dodaj zdjęcie map kosztów i zwykłej mapy)
 
3. Kolejnym rozwiązaniem jest wykorzystanie wielowarstwowych map dwuwymiarowych.
    Pozwalają one na wykrywanie przeszkód na wielu poziomach, dodatkowo można każdej warstwie przypisać inną cechę, przykładowo jedna mapa kosztów może odnosić się do tego czy powierzchnia jest śliska, czy może w danym miejscu znajduje się blat stołu. 
                (wstaw obrazek z Layered Costmaps for context-sensitive navigation)
                
### Lokalizacja

1. Zanim przedstawię bliżej problem lokalizacji omówię koncept operatorów przekształcenia jednorodnego oraz drzewa transformacji.
Operator przekształcenia jednorodnego jest macierzą przedstawiającą operację, którą należy wykonać aby przejść między dwoma punktami które dane przekształcenie opisuje.
W przypadkku punktów w trzech wymiarach ma ona wymiary 4x4. Można wyróżnić w niej macierz obrotu 3x3 oraz wektor translacji 3x1.
                            (WSTAW slajd anro-kin.pdf nr 34?)
2. W robotyce pozycję danego elementu przechowuje się wykorzystując układ współrzędnych, znając jego położenie oraz orientację można stwierdzić w jakim stanie znajduje się element, do którego dany układ współrzędnych jest przypisany.
W przypadku połączenia ze sobą dwóch elementów można ich relację przedstawić z pomocą parametrów Denavita-Hartenberga (obrót oraz przesunięcie względem osi x oraz z odpowiednich układów). 
Korzystając z tych czterech parametrów i macierzy transformacji jednorodnej można w przypadku robota szeregowego w prosty sposób obliczać pozycje kolejnych stawów przez mnożenie przez kolejne macierze wygenerowane z odpowiednich parametrów D-H.
                            (wstaw wzór 87 z anro-kin.pdf str 44)
W ten sposób powstaje drzewo transformacji. Ma ono swój początek w pierwszym wybranym układzie względem którego chcemy obliczyć pozycję elementu.
W przypadku mojej pracy jest to układ świata.
Drzewo transformacji składa się z układów współrzędnych jako węzłów oraz parametrów D-H opisujących ich relację. 
Drzewo robota szeregowego może mieć tylko jeden korzeń, a każdy węzeł tylko jednego rodzica, za to wiele dzieci.

3. W przypadku lokalizacji poszukiwana jest pozycja układu współrzędnych pierwszego węzła opisującego robota w układzie świata. 
Od niego potem odchodzą kolejne układy opisujące dalsze części robota, które jednak nie wnoszą żadnych użytecznych informacji do zadania lokalizacji.
W systemach nawigacji częstym widokiem jest przedstawienie początku drzewa jako sekwencja świat-map-odom w tej właśnie kolejności. 
Świat to globalny początkowy układ, mapa to początkowy układ współrzędnych względem którego jest zbudowana mapa, natomiast odometria to układ początku obliczania odometrii robota.
Znając przekształcenia świat-map, map-odom i odom-robot jesteśmy w stanie obliczyć położenie bazy mobilnej w układzie świata.

4. Tu przechodzimy do pierwszej metody lokalizacji którą omówię, odometrii.
Jest to metoda nawigacji zliczeniowej polegająca na ustaleniu pozycji robota na postawie jego oszacowanej prędkości, kierunku i czasu ruchu względem początkowego układu, o którym wspomniałem wcześniej.
                (Robotics Vision and Control)
W przypadku wielu robotów mobilnych obliczenie przesunięcia w ten sposób jest względnie prostym zadaniem, jednak wnosi do obliczeń wiele błędów zarówno systematycznych (niedokładnie zmierzony/obliczony obwód koła) jak i losowych (poślizgi). 
Stowosane są metody minimalizacji takich błędów, lecz nie da się ich pozbyć do końca, więc im dłużej dany robot będzie poruszał się po środowisku korzystając tylko i wyłącznie z odometrii, tym większy będzie błąd lokalizacji.
                        (Wstaw obrazek Robotics Vision and Control 6.4/159)
5. Kolejna metoda lokalizacji wykorzystuje punkty o znanym położeniu względem początkowego układu, czyli pewien rodzaj mapy.
Wiedząc który punkt robot bierze pod uwagę podczas lokalizacji i znając pozycję tego punktu względem robota można obliczyć pozycję bazy względem początkowego układu współrzędnych.
Ta metoda również generuje błędy lokalizacji, gdyż położenie punktów jest obarczone błędem zarówno jak zmierzenie ich pozycji względem robota, lecz ten błąd nie będzie rósł tak jak w przypadku odometrii.

6. W przypadku lokalizacji z wykorzystaniem czujników laserowych (LiDAR) problem lokalizacji jest podobny do poprzedniego, lecz w tym przypadku nie ma informacji o tym który konkretnie punkt na mapie jest aktualnie rozpatrywany, należy go dopasować razem ze wszystkimi innymi.
Algorytm ICP (iterative closed point) pozwala na dopasowanie aktualnego skanu z robota do stworzonej uprzednio mapy i zwraca najlepiej pasujące położenie i orientację w danej przestrzeni.
Metoda ta pozwala na lokalizację w przestrzeni bez umieszczania specjalnych znaczników, które robot byłby w stanie rozpoznać, lecz kosztem dużego nakładu obliczeniowego i bez gwarancji zwrócenia poprawnej pozycji jeżeli chmura punktów z czujnika lub zapisana w mapie nie zostaną odpowiednio przetworzone.
                    (https://www.ncbi.nlm.nih.gov/pmc/articles/PMC5949039/)
Lokalizacja z użyciem znaczników lub algorytmu ICP zwraca zwykle mniejsze błędy lokalizacji niż odometria, lecz są przypadki gdzie odometria ma również swoje zastosowania. 
W przypadku gdy pomieszczenie jest długim korytarzem lub innym pomieszczeniem gdzie algorytm nie jest w stanie znaleźć punktów szczególnych ostatnia metoda nie zwróci poprawnej pozycji.
Gdy dodatkowo nie mamy w pomieszczeniu żadnych rozpoznawalnych znaczników, druga metoda również niewiele zdziała.
w tym przypadku można skorzystać z odometrii, gdyż nie zależy ona od świata zewnętrznego, a jedynie stanu wewnętrznego robota.

### SLAM

1. Kolejnym krokiem jest połączenie dwóch powyższych działów, lokalizacji i budowania mapy, jest wykorzystanie ich jednocześnie. 
Podczas tworzenia mapy środowiska praktycznie zawsze w celu zobrazowania jego całości niezbędne jest poruszenie robotem, a gdy ten się poruszy, trzeba wiedzieć gdzie znajdował się tworząc dany element mapy, aby dołączyć ją do całości w odpowiednim miejscu.
SLAM (Simultaneous Localization and Mapping) jest rozwiązaniem tego problemu.
Dzięki temu możliwe jest nawet tworzenie map z wykorzystaniem wielu robotów na raz.

###   Planowanie Ścieżki

1. Przechodząc do ostatniego dużego punktu tej części mojej prezentacji warto wspomnieć że szukanie optymalnej ścieżki ma zastosowania również poza robotyką, przykładowo w grach komputerowych. 
Jeden z artykułów z którego korzystam w mojej pracy nosi tytuł  'A Comprehensive Study on Pathfinding Techniques for Robotics and Video Games'.
                        (A Comprehensive Study on Pathfinding Techniques for Robotics and Video Games-14)
2. Problem planowania ścieżki można rozdzielić na dwie sekcje. 
Pierwszą z nich jest sposób reprezentacji mapy w celu wykonania algorytmu poszukiwania bez zbędnych nakładów obliczeniowych.
Szkieletyzacja tworzy graf topologiczny zobrazowanego środowiska, po którym robot się przemieszcza, dekompozycja komórkowa dzieli mapę na komórki zawierające przestrzeń zdatną do poruszania. Mogą to być przykładowo wielokąty wypukłe, często wykorzystywane są kwadraty i sześciokąty.
                        (reprezentacja grafowa i komórkowa mapy)
3. Po otrzymaniu pożądanej reprezentacji środowiska należy dobrać odpowiedni algorymt wyszukiwania ścieżki.
Powszechne w wykorzystaniu są algorytm A*, genetyczny, wykorzystanie sztucznych pól potencjału, które generują wirtualne siły przyciągające robota do celu i odpychające od przeszkód.
W przypadku dekompozycji mapy reprezentacja sześciokątna znacznie zwiększyła wydajnośc niektórych algorytmów, 
4. W celu poprawnego wygenerowania ścieżki należy zdefiniować dokładnie skąd dokąd ścieżka ma prowadzić.
Punkt docelowy jest raczej oczywisty, użytkownik algorytmu sam definiuje końcowy punkt ścieżki, natomiast za punkt początkowy zwykle uznaje się aktualną pozycję bazowego układu współrzędnych robota, o którym była mowa w punkcie o lokalizacji. 
Po wyznaczeniu ścieżki może się jednak okazać, że na zakręcie jest ona zbyt blisko ściany i robot próbując ją wykonać uderzy w nią.
W celu uniknięcia tej katastrofy wykorzystuje się wspomniane wcześniej mapy kosztów, zwiększając wartości dookoła przeszkód aby ścieżka była generowana dalej od nich, oraz ślad robota (ang. footprint) będący rzutem pionowym postaci robota na podłoże, dzięki któremu wyznaczanie ścieżki w pobliżu przeszkód będzie mniej prawdopodobne.
                    (??????????????????????????????????????????????)

### ROS Navigation Stack

1. Po omówieniu głównych elementów wchodzących w skład problemu nawigacji możemy przejść do sposobu jej implementacji za pomocą systemu ROS.
ROS Navigation Stack przyjmuje popularne podejście podziału algorytmu na dwie części: globalną i lokalną.
Globalna ścieżka jest tworzona na podstawie mapy całego środowiska załadowanej do pamięci, ponieważ bardziej skomplikowane algorytmy mogłyby zająć zbyt wiele czasu, w tym przypadku wykorzystuje się proste algorytmy, które jednak często nie biorą pod uwagę stanu środowiska zewnętrznego. (????????????????????)
Reakcją na to jest planer lokalny, który dzięki przetwarzaniu jedynie przestrzeni w sąsiedztwie robota jest w stanie wykorzystać bardziej zaawansowane algorytmy do optymalizacji ścieżki i reakcji na bodźce zewnętrzne. 
Dodatkowo do systemu nawigacji powinien zostać zaimplementowany system odratowania (ang. recovery). 
Ma on pozwolić bazie mobilnej na wyrwanie się z sytuacji w której plan lokalny, który steruje ruchami robota, nie jest w stanie znaleźć wyjścia z sytuacji.
Takim zachowaniem odratowania może być obrót w miejscu dopóki ruch nie stanie się możliwy, czy cofnięcie i przesunięcie losowo w innym kierunku (z uwzględnieniem przeszkód)
                        (obrazek navigation stack)

## Stan pracy
### Sformułowanie zadań
1. Na początku pracy z tematem musiałem sformułować problem inżynierski, który przedstawię następująco:
moim zadaniem jest implementacja mechanizmu nawigacji do symulacji mobilnej bazy wielokierunkowej spełniająca następujące wymagania:
    1. zapewnienie możliwości budowania mapy na płaskim podłożu w zamkniętych pomieszczeniach
    2. zapewnienie mechanizmu lokalizacji robota w stworzonym uprzednio środowisku i mechanizmu jednoczesnej lokalizacji i budowy mapy (SLAM)
    3. zapewnienie mechanizmu stworzenia i wykonania ścieżki między dowolnymi dwoma punktami stworzonego środowiska i pokonanie jej w sposób bezpieczny dla robota
2. Zadanie wykonuję przy wykorzystaniu opisanych już symulatora Gazebo i systemu ROS, pisząc kod w językach Python i C++
    
### Dotychczasowe wyniki pracy

1. 
    1. Gdy zacząłem prace nad implementacją lokalizacji okazało się, że drzewo transformacji jest zbudowane niepoprawnie. 
Pierwszym punktem było więc naprawienie go tak, aby miało konfigurację omówioną już wcześniej w trakcie prezentacji, co pozwoliło uniknąć tworzenie nieporządanych zapętleń w drzewie.
Dzięki temu możliwa stała się lokalizacja, zaczęła działać odometria udostępniana przez sterownik bazy.
    2. Zaimplementowałem budowę mapy 2D z pomocą dwóch czujników laserowych obecnych na bazie mobilnej oraz mapy 3D z pomocą kamery Kinect.
    3. Zaimplementowałem algorytm lokalizacji z pomocą czujników laserowych

### Zadania do wykonania
1. Jak wspomniałem na początku prezentacji, w przypadku robota Velmy na bazie mobilnej ważne jest wykrycie przeszkód znajdujących się powyżej poziomu podestu.
Dlatego planuję wykorzystać zarówno mapę z czujników laserowych jak i kamery Kinect do stworzenia wielowarstwowej mapy kosztów. 
Zbadam implementację dwóch warstw o różnych parametrach oraz jednej stworzonej przez połączenie wyżej wspomnianych.
                        (mapa kosztów z lidarów i z kinecta)
2. Następnie sprawdzę obecny system planowania ścieżki i poprawię jego wariant lokalny, który w tym momencie traktuje bazę robota jako różnicową, nie wykorzystując jego możliwości w poruszaniu się w dowolnym kierunku.
                        (baza różnicowa i holonomiczna)
3. Na koniec przeprowadzę testy z różnymi metodami lokalizacji, budowy mapy i planowania.
W przypadku budowy mapy będę brał pod uwagę czas, poziom skomplikowania procesu jej budowy oraz ilość informacji którą przekazuje.
W przypadkku lokalizacji będę brał pod uwagę błąd względem idealnej pozycji uzyskanej z symulatora, sprawdzę odporność na słabości zarówno odometrii jak i lokalizacji z pomocą algorytmu ICP (długi korytarz, labirynt bez punktów szczególnych).
W przypadku planowania ścieżki porównam obecny silnik jej generowania z innymi dostępnymi, głównymi czynnikami oceny będzie czas wykonania ścieżki, reakcja na przeszkody dynamiczne, wpadanie w ślepe zaułki oraz wykorzystanie wszystkich stopni swobody bazy mobilnej.

## Zakończenie

1. Dziękuję wszystkim za uwagę.