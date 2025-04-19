# Baza Danych SigmaGym
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

```sql
-- Rejestracja wejscia
CREATE TABLE EntranceLog (
    logID INT AUTO_INCREMENT PRIMARY KEY,
    numerKarty VARCHAR(50),
    dataWejscia DATETIME,
    FOREIGN KEY (numerKarty) REFERENCES MembershipCard(numerKarty)
) ENGINE=InnoDB;  
```

```sql
-- "Konto lajalnościowe"
CREATE TABLE LoyaltyAccount (
    klientID INT UNSIGNED PRIMARY KEY,
    punkty INT DEFAULT 0,
    liczbaWizyt INT DEFAULT 0,
    FOREIGN KEY (klientID) REFERENCES Klient(klientID)
) ENGINE=InnoDB;
```

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

```sql
-- Operator płatnosci 
CREATE TABLE PaymentOperator (
    operatorID INT AUTO_INCREMENT PRIMARY KEY,
    nazwa VARCHAR(100)
) ENGINE=InnoDB;
```

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

```sql
-- Recepcjonista
CREATE TABLE Recepcjonista (
    pracownikID INT AUTO_INCREMENT PRIMARY KEY,
    imie VARCHAR(100),
    nazwisko VARCHAR(100),
    email VARCHAR(100)
) ENGINE=InnoDB;
```

```sql
-- Bank
CREATE TABLE Bank (
    bankID INT AUTO_INCREMENT PRIMARY KEY,
    nazwa VARCHAR(100)
) ENGINE=InnoDB;
```

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

```sql
-- Sprzęt
CREATE TABLE Equipment (
    equipmentID INT AUTO_INCREMENT PRIMARY KEY,
    typ VARCHAR(100),
    status VARCHAR(50),
    ostatniSerwis DATE
) ENGINE=InnoDB;
```

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

```sql
-- Administrator
CREATE TABLE Administrator (
    adminID INT UNSIGNED PRIMARY KEY,
    imie VARCHAR(100),
    nazwisko VARCHAR(100)
) ENGINE=InnoDB;
```

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

```sql
-- Dostawca
CREATE TABLE Supplier (
    supplierID INT AUTO_INCREMENT PRIMARY KEY,
    nazwa VARCHAR(100),
    kontakt VARCHAR(255)
) ENGINE=InnoDB;
```

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

```sql
-- Firma sprzątająca
CREATE TABLE CleaningService (
    serviceID INT AUTO_INCREMENT PRIMARY KEY,
    nazwaFirmy VARCHAR(100),
    harmonogram VARCHAR(255)
) ENGINE=InnoDB;
```
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