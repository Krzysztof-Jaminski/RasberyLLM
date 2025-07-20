# Jak naprawić błąd LLama.Native.NativeApi w MaIN.NET / LLamaSharp na RASBERYLLM2 (Raspberry Pi)

## Problem

Podczas uruchamiania aplikacji .NET (np. MaIN.NET, LLamaSharp) z lokalnym modelem LLM na Raspberry Pi pojawia się błąd:

```
Error: The type initializer for 'LLama.Native.NativeApi' threw an exception.
```
lub aplikacja wyłącza się bez komunikatu przy próbie uruchomienia modelu.

---

> **WAŻNE!**
> Zwykłe polecenia `dotnet restore`, `dotnet build`, `dotnet run` **nie wystarczą** na Raspberry Pi i wygenerują powyższy problem, jeśli nie zadbasz o zgodność natywnych bibliotek. 
> 
> **Na dzień 19.07.2025 uruchamianie przez CLI MaINa (mcli) na Raspberry Pi wciąż nie działa poprawnie.**

---

## Szczegółowy opis problemu

Na Windowsie MaIN.NET/LLamaSharp automatycznie pobiera i ładuje natywne biblioteki (np. `llama.dll`) przez NuGet, więc nie trzeba się tym przejmować. Na Raspberry Pi (Linux ARM64) NuGet również umieszcza plik `libllama.so` w katalogu builda (`bin/Debug/net8.0/runtimes/linux-arm64/native/`), ale ta wersja może być niezgodna z wrapperem .NET (MaIN.NET/LLamaSharp)?? W efekcie pojawia się błąd inicjalizacji natywnego API lub aplikacja wyłącza się bez komunikatu.

## Rozwiązanie

### 1. Zbuduj natywną bibliotekę `libllama.so` z odpowiedniej wersji llama.cpp

- **Wersja wymagana przez LLamaSharp v0.24.0 i MaIN.NET 0.2.9:**
  - **llama.cpp commit:** `ceda28ef8e310a8dee60bf275077a3eedae8e36c`

#### Krok po kroku:

```bash
# Przenieś starą wersję, jeśli istnieje
mv ~/llama.cpp ~/llama.cpp-old

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

*(lub do katalogu z aplikacją .NET, np. `~/RasberyLLM/bin/Debug/net8.0/runtimes/linux-arm64/native/`)*

---

### 2. Uruchom ponownie aplikację .NET

```bash
cd ~/RasberyLLM
dotnet run --urls "http://0.0.0.0:5000"
```

---

## Jak wywołać błąd ponownie (do testów lub debugowania)

1. **Usuń plik `libllama.so` z `/usr/lib/`:**
   ```bash
   sudo rm /usr/lib/libllama.so
   ```
2. **(Opcjonalnie) Usuń też z katalogu builda aplikacji:**
   ```bash
   rm ~/RasberyLLM/bin/Debug/net8.0/runtimes/linux-arm64/native/libllama.so
   ```
3. **Uruchom aplikację:**
   ```bash
   cd ~/RasberyLLM
   dotnet run
   ```
4. **Efekt:**
   Powinien pojawić się błąd:
   ```
   Error: The type initializer for 'LLama.Native.NativeApi' threw an exception.
   ```

---

## Dzięki temu:
- MaIN.NET/LLamaSharp i natywna biblioteka będą w tej samej, zgodnej wersji.
- Błąd `LLama.Native.NativeApi` zniknie, a aplikacja nie będzie się wyłączać przy uruchamianiu modelu.

---

# Standardowa instrukcja uruchomienia projektu na Raspberry Pi

1. **Przejdź do katalogu projektu:**
   ```bash
   cd ~/RasberyLLM
   ```
2. **Przygotuj model GGUF i umieść go w odpowiednim katalogu (np. `/home/pi/models/`).**
3. **Upewnij się, że plik `libllama.so` jest w `/usr/lib` lub w katalogu aplikacji.**
4. **Uruchom aplikację:**
   ```bash
   dotnet run --urls "http://0.0.0.0:5000"
   ```
5. **Otwórz przeglądarkę na innym urządzeniu w sieci i wejdź na:**
   ```
   http://<IP_RaspberryPi>:5000
   ```

---

**Notatka:**
- Te instrukcje są specjalnie przygotowane dla Raspberry Pi (ARM64, np. Pi 4/5, system 64-bitowy).
- Jeśli pojawią się inne błędy, sprawdź zgodność wersji bibliotek i modelu. 
