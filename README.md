# Konfiguracja Infrastruktury dla Aplikacji CNApp

## Opis Projektu

To repozytorium jest centralnym miejscem przechowywania całej konfiguracji infrastruktury potrzebnej do wdrożenia i uruchomienia systemu mikrousług **CNApp** w środowisku Kubernetes. Celem jest utrzymanie "Infrastruktury jako Kodu" (Infrastructure as Code), co zapewnia powtarzalność, wersjonowanie i automatyzację wdrożeń.

Architektura opiera się na zestawie mikrousług (`Register`, `Login`, `Posts`, `File Transfer` itd.) komunikujących się z jedną, współdzieloną bazą danych `MySQL`.

## Struktura Repozytorium

* **/k8s/**: Zawiera wszystkie pliki manifestu Kubernetes (`.yaml`) dla poszczególnych komponentów aplikacji.
* **README.md**: Ten plik, czyli główna strona informacyjna repozytorium.

## Jak Wdrożyć Aplikację?

### Krok 1: Przygotowanie Pliku z Sekretami

Przed wdrożeniem, należy stworzyć plik z sekretami dla bazy danych na podstawie szablonu.

1.  W katalogu `k8s/` zmień nazwę pliku z `00-mysql-deployment.template.yaml` na `00-mysql-deployment.yaml`.
2.  Wygeneruj swoje hasła w formacie Base64. Przykładowe polecenia:
    ```bash
    # Hasło roota
    echo -n 'moje-super-tajne-haslo-roota' | base64
    # Hasło użytkownika aplikacji
    echo -n 'haslo-dla-mojej-aplikacji' | base64
    ```
3.  Otwórz nowo utworzony plik `k8s/00-mysql-deployment.yaml` i wklej w nim swoje zakodowane hasła w odpowiednie miejsca.

**UWAGA:** Plik `k8s/00-mysql-deployment.yaml` jest ignorowany przez Git (`.gitignore`) i **nigdy** nie powinien być umieszczany w publicznym repozytorium.

### Krok 2: Wdrożenie w Kubernetesie

Po przygotowaniu pliku z sekretami, wdróż wszystkie manifesty za pomocą jednego polecenia, wykonanego z głównego katalogu tego repozytorium:

```bash
kubectl apply -f k8s/