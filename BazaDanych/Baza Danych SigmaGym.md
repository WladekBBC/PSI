# Baza Danych SigmaGym
### Klient
-- Tabela przechowuje dane osobowe i kontaktowe klientów siłowni.
-- Używana jako punkt odniesienia w wielu innych tabelach (np. karta, faktura, feedback).
```sql
-- Klient
CREATE TABLE Klient (
    klientID INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    imie VARCHAR(100),
    nazwisko VARCHAR(100),
    email VARCHAR(100),
    telefon VARCHAR(20),
    adres VARCHAR(255),
    miasto VARCHAR(100),
    kodPocztowy VARCHAR(10),
    kraj VARCHAR(100),
    dataRejestracji DATE
) ENGINE=InnoDB;
```
### Karta SigmaGym
-- Powiązana z klientem, reprezentuje członkostwo w siłowni.
-- Zawiera informacje o typie karty, jej ważności oraz statusie.
```sql
-- Karta SigmaGym
CREATE TABLE MembershipCard (
    numerKarty VARCHAR(50) PRIMARY KEY,
    klientID INT UNSIGNED UNIQUE,
    typKarty VARCHAR(50),
    dataWaznosci DATE,
    status VARCHAR(50) DEFAULT 'aktywny',
    FOREIGN KEY (klientID) REFERENCES Klient(klientID)
) ENGINE=InnoDB;
```
### Rejestracja wejść
-- Rejestruje każde wejście klienta na siłownię.
-- Służy do analizy aktywności i może wspierać system lojalnościowy.
```sql
-- Rejestracja wejscia
CREATE TABLE EntranceLog (
    logID INT AUTO_INCREMENT PRIMARY KEY,
    numerKarty VARCHAR(50),
    dataWejscia DATETIME,
    FOREIGN KEY (numerKarty) REFERENCES MembershipCard(numerKarty)
) ENGINE=InnoDB;  
```
### "Konto lojalnościowe"
-- Zlicza punkty i wizyty klienta, wspiera programy nagród i zniżek.
```sql
-- "Konto lajalnościowe"
CREATE TABLE LoyaltyAccount (
    klientID INT UNSIGNED PRIMARY KEY,
    punkty INT DEFAULT 0,
    liczbaWizyt INT DEFAULT 0,
    FOREIGN KEY (klientID) REFERENCES Klient(klientID)
) ENGINE=InnoDB;
```
### Feedback
-- Klienci mogą zostawić ocenę i komentarz nt. usług siłowni.
-- Może służyć do poprawy jakości obsługi i infrastruktury.

```sql
-- Feedback
CREATE TABLE Feedback (
    feedbackID INT AUTO_INCREMENT PRIMARY KEY,
    klientID INT UNSIGNED,
    ocena INT CHECK (ocena BETWEEN 1 AND 10),
    komentarz TEXT,
    data DATE,
    FOREIGN KEY (klientID) REFERENCES Klient(klientID)
) ENGINE=InnoDB;
```
### Operator płatności
-- Przechowuje informacje o zewnętrznych dostawcach usług płatniczych.
```sql
-- Operator płatnosci 
CREATE TABLE PaymentOperator (
    operatorID INT AUTO_INCREMENT PRIMARY KEY,
    nazwa VARCHAR(100)
) ENGINE=InnoDB;
```
### Faktura
-- Przechowuje informacje o fakturach wystawianych klientom.
```sql
-- Faktura
CREATE TABLE Faktura (
    fakturaID INT AUTO_INCREMENT PRIMARY KEY,
    klientID INT UNSIGNED,
    dataWystawienia DATE,
    dataPlatnosci DATE,
    kwota DECIMAL(10,2),
    status VARCHAR(50) DEFAULT 'oczekuje',
    nip VARCHAR(20),
    uwagi TEXT,
    FOREIGN KEY (klientID) REFERENCES Klient(klientID)
) ENGINE=InnoDB;
```
### Płatność
-- Powiązana z fakturą, karta klienta oraz operatorem.
-- Służy do ewidencji wpłat.
```sql
-- Płatność
CREATE TABLE Payment (
    paymentID INT AUTO_INCREMENT PRIMARY KEY,
    fakturaID INT,
    numerKarty VARCHAR(50),
    kwota DECIMAL(10,2),
    metodaPlatnosci VARCHAR(50),
    dataPlatnosci DATE,
    operatorID INT,
    FOREIGN KEY (fakturaID) REFERENCES Faktura(fakturaID),
    FOREIGN KEY (numerKarty) REFERENCES MembershipCard(numerKarty),
    FOREIGN KEY (operatorID) REFERENCES PaymentOperator(operatorID)
) ENGINE=InnoDB;
```
### Recepcjonista
-- Tabela przechowuje dane pracowników recepcji.
-- Recepcjoniści mogą być przypisani do konkretnych operatorów płatności.
```sql
-- Recepcjonista
CREATE TABLE Recepcjonista (
    pracownikID INT AUTO_INCREMENT PRIMARY KEY,
    imie VARCHAR(100),
    nazwisko VARCHAR(100),
    email VARCHAR(100)
) ENGINE=InnoDB;
```
### Bank
-- Lista banków współpracujących z operatorami płatności.
```sql
-- Bank
CREATE TABLE Bank (
    bankID INT AUTO_INCREMENT PRIMARY KEY,
    nazwa VARCHAR(100)
) ENGINE=InnoDB;
```
### Recepcjonista-OperatorPłatności
-- Tabela pośrednia określająca, który recepcjonista obsługuje którego operatora płatności.
```sql
-- Recepcjonista_PaymentOperator
CREATE TABLE Recepcjonista_PaymentOperator (
    recepcjonistaID INT,
    operatorID INT,
    PRIMARY KEY (recepcjonistaID, operatorID),
    FOREIGN KEY (recepcjonistaID) REFERENCES Recepcjonista(pracownikID),
    FOREIGN KEY (operatorID) REFERENCES PaymentOperator(operatorID)
) ENGINE=InnoDB;
```
### Autoryzacja płatności
-- Relacja między operatorami płatności a bankami, z którymi mają autoryzację.
```sql
-- Autoryzacja płatności
CREATE TABLE PaymentAuthorization (
    operatorID INT,
    bankID INT,
    PRIMARY KEY (operatorID, bankID),
    FOREIGN KEY (operatorID) REFERENCES PaymentOperator(operatorID),
    FOREIGN KEY (bankID) REFERENCES Bank(bankID)
) ENGINE=InnoDB;
```
### Sprzęt
-- Lista sprzętów dostępnych w siłowni. Zawiera typ, status i datę ostatniego serwisu.
```sql
-- Sprzęt
CREATE TABLE Equipment (
    equipmentID INT AUTO_INCREMENT PRIMARY KEY,
    typ VARCHAR(100),
    status VARCHAR(50),
    ostatniSerwis DATE
) ENGINE=InnoDB;
```
### Zgłoszenie usterki
-- Rejestruje zgłoszenia usterek danego sprzętu.
```sql
-- Zgłoszenie usterki
CREATE TABLE EquipmentReport (
    reportID INT AUTO_INCREMENT PRIMARY KEY,
    equipmentID INT,
    opis TEXT,
    status VARCHAR(50),
    dataZgloszenia DATE,
    FOREIGN KEY (equipmentID) REFERENCES Equipment(equipmentID)
) ENGINE=InnoDB;
```
### Administrator
-- Tabela przechowuje administratorów zarządzających sprzętem i infrastrukturą siłowni.
```sql
-- Administrator
CREATE TABLE Administrator (
    adminID INT UNSIGNED PRIMARY KEY,
    imie VARCHAR(100),
    nazwisko VARCHAR(100)
) ENGINE=InnoDB;
```
### Relacja Administrator-Sprzęt
-- Określa, który administrator odpowiada za który sprzęt.
```sql
-- Relacja Administrator_Sprzęt
CREATE TABLE Administrator_Equipment (
    adminID INT,
    equipmentID INT,
    PRIMARY KEY (adminID, equipmentID),
    FOREIGN KEY (adminID) REFERENCES Administrator(adminID),
    FOREIGN KEY (equipmentID) REFERENCES Equipment(equipmentID)
) ENGINE=InnoDB;
```
### Dostawca
-- Dane kontaktowe dostawców sprzętu lub usług dla siłowni.
```sql
-- Dostawca
CREATE TABLE Supplier (
    supplierID INT AUTO_INCREMENT PRIMARY KEY,
    nazwa VARCHAR(100),
    kontakt VARCHAR(255)
) ENGINE=InnoDB;
```
### Zapis zgłoszenia usterki
-- Powiązanie zgłoszenia sprzętu z odpowiedzialnym dostawcą.
```sql
-- Zgłoszenie usterki do dostawcy
CREATE TABLE EquipmentSupplierReport (
    reportID INT,
    supplierID INT,
    PRIMARY KEY (reportID, supplierID),
    FOREIGN KEY (reportID) REFERENCES EquipmentReport(reportID),
    FOREIGN KEY (supplierID) REFERENCES Supplier(supplierID)
) ENGINE=InnoDB;
```
### Firma sprzątająca
-- Dane firm sprzątających współpracujących z siłownią, wraz z harmonogramem.
```sql
-- Firma sprzątająca
CREATE TABLE CleaningService (
    serviceID INT AUTO_INCREMENT PRIMARY KEY,
    nazwaFirmy VARCHAR(100),
    harmonogram VARCHAR(255)
) ENGINE=InnoDB;
```
### Relacja FirmaSprząt-Sprzęt
-- Informuje, która firma sprząta który sprzęt lub strefę.
```sql
-- Relacja FirmaSprzątająca_Sprzęt
CREATE TABLE CleaningService_Equipment (
    serviceID INT,
    equipmentID INT,
    PRIMARY KEY (serviceID, equipmentID),
    FOREIGN KEY (serviceID) REFERENCES CleaningService(serviceID),
    FOREIGN KEY (equipmentID) REFERENCES Equipment(equipmentID)
) ENGINE=InnoDB;
```
### Formularz rejestracji
-- Formularze zgłoszeniowe klientów – powiązane z klientem i jego kartą.
```sql
-- Formularz rejestracji
CREATE TABLE RegistrationForm (
    formID INT AUTO_INCREMENT PRIMARY KEY,
    klientID INT UNSIGNED,
    numerKarty VARCHAR(50),
    dataZlozenia DATE,
    FOREIGN KEY (klientID) REFERENCES Klient(klientID),
    FOREIGN KEY (numerKarty) REFERENCES MembershipCard(numerKarty)
) ENGINE=InnoDB;
```