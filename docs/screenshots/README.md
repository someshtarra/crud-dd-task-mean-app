# Deliverable Screenshots Directory

Place the verification screenshots in this folder using the filenames below:

| File Name | Description | Key Verification Details |
|-----------|-------------|--------------------------|
| `01-cicd-workflow.png` | GitHub Actions Pipeline Run | Shows successful runs of `build-and-push` and `deploy` jobs in GitHub Actions. |
| `02-dockerhub-repositories.png` | Docker Hub Images | Shows the pushed `mean-backend` and `mean-frontend` images with tags. |
| `03-vm-docker-compose-ps.png` | Ubuntu VM Container Status | Output of `docker compose ps` showing `mean-mongodb`, `mean-backend`, and `mean-frontend` running. |
| `04-application-ui.png` | MEAN Application Working UI | Web browser accessing `http://<VM_IP>/` showing tutorials list, add tutorial, and search. |
| `05-nginx-routing-test.png` | Nginx Port 80 Routing Verification | Terminal `curl -I http://<VM_IP>/` and `curl -I http://<VM_IP>/api/tutorials` showing HTTP 200 responses. |
