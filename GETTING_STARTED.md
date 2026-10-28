# Getting Started — Publishing This Repo to GitHub (Fedora)

_Personal setup notes. Delete this file before publishing if you like._

## 1. Create your GitHub account
1. Go to https://github.com/signup
2. Pick a professional username — e.g. your name, or something like `nm-networking`. This becomes your portfolio URL: `github.com/<username>`.
3. Verify your email, then turn on **two-factor authentication** (Settings → Password and authentication).
4. Fill in your profile: photo, bio ("Network Engineer | Telecom | CCNA in progress"), location, LinkedIn link.

## 2. Install and configure Git on Fedora
```bash
sudo dnf install git gh
git config --global user.name  "Your Name"
git config --global user.email "you@example.com"
git config --global init.defaultBranch main
```

## 3. Log in from the terminal (easiest way)
```bash
gh auth login
```
Choose: **GitHub.com → HTTPS → Login with a web browser**, then paste the code it shows into the browser.

## 4. Fill in your lab files
Inside `01-router-on-a-stick/`:
- `configs/` — one file per device, e.g. `R1.txt`, `SW1.txt` (from `show running-config`)
- `verification/` — outputs like `show-ip-nat-translations.txt`
- `diagrams/topology.png` — screenshot of your GNS3 topology
- `gns3/` — GNS3 → **File → Export portable project** (untick "include base images")

**Before committing:** remove `enable secret`, passwords, SNMP communities and any real BSNL IPs from configs.

Also replace `YOUR-USERNAME` in the main `README.md`.

## 5. Push to GitHub
```bash
cd networking-portfolio
git init
git add .
git commit -m "Lab 01: Router-on-a-Stick inter-VLAN routing"
gh repo create networking-portfolio --public --source=. --push
```

## 6. Adding the next lab later
```bash
cp -r templates/LAB_README_TEMPLATE.md 02-<lab-name>/README.md   # after mkdir
git add .
git commit -m "Lab 02: <name>"
git push
```
Then add a row to the Labs table in the main README.

## 7. Optional: profile README
Create a public repo named exactly `<your-username>` with a `README.md` — GitHub shows it on your profile page. Good place for a short intro and a link to this portfolio.
