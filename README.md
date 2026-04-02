# Iris ML (LAB4)
Prosta aplikacja oparta o FastAPI oraz model regresji logistycznej. Model został wytrenowany na zbiorze Iris. Aplikacja udostępnia endpointy do sprawdzania stanu API oraz wykonywania predykcji na podstawie danych wejściowych.

## Funkcjonalności:
* sprawdzenie działania API - `/health`
* informacja o modelu - `/info`
* wykonanie predykcji - `/predict`
* dokumentacja Swagger - `/docs`

## Wymagania: 
* Python 3.9 lub nowszy
* pip
* Docker
* Docker Compose

## Spososb uruchomienia:

### Uruchomienie lokalne:

#### Instalacja zależności:
```bash
pip install -r requirements.txt
```
#### Start aplikacji:
```bash
python app.py
```
Aplikacja dostępna będzie pod adresem `http://localhost:8000`.

### Uruchomienie z Dockerem:
#### Budowa obrazu:
```bash
docker build -t iris-api .
```
#### Uruchomienie kontenera:
```bash
docker run -p 8000:8000 iris-api
```
Aplikacja dostępna będzie pod adresem `http://localhost:8000`.

### Uruchomienie z Docker Compose:
#### Start aplikacji:
```bash
docker-compose up -- build
```
Aplikacja dostępna będzie pod adresem `http://localhost:8000`.

#### Zatrzymanie aplikacji:
```bash
docker compose down
```

## Predykcja
Aby wykonać predykcję, można użyć narzędzia cURL lub Postman. Przykładowe zapytanie cURL do endpointu `/predict` wygląda następująco:

```bash
curl -X POST "http://localhost:8000/predict" -H "Content-Type: application/json" 
-d '{"sepal_length": 5.1, "sepal_width": 3.5, "petal_length": 1.4,"petal_width": 0.2}'
```
Przykładowa odpoiwedź:
```json
{"prediction": "setosa"}
```