# LAB03 - API do serwowania modelu ML

Projekt laboratoryjny z przedmiotu **Nowoczesne Technologie Przetwarzania Danych**.
Celem projektu jest przygotowanie prostego API w **FastAPI** do serwowania modelu ML, przetestowanie endpointów oraz zwracanie predykcji w formacie JSON.

## Zakres projektu

W projekcie zaimplementowano:

- endpoint główny `GET /`,
- endpoint kontrolny `GET /health`,
- endpoint informacyjny `GET /info`,
- endpoint predykcji `POST /predict`,
- walidację danych wejściowych przy użyciu **Pydantic**,
- prosty model klasyfikacyjny **LogisticRegression** z biblioteki **scikit-learn**,
- testy działania w **Postmanie** oraz przy użyciu **cURL**.

## Struktura projektu

```text
lab03ntpd/
├── app.py
├── requirements.txt
├── README.md
└── artifacts/
    └── model_v1.joblib
```

## Wymagania

- Python 3.13
- pip
- system Windows / Linux / macOS

## Utworzenie i aktywacja środowiska wirtualnego

Zalecana nazwa środowiska w tym projekcie: `.venv`.

### Windows PowerShell

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
```

### Linux / macOS

```bash
python3 -m venv .venv
source .venv/bin/activate
```

## Instalacja zależności

```bash
pip install --upgrade pip
pip install -r requirements.txt
```

## Uruchomienie aplikacji

```bash
uvicorn app:app --reload
```

Po uruchomieniu aplikacja będzie dostępna pod adresami:

- `http://127.0.0.1:8000/`
- `http://127.0.0.1:8000/docs`

## Opis endpointów

### `GET /`
Zwraca prostą wiadomość potwierdzającą działanie aplikacji.

Przykładowa odpowiedź:

```json
{
  "message": "API dziala"
}
```

### `GET /health`
Zwraca informację o stanie działania serwera.

Przykładowa odpowiedź:

```json
{
  "status": "ok"
}
```

### `GET /info`
Zwraca podstawowe informacje o modelu.

Przykładowa odpowiedź:

```json
{
  "model_type": "LogisticRegression",
  "number_of_features": 2,
  "classes": ["klasa_0", "klasa_1"],
  "model_path": "artifacts/model_v1.joblib"
}
```

### `POST /predict`
Przyjmuje dane wejściowe w formacie JSON i zwraca predykcję modelu.

Przykładowe dane wejściowe:

```json
{
  "feature_1": 1.0,
  "feature_2": 1.1
}
```

Przykładowa odpowiedź:

```json
{
  "prediction": 0,
  "predicted_class_name": "klasa_0",
  "probability_class_0": 0.8486,
  "probability_class_1": 0.1514,
  "input_data": {
    "feature_1": 1.0,
    "feature_2": 1.1
  }
}
```

## Testowanie endpointów

### Uwaga dla Windows PowerShell

W PowerShellu należy używać polecenia `curl.exe`, ponieważ samo `curl` bywa mapowane na `Invoke-WebRequest` i może działać inaczej niż klasyczny cURL.

### Test `GET /`

```powershell
curl.exe http://127.0.0.1:8000/
```

### Test `GET /health`

```powershell
curl.exe http://127.0.0.1:8000/health
```

### Test `GET /info`

```powershell
curl.exe http://127.0.0.1:8000/info
```

### Test `POST /predict`

```powershell
curl.exe --% -X POST http://127.0.0.1:8000/predict -H "Content-Type: application/json" -d "{\"feature_1\": 1.0, \"feature_2\": 1.1}"
```

### Test błędu walidacji

Przykład wysłania niepełnych danych wejściowych:

```powershell
curl.exe --% -X POST http://127.0.0.1:8000/predict -H "Content-Type: application/json" -d "{\"feature_1\": 1.0}"
```

Dla takiego żądania FastAPI powinno zwrócić błąd walidacji `422 Unprocessable Entity`, ponieważ brakuje pola `feature_2`.

## Uruchomienie z parametrami zbliżonymi do produkcyjnych

Przykład uruchomienia:

```bash
uvicorn app:app --host 0.0.0.0 --port 8000
```

Wariant z większą liczbą workerów:

```bash
uvicorn app:app --host 0.0.0.0 --port 8000 --workers 2
```

## Wykorzystane biblioteki

- fastapi
- uvicorn
- scikit-learn
- numpy
- joblib
- pydantic

## Wnioski

Projekt realizuje wszystkie podstawowe wymagania laboratorium:

- aplikacja API została uruchomiona poprawnie,
- model ML został załadowany i udostępniony przez endpoint `/predict`,
- przygotowano dodatkowe endpointy `/info` i `/health`,
- obsłużono walidację danych wejściowych,
- wykonano testy zarówno w Postmanie, jak i przy użyciu cURL.


# LAB04 - Docker i konteneryzacja modelu ML

W ramach kolejnego laboratorium projekt został rozszerzony o konteneryzację aplikacji przy użyciu **Dockera**.  
Celem tej części projektu było przygotowanie obrazu Dockera z aplikacją ML z poprzednich zajęć, uruchomienie kontenera, przetestowanie endpointów oraz skonfigurowanie środowiska wieloserwisowego.

## Dodatkowe pliki projektu

Po rozszerzeniu projektu repozytorium zawiera:

```text
lab03ntpd/
├── app.py
├── requirements.txt
├── Dockerfile
├── docker-compose.yml
├── README.md
└── artifacts/
    └── model_v1.joblib
```

## Uruchomienie aplikacji za pomocą Dockera

### Budowa obrazu

```bash
docker build -t fastapi-ml-api .
```

### Uruchomienie kontenera

```bash
docker run -d -p 8000:8000 --name fastapi-ml-api-container fastapi-ml-api
```

### Sprawdzenie działania

```bash
docker ps
```

Po uruchomieniu aplikacja będzie dostępna pod adresem:

- `http://127.0.0.1:8000/`
- `http://127.0.0.1:8000/docs`

Przykładowy test endpointu:

```bash
curl http://127.0.0.1:8000/health
```

## Uruchomienie aplikacji za pomocą Docker Compose

### Start usług

```bash
docker compose up -d --build
```

### Sprawdzenie uruchomionych usług

```bash
docker compose ps
```

W aktualnej konfiguracji uruchamiane są dwa serwisy:

- `api` – aplikacja FastAPI z modelem ML,
- `mongodb` – dodatkowy serwis działający w tej samej sieci dockerowej.

### Zatrzymanie środowiska

```bash
docker compose down
```

Po uruchomieniu aplikacja będzie dostępna na porcie `8000`, a dodatkowy serwis MongoDB na porcie `27017`.

## Konfiguracja parametrów aplikacji

W aktualnej wersji projektu aplikacja nie wymaga definiowania własnych zmiennych środowiskowych do działania.  
Najważniejsze parametry konfiguracyjne wynikają z plików `Dockerfile` oraz `docker-compose.yml`.

### Najważniejsze ustawienia

- aplikacja HTTP nasłuchuje na porcie `8000`,
- kontener API mapuje port `8000:8000`,
- serwis MongoDB mapuje port `27017:27017`,
- oba serwisy działają w tej samej sieci dockerowej.

### Zmienne środowiskowe

W tej wersji projektu nie zdefiniowano własnych zmiennych środowiskowych, ponieważ aplikacja działa poprawnie bez dodatkowej konfiguracji.  
Jeśli projekt byłby dalej rozwijany, do zmiennych środowiskowych można byłoby przenieść na przykład:

- numer portu aplikacji,
- nazwę hosta,
- dane połączenia do bazy danych,
- tryb uruchomienia aplikacji.

Takie podejście ułatwia późniejszą rozbudowę projektu i dostosowanie konfiguracji do różnych środowisk.

## Wymagane zasoby

Aplikacja jest lekka i do działania potrzebuje podstawowych zasobów:

- Python 3.13 lub środowisko Docker,
- biblioteki z pliku `requirements.txt`,
- dostępu do portu `8000`,
- pliku modelu `artifacts/model_v1.joblib`,
- w przypadku Docker Compose: wolnego portu `27017` dla serwisu MongoDB.

Ze względu na prosty model ML i niewielką liczbę zależności projekt nie wymaga dużej ilości pamięci RAM ani mocy obliczeniowej.