# Sprawozdanie nr 4
![PBS WTIiE logo CMYK poziom PL-01.png](https://pbs.edu.pl/upload/media/default/0001/01/b6cecfbda538ed95961609f41725eeecb1c0aaff.svg) <br>
**Nazwa ćwiczenia:** Docker i konteneryzacja modelu ML <br>
**Przedmiot:** Nowoczesne techniki przetwarzania danych [LAB]<br>
**Student grupa:** Szymon Piórkowski, gr. I <br>
**Data ćwiczeń:** 25.03.2026 r. <br>
**Data oddania sprawozdania:** 4.04.2026 r. <br>

## 1. Cel ćwiczenia
Celem ćwiczenia było stworzenie obrazu Docker z modelem ML, uruchomienie kontenera oraz testowanie endpointu API. Dodatkowo zadaniem było skonfigurowanie Docker Compose do zarządzania wieloma serwisami.
## 2. Przebieg ćwiczenia

### **Część właściwa zadania**

### Zadanie 1: Przygotowanie aplikacji API

Korzystając z aplikacji z poprzednich zajęć – przygotuj plik „requirements.txt” z potrzebnymi bibliotekami (m.in. flask lub fastapi, scikit-learn, itp.) oraz zainstaluj Docker; 

Przygotowano plik `requirements.txt` zawierający wszystkie potrzebne biblioteki do uruchomienia aplikacji. Plik ten został wygenerowany za pomocą polecenia:

```bash
pip freeze > requirements.txt
```

```text
annotated-doc==0.0.4
annotated-types==0.7.0
anyio==4.13.0
click==8.3.1
colorama==0.4.6
fastapi==0.135.2
h11==0.16.0
idna==3.11
joblib==1.5.3
numpy==2.4.3
pydantic==2.12.5
pydantic_core==2.41.5
scikit-learn==1.8.0
scipy==1.17.1
starlette==1.0.0
threadpoolctl==3.6.0
typing-inspection==0.4.2
typing_extensions==4.15.0
uvicorn==0.42.0

```
Sprawdzono poprawność instalacji Dockera, uruchamiając polecenie:
```bash
docker --version
```
![Zrzut ekranu 2026-03-25 142443.png](Zrzuty%20ekranu/Zrzut%20ekranu%202026-03-25%20142443.png)
### Zadanie 2: Dockerfile i budowa obrazu 

1. Utwórz plik Dockerfile, który będzie zawierał instrukcje do zbudowania obrazu z twoją aplikacją ML; 

W katalogu głównym projektu utworzono plik DockerFile, który zawiera instrukcje potrzebne do zbudowania obrazu aplikacji:
```dockerfile
FROM python:3.13-slim

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 8000

CMD ["uvicorn", "app:app", "--host", "0.0.0.0", "--port", "8000"]

```

2. Skonfiguruj Dockerfile, tak aby:

* Oparty był na oficjalnym obrazie z Pythonem (np. python:3.9-slim); 

Jako bazę wykorzystano obraz `python:3.13-slim`, a następnie ustawiono katalog roboczy wewnątrz kontenera:
```Dockerfile
FROM python:3.13-slim
WORKDIR /app
```

* Kopiował pliki aplikacji i plik requirements.txt do kontenera; 

Do kontenera skopiowano zarówno plik `requirements.txt`, a następnie cały kod aplikacji:
```Dockerfile
COPY requirements.txt .
COPY . .
```

* Instalował niezbędne zależności Pythona; 

Za pomocą `pip` zainstalowano wszystkie zależności wymienione w `requirements.txt`:
```Dockerfile
RUN pip install --no-cache-dir -r requirements.txt
```

* Uruchamiał serwer; 

Skonfigurowano wstawienie portu 8000 oraz uruchomienie serwera FastAPI za pomocą Uvicorna:
```Dockerfile
EXPOSE 8000

CMD ["uvicorn", "app:app", "--host", "0.0.0.0", "--port", "8000"]
```

3. Zbuduj obraz lokalnie i sprawdź, czy tworzy się bez błędów; 

Uruchomiono polecenie budowania obrazu:
```bash
docker build -t iris-app .
```

![Zrzut ekranu 2026-03-25 144243.png](Zrzuty%20ekranu/Zrzut%20ekranu%202026-03-25%20144243.png)

### Zadanie 3: Uruchamianie kontenera i testowanie endpointu 

1. Uruchom kontener z zbudowanym obrazem; 

Kontener został uruchomiony za pomocą polecenia z mapowaniem portów 8000 na 8000:
```bash
docker run -p 8000:8000 iris-app
```


2. Upewnij się, że kontener wystawia port, na którym działa Twój serwer (np. 5000 albo 8000); 

W celu weryfikacji poprawnego działania kontenera i wystawienia portu, użyto polecenia:
```bash
docker ps
```
![Zrzut ekranu 2026-03-31 221542.png](Zrzuty%20ekranu/Zrzut%20ekranu%202026-03-31%20221542.png)


3. Przetestuj endpoint POST „/predict” za pomocą narzędzia takiego jak cURL lub programu typu Postman; 

Przetestowano endpoint `/predict` za pomocą cURL, wysyłając przykładowe dane w formacie JSON:
```bash
curl -X POST "http://localhost:8000/predict" -H "Content-Type: application/json" -d "{\"sepal_length\":5.1,\"sepal_width\":3.5,\"petal_length\":1.4,\"petal_width\":0.2}"
```
![Zrzut ekranu 2026-03-31 221709.png](Zrzuty%20ekranu/Zrzut%20ekranu%202026-03-31%20221709.png)

### Zadanie 4: Konfiguracja Docker Compose 

1. Stwórz plik docker-compose.yml, który:
* Uruchomi kontener z Twoją aplikacją ML;

W pliku `docker-compose.yml` zdefiniowano serwis `app`, który buduje obraz na podstawie lokalnego Dockerfile i mapuje port 8000:
```dockercompose
app:
    build: .
    container_name: iris_api
    ports:
    - "8000:8000"
```

* Przypisze go do odpowiedniej sieci dockerowej; 

Aplikacja została przypisana do sieci `lab_network`, która została zdefiniowana w sekcji `networks`:
```dockercompose
app:
    networks:
      - lab_network
networks:
    lab_network:
        driver: bridge
```


2. Dodaj drugi serwis, np. bazę danych (Redis, PostgreSQL lub inny serwis) i znowu przetestuj aplikację; 

Dodano serwis `db`, który korzysta z obrazu PostgreSQL, ustawia odpowiednie zmienne środowiskowe oraz mapuje port 5432:
```dockercompose
db:
    image: postgres:16
    container_name: iris_postgres
    environment:
        POSTGRES_USER: postgres
        POSTGRES_PASSWORD: password
        POSTGRES_DB: iris_db
    ports:
    - "5432:5432"
    networks:
        - lab_network
    volumes:
      - postgres_data:/var/lib/postgresql/data

```

Sprawdzono działanie obu serwisów za pomocą polecenia:
```bash
docker ps
```

![Zrzut ekranu 2026-04-01 222519.png](Zrzuty%20ekranu/Zrzut%20ekranu%202026-04-01%20222519.png)

Przetestowano ponownie endpoint `/predict` po uruchomieniu aplikacji za pomocą Docker Compose:
```bash
curl http://localhost:8000/health
```
![Zrzut ekranu 2026-04-02 131900.png](Zrzuty%20ekranu/Zrzut%20ekranu%202026-04-02%20131900.png)
### Zadanie 5: Uruchomienie aplikacji w trybie produkcyjnym

1. Dodaj do swojego repozytorium krótkie instrukcje uruchamiania aplikacji: Lokalnie; Za pomocą Dockera; Za pomocą Docker Compose; Opisz, w jaki sposób skonfigurować parametry (np. zmienne środowiskowe) i jakich zasobów potrzebuje aplikacja;


W repozytorium dodano plik `README.md`, który zawiera instrukcje uruchamiania aplikacji w różnych trybach oraz wszelkie informacje o zasobach.

3. Stwórz repozytorium na GitHubie i dodaj link do sprawozdania;

Link do repozytorium na GitHubie: [Link do repozytorium](https://github.com/piuras-oguras/NTPD_LAB4)

## 3. Wnioski

* Docker umożliwia łatwe przenoszenie aplikacji między środowiskami bez konieczności ręcznej konfiguracji zależności.
* Przygotowanie Dockerfile pozwala w prosty sposób zautomatyzować proces budowy aplikacji oraz jej uruchamiania. 
* Konteneryzacja aplikacji eliminuje problemy związane z różnymi wersjami bibliotek oraz konfiguracją systemu.
* Docker Compose znacząco upraszcza zarządzanie wieloma kontenerami oraz umożliwia łatwe tworzenie środowisk wieloserwisowych.
* Dodanie bazy danych jako osobnego serwisu pokazuje, jak można rozszerzać aplikację o kolejne komponenty bez zmiany jej struktury.