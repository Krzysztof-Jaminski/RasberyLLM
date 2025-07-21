# Naprawa błędu LLama.Native.NativeApi na Raspberry Pi (LLamaSharp / MaIN.NET)

## Problem

Podczas uruchamiania aplikacji .NET z modelem LLM na Raspberry Pi pojawia się błąd:

```
Error: The type initializer for 'LLama.Native.NativeApi' threw an exception.
```

lub aplikacja wyłącza się bez komunikatu 

---

## Przyczyna

.NET/LLamaSharp na Raspberry Pi (ARM64) ma jednocześnie dwa problemy:

1. **Nie ładuje** `libllama.so` z katalogu aplikacji (`bin/Debug/net8.0/runtimes/linux-arm64/native/` itd.) – wymaga obecności biblioteki w systemowym katalogu `/usr/lib/`.
2. **Domyślnie pobierana lub budowana biblioteka jest niezgodna** – nie działa nawet po przeniesieniu do `/usr/lib/`. Działa wyłącznie własnoręcznie zbudowana wersja z odpowiedniego commita llama.cpp.

---

## Szybkie rozwiązanie

1. Zbuduj poprawną wersję `libllama.so` (zgodną z LLamaSharp/MaIN.NET, commit: `ceda28ef8e310a8dee60bf275077a3eedae8e36c`).
2. Skopiuj ją do katalogu systemowego:
   ```bash
   sudo cp ~/llama.cpp/build/bin/libllama.so /usr/lib/
   ```
3. Uruchom aplikację:
   ```bash
   cd ~/RasberyLLM
   dotnet restore
   dotnet build
   dotnet run --urls "http://0.0.0.0:5000"
   ```
4. Otwórz w przeglądarce:
   ```
   http://<IP_RaspberryPi>:5000
   ```


## Jak zbudować poprawną bibliotekę `libllama.so`

```bash
# Sklonuj repozytorium i przejdź do odpowiedniego commita

git clone https://github.com/ggerganov/llama.cpp.git
cd llama.cpp
git checkout ceda28ef8e310a8dee60bf275077a3eedae8e36c

# Zbuduj bibliotekę współdzieloną
mkdir build
cd build
cmake .. -DBUILD_SHARED_LIBS=ON
cmake --build . --config Release

# Skopiuj plik do katalogu systemowego
cd bin
sudo cp libllama.so /usr/lib/
```
---
## Lokalizacja modeli GGUF

Modele GGUF należy domyślnie umieszczać w katalogu:

```
/home/pi/models
```

czyli **w katalogu domowym użytkownika**.

Lokalizację katalogu modeli można zmienić w pliku `appsettings.json`.


## Wnioski z testów

- .NET na Raspberry Pi (ARM64) **ignoruje** nawet poprawną wersję `libllama.so` w katalogach aplikacji (`bin/Debug/net8.0/runtimes/linux-arm64/native/` itd.).
- Nawet jeśli podmienisz wszystkie lokalne pliki na poprawne, aplikacja nie działa bez obecności biblioteki w `/usr/lib/`.
- Oryginalne biblioteki pobierane przez `dotnet restore`/`build` są niezgodne z wrapperem
