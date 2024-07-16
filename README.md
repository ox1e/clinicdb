

Тестовое задание для ГК АДЕПТ
# Диаграмма 
![Диаграмма базы данных](https://github.com/user-attachments/assets/b158d50d-ffb9-4ddc-ba52-05d602e2b3ee)

# Описание таблиц и связей

1. **Role**

- RoleID (INT, PK, AUTO_INCREMENT)
- RoleName (VARCHAR(50), NOT NULL)

2. **User**

- UserID (INT, PK, AUTO_INCREMENT)
- Username (VARCHAR(50), UNIQUE, NOT NULL)
- Password (VARCHAR(100), NOT NULL)
- RoleID (INT, FK, NOT NULL)
- **Связь с таблицей Role через RoleID**

3. **Doctor**

- DoctorID (INT, PK, AUTO_INCREMENT)
- FirstName (VARCHAR(50), NOT NULL)
- LastName (VARCHAR(50), NOT NULL)
- Specialization (VARCHAR(50))
- ContactInfo (VARCHAR(100))
- UserID (INT, FK)
- **Связь с таблицей User через UserID**

4. **Patient**

- PatientID (INT, PK, AUTO_INCREMENT)
- FirstName (VARCHAR(50), NOT NULL)
- LastName (VARCHAR(50), NOT NULL)
- ContactInfo (VARCHAR(100))
- UserID (INT, FK)
- **Связь с таблицей User через UserID**

5. **WorkSchedule**

- ScheduleID (INT, PK, AUTO_INCREMENT)
- DoctorID (INT, FK, NOT NULL)
- WorkDate (DATE, NOT NULL)
- StartTime (TIME, NOT NULL)
- EndTime (TIME, NOT NULL)
- **Связь с таблицей Doctor через DoctorID**

6. **TimeSlot**

- SlotID (INT, PK, AUTO_INCREMENT)
- ScheduleID (INT, FK, NOT NULL)
- StartTime (DATETIME, NOT NULL)
- EndTime (DATETIME, NOT NULL)
- Status (VARCHAR(20), NOT NULL) -- 'available', 'booked', 'missed'
- **Связь с таблицей WorkSchedule через ScheduleID**

7. **Appointment**

- AppointmentID (INT, PK, AUTO_INCREMENT)
- SlotID (INT, FK, NOT NULL)
- PatientID (INT, FK, NOT NULL)
- Status (VARCHAR(20), NOT NULL) -- 'scheduled', 'completed', 'missed'
- **Связь с таблицей TimeSlot через SlotID**
- **Связь с таблицей Patient через PatientID**

8. **MissedAppointment**

- MissedID (INT, PK, AUTO_INCREMENT)
- AppointmentID (INT, FK, NOT NULL)
- Compensation (VARCHAR(100), NOT NULL)
- **Связь с таблицей Appointment через AppointmentID**

# SQL

1. **Создание БД**
``` 
CREATE DATABASE IF NOT EXISTS ClinicDB;
USE ClinicDB;
```
2.  **Таблица Role**
``` 
CREATE TABLE Role (
    RoleID INT AUTO_INCREMENT,
    RoleName VARCHAR(50) NOT NULL,
    PRIMARY KEY (RoleID)
);
```
3.  **Таблица User**
``` 
CREATE TABLE User (
    UserID INT AUTO_INCREMENT,
    Username VARCHAR(50) UNIQUE NOT NULL,
    Password VARCHAR(100) NOT NULL,
    RoleID INT NOT NULL,
    PRIMARY KEY (UserID),
    FOREIGN KEY (RoleID) REFERENCES Role(RoleID)
);
```
4.  **Таблица Doctor**
``` 
CREATE TABLE Doctor (
    DoctorID INT AUTO_INCREMENT,
    FirstName VARCHAR(50) NOT NULL,
    LastName VARCHAR(50) NOT NULL,
    Specialization VARCHAR(50),
    ContactInfo VARCHAR(100),
    UserID INT,
    PRIMARY KEY (DoctorID),
    FOREIGN KEY (UserID) REFERENCES User(UserID)
);
```
5.  **Таблица Patient**
``` 
CREATE TABLE Patient (
    PatientID INT AUTO_INCREMENT,
    FirstName VARCHAR(50) NOT NULL,
    LastName VARCHAR(50) NOT NULL,
    ContactInfo VARCHAR(100),
    UserID INT,
    PRIMARY KEY (PatientID),
    FOREIGN KEY (UserID) REFERENCES User(UserID)
);
```
6.  **Таблица WorkSchedule**
``` 
CREATE TABLE WorkSchedule (
    ScheduleID INT AUTO_INCREMENT,
    DoctorID INT NOT NULL,
    WorkDate DATE NOT NULL,
    StartTime TIME NOT NULL,
    EndTime TIME NOT NULL,
    PRIMARY KEY (ScheduleID),
    FOREIGN KEY (DoctorID) REFERENCES Doctor(DoctorID)
);
```
7.  **Таблица TimeSlot**
``` 
CREATE TABLE TimeSlot (
    SlotID INT AUTO_INCREMENT,
    ScheduleID INT NOT NULL,
    StartTime DATETIME NOT NULL,
    EndTime DATETIME NOT NULL,
    Status VARCHAR(20) NOT NULL,  -- 'available', 'booked', 'missed'
    PRIMARY KEY (SlotID),
    FOREIGN KEY (ScheduleID) REFERENCES WorkSchedule(ScheduleID),
    CONSTRAINT UC_TimeSlot UNIQUE (ScheduleID, StartTime, EndTime)
);
```
8.  **Таблица Appointment**
``` 
CREATE TABLE Appointment (
    AppointmentID INT AUTO_INCREMENT,
    SlotID INT NOT NULL,
    PatientID INT NOT NULL,
    Status VARCHAR(20) NOT NULL,  -- 'scheduled', 'completed', 'missed'
    PRIMARY KEY (AppointmentID),
    FOREIGN KEY (SlotID) REFERENCES TimeSlot(SlotID),
    FOREIGN KEY (PatientID) REFERENCES Patient(PatientID)
);
```
9.  **Таблица MissedAppointment**
``` 
CREATE TABLE MissedAppointment (
    MissedID INT AUTO_INCREMENT,
    AppointmentID INT NOT NULL,
    Compensation VARCHAR(100) NOT NULL,
    PRIMARY KEY (MissedID),
    FOREIGN KEY (AppointmentID) REFERENCES Appointment(AppointmentID)
);
```


# Описание и дополнение к бизнес-процессу:

1. **Регистрация врачей и пациентов**
  -   **Описание**: Процесс регистрации новых врачей и пациентов в системе.
  -   **Детали**: Администратор вводит данные новых врачей и пациентов, такие как имя, контактная информация, специализация врача и т.д.
  -   **Таблицы**: `Doctor`, `Patient`

2.   **Управление расписанием врачей**
 -   **Описание**: Процесс создания и управления расписанием работы врачей.
 -   **Детали**: Врачи или администраторы могут задавать и изменять время работы на каждый день, включая форс-мажорные ситуации.
  -    **Таблицы**: `WorkSchedule`
  
3.   **Формирование и управление слотами**
   -   **Описание**: Процесс создания слотов для приема пациентов в зависимости от расписания врачей.
   -   **Детали**: Генерация слотов на основе заданного расписания, учитывая возможные изменения и длину слотов. (Можно реализовать **динамически** или **статически**)
   -   **Таблицы**: `TimeSlot`

4.   **Запись на прием**
- **Описание**: Процесс записи пациентов на прием к врачам.
-   **Детали**: Пациенты или администраторы могут выбирать свободные слоты для записи на прием. При успешной записи слот помечается как "booked".
  -   **Таблицы**: `Appointment`
 
5.  **Отмена и перенос записи**
-   **Описание**: Процесс отмены или переноса записи пациента на другой слот.
-   **Детали**: Пациенты или администраторы могут отменить запись или перенести ее на другое время. При отмене слот освобождается.
 -   **Таблицы**: `Appointment`, `TimeSlot`

6. **Учет пропущенных приемов и компенсации**   
 -   **Описание**: Процесс учета пропущенных приемов и предоставления компенсации пациентам.
 -   **Детали**: Если врач не может принять пациента, администратор это фиксирует, и пациенту предоставляется компенсация или скидка на следующий прием.
-   **Таблицы**: `MissedAppointment`, `Appointment`

7.    **Управление пользователями и ролями**
    
-   **Описание**: Процесс управления пользователями системы и назначение ролей (врач, администратор, пациент).
-   **Детали**: Администраторы могут создавать учетные записи пользователей и назначать им роли с разными уровнями доступа.
-   **Таблицы**: Таблицы `User` и `Role`
