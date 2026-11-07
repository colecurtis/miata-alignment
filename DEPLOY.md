# Publish the phone-ready site

The app is static: it has no server or database. A static host that serves HTTPS is enough for Galaxy S21 sensor access and PWA installation.

## GitHub Pages

The `.github/workflows/pages.yml` workflow is ready to deploy these files from the root of a GitHub repository.

The site and repository code will be public. Measurement data stays in each browser and is not uploaded by the app.

1. Create a GitHub repository named `miata-alignment` (or another name you prefer).
2. Upload the contents of this folder to the repository root. Include the hidden `.github` folder and `.nojekyll` file.
3. In the repository, open **Settings > Pages** and set the publishing source to **GitHub Actions**.
4. Push the files to the `main` branch, or run **Publish Miata Alignment Desk** from the Actions tab.
5. After the workflow completes, GitHub Pages shows the live HTTPS address. For a project repository it is usually `https://YOUR-USERNAME.github.io/REPOSITORY/`.
6. Open that address in Chrome on the S21 and choose **Add to Home screen**.

The first deployment publishes the current code. Later pushes to `main` update the hosted site. The pages workflow follows GitHub's documented static publishing flow: check out, configure Pages, upload the static artifact, and deploy it.

## Session privacy

The hosted files contain only the app. Vehicle details and measurements stay in local storage in each browser; the app does not send them to the hosting provider. Use **Export session** to save a JSON copy. Browser data is not synchronized between devices unless you manually export and transfer it.
