# Baza danych e-handel
Relacyjna baza danych służy do obsługi portalu internetowego do e-handlu.

Projekt został stworzony korzystając z oprgramowania `SQL Developer Data Modeler 21.4.1`.

Aby przetestować bazę danych należy na serwererze MySQL wykonać dany [SRYPT](skryptDDL.ddl), który stworzy [tabelę](Tabele), relację między nimi, sekwencję oraz wyzwalacz.


## Założenia funkcjonalne:
- Użytkownik musi mieć przypisany jeden z kodów pocztowych znajdujących się w tabeli `KodPocztowy`.
- Każdy użytkownik może zarówno tworzyć aukcje, kupować przedmioty z aukcji oraz licytować na aukcjach.
- Użytkownik może sprzedawać wiele produktów, kupić wiele produktów, licytować wiele produktów, może też tworzyć wiele ofert kupna dla pojedynczej licytacji.
- Aby kupić produkt, należy pierwszy utworzyć zamówienia, a następnie dodawać do niego kupione przedmioty.
- W skład pojedynczego zamówienia może wchodzić wiele produktów, pochodzących od wielu sprzedawców (funkcja koszyka).
- Aukcja musi mieć przypisaną kategorię znajdującą się w tabeli `Kategoria`.
- Aby dodać przypisać aukcji parametr, wraz z jego wartość, nazwa parametru musi występować w tabeli `Parametr`.
- Każda aukcja może mieć przypisanych wiele parametrów.
- Każda kategoria może mieć przypisanych wiele parametrów.
- Parametry w poszczególnych kategoriach mogą mieć te same nazwy.
- Dodawanie rekordów do tabeli `KupionePrzedmioty` automatycznie zmniejsza ilość sztuk dla odpowiedniej aukcji o ilość zakupionych sztuk (trigger).
- Aby przypisać klucz główny tabeli należy skorzystać skorzystać z odpowiedniej z wcześniej zdefiniowanych sekwencji.

## Schemat logiczny
![logical](images/logical.png)

## Schemat relacyjny
![Relational](images/relational.png)

## Skrypty DDL SQL

## Tabele

### Aukcja:
```sql
CREATE TABLE aukcja (
    aid         NUMBER NOT NULL,
    uzid        NUMBER NOT NULL,
    anazwa      VARCHAR2(100),
    acena       NUMBER,
    ailoscsztuk NUMBER,
    alicytacja  CHAR(1) NOT NULL,
    katnazwa    VARCHAR2(20) NOT NULL
);

ALTER TABLE aukcja ADD CONSTRAINT aukcja_pk PRIMARY KEY ( aid );
```

### Kategoria:
```sql
CREATE TABLE kategoria (
    katnazwa VARCHAR2(20) NOT NULL,
    katinfo  VARCHAR2(100) NOT NULL
);

ALTER TABLE kategoria ADD CONSTRAINT kategoria_pk PRIMARY KEY ( katnazwa );
```

### Kod Pocztowy:
```sql
CREATE TABLE kodpocztowy (
    kodpocztowy VARCHAR2(6) NOT NULL,
    kodmiasto   VARCHAR2(20)
);
ALTER TABLE kodpocztowy ADD CONSTRAINT kodpocztowy_pk PRIMARY KEY ( kodpocztowy );

CREATE TABLE kupioneprzedmioty (
    zid         NUMBER NOT NULL,
    aid         NUMBER NOT NULL,
    kiloscsztuk NUMBER NOT NULL
);
```

### Oferta:
```sql
CREATE TABLE oferta (
    ooferowanacena NUMBER,
    oczaszlozenia  DATE,
    aid            NUMBER NOT NULL,
    uzid           NUMBER NOT NULL
);
```

### Parametr:
```sql
CREATE TABLE parametr (
    parid    NUMBER NOT NULL,
    parnazwa VARCHAR2(20) NOT NULL,
    katnazwa VARCHAR2(20) NOT NULL
);

ALTER TABLE parametr ADD CONSTRAINT parametr_pk PRIMARY KEY ( parid );
```

### Użytkownik:
```sql
CREATE TABLE uzytkownik (
    uzid         NUMBER NOT NULL,
    uzimie       VARCHAR2(20),
    uznazwisko   VARCHAR2(20),
    uzulica      VARCHAR2(20),
    uznrdomu     VARCHAR2(8),
    uznrtelefonu NUMBER,
    kodpocztowy  VARCHAR2(6) NOT NULL
);

ALTER TABLE uzytkownik ADD CONSTRAINT uzytkownik_pk PRIMARY KEY ( uzid );
```

### Wartość Parametru:
```sql
CREATE TABLE wartoscparametru (
    wpwartosc VARCHAR2(20),
    aid       NUMBER NOT NULL,
    parid     NUMBER NOT NULL
);
```

### Zamówienia:
```sql
CREATE TABLE zamowienia (
    zid           NUMBER NOT NULL,
    uzid          NUMBER NOT NULL,
    zczaszlozenia DATE,
    zczasstworzenia DATE
);

ALTER TABLE zamowienia ADD CONSTRAINT zamowienia_pk PRIMARY KEY ( zid );
```

### Relacje:
```sql
ALTER TABLE aukcja
    ADD CONSTRAINT aukcja_kategoria_fk FOREIGN KEY ( katnazwa )
        REFERENCES kategoria ( katnazwa );

ALTER TABLE aukcja
    ADD CONSTRAINT aukcja_uzytkownik_fk FOREIGN KEY ( uzid )
        REFERENCES uzytkownik ( uzid );

ALTER TABLE kupioneprzedmioty
    ADD CONSTRAINT kupioneprzedmioty_aukcja_fk FOREIGN KEY ( aid )
        REFERENCES aukcja ( aid );

ALTER TABLE oferta
    ADD CONSTRAINT oferta_aukcja_fk FOREIGN KEY ( aid )
        REFERENCES aukcja ( aid );

ALTER TABLE oferta
    ADD CONSTRAINT oferta_uzytkownik_fk FOREIGN KEY ( uzid )
        REFERENCES uzytkownik ( uzid );

ALTER TABLE parametr
    ADD CONSTRAINT parametr_kategoria_fk FOREIGN KEY ( katnazwa )
        REFERENCES kategoria ( katnazwa );

ALTER TABLE uzytkownik
    ADD CONSTRAINT uzytkownik_kodpocztowy_fk FOREIGN KEY ( kodpocztowy )
        REFERENCES kodpocztowy ( kodpocztowy );

ALTER TABLE wartoscparametru
    ADD CONSTRAINT wartoscparametru_aukcja_fk FOREIGN KEY ( aid )
        REFERENCES aukcja ( aid );

ALTER TABLE wartoscparametru
    ADD CONSTRAINT wartoscparametru_parametr_fk FOREIGN KEY ( parid )
        REFERENCES parametr ( parid );

ALTER TABLE kupioneprzedmioty
    ADD CONSTRAINT zamowienia_kupione_fk FOREIGN KEY ( zid )
        REFERENCES zamowienia ( zid );

ALTER TABLE zamowienia
    ADD CONSTRAINT zamowienia_uzytkownik_fk FOREIGN KEY ( uzid )
        REFERENCES uzytkownik ( uzid );
```

### Sekwencje
Poniższe sekwencję służą do generowania kluczy głównych dla poszczególnych tabel.
```sql
CREATE SEQUENCE SUZID INCREMENT BY 1 MAXVALUE 9999999999999999999999999999 MINVALUE 1;

CREATE SEQUENCE SZID INCREMENT BY 1 MAXVALUE 9999999999999999999999999999 MINVALUE 1;

CREATE SEQUENCE SAID INCREMENT BY 1 MAXVALUE 9999999999999999999999999999 MINVALUE 1;

CREATE SEQUENCE SPARID INCREMENT BY 1 MAXVALUE 9999999999999999999999999999 MINVALUE 1;
```

### Wyzwalacze
Poniższy wyzwalacz automatycznie zmniejsza ilość sztuk dla odpowiedniej aukcji, gdy dodany zostanie rekord do tabeli ‘KupionePrzedmioty’.
```sql
CREATE OR REPLACE TRIGGER AKUALIZACJA_SZTUK 
AFTER INSERT ON KUPIONEPRZEDMIOTY 
REFERENCING OLD AS OLD NEW AS NEW
FOR EACH ROW
BEGIN
    update aukcja
    set ailoscsztuk = (select ailoscsztuk from aukcja where aid=:NEW.aid) - :NEW.kiloscsztuk
    where aid = :NEW.aid;
    
END;
/
```

## Przykładowe zapytania SQL:

<hr/>


**1. Najwyższa ofertę, czas i datę jej złożenia dla aukcji (licytacji) o nazwie ‘Opel licytacja’ wraz z nazwa i opisem kategori do której należy.**
```sql
select max(ooferowanacena), to_char(oczaszlozenia, 'dd-mm-yy hh24:mi'), anazwa, katnazwa, katinfo
from oferta 
natural join (
    select anazwa, katnazwa, katinfo
    from aukcja 
    natural join 
    kategoria
    where anazwa='Opel licytacja' 
) 
group by anazwa, katnazwa, katinfo, oczaszlozenia;
```

<hr/>


**2. Historia ofert składanych przez użytkownika Jan Kowalski posortowana wg. nazwy oferty oraz czasu złożenia.**
```sql
SELECT anazwa, ooferowanacena, czas  FROM
(
    select aid, ooferowanacena, to_char(oczaszlozenia, 'dd-mm-yy hh24:mi:ss') as czas
    from oferta
    natural join (
        select uzid, uzimie, uznazwisko
        from uzytkownik 
        where uzimie = 'Jan' and uznazwisko = 'Kowalski'
    )
)
natural join aukcja
order by anazwa, czas
```

<hr/>


**3. Imiona, nazwiska oraz pełne adresy użytkowników, którzy kupili w sumie przynajmniej 3 sztuki dowolnych przedmiotów po 01.01.2022.**
```sql
select uzimie, uznazwisko, kodmiasto, kodpocztowy, uzulica, uznrdomu from 
    (
    select *
    from uzytkownik
    inner join (
        select uzid, sum(sztuki_dla_zam) as sztuki_klinet
        from zamowienia
        natural join (
            select  sum(kiloscsztuk) as sztuki_dla_zam, zid
            from kupioneprzedmioty
            group by zid
        )
        where zczaszlozenia > TO_DATE('2021/01/01', 'yyyy/mm/dd')
        group by uzid
    )sztuki_suma
    on uzytkownik.uzid = sztuki_suma.uzid
    where sztuki_klinet >= 3
    )
natural join kodpocztowy;
```

<hr/>


**4. Identyfikatory użytkowników, którzy licytowali dowolny przedmiot z kategori zaczynającej sie na litere ‘S’.**
```sql
select DISTINCT uzid
from oferta
inner join (
    select aid, katnazwa, anazwa
    from aukcja
    where katnazwa LIKE 'S%'
)aukcje_S
on oferta.aid = aukcje_S.aid
```

<hr/>


**5. Nazwy aukcji oraz jej wszystkie wartości parametrów licytowanych przez największą liczbą osób.**
```sql
select an, p.parnazwa, wwp
from(
    select aid_t.aid as aid, a.anazwa as an, w.wpwartosc as wwp, w.parid as wp
    from(
        select max(count_uzid) as  max_count_uzid
        from(
            select count(uzid) as count_uzid, aid
            from(
                select uzid, aid
                from oferta
                group by aid, uzid
            )
            group by aid
        )
    )
    
    natural join(
        select count(uzid) as max_count_uzid, aid
        from(
            select uzid, aid
            from oferta
            group by aid, uzid
        )
        group by aid
    )aid_t
    
    left join wartoscparametru w
    on aid_t.aid = w.aid
    
    left join aukcja a
    on aid_t.aid = a.aid
)
join parametr p
on p.parid = wp
```

<hr/>




