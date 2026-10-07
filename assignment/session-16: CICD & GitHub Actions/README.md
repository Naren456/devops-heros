# session-16: CICD & GitHub Actions

## Overview
Complete CI/CD demo project (`demo/`: Python calculator app + Dockerfile +
`workflows/ci.yml` + `workflows/cd.yml`). Every CI step was executed locally
for real on 2026-10-07 (venv at `/tmp/s16-venv`, Docker 29.8.2); the workflow
files are YAML-validated and push-ready.

## Demo project
```
demo/
├── app.py                 calculator (add/subtract/divide) + __main__ demo
├── tests/test_app.py      4 pytest cases (incl. divide-by-zero)
├── requirements.txt       pytest, pytest-cov, flake8
├── Dockerfile             python:3.12-slim, pip install, CMD python app.py
└── workflows/
    ├── ci.yml             lint -> test -> docker build (push/PR to main)
    └── cd.yml             publish to GHCR -> deploy to staging (needs GHCR_TOKEN, STAGING_KUBECONFIG)
```

## Concepts covered (mapped to the project)
- **CI vs CD:** CI = every push is linted/tested/built (`ci.yml`); CD = main
  branch automatically ships the image and deploys it (`cd.yml`).
- **Workflow:** a YAML file under test — here `ci.yml` / `cd.yml` (kept in
  `demo/workflows/`; copy to `.github/workflows/` to activate).
- **Jobs:** `lint`, `test`, `build` (CI); `publish`, `deploy-staging` (CD).
  `needs:` chains them so test can't run on unlinted code.
- **Steps:** individual `run:`/`uses:` items inside a job (checkout, setup-python…).
- **Runners:** `runs-on: ubuntu-latest` — GitHub-hosted VMs executing the jobs.
- **Secrets:** `${{ secrets.GHCR_TOKEN }}` / `${{ secrets.STAGING_KUBECONFIG }}`
  inject credentials without hardcoding them (note: never commit real secrets —
  see session 17 secret scanning).
- **Artifacts:** coverage output (`--cov-report=term`, uploadable via
  `actions/upload-artifact`) and the built image tagged `${{ github.sha }}`.
- **Build / Test / Pipeline execution:** verified below, in order, exactly as
  the pipeline would run them.

## Commands Used (local pipeline execution)
```bash
python3 -m venv /tmp/s16-venv          # system pip is PEP 668-protected, so venv
/tmp/s16-venv/bin/pip install -r requirements.txt
python3 -m flake8 app.py tests/ --max-line-length=100   # lint job
python3 -m pytest -v --cov=app --cov-report=term        # test job
docker build -t s16-calc-demo .                         # build job
docker run --rm s16-calc-demo
python3 -c "import yaml,glob; [yaml.safe_load(open(f)) for f in glob.glob('demo/workflows/*.yml')]"  # workflow syntax check
```

## Screenshots

### CI steps executed locally
![ci local](screenshots/01-ci-local.png)
`flake8` PASS, `pytest` 4 passed (69% coverage), image builds and the
container prints the calculator output. (First `pip install` attempt failed on
the system interpreter per PEP 668 — fixed with the venv, as documented.)

## Outcome
- App, tests, Dockerfile, CI and CD workflows all present and verified
  (workflows parse as valid YAML).
- Lesson learned: keep the pipeline's steps runnable locally with one command
  each — debugging CI through push-and-wait is slow; the venv + Docker flow
  here mirrors the runner 1:1.
