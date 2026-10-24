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

## Funkcje

- Administrator dodaje użytkowników (login, imię, nazwisko i hasło), zmienia ich hasła, zawiesza, wznawia i usuwa konta.
- Administrator i użytkownicy mogą zmienić hasło oraz motyw: zielony, niebieski, czerwony lub pomarańczowy.
- Użytkownicy tworzą quizy z wklejonego tekstu lub przesłanych plików, rozwiązują je online i pobierają quiz z kluczem odpowiedzi.
- Konta i quizy są zapisywane w lokalnej bazie SQLite (`quizlab.sqlite3`).
- Generowanie pytań wymaga klucza API dostawcy AI ustawionego w `OPENAI_API_KEY`; model można wskazać przez `OPENAI_MODEL` (domyślnie `gpt-4.1-mini`). Klucz API pozostaje po stronie backendu.
- PDF-y tekstowe są odczytywane przez `pypdf`. Zdjęcia wymagają pakietów Pillow i pytesseract oraz systemowego Tesseract OCR z językiem polskim.
- Obsługiwane formaty: PDF, JPG, PNG, WEBP i TXT. Limit aplikacji to 20 MB na żądanie.

## Dane i wdrożenie

Przesłany materiał jest przekazywany do skonfigurowanego dostawcy AI w celu utworzenia pytań. Nie przesyłaj tam poufnych danych bez upewnienia się, że jest to zgodne z Twoimi zasadami.

Serwer nasłuchuje lokalnie na `127.0.0.1`. Do publicznego wdrożenia użyj HTTPS, mocnego `QUIZLAB_SECRET_KEY`, serwera WSGI za reverse proxy, kopii zapasowych i zmień hasło początkowego administratora. Sesje i formularze mają ochronę CSRF; w produkcji włącz również ciasteczka `Secure` za HTTPS.
