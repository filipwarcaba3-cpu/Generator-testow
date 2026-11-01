# QuizLab

Aplikacja webowa w Pythonie (Flask) z panelem administratora, kontami użytkowników, profilami, motywami kolorystycznymi oraz quizami generowanymi przez AI.

## Uruchomienie w Windows PowerShell

Wymagany Python 3.10 lub nowszy.

```powershell
cd outputs/quizlab_app
py -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -r requirements.txt
$env:QUIZLAB_SECRET_KEY = (py -c "import secrets; print(secrets.token_hex(32))")
$env:OPENAI_API_KEY = "wstaw-swoj-klucz"
py app.py
```

Otwórz http://127.0.0.1:5000. Konto administratora utworzy się automatycznie przy pierwszym uruchomieniu:

- login: `admin`
- hasło: `1234`

Zmień to hasło po pierwszym zalogowaniu w zakładce **Profil**. Administrator dodaje konta w swoim panelu i przekazuje użytkownikom ich loginy oraz hasła.

## Wdrożenie na Renderze

1. Umieść folder `quizlab_app` w repozytorium GitHub i wypchnij zmiany.
2. W Render wybierz **New → Web Service** i połącz repozytorium.
3. Ustaw **Root Directory** na `outputs/quizlab_app` (jeśli ten folder znajduje się w repozytorium).
4. Wpisz **Build Command**: `pip install -r requirements.txt`.
5. Wpisz **Start Command**: `gunicorn --timeout 180 app:app`.
6. W **Environment Variables** dodaj `QUIZLAB_SECRET_KEY` z długą losową wartością i `OPENAI_API_KEY` ze swoim kluczem API. Możesz też ustawić `OPENAI_MODEL`.
7. Ustaw health check path na `/health`, wybierz plan i kliknij **Create Web Service**.

Renderowy system plików jest tymczasowy. W tej wersji konta i quizy są w SQLite, więc przed używaniem przez innych dodaj do usługi płatny **Persistent Disk** z mount path `/var/data` i ustaw zmienną `QUIZLAB_DB=/var/data/quizlab.sqlite3`. Bez trwałego dysku baza może zniknąć przy restarcie lub wdrożeniu. Pamiętaj, by zmienić początkowe hasło `admin` / `1234` po pierwszym logowaniu.

## Aktualizacja istniejącego repozytorium GitHub

Skopiuj do katalogu głównego repozytorium `app.py`, `requirements.txt` oraz `README.md`, a folder `templates` zastąp folderem z tej wersji. Zatwierdź zmiany w GitHubie; Render rozpocznie automatyczne wdrożenie. Zachowaj ustawione w Renderze sekrety `QUIZLAB_SECRET_KEY` i `OPENAI_API_KEY`.

## Funkcje

- Administrator dodaje użytkowników i administratorów (login, imię, nazwisko i hasło), zmienia hasła, zawiesza, wznawia i usuwa konta. Ostatniego aktywnego administratora nie da się usunąć ani zawiesić.
- Administrator i użytkownicy mogą zmienić hasło oraz motyw: zielony, niebieski, czerwony lub pomarańczowy.
- Użytkownicy tworzą testy z tekstu, zdjęć, skanów PDF i tekstowych PDF-ów; mogą wybrać od 1 do 50 pytań i typy ABCD, prawda/fałsz, otwarte oraz dopasowywanie.
- Zdjęcia i skany są analizowane przez AI. Przy zaznaczonej opcji ilustracji model może wygenerować maksymalnie trzy obrazy na test; generowanie grafik może zwiększyć koszt API.
- Przycisk „Wyjaśnij materiał” prosi AI o objaśnienie tekstu lub przesłanych obrazów. „Zapisz jako PDF” otwiera systemowe okno drukowania — wybierz w nim „Zapisz jako PDF”; dokument zawiera test i klucz.
- Konta i quizy są zapisywane w lokalnej bazie SQLite (`quizlab.sqlite3`).
- Generowanie pytań wymaga klucza API dostawcy AI ustawionego w `OPENAI_API_KEY`; model można wskazać przez `OPENAI_MODEL` (domyślnie `gpt-4.1-mini`). Klucz API pozostaje po stronie backendu.
- Tekstowe PDF-y są odczytywane przez `pypdf`; skanowane PDF-y są renderowane przez `PyMuPDF` i analizowane przez model AI. Zdjęcia trafiają bezpośrednio do analizy obrazu AI, więc serwer nie potrzebuje instalacji Tesseract OCR.
- Obsługiwane formaty: PDF, JPG, PNG, WEBP i TXT. Limit aplikacji to 20 MB na żądanie.

## Dane i wdrożenie

Przesłany materiał jest przekazywany do skonfigurowanego dostawcy AI w celu utworzenia pytań. Nie przesyłaj tam poufnych danych bez upewnienia się, że jest to zgodne z Twoimi zasadami.

Serwer nasłuchuje lokalnie na `127.0.0.1`. Do publicznego wdrożenia użyj HTTPS, mocnego `QUIZLAB_SECRET_KEY`, serwera WSGI za reverse proxy, kopii zapasowych i zmień hasło początkowego administratora. Sesje i formularze mają ochronę CSRF; w produkcji włącz również ciasteczka `Secure` za HTTPS.
