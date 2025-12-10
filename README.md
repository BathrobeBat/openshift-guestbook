# Guestbook – Inlämningsuppgift OpenShift / Helm / GHCR

**Gruppmedlemmar:**  
- Thed Rudebeck
- Andres Nyman  

Detta repo är vår inlämningsuppgift där vi bygger två containers (backend + frontend), publicerar dem till GitHub Container Registry (GHCR) och deployar dem till OpenShift med Helm.

---

## Vad som händer vid `git push`

När kod pushas till `main` sker följande automatiskt via GitHub Actions:

1. Backend-container byggs
2. Frontend-container byggs
3. Containers publiceras till GHCR
4. Helm-chart uppdateras (nya image-tags)
5. Deployment i OpenShift uppdateras via Helm (`helm upgrade`)

På detta sätt får vi ett automatiserat CI/CD-flöde som deployar nya versioner utan manuell handpåläggning.

---

## Våra två containers

| Del | Tech | Image (GHCR) |
|-----|------|--------------|
| Backend | Go | `ghcr.io/bathrobebat/backend:latest` |
| Frontend | Nginx | `ghcr.io/bathrobebat/frontend:latest` |

---

## Deployment i OpenShift (Helm)

All deployment hanteras av Helm-chartet i `helm/guestbook/` och vi valde att testa deploya med OpenShift Web Terminal:
```
cd ~
git clone https://github.com/BathrobeBat/openshift-guestbook.git
cd openshift-guestbook/helm/guestbook
```
```
helm upgrade --install guestbook . \
  --values values.yaml
```
Om images är privata används `imagePullSecrets` i OpenShift.

Om images är publika behövs det inte.

## Vilken YAML ändras?

När en container byggs med ny tag (t.ex. `v1.0.1`) så ändras bara image-tag i values.yaml.

Exempel:
```
backend:
  image:
    repository: ghcr.io/bathrobebat/backend
    tag: v1.0.1
```

När vi kör `helm upgrade` så uppdateras bara backend-podden – vi behöver inte apply:a alla YAML, eftersom Helm håller koll på diffen.

## Behöver vi starta om deployment?

Nej – `helm upgrade` rullar automatiskt nya pods.

dvs. Helm sköter restart och rolling update.


Om vi ändrar ConfigMap kan Helm generera ny checksum så att podden startas om.

Annars kan man manuellt rulla om med:
```
oc rollout restart deployment backend
```

## GHCR-auth i OpenShift

Om images är privata skapar vi `imagePullSecret`:
```
oc create secret docker-registry ghcr-pull-secret \
  --docker-server=ghcr.io \
  --docker-username=bathrobebat \
  --docker-password="<PAT>" \
  --docker-email="dummy@example.com"

oc secrets link default ghcr-pull-secret --for=pull
```

## Tekniker vi använder

- GitHub Actions
- Docker
- GHCR
- Helm
- OpenShift
- Go backend
- Nginx frontend

## Vad vi lärt oss
- Automatisera byggsteg med GitHub Actions
- Pusha images till GHCR
- Deploya via Helm i OpenShift
- Hantera imagePullSecrets
- Rolling updates utan downtime  
