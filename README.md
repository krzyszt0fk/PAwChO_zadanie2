# SPRAWOZDANIE z wykonania zadania 2
### 1. Na podstawie  Dockerfile-a oraz kodów źródłowych aplikacji opracowanej jako rozwiązanie zadania nr 1 stworzony został pipeline w usłudze GitHub Actions, który buduje obraz kontenera.
### Opracowany łańcuch został umieszczony w *.github/workflows/docker-multiarch-ci.yml*
### Spełnia on warunki:
- wsparcie architektur: linux/arm64 oraz linux/amd64. Aby zbudować obraz na wiele architektur instalujemy QEMU oraz narzędzie Buildx:
```shell 
      - name: Set up QEMU
        uses: docker/setup-qemu-action@v3 

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3
      # Budowanie obrazu z cache
      - name: Build multi-arch image (amd64, arm64) with cache
        uses: docker/build-push-action@v6
        with:
          context: .
          file: ./Dockerfile
          platforms: linux/amd64,linux/arm64
          push: true
          #tagujemy obraz commitem (sha) - unikalny identyfikator builda
          tags: ghcr.io/krzyszt0fk/pawcho_zadanie2:sha-${{ github.sha }}
          # Użycie cache z DockerHub
          cache-from: type=registry,ref=krzyszt0fk/pawcho_zadanie2_cache:buildcache
          cache-to: type=registry,ref=krzyszt0fk/pawcho_zadanie2_cache:buildcache,mode=max
```
Tag typu commit SHA to unikalny identyfikator na podstawie pełnego hash commita. Zapewnia to jednoznaczne powiązanie obrazu z konkretnym stanem repozytorium. Tagi oparte na commitach są niezmienne i umożliwiają śledzenie, jaka wersja kodu odpowiada danemu obrazowi

- Wykorzystuje dane cache przechowywane w publicznym repozytorium DockerHub (https://hub.docker.com/repository/docker/krzyszt0fk/pawcho_zadanie2_cache/general)
- Wykonuje test CVE. Zapewnia to przesłanie obrazu do repozytorium tylko gdy nie będzie zawierał zagrożeń
  sklasyfikowanych jako krytyczne lub wysokie:
```shell
      - name: Security scan with Trivy
        uses: aquasecurity/trivy-action@0.30.0
        with:
          scan-type: image
          image-ref: ghcr.io/krzyszt0fk/pawcho_zadanie2:sha-${{ github.sha }}
          format: table
          severity: CRITICAL,HIGH       
          ignore-unfixed: true          
          exit-code: 1 
```

### 2. Tworzenie tokenów i repozytorów
Aby powyższy workflow działał został przygotowany:
- GHCR_TOKEN
- DOCKERHUB_TOKEN
- DOCKERHUB_USERNAME
  ![secrets](img/secrets.PNG)
Powyższe secrety dodane zostały do repozytorium na GitHub. 

## 3. Uruchomienie
Utworzenie nowego tag-a:
```shell
git tag v0.1.2
```
Przesłanie tag-a na repozytorium Github
```shell
git push origin v0.1.2
```

## 4. Sprawdzenie poprawności działania
W zakładce Actions logi potwierdzają działanie łańcucha:

![jobs](img/jobs.PNG)

W repozytorium Dockerhub pojawił się obraz buildcache:
![DockerHub](img/dh.PNG)

W pakietach Github pojawił się nowy pakiet z odpowiednim tagiem
![package](img/package.PNG)
