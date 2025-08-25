# Konfiguracja Infrastruktury dla Aplikacji CNApp

## Opis Projektu

To repozytorium jest centralnym miejscem przechowywania całej konfiguracji infrastruktury potrzebnej do wdrożenia i uruchomienia systemu mikrousług **CNApp** w środowisku Kubernetes. Celem jest utrzymanie "Infrastruktury jako Kodu" (Infrastructure as Code).

Architektura opiera się na zestawie mikrousług (`Register`, `Login`, `Posts`, `File Transfer` itd.) komunikujących się z jedną, współdzieloną bazą danych `MySQL`.

---

## Wymagania Wstępne

Przed wdrożeniem aplikacji, upewnij się, że Twój klaster Kubernetes ma zainstalowany **dostawcę pamięci masowej (storage provisioner)**. Poniższe manifesty są przystosowane do pracy z `local-path-provisioner`.

Jeśli nie masz go w klastrze, zainstaluj go za pomocą poniższej komendy:
```bash
kubectl apply -f https://raw.githubusercontent.com/rancher/local-path-provisioner/v0.0.26/deploy/local-path-storage.yaml
```

---

## Jak Wdrożyć Aplikację?

### Krok 1: Stworzenie Przestrzeni Nazw (Namespace)

Aby utrzymać porządek w klastrze, wszystkie komponenty aplikacji zostaną umieszczone w dedykowanej przestrzeni nazw `cnapp`. Należy ją stworzyć (tylko raz) za pomocą polecenia:

```bash
kubectl create namespace cnapp
```

### Krok 2: Przygotowanie Pliku z Sekretami

Przed wdrożeniem należy stworzyć plik z sekretami dla bazy danych na podstawie dołączonego szablonu.

1.  W katalogu `k8s/` zmień nazwę pliku `00-mysql-deployment.template.yaml` na `00-mysql-deployment.yaml`.
2.  Wygeneruj swoje hasła w formacie **Base64**. Przykładowe polecenia:
    ```bash
    # Hasło roota
    echo -n 'moje-super-tajne-haslo-roota' | base64

    # Hasło użytkownika aplikacji
    echo -n 'haslo-dla-mojej-aplikacji' | base64
    ```
3.  Otwórz nowo utworzony plik `k8s/00-mysql-deployment.yaml` i wklej w nim swoje zakodowane hasła w odpowiednie miejsca.

**UWAGA:** Plik `k8s/00-mysql-deployment.yaml` jest ignorowany przez Git (`.gitignore`) i nigdy nie powinien być umieszczany w publicznym repozytorium.

### Krok 3: Wdrożenie w Kubernetesie

Po przygotowaniu pliku z sekretami wdróż wszystkie manifesty do dedykowanej przestrzeni nazw. Użyj jednego polecenia, wykonanego z głównego katalogu repozytorium:

```bash
kubectl apply -f k8s/ -n cnapp
```

---

## Weryfikacja Wdrożenia

Aby sprawdzić, czy wszystkie zasoby aplikacji poprawnie się uruchomiły, użyj poniższych poleceń.

**Sprawdzenie statusu Podów:**
```bash
# Wyświetl wszystkie pody w przestrzeni nazw 'cnapp'
kubectl get pods -n cnapp -o wide
```

**Sprawdzenie statusu wolumenów (Persistent Volume Claims):**
```bash
# Status powinien zmienić się na 'Bound'
kubectl get pvc -n cnapp
```